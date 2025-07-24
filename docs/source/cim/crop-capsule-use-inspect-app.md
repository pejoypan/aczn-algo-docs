# 使用 inspect_app 切图功能说明

## 修订记录

| 日期       | 描述                       | 作者                                  |
| ---------- | ----------- | ------------------------- |
| 2024-05-11 | initial               | [@pejoypan](panjianyue@aiaocheng.com) |

## Prerequisites

- aczn-algo >= ```a29fc2e: feat(CIM): add crop and save feature```
- aczn-inspect-app >= ```1b9a99f: add args crop crop_NA```

## Usage

```shell
inspect_app -c <flow_yaml> -p <params_yaml> -l <imglist> --crop --crop_NA=false
```

```{note}
不需要对 flow 做额外修改
```

- --crop: 如果为true，此时算法处于裁切模式，**没有**检测功能，将会裁切并保存每个object的bounding box的小图
- --crop_NA: 如果处于裁切模式，可以通过此值配置是否要裁切并保存**NA**的情况；如果是**NA**且crop_NA==true，则会按照默认位置切图

## 保存格式

<basename>-<cap_id><is_upper ? a : b>.png


```{important}
- 目前只支持胶囊检算法
- multi_thread模式下不可用
```