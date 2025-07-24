# 踢废调试模板生成

## 常规模板加工成"有片踢没片不踢"

detectors#0.yaml 

```yaml
tablet_detector:
  use: 1
  parameters:
    front:
      area:
        is_use: true
        ub: 1.2
        lb: 2.0 # 0.9->2.0
```

conf_inspect.yaml

```yaml
init:
  mode: inspect
  parameter: params00.yaml
  detectors: detectors00.yaml
  result_yaml: result.yaml
  test_mode: 0
  missing_return_ng: 0 # 1->0
  can_judge_NA: true
```



## 常规模板加工成"有片不踢没片踢"

所有detectors#.yaml中的```is_use```改为false