# 📑 Changelog

### 2025-09-29 (3.11.0)

合并 counting_machine 分支！

- 新增command `counter_detect`
- 新增command `update_SIM_result`
- 支持读取 params#.counter.num_channel 作为 m_num

### 2025-09-24

- 增加胶囊检新算法相关说明

### 2025-08-21

- 开发 background_subtract 命令
- 开发 distance_transform 命令
- 开发 count 命令

### 2025-07-30

- 优化 extract_objects 命令：
    - 现在支持 xywh 输入，使用相对值，范围为 [0, 1]
    - 以适应不同的输入图片尺寸

### 2025-07-18

- 针对 InferEngine 的重构：
    - 引入枚举量 ShapeMode, 代替并统一 fixed_roi, dynamic_shapes 等接口
    - 引入 DefectPolicyTable，代替并统一 not_defect, just_infer 等接口
    - 将 input 的 imgs 和 rois 变为类内成员，以防止异步推理时需要call两次 prepare_imgs_and_rois
        - run, submit, wait_and_collect 等函数的接口已做相应调整
    - 在 set_infer_outputs 时直接将 roi 设置上
    - 重构 run，现在与异步实现完全统一
    - 重构 examine_one_slice 内部逻辑

### 2025-05-30

在 commit 5dda3b56 中，`capsule_detect` 引入了 `tbb::parallel_for` 用于节约计算时间

为方便算法开发调试 

- 可在编译选项中增加 `-DCAPSULE_USE_TBB_PARALLEL=OFF`
- 或可直接使用 build_for_app.cmd 进行编译

即为原始的不使用 parallel_for 的版本

```{note}
如不增加该编译选项，默认为开启
```
