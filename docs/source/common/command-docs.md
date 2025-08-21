# Command Usage

This document records the image processing commands, as well as the optional parameters, parameter types, default values, data ranges, precautions and other information that may need to be entered in each processing flow.

## load_image

读取单张图片

```yaml
    - load_image:
        image: image  # input
        path: xxxx    # loading path
```

- image: (*required*) [scalar] 
- path: (*required*) [scalar] specify image loading path.

## load_images

读取图片列表

```yaml
    - load_images:
        images: [image, image2]  # input
        paths: [XX/XX/XX,XX/XX/XX]    # loading path
```

- images: (*required*) [sequence]
- paths: (*required*) [sequence] specify images loading paths.

## save_image

保存图片

```yaml
    - save_image:
        image: image  # input
        out_file: "X:/xxx/xxx/文件名" # output path
        tag: abc    # image label
        with_uuid: true # uuid switch
        format: jpg/png # extension
```

- image: (*required*) [scalar]
- out_file: (*optional*) [scalar] [default: current directory/xx-image_xx.png] / or \ ? should be /, (will still add 'png').
- with_uuid: (*optional*) [scalar] [default: empty] [bool] 如果开启此功能，会在图片的名字基础上增加uuid.
- format: (*optional*) [scalar] [default: png] 支持的类型：[bmp, jpg, png, ...].

## clone_image

图片拷贝

```yaml
    - clone_image:
        src: image  # input
        dst: image   # output
```
- src/dst: (*required*) [scalar]. 

## show_image

图片显示

```yaml
    - show_image:
        src: image   # input
```
- src: (*required*) [scalar].

## normalize

归一化


```yaml
    - normalize:
        src: image  # input       
        dst: image  # output    
        bounds: [lb,ub] # lower and upper bounds of the normalized data
```
- src/dst: (*required*) [scalar].   
- bounds: (*optional*) [sequence] [default: [0,255]]. 

## cvt_color

修改图片颜色空间

```yaml
    - cvt_color:
        src: image  # input       
        dst: image  # output
        dst_type: GRAY/BGR/RGB/HSV/...   # The color space type of the image
```

- src/dst: (*required*) [scalar].
- dst_type: (*required*) [scalar] the color space type should correspond to the number of image channels.

## histogram_equalizer

直方图均衡化

```yaml
    - histogram_equalizer:
        src: image  # input       
        dst: image  # output
```

- src: (*required*) [scalar] [grayscale image].
- dst: (*required*) [scalar]. 

## threshold_image 

图像阈值化

```yaml
    - threshold_image:
        src: image  # input       
        dst: image  # output
        tag: abc # thresh_str label
        thresh: digit # threshold used for binarization
        auto_mode: otsu/triangle/huang # method for automatically selecting thresholds
        type: binary/binary_inv/otsu/triangle/tozero/tozero_inv/trunc/huang # thresholding method
```

- src/dst: (*required*) [scalar].
- tag: (*optional*) [scalar] [default : empty]. 
- auto_mode: (*optional*) [scalar] [default : otsu], valid only when type is empty and thresh <= 0 and srcobj.m_data.channels() == 1.
- thresh: (*required*) [scalar].
- type: (*optional*) [sequence] [default : empty] sequence size should <= 2,requires srcobj.m_data.channels() == 1.

## dynamic_threshold

图像动态阈值化

```yaml
    - dynamic_threshold:
        src: image  # input       
        dst: image  # output
        mask: image # mask image 
        tag: abc # str_params label
        params: 
          ksize: [XX, XX] # mean filter kernel size
          light_dark: dark/light 
```
- src/dst: (*required*) [scalar].
- mask: (*optional*) [scalar] [default : empty].
- tag: (*optional*) [scalar] [default : empty]. 
- params: (*required*) [map].
- ksize: (*required*) [sequence].
- light_dark: (*optional*) [scalar] [default : dark].

## fill_image

图像元素填充

```yaml
    - fill_image:
        src: image  # input       
        dst: image  # output
        mask: image # mask image 
        scalar: [XX，XX,XX] # fill color
        tag: abc # scalar label
```
- src/dst: (*required*) [scalar]. 
- sclar: (*optional*) [scalar or sequence] [default : empty] the size of Sequence should >=3.
- mask: (*optional*) [scalar] [default : empty]，the size of mask and src must be the same.
- tag: (*optional*) [scalar] [default : empty]. 

```{note}
sclar and mask must have at least one. 
```

## image_inrange

图像像素值范围限定以及颜色空间转换

```yaml
    - image_inrange:
        src: image  # input       
        dst: image  # output
        tag: abc    # color_range or colorspace label 
        color_range: [GRAY/BGR/...,XX,XX] 
        or [GRAY/BGR/...,XX,XX,XX,XX,XX,XX] #The color space type of the image and range sequence
        colorspace: GRAY/BGR/HSV/... #The color space type of the image
        colorspace_range: [XX,XX] or [XX,XX,XX,XX,XX,XX] # range sequence

```
- src/dst:(*required*) [scalar]. 
- tag: (*optional*) [scalar] [default : empty]. 
- color_range: (*optional*) [sequence] [default : empty]，the sequence must be 3 or 7 pieces of data, the first of which is the color space type.
- colorspace: (*optional*) [scalar] [default : HSV]
- colorspace_range: (*optional*) [sequence] [default : empty]，the sequence must be 2 or 6 pieces of data, can be GRAY_range/BGR_range/HSV_range/...

```{note}
color_range and colorspace_range must have at least one. 
```

## image_roi

图像ROI区域提取

```yaml
    - image_roi:
        src: image  # input      
        dst: image  # output
        tag: abc    # ROI label 
        ROI: [[XX,XX,XX,XX]] # Each sequence includes the top-left coordinates of the ROI region and the length and width
```
- src/dst: (*required*) [scalar].
- tag: (*optional*) [scalar] [default : empty].
- ROI: (*required*) [sequence]，must be a two-dimensional sequence，each subsequence must have 4 elements.

## compare_images

图像比较

```yaml
    - compare_images:
        lhs: image   # the first comparison image
        rhs: image   # the second comparison image
        dst: image   # the result of comparison
        offset: digit # the offset for the second image 
        cmpop: eq/gt/ge/lt/le/ne/==/>/>=/</<=/!= # the type of compare operation
```
- src/dst/dst: (*required*) [scalar].
- offset: (*optional*) [scalar] [default : 0.0].
- cmpop: (*optional*) [scalar] [default : lt or < ].

```{note}
lhs and rhs must have the same size. 
```

## image_smoothing

图像平滑

```yaml
    - image_smoothing:
        src: image  # input      
        dst: image  # output
        tag: abc    # params label 
        params: 
          type: gaussian/median/custom_sym/... # filter type
          dst_size: XX or [XX, XX] # pyrdown times or size for output image
          kernel_size: XX or [XX, XX] # filter size
          sigma: XX or [XX, XX] 
          theta: XX 
          lambda: XX
          gamma: XX
          kenel: [XX, XX, XX, ...]
```
- src/dst: (*required*) [scalar].
- tag: (*optional*) [scalar] [default : empty].
- params: (*required*) [map].
  - type：(*required*) [scalar].
  - dst_size: (*required when node["type"] is pyrdown*) [scalar or sequence]; If dst_size is scalar, it represents pyrdown times; if dst_size is sequence, it represents the size for output image，and requires |dst_size[0] - src.width| <= 2，|dst_size[2] - src.height| <= 2，the size of sequence must be 2.
  - kernel_size: (*required when node["type"] is not pyrdown*) [scalar or sequence],must be odd,if scalar, the kernel will be [kernel_size, kernel_size];if sequence, the size of sequence must be 2.
  - sigma: (*optional*) [scalar or sequence] [default: 0 when type=gaussian or bilateral; when type=gaussian, default value is calculated using kernel_size],specifies sigma only when type=gaussian or bilateral or gabor.
  - theta: (*optional*) [scalar] [default : 0] specifies sigma only when type=gabor.
  - lambda: (*optional*) [scalar] [default : default value is calculated using kernel_size] specifies sigma only when type=gabor.
  - gamma: (*optional*) [scalar] [default : 0] specifies sigma only when type=gabor.
  - kernel: (*required when node["type"] is custom_sym or custom_asym*)

## image_gradient

图像梯度运算

```yaml
    - image_gradient:
        src: image  # input      
        dst: image  # output
        params: 
          type: sobel/scharr/... # gradient type
          kernel_size: digit # filter size
```
- src/dst: (*required*) [scalar].
- params: (*required*) [map].
  - type：(*required*) [scalar], currently only support: sobel, scharr.
  - kernel_size: (*required*) [scalar], scharr must use kernel size 3.

## morphology_op

图像形态学操作

```yaml
    - morphology_op:
        src: image  # input      
        dst: image  # output
        op: dilate/erode/open/close/gradient/tophat/blackhat # morphological operation type
        tag: abc # kernel_size or kernel_shape label
        kernel_size: XX, [XX, XX] # filter size
        kernel_shape: rect/ellipse/cross # type of structural element of morphological operation
```
- src/dst: (*required*) [scalar].
- op: (*required*) [scalar]. 
- tag: (*optional*) [scalar] [default : empty].
- kernel_size: (*optional*) [scalar or sequence] [default : [3, 3]].
- kernel_shape: (*optional*) [scalar] [default : rect].

## omni_morph
指定核类型和尺寸的图像形态学操作

```yaml
    - omni_morph:
        src: image  # input      
        dst: image  # output
        tag: abc    # params label 
        params: 
          ops: dilate/erode/close/gradient # morphological operation type
          shapes: rect/ellipse/cross # kernel type
          sizes: [[XX,XX]] ？ #kernel size
```
- src/dst: (*required*) [scalar].
- tag: (*optional*) [scalar] [default : empty].
- params: (*required*) [map].
  - ops: (*required*) [sequence] should be sequence but without check.
  - shapes: (*optional*) [sequence] [default : rect] should be sequence but without check.
  - sizes: (*required*) [sequence] should be two-dimensional sequence. 

## pixel_op

图像像素级操作

```yaml
    - pixel_op:
        src: image  # input
        src1: image  # input
        src2: image  # input 
        dst: image  # output
        mask: image # mask image
        op: not/abs/scaled_abs/offset,add/subtract/absdiff/and/or/xor
        params:
          alpha: digit
          beta: digit
          gamma: digit
```
- src: (*required when op is not/abs/scaled_abs/offset *) [scalar].
- src1/src2: (*required when op is add/subtract/absdiff/and/or/xor *) [scalar].
- dst: (*required*) [scalar].
- mask: (*optional*) [scalar] [default: NULL], mask image should be single channel 8-bit image.
- op: (*required*) [scalar]. 
- params: (*optional*) [map] 
  - alpha: (*optional*) [scalar] [default: 1.0]
  - beta: (*optional*) [scalar] [default: 1.0]
  - gamma: (*optional*) [scalar] [default: 0.0]

## convex_contour

图像轮廓的多边形凸包绘制

```yaml
    - convex_contour:
        src: image  # input
        dst: image  # output
```
- src/dst: (*required*) [scalar], src must be grayscale image .

## convexity_detect

凸包缺陷检测

```yaml
    - convexity_detect:
        src: image  # input
        def_img: image 
        save_defect: digit
        def_src: image
```

- src: (*required*) [scalar].
- def_img: (*optional*) [scalar]
- save_defect: (*optional*) [scalar] [default: empty].
- def_src: (*optional*) [scalar] [default: NULL].

## get_L

将图像从 RGB 色彩空间转换到 CIE Lab 色彩空间，并提取 L* 分量。

```yaml
    - get_L:
        src: image  # input
        dst: image  # output
```
- src/dst: (*required*) [scalar].

## dilate_circle

使用直接绘制圆的方式，替代使用圆形卷积核dilate，从而提高效率

```yaml
    - dilate_circle:
        src: image  # input
        dst: image  # output
        kernel_size: digit 
        tag: abc # kernel_size label
```
- src/dst: (*required*) [scalar]，src image must be binary image.
- kernel_size：(*optional*) [scalar or sequence] [default: 0], kernel_size is perferred to be an odd integer, if sequence,should kernel_size[0] == kernel_size[1].
- tag: (*optional*) [scalar] [default : empty].

## rotate_image

旋转图像

```yaml
    - rotate_image:
        src: image  # input
        dst: image  # output
        tag: abc # degreename label
        degree: 90/180/270/-90 # rotation degree
```

- src/dst: (*required*) [scalar].
- tag: (*optional*) [scalar] [default : empty].
- degree: (*required*) [scalar]  

## clean_salt

用于消除椒盐噪声

```yaml
    - clean_salt:
        src: image  # input
        dst: image  # output
        tag: abc # min_area label
        min_area: digit # 
```
- src/dst: (*required*) [scalar],src image must be binary image.
- tag: (*optional*) [scalar] [default : empty].
- min_area: (*optional*) [scalar] [default : 0],prefer min_area  > 0.

## fill_holes

填补图像空洞

```yaml
    - fill_holes:
        src: image  # input
        dst: image  # output
        tag: abc # max_area label
        max_area: digit # 
```
- src/dst: (*required*) [scalar],src image must be binary image.
- tag: (*optional*) [scalar] [default : empty].
- max_area: (*optional*) [scalar] [default : 0], prefer max_area > 0.

## crop_image

图片裁剪

```yaml
    - crop_image:
        src: image  # input
        dst: image  # output
        mode: THRESH_OTSU/THRESH_TRIANGLE # threshold selection method for image binarization
        roi: [XX,XXX,XX,XX] # clipping region
        dst_size: [XX,XX] # output image size
```
- src/dst: (*required*) [scalar].
- mode: (*optional*) [scalar] [default: triangle], currently support ostu/triangle.
- roi: (*optional*) [sequence] [default : empty] should be less than the length and width of the src image.
- dst_size: (*optional*) [sequence] [default : [src.cols, src.rows]] should be less than the length and width of the src image.

## extract_objects

提取前景目标

```yaml
    - extract_objects:
        bgr_src: image       # input
        bin: image           # input
        dst: [ch0, ch1, ch2] # output
        xywh:
          - [0.000, 0.2, 0.333, 0.6]
          - [0.333, 0.2, 0.333, 0.6]
          - [0.667, 0.2, 0.333, 0.6]
        # rois:
        #   - [x1, y1, x2, y2]
        #   - [x1, y1, x2, y2]
        #   - [x1, y1, x2, y2]
        margin: 5  # [5, 5, 10, 10] # [top, bottom, left, right]
```
|         | required? | description                                                  | format                                              |
| ------- | --------- | ------------------------------------------------------------ | --------------------------------------------------- |
| bgr_src | ✅         | 用于被裁切的三通道原图                                       | scalar                                              |
| bin     | ✅         | 前景的二值化图像                                             | scalar                                              |
| dst     | ✅         | 用于存放裁切的结果                                           | sequence                                            |
| xywh    | ⚠️         | 和`rois`至少要有一个存在，优先级高<br />x: roi左上角x坐标的相对值 (0.0~1.0)<br />y: roi左上角y坐标的相对值 (0.0~1.0)<br />w: roi总宽度的相对值 (0.0~1.0)<br />h: roi总高度的相对值 (0.0~1.0) | - [x, y, w, h]<br />...<br />- [x, y, w, h]         |
| rois    | ⚠️         | 和`xywh`至少要有一个存在，优先级低<br />x1: roi左上角点x坐标的整数绝对值<br />y1: roi左上角点y坐标的整数绝对值<br />x2: roi右下角点x坐标的整数绝对值<br />y2: roi右下角点y坐标的整数绝对值 | - [x1, y1, x2, y2]<br />...<br />- [x1, y1, x2, y2] |
| margin  |           | 可以是整数scalar，此时上下左右margin相同<br />也可以是整数sequence，顺序为 [top, btm, left, right] | scalar or sequence                                  |

```{note}
`xywh` 在 3.9.7 版本开始引入
```

## warp

对图像的部分区域应用 warpAffine

```yaml
    - warp:
        src: image
        dst: image_dst
        params:
          dst_size: [1024, 480]
          src_rrect:
            - [270, 290, 480, 224, -45.0]   # tl
            - [1682, 276, 480, 224, 45.0]   # tr
            - [270, 1672, 480, 224, -135.0] # bl
            - [1676, 1665, 480, 224, 135.0] # br
          # src_pts:
          #   - [[x1, y1], [x2, y2], [x3, y3]]
          dst_rect:
            - [0, 0, 480, 224]
            - [544, 0, 480, 224]
            - [0, 256, 480, 224]
            - [544, 256, 480, 224]
          # dst_pts:
          #   - [[x1', y1'], [x2', y2'], [x3', y3']]
```

|           | required? | description                        | format                                   |
| --------- | --------- | ---------------------------------- | ---------------------------------------- |
| dst_size  | ✅         | The size of dst image              | [w, h]                                   |
| src_rrect | ✅         | The list of cv::RotatedRect on src | [x, y, w, h, ɵ]                          |
| src_pts   | ✅         | The list of points on src          | [[x1, y1], [x2, y2], [x3, y3]] (size==3) |
| dst_rect  | ✅         | The list of Rect on dst            | [x, y, w, h]                             |
| dst_pts   | ✅         | The list of points(size:3) on dst  | [[x1, y1], [x2, y2], [x3, y3]] (size==3) |

## background_subtract

基于背景减法的运动前景分割，用于从图像序列中提取运动前景目标。该算法通过建立背景模型，将当前帧与背景模型进行比较，检测出前景运动物体。

```yaml
    - background_subtract:
        src: image            # input, should be 8-bit
        dst: fg_mask          # output foreground, 8-bit binary
        params:
          type: MOG2          # algo types, support: [MOG2, KNN]
          history: 500        # last N frames that affect background model
          var_threshold: 20   # variance threshold for background model
          n_mixtures: 2       # number of Gaussian components in the background model
          learning_rates: [1.0, 0.95, 0.8, 0.6, 0.4, 0.25, 0.135, 0.065, 0.03, 0.01] 
          detect_shadows: false     # shadow detection
          min_learning_rate: 0.001  # the stable learning rate after learning_rates
          pre_learn_datas:          # optional. If set, will pre-learn from with these datas
            - D:/image_data/v-counter/1/0000.bmp
```

| 参数名 | 类型 | 取值范围 | 默认值 | 描述 |
|--------|------|----------|--------|------|
| **type** | string | [MOG2, KNN] | MOG2 | 背景建模算法类型 |
| **history** | int | [1, 10000] | 500 | 用于训练背景模型的历史帧数，值越大背景更新越慢 |
| **var_threshold** | float | [1, 1000] | 16.0 | 方差阈值，用于像素分类为前景/背景，值越小检测越敏感 |
| **n_mixtures** | int | [1, 5] | 2 | 高斯混合模型中的高斯分量数量 |
| **learning_rates** | array | [0.0, 1.0] | [1.0] | 初始学习率序列（假定刚开始的时候通常没有运动目标） |
| **min_learning_rate** | float | [0.0, 1.0] | 0.001 | 初始学习率之后的稳定学习率 |
| **pre_learn_datas** | array | - | [] | 预训练图像路径列表，用于初始化背景模型 |
| **detect_shadows** | bool | - | false | 是否检测阴影 |

在最初始的N帧(N==learning_rates.size)中，模型会根据 learning_rates 对每一帧进行学习。学习完成后，模型会根据 min_learning_rate 进行稳定学习。

如果存在 pre_learn_datas, learning_rates 也会被消耗