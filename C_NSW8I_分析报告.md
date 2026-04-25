# C_NSW8I 功能块分析报告

## 基本信息

| 项目 | 内容 |
|------|------|
| 功能块名称 | C_NSW8I |
| 功能描述 | Numerical Changeover Switch, 8 Branch Selector(INT type)（数值选择开关，8分支选择器，INT类型） |
| 最后修改 | 2017.10.12 |
| 作者 | Hu Jing Qi |
| 页数 | 1页 |

## 功能概述

C_NSW8I 是一个8分支数值选择开关功能块，用于根据选择信号在八个INT类型输入值之间切换输出。选择信号I1~I8按优先级顺序选择对应的输入值输出。

**主要应用场景**：
- 多档位速度选择（8档）
- 多模式参数切换
- 多数据源选择
- 优先级选择器

**选择优先级说明**：
- I1优先级最高，I1=TRUE时输出X1
- I2优先级次之，I1=FALSE且I2=TRUE时输出X2
- 依次类推...
- I8优先级最低，I1~I7都为FALSE且I8=TRUE时输出X8
- 所有选择信号都为FALSE时输出0

## 思维导图

```mermaid
graph TD
    A[C_NSW8I 8分支选择器] --> B[输入模块]
    A --> C[处理模块]
    A --> D[输出模块]
    
    B --> B1[X1~X8]
    B --> B2[I1~I8]
    
    C --> C1[优先级判断]
    C --> C2[数据传送]
    
    D --> D1[Y]
    
    B2 --> C1
    C1 --> C2
    B1 --> C2
    C2 --> D1
    
    style A fill:#e1f5ff
    style B fill:#fff4e1
    style C fill:#e1ffe1
    style D fill:#ffe1e1
```

## 流程路径描述

### 选择X1路径：
开始 → I1=TRUE → 输出X1
**功能**: 选择输入X1输出（最高优先级）

### 选择X2路径：
开始 → I1=FALSE AND I2=TRUE → 输出X2
**功能**: 选择输入X2输出（次优先级）

### 选择X3~X8路径：
依次类推，按优先级顺序选择

### 默认输出路径：
开始 → I1~I8=FALSE → 输出0
**功能**: 无选择时输出默认值0

## 逐帧功能分析

### Rung 7: 选择X1

**触发逻辑**:
- IF I1 = TRUE THEN Y = X1

### Rung 8: 选择X2

**触发逻辑**:
- IF I1 = FALSE AND I2 = TRUE THEN Y = X2

### Rung 9: 选择X3

**触发逻辑**:
- IF I1 = FALSE AND I2 = FALSE AND I3 = TRUE THEN Y = X3

### Rung 10: 选择X4

**触发逻辑**:
- IF I1~I3 = FALSE AND I4 = TRUE THEN Y = X4

### Rung 11: 选择X5

**触发逻辑**:
- IF I1~I4 = FALSE AND I5 = TRUE THEN Y = X5

### Rung 12: 选择X6

**触发逻辑**:
- IF I1~I5 = FALSE AND I6 = TRUE THEN Y = X6

### Rung 13: 选择X7

**触发逻辑**:
- IF I1~I6 = FALSE AND I7 = TRUE THEN Y = X7

### Rung 14: 选择X8

**触发逻辑**:
- IF I1~I7 = FALSE AND I8 = TRUE THEN Y = X8

### Rung 15: 默认输出

**触发逻辑**:
- IF I1~I8 = FALSE THEN Y = 0

## 触发条件总结

### 选择逻辑
| I1 | I2 | I3 | I4 | I5 | I6 | I7 | I8 | 输出Y |
|----|----|----|----|----|----|----|----|----|
| TRUE | X | X | X | X | X | X | X | X1 |
| FALSE | TRUE | X | X | X | X | X | X | X2 |
| FALSE | FALSE | TRUE | X | X | X | X | X | X3 |
| FALSE | FALSE | FALSE | TRUE | X | X | X | X | X4 |
| FALSE | FALSE | FALSE | FALSE | TRUE | X | X | X | X5 |
| FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | X | X | X6 |
| FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | X | X7 |
| FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | TRUE | X8 |
| FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | FALSE | 0 |

**注**: X表示任意值（TRUE或FALSE）

## 实现功能总结

### 主要功能
1. **8分支选择**: 根据选择信号选择八个输入之一
2. **优先级控制**: I1优先级最高，依次递减
3. **默认输出**: 无选择时输出0

## 关键信号说明

| 信号名称 | 信号描述 | 信号类型 | 用途 |
|----------|----------|----------|------|
| X1~X8 | 输入值1~8 | INT | 选择输入（按优先级排序） |
| I1~I8 | 选择信号1~8 | BOOL | 选择控制信号 |
| Y | 输出 | INT | 选择输出 |

## 调试技巧

### 调试步骤
1. 检查X1~X8值，确认输入正常
2. 检查I1~I8信号，确认选择信号正常
3. 监控Y值，观察输出是否正确
4. 验证优先级逻辑是否正确

### 常见问题
1. **输出不变化**: 检查选择信号
2. **优先级错误**: 检查选择信号逻辑
3. **输出为0**: 检查所有选择信号是否都为FALSE

### 监控信号列表
- X1~X8（输入值）
- I1~I8（选择信号）
- Y（输出）
