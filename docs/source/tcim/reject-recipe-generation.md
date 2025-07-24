# 踢废调试模板生成

## 常规模板加工成"有片踢没片不踢"

1. 将面积下限设置为2.0 (为了让所有合格片都报缺陷)
```yaml
# detectors#0.yaml
tablet_detector:
  use: 1
  parameters:
    front:
      area:
        is_use: true
        ub: 1.2
        lb: 2.0 # 0.9->2.0
```

2. 将 missing_return_ng 设为 0 （为了让NA返回Good结果）

```yaml
# conf_inspect_0.yaml
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

将所有detectors#.yaml中的```is_use```改为false（为了让所有片都不报缺陷）