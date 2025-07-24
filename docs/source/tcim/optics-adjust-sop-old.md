# TCIM光学调试SOP (老片剂)

## 修订记录

|日期|描述|作者|
|---|---|---|
|2024-03-07|初版|[@pejoypan](panjianyue@aiaocheng.com)|
|2024-03-21|增加所有工位的拍照偏移调整|[@pejoypan](panjianyue@aiaocheng.com)|
|2024-03-25|增加硬件设置|[@pejoypan](panjianyue@aiaocheng.com)|
|2024-04-07|完整版||
|2024-05-11|增加疑难解答|[@pejoypan](panjianyue@aiaocheng.com)|
|2024-06-14|增加 Debouncer Time|[@pejoypan](panjianyue@aiaocheng.com)|

## TODO

1. 硬件安装检查相机朝向（图）
2. 更多配图

## 疑难解答

### 为什么有些配置项在 pylon viewer 中找不到？

> 在"特征(Features)"页面的左下角，将级别设置为"大师级(Guru)"

### 在 pylon viewer 中打开相机会弹窗？

> basler相机同时只能有一个进程占用，需把我们的程序退到生产界面；还不行则完全退出UI

## 相机光源硬件安装

|位置|相机|镜头|光源|
|---|---|---|---|
|滚筒常规工位|acA1300-200uc|Basler C23-3520-5M-P|KM-BRD###30 x2|
|皮带常规工位|acA2500-60uc|Basler C11-5020-12M-P|KM-BRD###20 x4|
|滚筒同轴工位|acA1920-150um|Basler C23-5028-5M-P|BX-COL-###-40-HD-M-W x1|
|皮带同轴工位|acA1920-150um|Basler C23-5028-5M-P|BX-COL-###-40-HD-M-W x1|

## 光源控制器硬件安装与配置

### 博兴远志控制器

**BXXJ-VHC-KZQ-COM-4DF**

触发源：PLC

输出：无

[软件下载链接](https://pan.baidu.com/s/1Zo0CpFrM75RQS8NjiyWG_Q?pwd=aczn)

> 提取码：aczn，修改控制器参数需使用网线连接

|通道|连接到|模式|亮度|发光时间|发光延时|
|---|---|---|---|---|---|
|CH1|滚筒工位同轴光|上升|255|?|2|
|CH2|皮带工位同轴光|上升|255|?|2|

- 滤波脉宽：1
- 噪音平滑：1

### 科麦控制器

- 控制器本体型号：KM-DPSHG04CH-48/30W，数量：1
- 一拖二光源连接线，数量：2
- 开关电源：需至少240W
- 触发源：PLC
- 输出：无
  
|通道|连接到|设定频闪时间|
|---|---|---|
|CH1|滚筒工位条光1|200|
|CH2|滚筒工位条光2|200|
|CH3|皮带工位条光1&2 (使用一拖二连接线)|200|
|CH4|皮带工位条光3&4 (使用一拖二连接线)|200|

## 滚筒常规工位（条光）调整

### 镜头调整

1. 将光圈调到对准**5.6**的位置
2. 将光圈的螺丝拧紧
3. 将对焦的螺丝松开，以备后用

### 相机参数设置

1. UI退出或退到生产页面
2. 打开 pylon Viewer
3. 在顶部菜单栏，打开设备窗口和特征窗口
4. 在设备窗口，双击要配置的相机，打开特征窗口
5. Device Control - Device Reset - 执行
6. Image Format Control
   1. Width： 设为1280
   2. Height： 设为352
   3. 这里需要简单判断一下Height是否够大，如果不够大需将该值增大
   4. 勾选Center Y, 再取消勾选
   5. Pixel Format - 选择 YCbCr 422
7. Image Quality Control
   1. PGI Control - Demosaicing Mode - 选择 Basler PGI
   2. Light Source Preset - 选择 Daylight (6500 Kelvin)
   3. 白平衡此时先不做
8. Acquisition Control
   1. Trigger Mode - On
   2. Exposure Time - 150us
9. Digital I/O Control - Line Debouncer Time - 设为 10.0
10. User Set Control
   1. User Set Selector - User Set 1
   2. User Set Save - 执行
   3. User Set Default - User Set 1
11. 对每个该工位相机重复4~10步

### 对焦与位置微调

1. 退出pylon Viewer
2. 确认风机已打开
3. 高级设置 - 相机设置 - 选择对应的相机
4. 勾选频闪光源选项 开始采样
5. 此时凹槽应该没有正对，停止采样
6. 进入PLC设置 - 使用滚筒点动功能 - 直到将凹槽正对相机
7. 放置两粒药片到凹槽中
8. 旋转对焦环，对焦到最清晰状态后，拧紧螺丝
9. 将固定相机的三颗螺丝稍微拧松，左右调整相机，使其正对两个孔的正中央(重要)
10. 对每个一工位相机，重复3~9步

### 做白平衡

1. 将UI退出到生产页面
2. 打开pylon Viewer，打开相机
3. 特征窗口 - User Set Control - User Set Selector - 选择 User Set 1
4. 特征窗口 - User Set Control - User Set Load - 执行
5. 相机进入采集状态
6. 将白平衡卡 覆盖相机视野
7. pylon Viewer - 特征窗口 - Image Quality Control - Balance White Auto - 设为Once
8. UI - 高级设置 - PLC设置 - 点击一次手动拍照
9. pylon Viewer - 特征窗口 - Image Quality Control - Balance White Auto - 设为Off
10. UI - 高级设置 - PLC设置 - 再点击一次手动拍照
11. 特征窗口 - User Set Control - User Set Save - 执行
12. 白平衡完成，对每一个一工位相机，重复2~11步

### 拍照偏移调整

1. 确认pylon Viewer已退出
2. 高级设置 - PLC设置 - 前工位2D相机拍照偏移 - 设为0
3. 其他设置 - 算法使能 - 关
4. 其他设置 - 存图设置 - OK存图 - 开
5. 回到生产页面，开始运行
6. 待完全开始运行，放入一粒药片
7. 药片经过两次拍照后，停止运行
8. 找到对应的图片（以药片在上方的那张为准），此时药片应该是歪着的
9. 略微增加“前工位2D相机拍照偏移”（1脉冲=0.0025°≈6.3um）
10. 重复5~9步，直到拍照在正中为止

## 皮带常规工位（条光）调整

### 镜头调整

同上，光圈对准**8**

### 相机参数设置

1. UI退出或推到生产页面
2. 打开 pylon Viewer
3. 在顶部菜单栏，打开设备窗口和特征窗口
4. 在设备窗口，双击要配置的相机，打开特征窗口
5. Device Control - Device Reset - 执行
6. Image Format Control
   1. Width: 设为2144
   2. 勾选Center X, 再取消勾选
   3. Pixel Format - 选择 YCbCr 422
   4. ROI Zone 设置：
      1. ROI Zone Selector - Vertical Zone 0
      2. ROI Zone Mode - On
      3. ROI Zone Size - 288
      4. ROI Zone Offset - 0
      5. ROI Zone Selector - Vertical Zone 1
      6. ROI Zone Mode - On
      7. ROI Zone Size - 448
      8. ROI Zone Offset - 800
      9. ROI Zone Selector - Vertical Zone 2
      10. ROI Zone Selector - On
      11. ROI Zone Size - 288
      12. ROI ZOne Offset - 1760
7. Image Quality Control
   1. PGI Control - Demosaicing Mode - 选择 Basler PGI
   2. Light Source Preset - 选择 Daylight (6500 Kelvin)
   3. 白平衡此时先不做
8. Acquisition Control
   1. Trigger Mode - On
   2. Exposure Time - 150us
9. Digital I/O Control - Line Debouncer Time - 设为 10.0
10. User Set Control
   1. User Set Selector - User Set 1
   2. User Set Save - 执行
   3. User Set Default - User Set 1
11. 对每个该工位相机重复4~10步

### 对焦与位置微调

1. 退出pylon Viewer
2. 放置两粒药片到皮带上
3. 进入PLC设置 - 使用皮带点动功能 - 将药片运送到相机正下方
4. 高级设置 - 相机设置 - 选择对应的相机
5. 勾选频闪光源选项 开始采样
6. 如果药片不在中心，使用点动功能继续微调直到中心位置
7. 旋转对焦环，对焦到最清晰状态后，拧紧螺丝
8. 将固定相机的三颗螺丝稍微拧松，调整相机，使其上下、左右都对称(重要)
9. 对每个该工位相机，重复3~8步

### 做白平衡

1. 将UI退出到生产页面
2. 打开pylon Viewer，打开相机
3. 特征窗口 - User Set Control - User Set Selector - 选择 User Set 1
4. 特征窗口 - User Set Control - User Set Load - 执行
5. 相机进入采集状态
6. 将白平衡卡 覆盖相机视野
7. pylon Viewer - 特征窗口 - Image Quality Control - Balance White Auto - 设为Once
8. UI - 高级设置 - PLC设置 - 点击一次手动拍照
9. pylon Viewer - 特征窗口 - Image Quality Control - Balance White Auto - 设为Off
10. UI - 高级设置 - PLC设置 - 再点击一次手动拍照
11. 特征窗口 - User Set Control - User Set Save - 执行
12. 白平衡完成，对每个该工位相机，重复2~11步

### 拍照偏移调整

1.	高级设置 - PLC设置 - 后工位2D相机拍照偏移 -设为（前工位2D相机拍照偏移+27000）
2.	其他设置 - 算法使能 - 关
3.	其他设置 - 存图设置 - OK存图 - 开
4.	回到生产页面，开始运行
5.	待完全开始运行，放入一粒药片
6.	药片经过两次拍照后，停止运行
7.	找到对应的图片（以药片在图片下方的那张为准），此时图片应该与前工位2D的图片编号不一致，
8.	增加“后工位2D相机拍照偏移”（1脉冲≈17.4533um）
9.	重复5~8步，直到拍照在正中，且图片运行运行编号与前工位2D的图片一致为止

## （如果有）滚筒同轴工位调整

### 镜头调整

同上，光圈对准**4**

### 相机参数设置

1. UI退出或退到生产页面
2. 打开 pylon Viewer
3. 在顶部菜单栏，打开设备窗口和特征窗口
4. 在设备窗口，双击要配置的相机，打开特征窗口
5. Device Control - Device Reset - 执行
6. Image Format Control
   1. Width：设为 1920
   2. Height：设为 480
   3. 这里需要简单判断一下 Height 是否够大，如果不够大需将该值增大
   4. 勾选 Center Y, 再取消勾选
7. Acquisition Control
   1. Trigger Mode - On
   2. Exposure Time - 105us
8. Digital I/O Control - Line Debouncer Time - 设为 10.0
9. User Set Control
   1. User Set Selector - User Set 1
   2. User Set Save - 执行
   3. User Set Default - User Set 1
10. 对每个该工位相机重复4~9步

### 对焦与位置微调

1. 退出pylon Viewer
2. 确认风机已打开
3. 高级设置 - 相机设置 - 选择对应的相机
4. 勾选频闪光源选项 开始采样
5. 此时凹槽应该没有正对，停止采样
6. 进入PLC设置 - 使用滚筒点动功能 - 直到将凹槽正对相机
7. 放置两粒药片到凹槽中
8. 旋转对焦环，对焦到最清晰状态后，拧紧螺丝
9. 将固定相机的三颗螺丝稍微拧松，左右调整相机，使其正对两个孔的正中央(重要)
10. 对每个一工位相机，重复3~9步

### 拍照偏移调整

1.	高级设置-PLC设置-前工位黑白相机拍照偏移-设为（前工位2D相机拍照偏移+11000）
2.	其他设置 - 算法使能 - 关
3.	其他设置 - 存图设置 - OK存图 - 开
4.	回到生产页面，开始运行
5.	待完全开始运行，放入一粒药片
6.	药片经过两次拍照后，停止运行
7.	找到对应的图片（以药片在图片上方的那张为准），此时图片应该与前工位2D的图片编号不一致，
8.	增加“前工位黑白相机拍照偏移”（1脉冲=0.0025°≈6.3um）
9.	重复5~9步，直到拍照在正中，且图片运行运行编号与前工位2D的图片一致为止

## （如果有）皮带同轴工位调整

### 镜头调整

同上，光圈对准**4**

### 相机参数设置

1. UI退出或推到生产页面
2. 打开 pylon Viewer
3. 在顶部菜单栏，打开设备窗口和特征窗口
4. 在设备窗口，双击要配置的相机，打开特征窗口
5. Device Control - Device Reset - 执行
6. Image Format Control
   1. Width：设为 1920
   2. Height：设为 480
   3. 这里需要简单判断一下 Height 是否够大，如果不够大需将该值增大
   4. 勾选 Center Y, 再取消勾选
7. Acquisition Control
   1. Trigger Mode - On
   2. Exposure Time - 105us
8. Digital I/O Control - Line Debouncer Time - 设为 10.0
9. User Set Control
   1. User Set Selector - User Set 1
   2. User Set Save - 执行
   3. User Set Default - User Set 1
10. 对每个该工位相机重复4~9步

### 对焦与位置微调

1. 退出pylon Viewer
2. 放置两粒药片到皮带上
3. 进入PLC设置 - 使用皮带点动功能 - 将药片运送到相机正下方
4. 高级设置 - 相机设置 - 选择对应的相机
5. 勾选频闪光源选项 开始采样
6. 如果药片不在中心，使用点动功能继续微调直到中心位置
7. 旋转对焦环，对焦到最清晰状态后，拧紧螺丝
8. 将固定相机的三颗螺丝稍微拧松，调整相机，使其上下、左右都对称(重要)
9. 对每个该工位相机，重复2~8步

### 拍照偏移调整

1.	高级设置-PLC设置-后工位黑白相机拍照偏移-设为（后工位2D相机拍照偏移+8000）
2.	其他设置 - 算法使能 - 关
3.	其他设置 - 存图设置 - OK存图 - 开
4.	回到生产页面，开始运行
5.	待完全开始运行，放入一粒药片
6.	药片经过两次拍照后，停止运行
7.	找到对应的图片（以药片在图片下方的那张为准），此时图片应该与前工位2D的图片编号不一致，
8.	增加“后工位黑白相机拍照偏移”（1脉冲≈17.4533um）
9.	重复5~9步，直到拍照在正中，且图片运行运行编号与前工位2D的图片一致为止

## 软件配置

### Camera.yaml 配置

相机唯一ID和实体相机的对应关系：

|                         | 机器内侧 | <------> | 机器外侧 |
| :---------------------: | :------: | :------: | :------: |
| 正面检一工位 (滚筒条光) |    0     |    1     |    2     |
| 背面检一工位 (皮带条光) |    3     |    4     |    5     |
| 正面检二工位(滚筒同轴)  |    6     |    7     |    8     |
| 背面检二工位(皮带同轴)  |    9     |    10    |    11    |

#### 文件格式

```yaml
CAMERAPARAM: # 根节点
  CAMERA0:   # 格式必须为CAMERA+相机唯一ID
    CameraID:
      Value: 24674984
    Camera_Series:
      Value: acA1300-200uc
    ROI:
      Value:
        - 0
        - 350
        - 1280
        - 352
```

（示例）

- 有N个相机，就**必须**有N个```CAMERA#```节点
- 在```CAMERA#```节点下，```CameraID``` ```Camera_Series``` ```ROI``` 三项为**必填项**

> **相机唯一ID** 来源是数据库Camera_List表内的CAM_ID字段

#### ROI设置

在 pylon viewer 中，load之前设置好的参数

将 Features - Image Format 中的 ```Offset X```, ```Offset Y```, ```Width```, ```Height```抄写过来

对应方式如图所示


### 数据库配置

#### Camera_List配置

- CAM_ID: 相机唯一ID **必须从0开始**
- CAM_MAKER: 相机厂商 （Basler, Baumer, Keyence）
- CAM_MODEL: 相机型号（影响ROI的取值范围）
- CAM_SN: 相机S/N码
- IMAGE_NUM: 图片数量（特殊相机一次会产生多张图片）
- CAM_GROUP: 相机所属工位

#### camera_define 配置

```{important}
当该相机有两个像的时候，需要当成2个相机来配置
```

- cameraindex: 相机图片唯一ID **与Camera_List中的CAM_ID含义不同**
- cameragroup: 实际工位序号（药片从头到尾实际经过的顺序 **与Camera_List中的CAM_GROUP含义不同**）
- camerapos: 通道序号（不是实际的通道 比如说一个工位有3个相机则序号为0,1,2）
- backendindex: 实际工位对应的后台工位序号
	- 两工位：后台工位与实际工位一一对应
	- 四工位：
	
	  - | cameragroup | backendindex (新) | backendindex (旧100102) |
	    | ----------- | ----------------- | ----------------------- |
	    | 0           | 0                 | 2                       |
	    | 1           | 2                 | 0                       |
	    | 2           | 1                 | 1                       |
	    | 3           | 3                 | 3                       |
- tagnamefirst: 对应通道的英文名称
- tagnamefirstts: 对应通道的翻译名称
- tagnamesec: 工位相机英文名称
- tagnamesects: 工位相机翻译名称