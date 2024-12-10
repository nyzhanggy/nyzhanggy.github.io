---
title: CIColorKernel
categories: [根深柢固]
---

### 创建 default.ci.metal 文件

```c
#include <CoreImage/CoreImage.h>
using namespace metal;

extern "C" float4 transparentToWhite(coreimage::sample_t pixel) {
    float4 color = pixel.rgba;
    float result = 1.0 - step(0.5, color.a);
    return float4(result, result, result, 1.0);
}

```

### 添加Build Rules
第一条：
```
Process: Source files with names matching -> *.ci.metal

Using: Custom script -> xcrun metal -c -fcikernel "${INPUT_FILE_PATH}" -o "${SCRIPT_OUTPUT_FILE_0}"

Output Files: $(DERIVED_FILE_DIR)/$(INPUT_FILE_BASE).air
```
第二条：
```
Process: Source files with names matching -> *.ci.air

Using: Custom script -> xcrun metallib -cikernel "${INPUT_FILE_PATH}" -o "${SCRIPT_OUTPUT_FILE_0}"

Output Files: ${METAL_LIBRARY_OUTPUT_DIR}/${INPUT_FILE_BASE}.metallib
```

### 使用

```swift
if let url = Bundle.main.url(forResource: "default", withExtension: "ci.metallib"),
let data = try? Data(contentsOf: url) {
do {
    let kernel = try CIColorKernel(functionName: "transparentToWhite", fromMetalLibraryData: data)
    // 二值化
    let thresholdImage = kernel.apply(extent: inputImage.extent, arguments: [inputImage]) ?? outputImage
} catch let e {
    print(e)
}
}
```



