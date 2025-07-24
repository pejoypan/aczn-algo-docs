# 如何部署新深度模型？

## 常规内容
- 模型onnx文件
- 模型yaml文件
- 算法模板文件夹

## 模型onnx文件
位置放到 C:\opt\aczn_models

## 模型yaml文件

位置放到  C:\opt\aczn_models

其内容如下：

```yaml
name: CAPSCANADA_VEGGIE_20240802        # 需检查
version: 1
file_path: C:/opt/aczn_models/best.onnx # 需检查
mode: YoloV8ModelDetect                 # [YoloV8ModelSegment, YoloV8ModelClassify, MaskRcnnModelSegment]
class_threshold: 0.5                    # 默认值 0.5
nms_threshold: 0.7                      # 默认值 0.45
batch_size: 6                           # 根据情况配置
number_of_sessions: 10                  # 要根据情况配置 通常与该模型涉及的相机数量相同
in_gpu: true
device_id: 0                            # GPU index
runner: OnnxPredictRunner

# task_wait_timeout_ms: 500
# req_queue_wait_timeout_ms: 500
# merge_size: 1
# merge_distance_ms: 0
# session_options:
#   inter_op_num_threads: 1
#   intra_op_num_threads: 4
#   execution_mode: "ORT_PARALLEL"
#   graph_optimization_level: "ORT_ENABLE_ALL"
```

以下项需重点检查‼️：
- name
- file_path
- class_threshold
    - 通常设置为 0.1
- batch_size
    - 如果不确定，询问算法开发人员
- number_of_sessions
    - 如果不确定，询问后台开发人员

## 算法模板
如果由传统算法升级为带深度的算法，conf_inspect.yaml中会有些改动
因此如要使用深度算法，需使用对应的模板

## 注意事项 ⚠️
- 确保 C:/opt/aczn_models/ 中存在 infer_api_cfg.yaml 文件，其内容如下
```yaml
log_provider: h9log
log_level: ERROR
log_file: c:/opt/aczn_models/infer_api.log
```
- 如果为主从机的机器, 模型onnx文件 和 模型yaml文件, 所有从机都要有一份