# 00-使用webAudio播放音乐

## 摘要
Web Audio API紧紧围绕着一个概念设计：audio context，它就像是一个有向图，途中的每个节点都是一个audio node，音频数据从源节点按照程序中指定的，一步一步走向目的节点。
audio context就像filter manager，而audio node则是各种filter，如果你是个linux开发者，这看起来像是pipe。一个audio context中的audio node可以有很多，包括四种：
1. 源节点（source node）
2. 处理节点（gain node、panner node、wave shaper node、oscillator node、delay node、convolver node等）
4. 目的节点（destination node）

## 准备音频资源

### 加载音频资源方式一：本地加载

首先要准备一个mp3格式的音频，然后通过file表单元素将该音频文件加载进来，如下代码：

```html
<form name="myForm">
	<input type="file" name="uploadMusic" id="uploadMusic" />
</form>
```

```javascript
var uploadMusic = document.getElementById('uploadMusic');
uploadMusic.onchange = function (e) {
    var file = uploadMusic.files[0];
    var fr = new FileReader();
    
    fr.onload = function (e) {
        console.dir(this.result);
    }
    
    fr.readAsArrayBuffer(file);
}
```

代码说明: 
当file元素打开音频文件后，就会进入onchange事件，而文件信息会存入当前file元素的files数组中，数组的每一个元素都是一个File类型的对象，注意：此时只是加载到了文件的信息而不是文件本身的数据，如下图：
![Alt text](./20160902000607.jpg)
可以看到只有文件信息（size是文件的大小，单位是Byte），而没有文件数据，所以要想得到文件本身的数据，则需要文件读取器FileReader。
实例化一个文件读取器的时候是不接收任何参数的，实例化成功后，可以将文件内容以指定的类型读入，读取成功后，将保存在文件读取器的result属性中。

这里文件读取器的onload事件，表示音频资源读取完毕并且读取成功，事件对象e是ProgressEvent类型，记录加载进度的对象，其中loaded和total属性分别记录了加载了多少字节Byte的数据，如下图所示：
![Alt text](./20160902002708.jpg)
其中readyState: 2表示已完成全部的读取请求，1表示数据正在被加载，0表示还没有加载任何数据。

读取文件内容的方法有四种，这四个方法都需要传入Blob或File类型的对象作为参数：
1. readAsDataURL
2. readAsBinaryString
3. readAsText
4. readAsArrayBuffer

在上述代码中，采用的是readAsArrayBuffer方式读取的音频资源，因为webAudio在对音频资源进行解码的时候，需要接收的资源类型是arrayBuffer类型。

总结一下加载音频资源的过程：
1. 打开文件，获取文件信息
2. 创建文件读取器
3. 根据需要的类型读入文件内容
4. 从文件读取器的result属性中获取读入结果

以上是在本地加载音频资源的方式。

### 加载音频资源方式二：ajax加载

从服务器端加载音频资源，可以使用node.js快速搭建一个静态web资源服务器，例如使用express框架并使用静态资源中间件，前端发出请求，加载音频资源的代码如下：

```javascript
var audioURL = 'http://localhost:3000/GoTime.mp3';
var xhr = new XMLHttpRequest();
xhr.open('GET', audioURL, true);
xhr.responseType = 'arraybuffer';

xhr.onload = function (e) {
    // this.response // 音频数据
}

xhr.send(); // 最后别忘了发送
```

## 播放加载的音频资源

代码如下：

```javascript
var ctx = new AudioContext();
ctx.decodeAudioData(this.result, function (buffer) {
    var source = ctx.createBufferSource();
    source.buffer = buffer;
    source.connect(ctx.destination);
    source.start(0);
});
```

要播放音频资源，首先要创建上下文对象，随后对加载的音频数据进行解码，解码的过程是异步的，解码成功后，将会返回解码后的buffer对象。
随后创建bufferSource对象，并且挂载上buffer数据，将音频资源连接到最终的目的地也就是输出设备上destination，然后调用start方法播放音频。

decodeAudioData方法的语法：

```javascript
ctx.decodeAudioData(audioData, function (decodedData) {
    // ...
});

ctx.decodeAudioData(audioData).then((decodedData)=> {
    // ...
});
```