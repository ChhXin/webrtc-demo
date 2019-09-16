# 相关内容

### 获取可访问的媒体设备
`MediaDevices.enumerateDevices`请求一个可用的媒体输入和输出设备的列表，例如麦克风，摄像机，耳机设备等。 返回的 Promise 完成时，会带有一个描述设备的 MediaDeviceInfo 的数组。


 使用`enumerateDevices`输出一个带有标签（有标签的情况下）的 device ID的列表：
```
    if (!navigator.mediaDevices || !navigator.mediaDevices.enumerateDevices) {
      console.log("不支持 enumerateDevices() .");
      return;
    }
    // 列出相机和麦克风.
     navigator.mediaDevices.enumerateDevices()
        .then(function (devices) {
            devices.forEach(function (device) {
                console.log(device.kind + ": " + device.label +
                    " id = " + device.deviceId);
            });
        })
        .catch(function (err) {
            console.log(err.name + ": " + err.message);
        });
```

### 获取可使用的媒体流

##### 1. MediaDevices.getUserMedia()

getUserMedia API是由最初的`navigator.getUserMedia`（已被最新Web标准废弃），变更为`navigator.mediaDevices.getUserMedia`。`getUserMedia`使用时，首先会提示用户授权媒体输入许可，然后生成`MediaStream`。

一个 `MediaStream` 包含零个或更多的 `MediaStreamTrack` 对象，代表着各种视频轨（来自硬件或者虚拟视频源，比如相机、视频采集设备和屏幕共享服务等）和声轨（来自硬件或虚拟音频源，比如麦克风、A/D转换器等）。 每一个 `MediaStreamTrack` 可能有一个或更多的通道。这个通道代表着媒体流的最小单元，比如一个音频信号对应着一个对应的扬声器，像是在立体声音轨中的左通道或右通道。


```
var video = document.createElement('video');
var constraints = {
  audio: false,
  video: true
};

function successCallback(stream) {
  window.stream = stream;  //MediaStream对象
  video.srcObject = stream;
}

function errorCallback(error) {
  console.log('navigator.getUserMedia error: ', error);
}

function getMedia(constraints) {
  if (window.stream) {
    video.src = null;
    window.stream.getVideoTracks()[0].stop();
  }
  navigator.mediaDevices.getUserMedia(
    constraints
  ).then(
    successCallback,
    errorCallback
  );
}
```


**注意：**
*Chrome 47以后，getUserMedia API只能允许来自“安全可信”的客户端的视频音频请求，如HTTPS和本地的Localhost。如果页面的脚本从一个非安全源加载，`navigator`对象中则没有可用的`mediaDevices`对象，Chrome抛出错误。*


##### 2. MediaDevices.getDisplayMedia()


这个 MediaDevices  接口的 `getDisplayMedia` 方法提示用户去选择和授权捕获展示的内容或部分内容（如一个窗口）在一个  MediaStream 里。其中包含一个视频轨道（视频轨道的内容来自用户选择的屏幕区域以及一个可选的音频轨道），然后流能被用 MediaStream Recording API 记录或传输一部分去一个WebRTC 会话。

```
    navigator.getDisplayMedia({ video: true })
      .then(stream => {
        // 成功回调的流，将它赋给video元素；
        videoElement.srcObject = stream;
      }, error => {
        console.log("Unable to acquire screen capture", error);
      });
      
```

##### 3. getUserMedia与getDisplayMedia比较

这两个难兄难弟大部分操作是相同的，除了以下几点：

**getUserMedia**

 - 获取的mediaStream可以包括例如视频轨道（来自硬件或者虚拟视频源，比如相机、视频采集设备和屏幕共享服务等），音频轨道（来自硬件或虚拟音频源，比如麦克风、A/D转换器等）以及可能的其他轨道类型。
 - 可以实现约束，接受`MediaStreamConstraints`约束参数对捕获的`mediaStream`进行限制。

**getDisplayMedia**

 - `MediaStream`对象只有一个`MediaStreamTrack`用于捕获的视频流，没有`MediaStreamTrack`对应捕获的音频流。
 - 不能实现约束，`constraints`参数不接受`MediaTrackConstraints` 值。
 - 不能保持权限，确认要分享的屏幕内容后，不能发生更改，除非`reload`重新唤醒Screen Capture。
