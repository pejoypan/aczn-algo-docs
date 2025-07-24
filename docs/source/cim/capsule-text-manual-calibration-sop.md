# 字符检手工建模SOP

## 前置条件

1. 已有对应的传统模板
2. 转速与拍照间隔已调整至最佳
3. 有1000粒左右的数据集

```shell
│  cal_params0-printing.yaml
│  cal_params0.yaml
│  conf_cal-printing.yaml
│  conf_cal.yaml
│  conf_inspect-printing.yaml
│  conf_inspect.yaml
│  detectors0.yaml
│  list_images.py
│  params0.yaml
│  params1.yaml
│  params2.yaml
└─data
    ├─1
    ├─2
    └─3
```

## 准备unwrap图

### 跑图片生成unwrap图

```shell
inspect_app.exe -c conf_inspect.yaml -p params0.yaml -f data/1 --NA_return_NG=false --save_unwrap=true --save=false
```

```{important}
此时应该把缺陷检测放松，只检测很大的缺陷（待开发）
```

```shell
mkdir text
move *unwrap* text
```

将所有unwrap图移动到一文件夹中

## 建模

### 找一组正常的图片用于建模

一张图片为正常的标准为，其对应的 unwrap.yaml 中的内容：

```
!any_of(box==[0,0,0,0]) && !any_of(box_whole==[0,0,0,0]) && !any_of(is_positive==-1)
```

该组中所有图片都应符合正常的标准

并生成对应的图片路径列表 text_cal.txt

### 改 cal_params-printing.yaml 初始参数

#### num_texts

```yaml
num_texts: [1, 1] # [囊帽数量, 囊体数量]
```

这里指的是单词数量

#### repeat_times

```yaml
repeat_times: [1, 1] # [囊帽重复次数，囊体重复次数]
```

#### morph_ksize

```yaml
morph_ksize: [25, 25] # 一般可不调 用默认
```

#### no_stitching_mode

```yaml
no_stitching_mode: 1 # 如果字符为横排，填0；如果字符为竖排，填1
```

竖排即单张图片下能看到完整字符

#### text_positions

```yaml
text_positions:
    upper_positive: 
        - [0.00, 0.51]
        - [0.55, 1.00]
    upper_negative:
        - [0.00, 0.45]
        - [0.49, 1.00]
    lower_positive:
        - [0.00, 0.50]
        - [0.54, 1.00]
    lower_negative:
        - [0.00, 0.43]
        - [0.47, 1.00]
```

定义字符可能出现的Y范围

#### darker_text

```yaml
darker_text: 1 # 如果字符比背景深填1，比背景浅填0
```

### 初次建模

#### 生成建模图片列表

```shell
inspect_app.exe --list_images=text,text.txt,png
copy text.txt text_cal.txt
sed -i "7,$ d" text_cal.txt   # 假设拍照次数为6
```

#### 建模

```shell
inspect_app.exe -c conf_cal-printing.yaml --list ./text_cal.txt --textmode=true -p cal_params0-printing.yaml -r params0-printing.yaml --times=6   # 假设拍照次数为6
```

将输出结果 xxx-textsrc.png 和 xxx-textdef.png

### 参数调优

调整以下参数，优化效果

```yaml
threshold: 30         # 用于提取字符
close_ksize: [15, 15] # 用于将单词中的字连接起来
open_ksize: [3, 3]    # 用于去除噪声
```

### 最终建模

确定好参数后 再次建模 

并将结果复制给其他相机

```shell
inspect_app.exe -c conf_cal-printing.yaml --list ./text_cal.txt --textmode=true -p cal_params0-printing.yaml -r params0-printing.yaml --times=6  # 假设拍照次数为6
copy params0-printing.yaml params1-printing.yaml
copy params0-printing.yaml params2-printing.yaml
```

