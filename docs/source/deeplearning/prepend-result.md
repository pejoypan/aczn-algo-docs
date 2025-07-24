# 深度缺陷结果前置功能

## 主要功能
将深度算法检测出来的结果，在添加到最终结果时，插入到顶部，而不是默认的追加到尾部

## 适用场景
适用于需要将深度算法检测结果优先展示的场景，例如对结果归类统一性要求高的客户

## 开启方法
```{important}
前置条件：算法版本≥3.8.9
```

```yaml
    - async_infer_submit:
        INTERNAL_ENGINE_KEY: async_engine
        which_detector: infer_detector
        model: taiguo1_20250312232627
        uuid: 0
        prepend_result: true # 增加此项并设为 true
        src: [0a, 0b, 1a, 1b, 2a, 2b, 3a, 3b, 4a, 4b, 5a, 5b]
        skip_all_NA: true
        just_infer: false
        idxs: [0, 0, 1, 1, 2, 2, 3, 3, 4, 4, 5, 5]
```

```{note}
- 默认为关闭状态
- 若使用同步命令，接口相同
```
