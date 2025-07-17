# Predict Command Tutorial   
## predict 基本功能   
```
    - predict:
        model: model              # 在此处指定模型(infer_models中路径的stem)，支持tag
        src: [image]              # 输入图像
        # src: [img0, img1, img2] # or 图像组 
        skip_all_NA: true         # 配置是否跳过全部NA的情况
        just_infer: false         # 配置报缺陷还是只推理        
        not_defect: [class_0, class_1, ...]  # 指定哪些不当作 defect
        prepend_result: false     # if true，将结果插入到最前而不是最后
        dynamic_shapes: false     # 定义每个slice的尺寸是动态还是固定
        slices:                   # 定义 slices，支持tag
          - [x, y, w, h]
          - [x, y, w, h]
        idxs: [0, 1, 2, ...]      # 配置小图/slice与结果的映射关系，支持tag
        def_pos: [top, btm, ...]  # 定义每个slice的位置str，只在片剂检中使用
        crop:                     # 调试用，用于将prediction裁切并存储为png
          save: true/false        # crop save 开关
          padding: 5              # crop save 时的 padding
        which_detector: xxxx_detector # 指定 detectors.yaml 中的 key
        seg_mask: image_xxxx      # 如果存在，会将预测结果绘制在 image_xxxx 上
        min_contour_size: 10      # save yolo 时使用，用于控制最小的contour
        outputs: [out0, out1, out2, ...]     # 输出的 InferObject
        def_src: image            # 缺陷绘制源
        def_img: image_defect     # 缺陷绘制画布
```
## 重要概念   
### 动态模式   
> 输入图片为1~n个，图片尺寸不固定。多张图片batch推理   

当满足下列任一条件时进入动态模式：   
- src.size > 1   
- dynamic\_shapes == true   
  
### 固定模式   
> 输入图片必须为1个，图片固定为slice设置的尺寸。裁切后多张图片batch推理   

当满足下列全部条件时进入固定模式：   
- src.size == 1    
- dynamic\_shapes == false   
  
### num   
> 指单次提交推理的图片数量   

固定模式下：`num == slices.size`

动态模式下：`num == src.size`

## 使用场景   
### 场景A. 输入为1~N张图片 尺寸不确定    
> 如胶囊正面检、胶囊侧面检、片剂正面检、片剂侧面检。图片由前序步骤抠出来   

最小实现：   
```
    - predict:
        src: [img0, img1, img2, img3]
        idxs: [0, 1, 2, 3]
        model: best
        def_src: image
        def_img: image_def

```
### 场景B. 输入为1张图片 分块推理   
> 如泡罩检、数粒线、透明胶囊。结果可以是单通道也可以是多通道   

最小实现：   
```
    - predict:
        src: [image]
        slices:
          - [0, 0, 640, 640]
          - [560, 0, 640, 640]
          - [0, 560, 640, 640]
          - [560, 560, 640, 640]
        idxs: [0, 1, 2, 3]
        model: best
        def_src: image
        def_img: image_def
```
### 场景C. 输入为1张图片 直接推理   
最小实现：   
```
    - predict:
        src: [image]
        slices:
          - [0, 0, image.width, image.height]
        idxs: [0]
        model: best
        def_src: image
        def_img: image_def
```

## 输入&输出参数详细说明   
### model   
> 输入，用于标志该command要使用的模型，输入路径的stem   

比如在 init node 中：   
```
init:
  infer_models:
    - C:/opt/aczn_models/best_yolov8n.yaml
```
则这里填 `model: best\_yolov8n` 

:::{note}
command 并不负责模型的初始化工作
模型的初始化统一在 InspectProcessor::initialize\_generic 中进行
command 只是根据名字，取得对应 model 的指针
:::

- 该关键字支持tag   
  
### src   
> 输入，用于标志输入图片的名字，必须为 sequence 格式   

:::{note}
动态模式下，num == src.size
:::

- 单张输入：`src: [image]`
- 多张输入：`src: [img0, img1, img2]`

### skip\_all\_NA 
> 此项开启时，当前序flow判定所有通道都为NA时，直接跳过

- bool 型，可填 true or false
- 默认为 true

### just\_infer
> 此项开启时，command只做推理，不修改m_result_

- bool 型，可填 true or false
- 默认为 false
- 开启时：
    - 不会修改 m\_result\_ 
    - 会将 predictions 以 InferObject 形式放入 outputs 节点

### not\_defect   
> 用于标志哪些class不作为缺陷处理   

- 默认为空   
- 标准格式：`not\_defect: [class\_0, class\_1, …]`   
- not\_defect内的class：   
    - 不会画在画布上   
    - 不会进入 m\_result\_   
    - 会进入 outputs (如果存在) 

### prepend\_result   
> 若此项开启，在修改m_result_时，预测的结果会插入到头部，而不是尾部   

- bool 型，可填 true or false   
- 默认为 false 

### dynamic\_shapes   
> 此项开启时，强制进入动态模式   

- bool型，可填 true or false   
- 无默认值，没有该节点时不会触发相关逻辑

### slices   
> 在固定模式下，用于将输入的单张图片切成多份   

- **只在固定模式下使用**   
- 固定模式下必须输入，动态模式下会直接忽略   
- 支持 tag   
- 内部逻辑：   
    - slices.size 会赋值给 num   
    - 裁切时，只会取合法的slice，如果遇到out of的情况，会报错并 return false   
    - 如果存在合法的def\_pos，会修改对应的 m\_tablet\_bbox (用于跑马灯显示)   
- 标准格式:

```yaml
  slices:
  - [x, y, w, h]
  - [x, y, w, h]
```


### idxs   
> 用于标志每张图片/每个slice所属的通道，以修改m_result_中对应的位置   

- 支持 tag   
- 如果节点存在，长度必须 == num   
- 默认为: [0, 0, …, 0] (num个)   
- 标准格式：   
```
idxs: [0, 1, 2, ...] # size == num
```
- 动态模式下，映射src中元素的index   
- 固定模式下，映射slices中元素的index

### def\_pos   
> 标志每张图片/每个slice对应的def_pos，目前仅用于TCIM跑马灯显示效果   

- 支持 tag   
- 如果节点存在，长度必须 == num   
- 默认为空   
- 标准格式：   
```
def_pos: [top, bottom, ...]
```

### crop   
> 用于 crop_save 辅助功能，将prediction裁切并存储为png   

```
crop:                     
  save: true/false  # bool 型开关，默认为false
  padding: 5        # 边界 padding 的像素尺寸，默认为0
```
存储格式：`basename-s<slice\_id>-<box>.png` 

> [!WARNING]
>
> 2025-06-04 目前应该存在bug，会将所有predictions都裁切保存，而不是filter后的

### which\_detector   
> 用于指定该command在 detector#.yaml中对应哪个detector   

如果该节点不存在，默认为 `infer\_detector` 

### seg\_mask   
> 用于输出一张绘制着预测结果的mask   

标准格式：`seg\_mask: predict\_mask`   
如果该节点存在，每次执行时：   
1. 会生成一张8UC1的图像，放在 `m\_container[predict\_mask]` 中   
2. 对图像清零 （全部赋0）   
3. 对于不同的模型：   
    1. 分割模型：绘制轮廓（填充）   
    2. 检测模型：绘制方框（填充）   
4. 绘制使用的灰度为 `255 - label\_id`   
    > 即，对于class_0，以255绘制；对于class_1，以254绘制；… 


### min\_contour\_size   
> 仅用于分割模型的save_yolo功能，滤掉尺寸较小的contour   

- 默认值： 0 

### outputs   
> 用于 just_infer 或 not_defect 不为空模式下，输出 InferObject 的接口    

- 数量应 == num   
- 默认为空，即不使用   
- 满足以下**任一**条件时，将会向其放入 pass filter 的 predictions：   
    - command 工作在 just\_infer 模式   
    - 该 class 不是defect (not\_defect 中包含该 prediction 的 label)    

**InferObject** 

定义: 

> 某张图片/某个slice的所有推理结果

成员：   

- vector<InferOutput>   
- roi   
- InferOutput 定义：   
    - int label\_id   
    - float confidence   
    - Rect box   
    - contours   
    - string class\_name   
    - Scalar color   

用法举例：   
- 数量检测：搭配 validate\_detection\_box   
- 尺寸检测：透明空胶囊的box被检查出来后，检测其高度宽度   
  

## 同步推理 predict 与 异步推理的转换   
:::{note}
所有 predict 支持的选项，异步推理command也同步支持 
:::

假设现有同步模式flow:   

```
    - predict:
        model: best
        src: [img0, img1, img2]
        idxs: [0, 1, 2]
        def_src: image
        def_img: image_def
```
下面的异步command与之等价：   
```
    - async_infer_submit:        
        INTERNAL_ENGINE_KEY: async_engine
        model: best
        uuid: 0
        src: [img0, img1, img2]
        idxs: [0, 1, 2]
    
    - do something ...
    
    - async_infer_fetch:
        INTERNAL_ENGINE_KEY: async_engine
        uuid: 0
        src: [img0, img1, img2]  # save_yolo 模式下会用到
        def_src: image
        def_img: cap_defect
```
- async\_infer\_submit:   
    - 负责engine的初始化，因此应带上所有的参数选项   
    - 负责提交推理请求   
- async\_infer\_fetch：   
    - 负责获取推理结果   
- 每对 command 以下参数必须一致：   
    - `INTERNAL\_ENGINE\_KEY `保证其使用同一个engine   
    - `uuid ` 用于保证task的唯一性 （范围0~9）   


---

## 使用inspect\_app时的相关arguments 

### `--save\_yolo`
参见 [使用 inspect\_app 将 predictions 保存为 yolo 格式](https://pejoypan.any.org/inspect-app-save-yolo) 

## infer\_detector 设置

参见 [https://pejoypan.any.org/infer-detector](https://pejoypan.any.org/infer-detector)    

## TODOs 

- 合并 counting\_machine 分支的优化改动   
- 统一 fixed\_roi 和 dynamic\_shapes   
- 统一 just\_infer 和 not\_defect   
