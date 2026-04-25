# C_MN12 功能块分析报告

## 基本信息

| 项目 | 内容 |
|------|------|
| 功能块名称 | C_MN12 |
| 功能描述 | Manual Sequence of DS-3P Solenoid Valve with STOP Operation Device（DS-3P电磁阀手动顺序控制，带停止操作装置） |
| 最后修改 | 2015.12.25 |
| 作者 | Gao Weidi |
| 页数 | 1页 |

## 功能概述

C_MN12 是一个带停止操作装置的DS-3P（双线圈三位置）电磁阀手动顺序控制功能块。与C_MN11相比，该功能块增加了停止命令输入，可以在运行过程中随时停止，使电磁阀回到中间位置。

**主要应用场景**：
- 需要紧急停止功能的双线圈三位置电磁阀
- 安全要求较高的液压控制系统
- 需要中途停止的执行机构控制

**与C_MN11的区别**：
- C_MN11: 无停止操作装置，只能通过反向命令切换
- C_MN12: 带停止操作装置，可以随时停止到中间位置

## 思维导图

```mermaid
graph TD
    A[C_MN12 DS-3P手动顺序控制-带停止] --> B[输入模块]
    A --> C[处理模块]
    A --> D[输出模块]
    
    B --> B1[A]
    B --> B2[B]
    B --> B3[STOP]
    B --> B4[A_SIL]
    B --> B5[B_SIL]
    B --> B6[A_RIL]
    B --> B7[B_RIL]
    B --> B8[RDY]
    
    C --> C1[A线圈控制]
    C --> C2[B线圈控制]
    C --> C3[停止处理]
    C --> C4[运行检测]
    
    D --> D1[A_ON]
    D --> D2[B_ON]
    D --> D3[RUN]
    
    B1 --> C1
    B3 --> C1
    B4 --> C1
    B6 --> C1
    B8 --> C1
    C1 --> D1
    B2 --> C2
    B3 --> C2
    B5 --> C2
    B7 --> C2
    B8 --> C2
    C2 --> D2
    D1 --> C4
    D2 --> C4
    C4 --> D3
    
    style A fill:#e1f5ff
    style B fill:#fff4e1
    style C fill:#e1ffe1
    style D fill:#ffe1e1
```

## 流程路径描述

### A线圈控制路径：
开始 → A信号 AND A_SIL AND NOT B AND NOT STOP AND A_RIL AND RDY → A_ON输出
**功能**: 控制A线圈励磁

### B线圈控制路径：
开始 → B信号 AND B_SIL AND NOT A AND NOT STOP AND B_RIL AND RDY → B_ON输出
**功能**: 控制B线圈励磁

### 停止路径：
开始 → STOP信号 → A_ON和B_ON都复位 → 电磁阀回到中间位置
**功能**: 紧急停止功能

## 逐帧功能分析

### Rung 7: A线圈控制

**功能描述**: 控制A线圈励磁

**输入条件**:
| 信号名称 | 信号描述 | 信号类型 | 触发值 |
|----------|----------|----------|--------|
| A | A命令 | BOOL | TRUE |
| A_SIL | A启动联锁 | BOOL | TRUE |
| B | B命令 | BOOL | FALSE |
| STOP | 停止命令 | BOOL | FALSE |
| A_RIL | A运行联锁 | BOOL | TRUE |
| RDY | 准备就绪 | BOOL | TRUE |

**输出功能**:
| 信号名称 | 信号描述 | 信号类型 |
|----------|----------|----------|
| A_ON | A线圈输出 | BOOL |

**触发逻辑**:
- IF A = TRUE AND A_SIL = TRUE AND B = FALSE AND STOP = FALSE AND A_RIL = TRUE AND RDY = TRUE THEN A_ON = TRUE

**功能实现**: 
当所有条件满足时A线圈得电，当STOP命令有效时A线圈失电。

### Rung 8: B线圈控制

**功能描述**: 控制B线圈励磁

**输入条件**:
| 信号名称 | 信号描述 | 信号类型 | 触发值 |
|----------|----------|----------|--------|
| B | B命令 | BOOL | TRUE |
| B_SIL | B启动联锁 | BOOL | TRUE |
| A | A命令 | BOOL | FALSE |
| STOP | 停止命令 | BOOL | FALSE |
| B_RIL | B运行联锁 | BOOL | TRUE |
| RDY | 准备就绪 | BOOL | TRUE |

**输出功能**:
| 信号名称 | 信号描述 | 信号类型 |
|----------|----------|----------|
| B_ON | B线圈输出 | BOOL |

**触发逻辑**:
- IF B = TRUE AND B_SIL = TRUE AND A = FALSE AND STOP = FALSE AND B_RIL = TRUE AND RDY = TRUE THEN B_ON = TRUE

**功能实现**: 
当所有条件满足时B线圈得电，当STOP命令有效时B线圈失电。

### Rung 9: 运行检测

**功能描述**: 检测运行状态

**输入条件**:
| 信号名称 | 信号描述 | 信号类型 | 触发值 |
|----------|----------|----------|--------|
| A_ON | A线圈输出 | BOOL | TRUE |
| B_ON | B线圈输出 | BOOL | TRUE |

**输出功能**:
| 信号名称 | 信号描述 | 信号类型 |
|----------|----------|----------|
| RUN | 运行状态 | BOOL |

**触发逻辑**:
- IF A_ON = TRUE OR B_ON = TRUE THEN RUN = TRUE

**功能实现**: 
当A或B任一线圈得电时，输出运行状态信号。

## 触发条件总结

### 控制条件
| 线圈 | 触发条件 | 复位条件 |
|------|----------|----------|
| A_ON | A=TRUE AND A_SIL=TRUE AND B=FALSE AND STOP=FALSE AND A_RIL=TRUE AND RDY=TRUE | STOP=TRUE 或 B命令有效 |
| B_ON | B=TRUE AND B_SIL=TRUE AND A=FALSE AND STOP=FALSE AND B_RIL=TRUE AND RDY=TRUE | STOP=TRUE 或 A命令有效 |

### 停止功能
- **STOP信号**: 优先级最高，可以立即停止所有输出

## 实现功能总结

### 主要功能
1. **A线圈控制**: 控制A线圈励磁
2. **B线圈控制**: 控制B线圈励磁
3. **停止功能**: 支持紧急停止
4. **运行检测**: 检测运行状态
5. **互锁保护**: A和B命令互锁

## 关键信号说明

| 信号名称 | 信号描述 | 信号类型 | 用途 |
|----------|----------|----------|------|
| A | A命令 | BOOL | A方向控制命令 |
| B | B命令 | BOOL | B方向控制命令 |
| STOP | 停止命令 | BOOL | 紧急停止信号 |
| A_SIL | A启动联锁 | BOOL | A启动联锁信号 |
| B_SIL | B启动联锁 | BOOL | B启动联锁信号 |
| A_RIL | A运行联锁 | BOOL | A运行联锁信号 |
| B_RIL | B运行联锁 | BOOL | B运行联锁信号 |
| RDY | 准备就绪 | BOOL | 准备就绪信号 |
| A_ON | A线圈输出 | BOOL | A线圈励磁输出 |
| B_ON | B线圈输出 | BOOL | B线圈励磁输出 |
| RUN | 运行状态 | BOOL | 运行状态输出 |

## 调试技巧

### 调试步骤
1. 检查A和B信号，确认命令正常
2. 检查STOP信号，确认停止功能正常
3. 检查联锁信号，确认联锁条件满足
4. 检查RDY信号，确认准备就绪
5. 监控A_ON、B_ON、RUN信号，观察输出状态

### 常见问题
1. **线圈不励磁**: 检查命令信号和联锁信号
2. **停止不生效**: 检查STOP信号
3. **互锁失效**: 检查A和B命令逻辑

### 监控信号列表
- A、B、STOP（命令信号）
- A_SIL、B_SIL、A_RIL、B_RIL（联锁信号）
- RDY（准备就绪）
- A_ON、B_ON、RUN（输出信号）
