# Command Unified Interface

## Unified Format

```yaml
- xxxx_command:
    src: src_image
    dst: dst_image
    params:          # params.yaml: params_<tag>
      type: HSV_FULL # [open, close, open] 
      ksize: [w, h]  # [[w0, h0], [w1, h1], [w2, h2]]
      kshape: rect   # [rect, ellipse, cross]
      value: 100.0   # [255, 0, 0]
      range: [[0, 255], [0, 255], [0, 255]] # [0, 100]
```

