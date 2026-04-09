# Slat Mode 指南

## 1. 概述

**Slat Mode**是专为**数粒线模组**设计的特殊工作模式。

在数粒线中，药品通过振动盘或输送轨道进入计数通道，每个 Cavity 应该容纳一个药片。Slat Mode 可以智能地区分以下情况：
- 有 Cavity 且装有药片 → 正常
- 有 Cavity 但未装药片 → 漏装缺陷（Missing）
- 无 Cavity 且未装药片 → 空位（Blank，正常现象）
- 无 Cavity 但装有药片 → 多余药片缺陷（Extra_Cavity）

## 2. 与传统模式的区别

|        | 普通模式 | Slat Mode |
|--------|----------|-----------|
| 无药片处理 | 一律判为 N/A | 有 Cavity → N/A<br />无 Cavity → Blank（正常） |
| 区域管理 | 按单个药片 ROI 管理 | 支持按 Section 分组管理 |
| 裁剪输出 | 按单个药片裁剪 | 支持按区块裁剪 |

## 3. 核心概念

### 3.1 Cavity

Slat Mode 根据两个条件判断状态：

1. **是否有 Cavity**（由配置 `has_cavity` 指定）
2. **是否检测到药片**（由图像分析自动判断）

判断逻辑如下：

| Cavity 状态 | 药片状态 | 判定结果 | 说明 |
|----------|----------|----------|------|
| 有 Cavity | 有药片 | 正常检测 | 进入常规缺陷检测流程 |
| 有 Cavity | 无药片 | **Missing** | 漏装缺陷 |
| 无 Cavity | 无药片 | **Blank** | 空位，正常现象 |
| 无 Cavity | 有药片 | **Extra_Cavity** | 多余药片，异常 |

### 3.2 Section

在 Slat Mode 中，将多个药片位置划分为一个 **Section**：

- **原因**：踢废装置的物理结构决定，同一个section一起踢废，因此最终的结果要按section给

## 4. 配置参数

### 4.1 启用 Slat Mode

在 `tablet` 节点下添加以下参数：

```yaml
tablet:
  num: 6              # 总药片数量
  num_section: 2       # 分成 2 个区块（>0 即启用 Slat Mode）
  has_cavity: [...]    # 每个位置是否有 Cavity，详见下文
```

### 4.2 关键参数说明

#### `num_section`
- **类型**：整数
- **说明**：将药片分成多少个区块
- **注意**：
  - 设为 0 或不设置时，Slat Mode 不启用
  - flow 文件和 params 文件都可以配置，后者的优先级更高


#### `has_cavity`
- **类型**：bool 数组
- **长度**：必须等于 `num`（总药片数）
- **含义**：
  - `true`：该位置有 Cavity（应该有药片）
  - `false`：该位置无 Cavity（应该为空）

**示例**：

```yaml
# 6 个药片位置，第 1 个和最后 1 个无 Cavity
has_cavity: [false, true, true, true, true, false]
```

#### `section_rois`（自动生成）
- **说明**：系统会根据 `num_section` 自动计算每个区块的外接矩形
- **注意**：无需手动配置，自动生成到建模结果中

**示例**：

```yaml
section_rois:
  - [212, 40, 503, 207]
  - [1031, 34, 520, 202]
```

## 5. 检测结果说明

### 5.1 缺陷类型

在 Slat Mode 下，可能出现以下特殊缺陷类型：

| 缺陷类型 | 含义 | 触发条件 |
|----------|------|----------|
| **BLANK** | 空位 | 无 Cavity 位置未检测到药片（正常现象） |
| **Extra_Cavity** | 多余药片 | 无 Cavity 位置检测到药片 |
| **Missing** | 漏装 | 有 Cavity 位置未检测到药片 |

### 5.2 结果显示

- **Blank（空位）**：显示绿色 ROI 框，标注 "Blank"
- **Extra_Cavity（多余药片）**：显示红色 ROI 框，标注 "Extra_Cavity"
- **Missing（漏装）**：显示红色 X 标记，在 Slat Mode 下标注 "Empty"（而非普通模式的 "Missing"）

### 5.3 结果判定规则

在 Slat Mode 下，结果判定有以下特殊规则：

1. **Blank 不算缺陷**：所有位置都是 Blank 时，整体结果判定为 OK
2. **混合判定**：只要存在非 Blank 缺陷（如 Extra_Cavity、Missing），整体结果判定为 NG
3. **区块结果**：按 section 分组输出结果
