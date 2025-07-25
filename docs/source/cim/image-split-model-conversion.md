# CIM 图像拆分 模型转换

[convert_cim_params](image-split-model-conversion/convert_cim_params.exe)

## 用法

```shell
convert_cim_params.exe [-h] -n NUMBER -x OFFSETX [OFFSETX …] -w IMG_WIDTH [IMG_WIDTH …] input
```

| 参数 | 说明 |
| --- | --- |
| input | 待拆分的params#.yaml文件路径 |
| -n, --number | 要拆分成几份？ |
| -x, --offsetx | 每份拆分的零点在原始图像中的X轴偏移量，用空格分开 |
| -w, --img-width | 每份拆分的图像宽度，用空格分开 |

### 示例

要把模板文件 params0.yaml，拆分为3份。每份的x偏移量为 0, 682, 1364，每份图片的宽度为 682, 682, 684：

```shell
.\convert_cim_params.exe .\params0.yaml --number 3 --offsetx 0 682 1364 --img-width 682 682 684
```

得到三个文件： params0.0.yaml, params0.1.yaml, params0.2.yaml