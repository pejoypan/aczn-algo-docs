# 胶囊检新算法说明

## 版本要求

aczn-algo >= 3.10.1

## 路线图

![roadmap](new-cim-algo-instruction/cim-new-roadmap.jpg)

## 主要特性

同样的检测能力下：
- CPU占用率降低15%
- 耗时降低50%

更优的检测能力：
- 所有计算提升至亚像素精度
- 所有检测项均有单独的开关，且可分帽/体控制
- 宽度差引入基于梯度计算，双帽检测更准
- 黑点检测，参数配置更灵活
- 并行执行无锁设计

## 参数简述

### conf_inspect.yaml

```yaml
    - capsule_detect_new:
        raw_src: image              # 用于作为crop的原始图像
        bgr_src: image_smooth       # 1. gray图像提取的src 2. blob检测使用特殊colorspace的src
        hsv_src: image_hsv          # 1. gray图像使用V通道时的src 2. blob检测使用HSV时的src 3. 颜色检查的src
        all_mask: capsule_mask      # 1. 生成detect_mask.all的src 2. 判断[有无,多粒,内孔]的src 3. 计算整体的[contour,area,center,bbox,rrect,roi_roi,angle,trans,roi_trans]的src 4. case1情况下用于align的src
        cap_mask: cap_mask          # (双色)需提供 1. 生成detect_mask.cap的src 2. case2下判断cap[有无,多粒,内孔]的src 3. 计算帽的[contour,area,center,bbox,rrect]的src 4. case2下用于拼接整体mask的src
        body_mask: body_mask        # (双色)需提供 1. 生成detect_mask.cap的src 2. case2下判断cap[有无,多粒,内孔]的src 3. 计算帽的[contour,area,center,bbox,rrect]的src 4. case2下用于拼接整体mask的src
        def_src: image              # 用于绘制缺陷图的src
        cropped: [c00, c01, c10, c11, c20, c21, c30, c31, c40, c41, c50, c51] # crop的输出，顺序: 先列后行，size == 通道数x2
        imgsz: [1024, 896]          # 必填项 [宽, 高] 1. 检查输入图像的尺寸 2. 边界检查时的依据 3. 过程图像生成的尺寸
        execution_policy: par       # 执行策略，可选：[par, seq, par_unseq] (并行/顺序/无序并行)
        config:
          valid_lb:
            all: 2000               # 判断胶囊有无的阈值
            cap: 2000               # (双色)判断囊帽有无的阈值
            body: 2000              # (双色)判断囊体有无的阈值
          morphology_concat:        # case 2/3 情况下，对 concat 后的 mask 进行形态学操作的参数
            type: [open, close]
            ksize:
              - [1, 1]
              - [1, 1]
          joint_range:                    # 接缝的相对位置：格式: [ystart, yend] 相对值，范围: [0, 1] (图像中胶囊的最高点为0.0, 最低点为1.0)
            upper_positive: [0.53, 0.62]  # 上排帽在上
            upper_negative: [0.47, 0.56]  # 上排帽在下
            lower_positive: [0.43, 0.52]  # 下排帽在上
            lower_negative: [0.38, 0.45]  # 下派帽在下
          text_range:                     # 印字的相对位置：格式: [ystart, yend] 相对值，范围: [0, 1] (图像中胶囊的最高点为0.0, 最低点为1.0)
            upper_positive:               # 上排帽在上
              - [0.2, 0.4]                # 当有多行文字时，设置多个范围
            upper_negative:               # 上排帽在下
              - [0.6, 0.8]
            lower_positive:               # 下排帽在上
              - [0.2, 0.4]
            lower_negative:               # 下排帽在下
              - [0.6, 0.8]
        rule:
          mask:
            all_mask:               # 整体mask
              check: true           # 是否检查？
              concat: false         # (双色)是否用帽体拼接？
              concat_ksize: [3, 5]  # (双色)拼接的ksize
            cap_mask:               # (双色)囊帽mask
              check: false          # (双色)是否检查？
            body_mask:              # (双色)囊体mask
              check: false          # (双色)是否检查？
          detect_mask:              # 1. 定义检查的范围 2. 防止all_mask存在覆盖不到的区域
            all:                    # 定义整体detect_mask的生成方式
              type: [convex_hull, open, close]
              ksize:
                - [3, 3]
                - [15, 15]
                - [13, 13]
            cap:                    # (双色)定义囊帽detect_mask的生成方式
              type: [open, close]
              ksize:
                - [13, 13]
                - [13, 13]
            body:                   # (双色)定义囊体detect_mask的生成方式
              type: [open, close]
              ksize:
                - [13, 13]
                - [13, 13]
          width_diff:               # 宽度差计算
            check: true             # 是否检查？(对于单色胶囊，为了判断胶囊朝向，无论是否检查都将计算宽度差)
            method: gradient        # 检查方法 可选：[gradient, meanx] (基于梯度/老算法)
          color:                    # 颜色检查 (可按需开启)
            check_all: false        # 检查整体?
            check_cap: false        # (双色)是否检查囊帽？
            check_body: false       # (双色)是否检查囊体？
          crop:                     # crop的规则 (用于生成给深度推理用的小图)
            policy: disable         # 裁切策略，可选：[disable, na, ng, ok] 详见[裁切规则](###裁切规则)
            margin: 2               # 裁切时多留出的边界
          shape:                    # 形状检查
            align_norm:             # 轮廓插值、对齐，逐像素计算距离的算法（目前性能不好，尚在开发中）
              all: false            # 暂不检查
              cap: false            # (双色)暂不检查
              body: false           # (双色)暂不检查
            hu_moments:             # 基于Hu矩的算法 (虽然很快，但是检测效果一般)
              all: false            # 暂不检查
              cap: false            # (双色)暂不检查
              body: false           # (双色)暂不检查
            fit_ellipse: true       # 端部轮廓椭圆拟合后检查 建议开启
            use_old: true           # 老算法 建议开启
          blob:                     # 斑点检查 建议开启 (需详细修改detectors.yaml)
            check: true             # 是否检查？
          canny:                    # 条纹检查
            check: true             # 是否检查？
            ksize_erode: [9, 5]     # 检查范围的收缩程度 (防止胶囊的边缘被当作edge)
          topconcave:               # 顶凹检查
            check_cap: true         # 检查囊帽端？
            check_body: true        # 检查囊体端？

```

### detectors.yaml

可以直接沿用旧模板，但是黑点检测部分需要修改

```yaml

    blob_detectors:
      - colorspace: Lab
        coi: 0
        ksize: [23, 23]
        scalar: 49
        is_darker: true
        textbox_weakener: 0.8
        seambox_weakener: 0.8
        y_range: [0.0, 1.0]
        mask_from: all
        mask_morph:
          type: [erode]
          ksize:
            - [7, 1]
        params:
          - thresholdStep: 1
            minThreshold: 6
            maxThreshold: 9
            minRepeatability: 2
            minDistBetweenBlobs: 5
            filterByColor: false
            blobColor: 255
            filterByArea: true
            minArea: 15
            maxArea: 9999999
            filterByCircularity: true
            minCircularity: 0.3
            maxCircularity: 1.0
            filterByInertia: true
            minInertiaRatio: 0.1
            maxInertiaRatio: 1.0
            filterByConvexity: true
            minConvexity: 0.5
            maxConvexity: 1.0
```

### params.yaml 

可以直接沿用旧模板，需要增加 img_size 字段

## 概念

### 裁切规则

| crop规则 | 无胶囊裁切？ | ng胶囊裁切？ | ok胶囊裁切？ |
| -------- | ---------- | ------------ | ----------- |
| na       | ✅         | ✅          | ✅         |
| ng       | ❌         | ✅          | ✅         |
| ok       | ❌         | ❌          | ✅         |
| disable  | ❌         | ❌          | ❌         |

### mask case 1/2/3 说明

![case flowchart](new-cim-algo-instruction/case_flowchart.png)