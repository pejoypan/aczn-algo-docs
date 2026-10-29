# Tally / Tally Vote 命令

`tally` 和 `tally_vote` 用于「先收集证据，再统一裁决」的检测场景：

- 透明空胶囊印字有无：单次曝光看不到印字，不等于没有印字
- CapsCanada wrinkle：一圈都出现才算 NG，单次曝光判定过严
- 其他需要综合多张图、多个 slice 或多次曝光结果的情况

常规 flow 的每条结果只保存最终 verdict。需要多拍投票时，先用 `tally` 把每拍的候选证据写入结果，再由后台把同一批结果拼起来，交给 `tally_vote` 做最终裁决。

```{important}
版本要求

- `tally`：aczn-algo ≥ 3.17.1
- `tally_vote`：aczn-algo ≥ 3.17.2
- 无图 flow 的合成缺陷图接口 `GetDefectImage(..., true)`：aczn-algo ≥ 3.17.3

完整的 vote 链路建议统一使用 **≥ 3.17.3**。
```

## 1. 核心概念

### 1.1 tally 是证据，不是裁决

`tally` 写入 `SingleResult::tally`，类型为 `map<string, int>`：

```yaml
- id: 0
  type: Good
  tally: {text: 4}
```

| 字段 | 含义 |
|------|------|
| `type` | 本拍 / 本通道的裁决，例如 `Good`、`No_Text@` |
| `tally` | 候选证据，`key -> 命中数` |
| `tally` 缺失 | 该结果没有提供计数证据：老 recipe，或该通道为 NA |

```{important}
- `tally` 不修改 `type`，也不绘制缺陷
- `SingleResult::operator+=` 会合并 verdict 和 `def_pos`，但**不会**合并 `tally`
- `tally` 为空时不会序列化到 YAML
- `{text: 0}` 和「没有 text 键」语义不同：前者表示已计数但未命中，后者表示没有证据
```

### 1.2 两个命令的 tally 含义不同

| 命令 | `tally` 的含义 |
|------|----------------|
| `tally` | 单个通道内，class 的 **box 命中数** |
| `tally_vote` | 单个通道内，**出现过该 key 的拍数** |

`tally_vote` 不关心单拍命中了几次：一拍内 `tally: {text: 5}` 也只算 1 票。

### 1.3 数据流

```
曝光 e0 .. eN
      │
      ▼
常规 flow：predict + tally
      │  每拍输出 SingleResult（含 tally）
      ▼
后台按 [channel][shot] 拼装
      │
      ▼
vote flow：tally_vote
      │
      ▼
最终结果：id / type / def_pos（不含 tally）
```

## 2. tally 命令

### 2.1 示例

```yaml
    - tally:
        input: [text0, text1, text2, text3, text4, text5]
        idxs: [0, 1, 2, 3, 4, 5]
        keys: [text]
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `input` | sequence | ✅ | — | `InferObject` 名称列表，每个输入一项（通常对应一个 slice） |
| `idxs` | sequence | - | `[0, 1, ...]` | 每个输入映射到 `m_result_` 的哪个通道；长度必须与 `input` 一致 |
| `keys` | sequence | - | 无 | 启用「列出的 key」模式；必须是非空字符串序列 |

### 2.2 两种计数模式

**count-all 模式**（不配置 `keys`）

- 统计 `InferObject::m_outputs` 中每个非空 `class_name`
- 不做零初始化
- 没有命中时 `tally` 为空

```yaml
    - tally:
        input: [text0]
```

**listed-keys 模式**（配置 `keys`）

- 对每个被引用且非 NA 的通道，先把所有列出的 key 置为 `0`
- 只统计列出的 class，其他 class 忽略
- 没有命中时仍输出 `{text: 0}`

```{note}
`keys` 中的字符串必须与模型输出的 `class_name` 完全一致（大小写敏感），
也就是训练时 names 中指定的名字。
```

```yaml
    - tally:
        input: [text0]
        keys: [text, wrinkle]
```

```{tip}
下游需要区分「0」和「没有证据」时，使用 `keys`。
例如约定「有胶囊、无检出」也要输出 `tally: {text: 0}`，就必须配置 `keys`。
```

### 2.3 前置条件：被计数的 class 必须是非缺陷

`tally` 读取的是 `InferObject::m_outputs`。`predict` 只会把**非缺陷 class** 放进 `outputs`：

- `defect_policy: none`：所有 class 都是非缺陷
- `defect_policy: {text: false}`：`text` 是非缺陷，其余仍是缺陷
- `defect_policy: normal`：所有 class 都是缺陷，`outputs` 为空，`tally` 不会命中

因此需要计数的 class（例如 `text`、`wrinkle`）必须在 `predict` 中配置为非缺陷：

```yaml
    - predict:
        INTERNAL_ENGINE_KEY: text_infer
        model: korea_print
        src: [image]
        defect_policy:
          text: false
        outputs: [text0]
        def_src: image
        def_img: image_defect
```

同时，`predict` 的 `infer_detector` 过滤仍然生效：只有通过 `confidence`、`area`、`shortside` 等过滤的预测才会被 `tally` 统计。

### 2.4 完整示例：每次曝光输出 per-channel tally

以下示例假设一张图有 6 个 capsule 通道，每个通道一个 slice：

```yaml
flow:
  commands:
    - predict:
        INTERNAL_ENGINE_KEY: text_infer
        model: korea_print
        src: [image]
        slices:
          - [x0, y0, w, h]
          - [x1, y1, w, h]
          - [x2, y2, w, h]
          - [x3, y3, w, h]
          - [x4, y4, w, h]
          - [x5, y5, w, h]
        idxs: [0, 1, 2, 3, 4, 5]
        outputs: [text0, text1, text2, text3, text4, text5]
        defect_policy:
          text: false
        def_src: image
        def_img: image_defect

    - tally:
        input: [text0, text1, text2, text3, text4, text5]
        idxs: [0, 1, 2, 3, 4, 5]
        keys: [text]
```

输出示例：

```yaml
- id: 0
  type: Good
  tally: {text: 2}
- id: 1
  type: Good
  tally: {text: 0}
- id: 2
  type: "No Capsule"
  # NA 通道没有 tally
```

### 2.5 行为细节

- 多个 `input` 映射到同一个 `idxs` 时，命中数累加
- `results[ch].NA()` 为真时跳过该通道：不预填 0，也不累加
- 输入缺失或类型不对时按空对象处理；listed-keys 模式仍会为该通道预填 0
- `idxs` 越界会直接失败
- `input` 为空、`idxs` 长度不匹配、`keys` 格式非法都会导致命令失败
- `tally` 从不修改 `type`，也不会修改 `m_foundDefect`

## 3. tally_vote 命令

`tally_vote` 是一个**无图 flow**：不读取 `load_image`，输入是后台拼好的多拍结果。

### 3.1 vote flow 示例

```yaml
# conf_inspect-vote.yaml
init:
  mode: inspect
  test_mode: 0
  missing_return_ng: 1
flow:
  commands:
    - tally_vote:
        INTERNAL_ENGINE_KEY: vote
        rules:
          - key: text
            valid_num: {ge: 1}
            defect: No_Text
```

```yaml
# params.yaml
tally_vote:
  num_channels: 6
```

```{important}
`tally_vote.num_channels` 决定 `m_result_` 的通道数，必须等于输入结果的通道数。
不配置时 `m_num` 默认为 1，6 通道输入会报 `channels 6 != results 1`。
```

### 3.2 输入格式

外层是通道，内层是同一通道的多次曝光（shot）：

```yaml
- - {id: 0, type: Good, tally: {text: 2}}   # ch0 shot0（e0）
  - {id: 0, type: Good, tally: {text: 0}}   # ch0 shot1（e1）
  - {id: 0, type: "No Capsule"}             # ch0 shot2（NA，无 tally）
  # ... 共 6 次曝光
- - {id: 1, type: Good, tally: {text: 0}}   # ch1 ...
  # ...
```

约束：

- 外层数量等于通道数，也等于 `m_result_.size()`
- 每个通道的 shot 数必须一致
- `shots[ch][s].id` 必须等于 `ch`
- shot 数不能为 0
- 输入不能为空

### 3.3 rules 参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `key` | scalar | ✅ | 必须与 `tally` 中的键名一致；同一条 flow 内不能重复 |
| `valid_num` | map | ✅ | **通过条件**，只能包含一个比较符 |
| `defect` | scalar | ✅ | 条件不满足时追加的缺陷名 |

`valid_num` 的写法：

| 写法 | 通过条件 |
|------|----------|
| `{eq: 2}` | `hits == 2` |
| `{ge: 1}` | `hits >= 1` |
| `{gt: 1}` | `hits > 1` |
| `{le: 2}` | `hits <= 2` |
| `{lt: 4}` | `hits < 4` |
| `{range: [1, 4]}` | `1 <= hits <= 4` |

所有值必须 ≥ 0，`range` 必须满足 `min <= max`。

```{warning}
`valid_num` 是**通过条件**，不是缺陷条件。
例如「6 拍里至少 1 拍看到 text 才算通过」对应 `valid_num: {ge: 1}` 和 `defect: No_Text`；
命中 0 拍时条件不满足，才会追加 `No_Text`。
```

### 3.4 投票逻辑

对每个通道依次执行：

1. 排除 NA shot（`No Capsule`、`No_Tablet`、`pack missing`、`NA` 前缀）
2. 如果所有 shot 都是 NA：直接沿用第一个 shot 的结果（清空 `tally`），跳过规则
3. 如果该通道所有 shot 都没有 `tally`：跳过规则（兼容老 recipe）
4. 合并有效 shot 的 verdict；如果已经是 NG：保留原判，清空 `tally`，跳过规则
5. 对每条规则：统计 `tally[key] > 0` 的 shot 数，得到 `hits`
   - 条件满足：通过
   - 条件不满足：按规则顺序追加 `defect`
6. 输出 `id / type / def_pos`，**不输出 `tally`**

```{note}
- 一拍对一个 key 最多贡献 1 票，`tally` 的数值大小被忽略
- 缺少 `tally` 的单拍按 0 票处理
- 只要该通道有任意一拍带 `tally`，就会进入投票；新旧 recipe 混用时要注意这一点
- 条件在 `[0, shot 数]` 上恒真或恒假时，运行期会打印 warning
```

### 3.5 输出

```yaml
- id: 0
  type: Good
- id: 1
  type: No_Text@
# ... 共 6 通道
```

- 条件不满足时，`defect` 追加到 `type` 末尾，自动补 `@`
- 多条规则同时失败时，按 rules 顺序追加，例如 `A@B@`
- `def_pos` 来自参与合并的有效 shot 的并集
- 输出不含 `tally`

### 3.6 缺陷图

`tally_vote` 不绘制缺陷框。无图 flow 中如果最终结果非 `Good`，调用：

```cpp
std::vector<std::pair<std::string, ImageObject>> defects;
std::string tag;
bool ok = processor.GetDefectImage(defects, tag, /*allow_synthetic=*/true);
```

- `allow_synthetic = true` 时，没有缺陷画布会返回一张 320x320 BGR 的合成占位图，`pair.first` 为 `"DEFECT"`
- 结果全为 `Good` 时仍返回 `false`
- 默认 `allow_synthetic = false`，老调用方行为不变
- 该接口需要 aczn-algo ≥ 3.17.3

### 3.7 C++ 对接

```cpp
InspectProcessor vote(InspectProcessor::INSPECT, "vote");
// params / detectors 可沿用原 recipe；params 中需补 tally_vote.num_channels
if (!vote.Initialize(flow_yaml, params_yaml, detectors_yaml, "", InspectProcessor::INSPECT))
    return false;

// 每次投票前必须清理上一批的状态
vote.cleandata();

if (!vote.Execute(YAML::Load(shots_yaml)))
    return false;

const ImageResult& results = vote.getResult();
```

```{warning}
复用同一个 `InspectProcessor` 时，`Execute()` 本身不会清除 `m_foundDefect`。
如果上一批投票已经产生缺陷，下一批 `executeFlow()` 可能因为 `inspectionDone()` 提前返回，
导致结果保持全 `Good`。每次 `Execute()` 前调用 `cleandata()`。
```

## 4. 依赖关系与常见问题

### 4.1 常规 flow 与 vote flow 的约定

| 常规 flow 输出 | vote flow 行为 |
|----------------|----------------|
| 所有 shot 都没有 `tally` | 跳过规则，沿用合并后的 verdict（老 recipe） |
| 至少一拍有 `tally` | 进入投票；没有 `tally` 的拍按 0 票 |
| NA shot | 不参与投票 |
| 整通道全 NA | 原样输出 NA，不判 NG |
| 通道已经 NG | 保留原判，不再追加投票缺陷 |

### 4.2 常见错误

| 现象 | 原因 / 处理 |
|------|-------------|
| `tally` 一直为空 | `predict` 的 `defect_policy` 把目标 class 当成了缺陷；改成 `none` 或 `{text: false}` |
| `tally` 没有 `{text: 0}` | 使用了 count-all 模式；配置 `keys: [text]` |
| `channels 6 != results 1` | 没有配置 `tally_vote.num_channels` |
| `id 0 != channel 1` | shot 的 `id` 与外层通道顺序不一致 |
| `ragged shots at ch 0` | 各通道 shot 数不一致 |
| 规则从未触发 | `valid_num` 是**通过条件**，检查比较符方向；同时看运行期 constant rule warning |
| 规则总是触发 | 同上，条件可能恒假 |
| 已 NG 但 `found_defect` 为 false | 该 NG 来自 shot 原有 verdict，不是投票追加；最终判定以 `getResult()` 为准 |
| 无图 NG 但拿不到缺陷图 | `GetDefectImage` 需要显式传 `allow_synthetic = true`，且版本 ≥ 3.17.3 |

### 4.3 参考

- 推理与 `outputs` / `defect_policy`：[Predict Command Tutorial](predict-command-tutorial.md)
- 后端对接原始契约：aczn-algo 仓库 `docs/contract/tally_vote_ros.md`
