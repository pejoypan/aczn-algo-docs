# CIM 字符检参数说明

## conf_inspect-printing.yaml

```yaml
init:
  mode: text_inspect
  parameter: params0-printing.yaml
  detectors: detectors0.yaml
  result_yaml: result-text.yaml
  test_mode: 1
  missing_return_ng: 1
  can_judge_NA: false # 建议为false
flow:
  commands:
    - load_images: 
        images: [a, b, c, d, e, f]    # 输入，数组的size根据曝光次数调整
    
    - capsule_text_detect:
        src: [a, b, c, d, e, f]
        debug: 0
        output: blackspot_src         # 当启用capsule_text_blackspot_detect时，需确保输出该图像
        def_src: cap_defsrc
        def_img: text_defect
        INTERNAL_ENGINE_KEY: 0
    
    - capsule_text_blackspot_detect:
        src: blackspot_src
        debug: 0
        def_src: cap_defsrc
        def_img: text_defect
        INTERNAL_ENGINE_KEY: 1
```

## detectors0.yaml

```yaml
capsule_text_blackspot_detector:
  use: 1
  parameters:
    simple_blob_detector:       # 与其他的斑点检测器无异，参考其他文档
      thresholdStep: 1
      minThreshold: 6
      maxThreshold: 9
      minRepeatability: 2
      minDistBetweenBlobs: 5
      filterByColor: 1
      blobColor: 255
      filterByArea: 1
      minArea: 15
      filterByCircularity: 1
      minCircularity: 0.3
      filterByInertia: 1
      minInertiaRatio: 0.1
      filterByConvexity: 1
      minConvexity: 0.5
capsule_text_detector:
  use: 1
  parameters:
    align_ub:    # 当囊帽和囊体都存在字符时，此参数限制上下字符的对齐程度。参数越大越松
      val: 1.5
    width_lb:    # 单组字符的外接矩形的宽度下限
      val: 0.8
    width_ub:    # 单组字符的外接矩形的宽度上限
      val: 1.5
    shift_ub:    # 单组字符的质心在垂直方向偏移量的上限
      val: 2.0
    height_lb:   # 单组字符的外接矩形的高度下限
      val: 0.75
    height_ub:   # 单组字符的外接矩形高度上限
      val: 1.5
    area_lb:     # 单组字符的轮廓面积下限
      val: 0.7
    area_ub:     # 单组字符的轮廓面积上限
      val: 1.5
```

## params#-printing.yaml

```yaml
capsule_text:
  num_capsules: 6
  num_texts: [1, 1]      # [帽的字符组数量, 体的字符组数量]
  repeat_times: [1, 1]   # [帽的字符组重复次数, 提的字符组重复次数]
  enable_stitching: 0    # 基于角点检测的stitching算法，通常不开启。不开启时为直接按顺序摆放图片简单拼接
  no_stitching_mode: 0   # 完全不拼接的模式，当文字为竖直排布且宽度很小时，此模式效果更好
  morph_ksize: [25, 25]  # blackhat 操作的kernel size
  close_ksize: [17, 9]   # 阈值化以后链接字符轮廓的kernel size
  open_ksize: [1, 3]     # 阈值化以后消除噪声的kernel size
  darker_text: 1         # 定义文字是否比背景更黑或更亮
  threshold: 35          # blackhat操作后的二值化阈值，用于提取字符
  text:                  # 各种形态下，字符的标准归一化 [x, y, w, h]
    lower_negative:
      - [0.4081411, 0.2217678, 0.360973, 0.1560504]
      - [0.3598886, 0.5605797, 0.4213746, 0.1889091]
    lower_positive:
      - [0.5434783, 0.2363636, 0.4565217, 0.1939394]
      - [0.5507246, 0.6060606, 0.3731884, 0.1636364]
    upper_negative:
      - [0.4296031, 0.2342767, 0.3663405, 0.1603774]
      - [0.3671051, 0.581761, 0.4241728, 0.1839623]
    upper_positive:
      - [0.5381818, 0.2594937, 0.4618182, 0.1962025]
      - [0.5490909, 0.6265823, 0.3745455, 0.1518987]
  text_areas:            # 各种形态下，字符轮廓的标准面积
    lower_negative:
      - 1713
      - 2048.666666666667
    lower_positive:
      - 2238.5
      - 1834
    upper_negative:
      - 1705.75
      - 1932.25
    upper_positive:
      - 2152.5
      - 1748.5
text_positions:        # 各种形态下，字符的出现范围（主要为了规避接缝） 
    upper_positive: 
        - [0.00, 0.48]
        - [0.50, 1.00]
    upper_negative:
        - [0.00, 0.52]
        - [0.54, 1.00]
    lower_positive:
        - [0.00, 0.46]
        - [0.48, 1.00]
    lower_negative:
        - [0.00, 0.50]
        - [0.52, 1.00]
capsule_text_blackspot:
  num_capsules: 6
  kernel_size: [23, 23]   # 应用dynamic_threshold的kernel size 
  text_margin: [10, 10, 20, 20] # 字符的 [上, 下, 左, 右] 预留的像素数量（用于减少一些字符周围黑点误检，比如i的点）
  edge_margin: [-10, -10, -20, -20] # 整体的 [上, 下, 左, 右] 向内缩的像素数量 （用于减少位于边缘位置的黑点误检）
```