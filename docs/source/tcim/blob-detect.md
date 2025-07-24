# Blob Detect (Universal)

> required version: >= 3.9.5

## Format

```yaml
# detector#.yaml
tablet_detector:
  use: 1
  parameters:
    front:
      blob_detectors:
      - colorspace: Lab # Lab, Luv, HSV, HLS, BGR, XYZ, YUV, YCrCb, GRAY
        coi: 0 # 0, 1, 2 
        ksize: [49, 49]
        scalar: 125 # 0~255
        is_darker: true
        params:
        - thresholdStep: 1
          minThreshold: 10
          maxThreshold: 11
          minRepeatability: 1
          filterByArea: true
          minArea: 30
          filterByCircularity: true
          minCircularity: 0.3
        #   maxCircularity: 1.0
          filterByInertia: true
          minInertiaRatio: 0.1
        #   maxInertiaRatio: 1.0
          filterByConvexity: true
          minConvexity: 0.5
        #   maxConvexity: 1.0
```

### colorspace & coi

Choose the best channel to detect blob

You can use the inspect-toolbox or ImageJ to observe

### ksize

The kernel size for dynamic threshold.

### scalar

The scalar for filling the mask outside

### is_darker

If true, the blob will be detected in the darker region, else the brighter

### params

Same as simple blob detector

```{note}
There can be multiple params
```

### Tips

- There can also be multiple different detectors