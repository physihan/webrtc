# webrtc
web rtc实现实时视频
## 已经完成
- [x] 实现基本的视频读取

## todo
- [ ] 将视频存储为本地文件
- [ ] 本地实现双向视频

## 关键技术
webrtc使用了navigator对象，navigator对象是window对象的一个属性
### Navigator 对象属性
|属性|描述|
:----|:----
|appCodeName|返回浏览器的代码名。|
|appMinorVersion|返回浏览器的次级版本。|
|appName	|返回浏览器的名称。|
|appVersion	|返回浏览器的平台和版本信息。|
|browserLanguage	|返回当前浏览器的语言。|
|cookieEnabled	|返回指明浏览器中是否启用 cookie 的布尔值。|
|cpuClass	|返回浏览器系统的 CPU 等级。|
|onLine|	返回指明系统是否处于脱机模式的布尔值。|
|platform	|返回运行浏览器的操作系统平台。|
|systemLanguage	|返回 OS 使用的默认语言。|
|userAgent|	返回由客户机发送服务器的 user-agent 头部的值。|
|userLanguage	|返回 OS 的自然语言设置。|

### Navigator 对象方法
|方法|	描述|
:--|:--
javaEnabled()|	规定浏览器是否启用 Java。
taintEnabled()	|规定浏览器是否启用数据污点 (data tainting)。

### navigator webrtc API
webrtc使用了一个新的对象方法getUserMedia方法，在不同浏览器实现不同
```javascript
navigator.getUserMedia = navigator.getUserMedia ||
    navigator.webkitGetUserMedia || navigator.mozGetUserMedia;
```    
这样使用来消除不同浏览器中的差异


Navigator.getUserMedia()方法提醒用户需要使用音频（0或者1）和（0或者1）视频输入设备，比如相机，屏幕共享，或者麦克风。如果用户给予许可，successCallback回调就会被调用，MediaStream对象作为回调函数的参数。如果用户拒绝许可或者没有媒体可用，errorCallback就会被调用，类似的，PermissionDeniedError 或者NotFoundError对象作为它的参数。注意，有可能以上两个回调函数都不被调用，因为不要求用户一定作出选择（允许或者拒绝）。
```
navigator.getUserMedia ( constraints, successCallback, errorCallback );
```


>此API已更名为 MediaDevices.getUserMedia()。 请使用那个版本进行替代！ 这个已废弃的API版本仅为了向后兼容而存在。

新的api使用方法如下，使用的是promise这种写法
```
navigator.mediaDevices.getUserMedia(constraints)
.then(function(mediaStream) { ... })
.catch(function(error) { ... })
```
constraints
作为一个MediaStreamConstraints 对象，指定了请求的媒体类型和相对应的参数。

constraints 参数是一个包含了video 和 audio两个成员的MediaStreamConstraints 对象，用于说明请求的媒体类型。必须至少一个类型或者两个同时可以被指定。如果浏览器无法找到指定的媒体类型或者无法满足相对应的参数要求，那么返回的Promise对象就会处于rejected［失败］状态，NotFoundError作为rejected［失败］回调的参数。 

以下同时请求不带任何参数的音频和视频：


>```javascript
>{ audio: true, video: true }
>```

当由于隐私保护的原因，无法访问用户的摄像头和麦克风信息时，应用可以使用额外的constraints参数请求它所需要或者想要的摄像头和麦克风能力。下面演示了应用想要使用1280x720的摄像头分辨率：


>```javascript
>{
>  audio: true,
>  video: { width: 1280, height: 720 }
>}
>```
强制要求获取特定的尺寸时，可以使用关键字min, max, 或者 exact(就是 min == max). 以下参数表示要求获取最低为1280x720的分辨率。
```javascript
{
  audio: true,
  video: {
    width: { min: 1280 },
    height: { min: 720 }
  }
}
```
如果摄像头不支持请求的或者更高的分辨率，返回的Promise会处于rejected状态，NotFoundError作为rejected回调的参数，而且用户将不会得到要求授权的提示。

造成不同表现的原因是，相对于简单的请求值和ideal关键字而言，关键字min, max, 和 exact有着内在关联的强制性，请看一个更详细的例子：
```javascript
{
  audio: true,
  video: {
    width: { min: 1024, ideal: 1280, max: 1920 },
    height: { min: 776, ideal: 720, max: 1080 }
  }
}
```
当请求包含一个##ideal（应用最理想的）##值时，这个值有着更高的权重，意味着浏览器会先尝试找到最接近指定的理想值的设定或者摄像头（如果设备拥有不止一个摄像头）。

 

简单的请求值也可以理解为是应用理想的值，因此我们的第一个指定分辨率的请求也可以写成如下：
```javascript
{
  audio: true,
  video: {
    width: { ideal: 1280 },
    height: { ideal: 720 }
  }
}
```
并不是所有的constraints 都是数字。例如, 在移动设备上面，如下的例子表示优先使用前置摄像头（如果有的话）：

>`{ audio: true, video: { facingMode: "user" } }`
强制使用后置摄像头，请用：

>`{ audio: true, video: { facingMode: { exact: "environment" } } }`

### 还可以控制帧率
在某些情况下，比如WebRTC上使用受限带宽传输时，低帧率可能更适宜。

>`var constraints = { video: { frameRate: { ideal: 10, max: 15 } } };`
对于移动设备可以控制选择前置或者后置摄像头

```javascript
var front = false;
document.getElementById('flip-button').onclick = function() { front = !front; };

var constraints = { video: { facingMode: (front? "user" : "environment") } };
```

完整的例子

```javascript
var promisifiedOldGUM = function(constraints) {

  // First get ahold of getUserMedia, if present
  var getUserMedia = (navigator.getUserMedia ||
      navigator.webkitGetUserMedia ||
      navigator.mozGetUserMedia);

  // Some browsers just don't implement it - return a rejected promise with an error
  // to keep a consistent interface
  if(!getUserMedia) {
    return Promise.reject(new Error('getUserMedia is not implemented in this browser'));
  }

  // Otherwise, wrap the call to the old navigator.getUserMedia with a Promise
  return new Promise(function(resolve, reject) {
    getUserMedia.call(navigator, constraints, resolve, reject);
  });
		
}

// Older browsers might not implement mediaDevices at all, so we set an empty object first
if(navigator.mediaDevices === undefined) {
  navigator.mediaDevices = {};
}

// Some browsers partially implement mediaDevices. We can't just assign an object
// with getUserMedia as it would overwrite existing properties.
// Here, we will just add the getUserMedia property if it's missing.
if(navigator.mediaDevices.getUserMedia === undefined) {
  navigator.mediaDevices.getUserMedia = promisifiedOldGUM;
}


// Prefer camera resolution nearest to 1280x720.
var constraints = { audio: true, video: { width: 1280, height: 720 } };

navigator.mediaDevices.getUserMedia(constraints)
.then(function(stream) {
  var video = document.querySelector('video');
  video.src = window.URL.createObjectURL(stream);
  video.onloadedmetadata = function(e) {
    video.play();
  };
})
.catch(function(err) {
  console.log(err.name + ": " + err.message);
});
```