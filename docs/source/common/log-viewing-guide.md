# 日志查看指南（现场交付/调试/测试）

本文说明算法库日志的默认位置、配置方法和常见查看技巧，帮助现场快速定位问题。

## 1. 默认日志位置

| 平台 | 默认日志文件 |
|------|-------------|
| Windows | `C:/opt/aczn_logdir/inspect_algo.log` |
| Linux | `/opt/aczn_logdir/inspect_algo.log` |

启用滚动后，还会生成备份文件：

```text
inspect_algo.log
inspect_algo.log.1
inspect_algo.log.2
...
```

## 2. 修改日志配置

日志由 `algo_log_config.yaml` 控制。系统按以下顺序查找该文件：

1. 环境变量 `INSPECT_ALGO_LOG_CONFIG` 指定的路径
2. 默认路径：
   - Windows：`C:/opt/aczn_workdir/algo_log_config.yaml`
   - Linux：`/opt/aczn_workdir/algo_log_config.yaml`

### 常用配置项

```yaml
log_level: DEBUG
log_path: C:/opt/aczn_logdir
rotate_max_file_size: 1GB
rotate_max_backup_files: 10
open_mode: a
```

| 配置项 | 说明 |
|--------|------|
| `log_level` | 日志级别：`TRACE`（最详细）/ `DEBUG` / `INFO` / `WARNING` / `ERROR` |
| `log_path` | 日志输出目录，支持相对路径 |
| `rotate_max_file_size` | 单个日志文件上限，如 `100MB`、`1GB` |
| `rotate_max_backup_files` | 保留的备份数量 |
| `open_mode` | `a` 追加，`w` 覆盖。建议保持 `a` |

## 3. 查看日志

### 3.1 日志格式示例

```text
15:23:47.123456  InspectProcessor.cc:120  INFO  camera_01  Node(camera_01) Construction Complete
```

从左到右依次为：时间、源码位置、级别、通道名/实例名、日志内容。

## 4. 常见问题

| 问题 | 排查方法 |
|------|---------|
| 找不到日志文件 | 检查 `log_path` 目录是否存在、是否有写入权限；检查配置文件是否被环境变量指向其他位置 |
| 日志为空 | 日志是异步写入，短进程可能还没落盘；或 `log_level` 设置过高 |
| 日志增长太快 | 提高日志级别为 `INFO` 或 `ERROR`，或减小 `rotate_max_file_size` |
| 启动后日志被清空 | 检查 `open_mode` 是否为 `w`，建议改为 `a` |
| 多通道日志混在一起 | 按“实例名”列搜索，例如 `camera_01`、`line_02` |

## 5. 现场建议

- 复现问题时，将 `log_level` 临时改为 `DEBUG` 或 `TRACE`，问题定位完成后再改回 `INFO`。
- 不要把 `log_path` 指向系统盘根目录，避免权限问题。
- 保留 `open_mode: a`，防止历史日志被覆盖。
