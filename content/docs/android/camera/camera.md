---
title: 'android camera 相关知识'
date: 2018-11-28T15:14:39+10:00
weight: 1
summary: "android vm 和 java vm 的区别"
---

# Android Camera

> 工作中开发了一个相机应用，在开发过程中由于和evs框架冲突，导致经常发生打开camera失败。因此写下这篇文章，来简单的分析一下camera底层的内容<br>
[相机](https://developer.android.google.cn/training/camera) <br>
[CameraX](https://developer.android.google.cn/training/camerax)


## Camera2
> [camera 官方例子](https://github.com/android/camera-samples)


| Sample | Description |
| -- | -- |
| CameraXBasic | Demonstrates how to use CameraX APIs. |
| CameraXTfLite |	Demonstrates how to use TfLite with CameraX APIs. |
| Camera2Basic | Demostrates capturing JPEG, RAW and DEPTH images, e.g. unprocessed pixel data directly from the camera sensor. | |
| Camera2SlowMotion |	Demostrates capturing high-speed video in a constrained camera capture session. |
| Camera2Video | Demonstrates recording video using the Camera2 API and MediaRecorder. | |
| HdrViewfinder	|	Demonstrates use of RenderScript to display a live HDR feed from camera frames using Camera2 API. |

我们是为了分析camera底层的原理，所以我们来分析Camera2Basic的内容

### Camera2Basic
- 流程图 - 预览
{{<mermaid>}}
sequenceDiagram
    participant cm As CameraManager
    participant cd As CameraDevice
    participant ccs As CameraCaptureSession
    cm->>cd: openCamera(cameraId, callback)
    cd->>ccs: createCaptureSession(List of Surface, StateCallback)
    cd->>CaptureRequest: createCaptureRequest(CameraDevice.TEMPLATE_STILL_CAPTURE).apply { addTarget(imageReader.surface) }
    CaptureRequest->>ccs: build()
    ccs->>ccs:setRepeatingRequest(CaptureRequest, callback)
{{</mermaid>}}
- 解释
  - CameraManager 调用 `openCamera(cameraId, callback)` 获取一个cameradevice的对象实例。cameraid 为一个常量可以通过`CameraManager.getCameraIdList()` 获取
  - 打开成功将会回调在callback 中获取到CameraDevice
  - 使用CameraDevice 创建一个CaptureSession。传入接受数据的Textureview的surface
  - 成功后，可以发起获取画面的请求setRepeatingRequest

### 
