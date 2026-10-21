# Flow YAML Init Node

## Example

```yaml
init:
  mode: inspect
  parameter: ./params.yaml
  detectors: ./detectors.yaml
  result_yaml: ./result.yaml
  reference:
    - name: tmpl_main
      path: ./template/main.bmp
  test_mode: 0
  missing_return_ng: 1
  can_judge_NA: true
  use_infer_at_cal: false
  min_version: 3.15.3
  identifier: user-defined-string
  infer_models:
    - ./models/capsule_det.onnx
    - ./models/defect_seg.onnx
  force_kick:
    lb: 0.8
    ub: 1.25
  default_tablet_bbox:
    - top_left: [0, 0, 480, 240]
      top_right: [480, 0, 480, 240]
      btm_left: [0, 240, 480, 240]
      btm_right: [480, 240, 480, 240]

flow:
  commands:
    - load_image:
        image: image
    # ... other commands
```

## Field Reference

### mode
- **Required**: ❌
- **Type**: scalar (string)
- **Valid values**: `calibrate` | `cal` | `inspect` | `text_inspect` | `debug`
- **Description**: Sets the operating mode of the inspection pipeline. The value can also be overridden by the mode argument passed to `Initialize()` at runtime.
  - `calibrate` / `cal`: Calibration mode. The algorithm will generate reference parameters and save them to `result_yaml`.
  - `inspect`: Standard inspection mode.
  - `text_inspect`: Capsule text inspection mode.
  - `debug`: Debug mode with verbose logging and intermediate image saving.

### parameter
- **Required**: ✅ (when launched via `inspect_app`)
- **Type**: scalar (string)
- **Description**: Path to the parameter / design-template YAML file (`params.yaml`). This file contains model-specific calibration parameters and is loaded into `m_params` during initialization. It can be overridden by the `--param` CLI argument.

### detectors
- **Required**: ❌
- **Type**: scalar (string)
- **Description**: Path to the detector YAML file (`detectors.yaml`). This file defines the detection thresholds and rules used by various engines in the flow. It can be overridden by the `--detector` CLI argument.

### result_yaml
- **Required**: ❌
- **Type**: scalar (string)
- **Default**: `result.yaml`
- **Description**: Specifies the output YAML file path where calibration results (or final results in debug scenarios) are written.

### reference
- **Required**: ❌ (optional, but typically required in `inspect` mode)
- **Type**: sequence of maps
- **Description**: Loads template/reference images into the internal image container before the flow starts. Each item must contain:
  - `name`: the key used to store the image in the container
  - `path`: the file path to the template image (read as grayscale)

### test_mode
- **Required**: ❌
- **Type**: scalar (int)
- **Default**: `0` (disabled)
- **Description**: When set to a value greater than `0`, enables test mode. In test mode the processor skips early-exit optimizations (e.g. stopping on all-NG) and usually preserves more intermediate data for debugging.

### missing_return_ng
- **Required**: ❌
- **Type**: scalar (int)
- **Default**: `1` (enabled)
- **Description**: When enabled (`> 0`), any channel whose result is missing or `NA` will be treated as `NG`. This affects how `m_result_` is interpreted by downstream logic.

### can_judge_NA
- **Required**: ❌
- **Type**: scalar (bool)
- **Default**: `true`
- **Description**: Allows the processor to explicitly judge a channel as `NA` (Not Available). If set to `false`, the algorithm is forced to return either `Good` or a concrete defect type. This value may also be overridden by the same key in `params.yaml`.

### identifier
- **Required**: ❌
- **Type**: scalar (string)
- **Default**: empty string
- **Description**: A user-defined identifier string for the current inspection session. It is stored in `m_identifier` and typically used for logging or result tagging.

### use_infer_at_cal
- **Required**: ❌
- **Type**: scalar (bool)
- **Default**: `false`
- **Description**: When `true`, deep-learning inference models are loaded and executed even in calibration mode. By default (`false`), model initialization is skipped during calibration to save time.

### min_version / minimum_required
- **Required**: ❌
- **Type**: scalar (string)
- **Default**: empty (no check)
- **Description**: Minimum required algorithm library version, specified as a dot-separated string such as `"3.10.0"`. If the runtime library version is lower than the required version, initialization fails with an error. Both keys are accepted for backward compatibility; `min_version` takes precedence if both are present.

### infer_models
- **Required**: ❌ (required by `predict` & `async_infer_submit` command)
- **Type**: sequence of strings
- **Default**: empty
- **Description**: List of model file paths to register with the `infer_api` model manager at initialization time. Each path must point to an existing file. The model name (filename without extension) is used as the internal lookup key.

### force_kick
- **Required**: ❌
- **Type**: map
- **Default**: `{lb: 0.8, ub: 1.25}`
- **Description**: Thresholds used by capsule and tablet engines to force-tag a defect when a contour metric (area, width, or height ratio) deviates significantly from the reference. If the measured ratio is below `lb` or above `ub`, the defect type is prefixed with a category such as `Half - ` or `Multiple - `.
- **Sub-fields**:
  - `lb`: scalar (float), lower bound ratio. Must satisfy `0 < lb < 1`. Default `0.8`.
  - `ub`: scalar (float), upper bound ratio. Must satisfy `ub > 1`. Default `1.25`.

### default_tablet_bbox
- **Required**: ❌
- **Type**: sequence of maps
- **Default**: derived from `params.yaml` (`tablet.*.rois`) when available
- **Description**: Provides default tablet bounding boxes per channel before any detection runs. The sequence length must equal the number of tablet channels (`m_num`). Each map may contain orientation keys such as `front`, `top_left`, `top_right`, `btm_left`, `btm_right`, and `3D`, each mapping to a rectangle (`cv::Rect`).

## Params.yaml Overrides

Some fields in `init` can be supplemented or overridden by the corresponding `params?.yaml`:

| Field in params.yaml | Type | Description |
|----------------------|------|-------------|
| `can_judge_NA` | bool | Overrides the same key in `init`. |
| `img_size` | sequence `[w, h]` | Standard image size (`cv::Size`). |
| `tablet.num` | int | Number of tablet channels (TCIM model). |
| `counter.num_channel` | int | Number of counter channels (SIM model). |
| `capsule.num_capsules` | int | Number of capsule channels (CIM model). |
| `capsule_text.num_capsules` | int | Number of capsule text channels. |
| `capsule_new.num_capsules` | int | Number of channels for the new capsule model. |

The `m_num` internal variable is derived from the above model-specific fields and determines the size of the result array (`m_result_`).


