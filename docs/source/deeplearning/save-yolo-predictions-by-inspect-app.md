# 如何通过inspect-app保存yolo预测结果

## Prerequisites
- aczn-algo >= d04986e: feat(infer_api): develop save_yolo
- aczn-inspect-app >= 74ec883: add build with gui

## Usage
```{note}
以单张图片为例，图片list同理
```

```shell
inspect_app -c <flow_yaml> -p <params_yaml> -i <image>.bmp --save_yolo
```

## Result Structure
```shell
├── image.bmp
├── image_3.png
├── image_3.txt
├── image_7.png
├── image_7.txt
├── conf_inspect.yaml
├── detectors0.yaml
└── obj.names

# obj.names:
Joints

# image_3.txt
0 0.171875 0.590726 0.322917 0.157258
```

```{important}
- 只有在有输出的时候才会保存
- 存储的图片与推理的input一致 （通常不是整张图）
- multi_thread模式下功能不可用
```

## 后续需要手动的操作

1. 创建 obj_train_data 文件夹，将所有图片和txt放入
2. 生成 train.txt （obj_train_data中所有图片的list）
3. 写 obj.data

```
# obj.data:
classes = 1
names = obj.names
train = train.txt
```
