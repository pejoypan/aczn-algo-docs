# 📑 Changelog

### 2025-07-18

- 针对 InferEngine 的重构：
    - 引入枚举量 ShapeMode, 代替并统一 fixed_roi, dynamic_shapes 等接口
    - 引入 DefectPolicyTable，代替并统一 not_defect, just_infer 等接口
    - 将 input 的 imgs 和 rois 变为类内成员，以防止异步推理时需要call两次 prepare_imgs_and_rois
        - run, submit, wait_and_collect 等函数的接口已做相应调整
    - 在 set_infer_outputs 时直接将 roi 设置上
    - 重构 run，现在与异步实现完全统一
    - 重构 examine_one_slice 内部逻辑