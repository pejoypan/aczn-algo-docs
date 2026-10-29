# 如何通过inspect-app保存yolo预测结果

## Usage
```{note}
以单张图片为例，图片list同理。仅单线程可用。
```

```shell
# 默认：只导出有 YOLO 标签的 slice
inspect_app -c <flow_yaml> -p <params_yaml> -i <image>.bmp --save_yolo

# 可选：把无预测的 slice 也导出为空 txt（YOLO 背景/负样本）
inspect_app -c <flow_yaml> -p <params_yaml> -i <image>.bmp --save_yolo --save_yolo_keep_empty
```

| 选项 | 默认 | 说明 |
| --- | --- | --- |
| `--save_yolo` | off | 导出 post-filter 预测。默认 defect-only：只有产生 YOLO 标签的 slice 才会落盘。 |
| `--save_yolo_keep_empty` | off | 必须与 `--save_yolo` 同用。启用后，无预测的 slice 也保存空 txt 并进入 `train.txt`，作为背景/负样本。单独使用直接报错退出。 |
| `--yolo_dir` | `yolo` | bundle 根目录。`--save_yolo` 下为空串则报错退出。 |

## Result Structure
```shell
<yolo_dir>/
├── obj.names
├── obj.data
├── train.txt
└── obj_train_data/
    ├── image_3.jpg
    ├── image_3.txt
    ├── image_7.jpg
    └── image_7.txt

# obj.names:
Joints

# obj.data:
classes = 1
names = obj.names
train = train.txt

# image_3.txt
0 0.171875 0.590726 0.322917 0.157258
```

```{important}
- 默认只有产生 YOLO 标签的 slice 才会保存；`--save_yolo_keep_empty` 才会额外保存无预测的空样本
- 空 txt 是有意的 YOLO 背景/负样本，不是缺陷记录
- 有预测但无法用 YOLO 表达时（例如 polygon 点数不足），两种模式下都会跳过该 slice 并输出警告，不会退化成空样本
- 整轮没有任何样本时，不创建 `obj.names` / `obj.data` / `train.txt`，也不创建 bundle 目录
- 存储的图片与推理的input一致 （通常不是整张图）
- `multi_thread` 模式下功能不可用
```

`obj.names`、`obj.data`、`train.txt` 和 `obj_train_data/` 由 inspect-app 自动生成，不需要手工补齐。
