强踢参数调节说明
=============


# 强踢参数调节说明

## 原理

由算法判断当前目标是否为"**半粒**"或"**多对象**"情况，并给缺陷结果打标记，后续将根据标记触发相应的强踢动作

## 用法

在当前模板的```conf_inpsect.yaml```中的```init```节点，增加```force_kick```节点，如下

```yaml
init:
  mode: inspect
  parameter: params0.yaml
  detectors: detectors0.yaml
  result_yaml: result.yaml
  test_mode: 0
  missing_return_ng: 1
  force_kick:    # 强踢判定条件
    lb: 0.8      # 下界
    ub: 1.3      # 上界
```

定义 **ratio** = 面积 or 高度 or 宽度 的 当前值 / 标准值

- 如果 **ratio < lb**，则会在缺陷类型的头部增加 **"Half - "** 前缀
- 如果 **ratio > ub**，则会在缺陷类型的头部增加 **"Multiple - "** 前缀

## Tips

算法只负责添加前缀，最终强踢是否执行**不**受算法控制