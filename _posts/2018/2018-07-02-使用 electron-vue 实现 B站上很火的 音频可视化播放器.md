---
layout: post
title: 使用 electron-vue 实现 B站上很火的 音频可视化播放器
categories: Electron
---

项目地址：[https://github.com/geminate/mwave](https://github.com/geminate/mwave)

下载地址：[https://pan.baidu.com/s/1boIHExuBOkMqzXVEE4u3gA](https://pan.baidu.com/s/1boIHExuBOkMqzXVEE4u3gA) **仅为测试项目**

## 一. 启发

经常逛B站的小伙伴们中应该不少看到过 使用 AE 制作的音频可视化视频，例如

![audio-v-3-300x249.png](https://geminate.github.io/assets/images/2018/audio-v-3-300x249.png)

![audio-v-2-300x190.png](https://geminate.github.io/assets/images/2018/audio-v-2-300x190.png)

![audio-v-1-300x168.png](https://geminate.github.io/assets/images/2018/audio-v-1-300x168.png)

看起来是不是很酷炫，当初我还傻傻的在视频下面问别人这是什么播放器QAQ，其实这种视频都是使用 AE 做的视频，并没有相关的播放器实现这种效果。

由于本人的开发方向是前端，猛然想起之前看到的 [electron](https://electronjs.org/) 这个使用前端语言写桌面程序的开源项目，之后点进去了解了一下，感觉可以尝试实现一下上图中的效果。于是花了几天的闲暇时间搞出了这么个 ***仅服务于本人兴趣与学习 ***的 玩意。

本人参照的模板是上图中的第三个，未闻花名那张（[B站上的视频连接](https://www.bilibili.com/video/av14342490)），最终完成的播放器效果上来看大约有 视频中的 70% 左右，没做到视频里那么好看（主要是时间原因，粒子效果、彩色渐变和波形的细致过滤没做），如果硬要做的话应该能实现的差不多，有兴趣的前端小伙伴可以自己尝试下。

下面这张是最终的实现效果Gif

![audio-v-4.gif](https://geminate.github.io/assets/images/2018/audio-v-4.gif)

## 二. Electron 相关

#### 1\. 项目结构

![audio-v-5.png](https://geminate.github.io/assets/images/2018/audio-v-5.png)

项目使用的是 electron-vue 作为骨架，如上图。其中renderer文件夹中的东西和Vue项目基本一致，多出来的东西是main文件夹和外面的IPC.js。main文件夹里面的东西是主进程文件，renderer则是渲染进程文件。IPC.js用来在在两个进程中定义通信渠道。

#### 2\. 主进程与渲染进程

electron中一个很重要的概念就是主进程与渲染进程，简单来说 主进程负责操作系统相关的操作，而渲染进程则负责使用前端语言实现界面展示，运行在webview中。而两个进程之间的通信则要通过 IPC 通道实现。在任意一端监听一条消息，之后在另一端发出这条消息即可，需要注意的是，渲染进程->主进程、主进程->渲染进程的消息发送略有不同。

主进程发消息，渲染进程接收消息：

```javascript
// 主进程使用minWindow发送消息
mainWindow.webContents.send(IPC.SET_MUSIC_LIST, {message:'message'});

// 渲染进程使用 electron.ipcRenderer 监听消息
import electron from 'electron';

electron.ipcRenderer.on(IPC.SET_MUSIC_LIST, (event, message) => {
    console.log(message);
});
```

渲染进程发消息，主进程接收消息：

```javascript
// 渲染进程使用 electron.ipcRenderer 发送消息
import electron from 'electron';

electron.ipcRenderer.send(IPC.RENDER_READY,{message:'message'});

//主进程使用 electron.ipcMain 监听消息
import electron from 'electron';

electron.ipcMain.on(IPC.RENDER_READY, (event, arg) => {
    console.log(arg);
});
```

使用起来相当方便，而主进程和渲染进程内部通信与状态管理则分别用各自的store实现。主进程涉及用户配置的可直接以文件形式保存在用户文件夹，而Vue的状态管理直接使用Vuex即可。

#### 3\. window 创建

```javascript
function createWindow() {
    mainWindow = new BrowserWindow({
        height: 600,
        width: 600,
        titleBarStyle: 'hidden-inset',
        frame: false,
        transparent: true,
    });
    mainWindow.loadURL(winURL);
    mainWindow.on('closed', () => {
        mainWindow = null
    });
}
```

上面为主窗体创建的配置，由于我们的播放器需要整体透明且无上部的标题栏，因此设置 titleBarStyle: ‘hidden-inset’ 和 transparent: true ，注意在这样设置之后，我们还需要对窗口内的 body 设置 -webkit-app-region: drag; 的Css 使整个窗口可拖动，之后在对需要有点击效果的地方(不需要拖动)设置-webkit-app-region: no-drag;

#### 4\. 右下角托盘图标 创建


```javascript
/**
 * Create Tray
 */
function createTray() {
    let iconPath = path.join(__static, 'icons/256x256.png');
    tray = new Tray(iconPath);
    const contextMenu = Menu.buildFromTemplate([
        {
            label: '选择文件夹', type: 'normal', click: onChooseFolderClick
        },
        {label: '退出', type: 'normal', role: 'quit'}
    ]);
    contextMenu.items[1].checked = false;
    tray.setContextMenu(contextMenu);
    tray.setToolTip("mwave");
}

/**
 * when choose folder btn click
 */
function onChooseFolderClick() {
    const musicPaths = dialog.showOpenDialog({
        properties: ['openDirectory']
    });
    if (musicPaths != null && musicPaths != 'undefined') {
        sendMusicList(musicPaths);
    }
}
```

由于播放器上的按钮有限，需要将一些功能性的按钮放在 托盘图标的右键菜单中，可使用electron的Tray对象实现，这里主要是将文件夹选择的功能放在了这里，文件选择可用dialog.showOpenDialog 实现。

#### 5\. Vue 相关组件

Vue组件并没有什么特殊的，我这里拆成了musicAudio、musicCanvas、musicControl、musicInfo、musicName、musicProgress这几个组件，其中进度条的处理需要稍加留意，因为涉及点击与拖动，导致需要判断的逻辑比较多。

## 三. 音频 相关

由于electron中无论是主进程还是渲染进程中均支持node模块，因此播放音频十分方便，我这里是使用 mediaserver 在主进程中创建了一个音乐server，之后在渲染进程中使用audio标签即可。

```javascript
class MusicServer {

    start() {
        const server = http.createServer((req, res) => {
            this.pipeMusic(req, res);
        }).listen(8580);
        return server;
    }

    pipeMusic(req, res) {
        if (store.get("MUSIC_PATHS") == undefined || store.get("MUSIC_PATHS").length <= 0) {
            return this.notFound(res);
        }
        const musicUrl = decodeURIComponent(req.url);
        const fileUrl = path.join(store.get("MUSIC_PATHS")[0], musicUrl.substring(1));
        if (musicUrl.substring(1) == '' || !fs.existsSync(fileUrl)) {
            return this.notFound(res);
        }
        ms.pipe(req, res, fileUrl);
    }

    notFound(res) {
        res.writeHead(200);
        res.end('not found');
    }
}
```

## 四. Canvas 相关

播放器使用 Canvas 实现那一圈 音频可视化效果，主要有两个部分，外圈的柱状条和内圈的跳动颗粒。这里是使用了 WebAudio Api 实现的。

```javascript
createAnalyser() {
    const AC = new (window.AudioContext || window.webkitAudioContext)();
    const analyser = AC.createAnalyser();
    const gainnode = AC.createGain();
    gainnode.gain.value = 1;
    const source = AC.createMediaElementSource(this.$refs.audio);
    source.connect(analyser);
    analyser.connect(gainnode);
    gainnode.connect(AC.destination);
    return analyser;
}
```

常用的api中我们可以用 AC.createGain() 控制音频增益(即音量大小)，可以使用AC.createAnalyser()对音频进行分析。我们在实现音频可视化的时候就是使用 AC.createAnalyser().getByteFrequencyData() 生成频率数组。具体使用方式如下

```javascript
this.analyser.fftSize = 1024;
const arrayLength = this.analyser.frequencyBinCount;
const array = new Uint8Array(arrayLength);
this.analyser.getByteFrequencyData(array);
```

之后我们可以根据 Array 里面的 频率数据进行取值，然后绘制Canvas。绘制的过程就不再详细说明，主要是数学上的计算，涉及到围绕圆的半圈进行绘制，之后取镜像。在处理时为了美观，对内圈数值做过滤处理，对外圈数值做发散处理。我这里只是简单处理了一下，想要更加美观还需要更多的数学处理。

```javascript
/**
 * 绘制内圈 point
 */
drawInner(array, i, ctx) {
    if (i < 136) {
        var point = i % 9 > 4 ? (9 - i % 9) : (i % 9);
        var value = (array[i]) * 120 / 256 * ((5 - point) / 5);
        if (value > 70) {
            value = ((value - 70) * 120 / 50);
        } else {
            value = 0;
        }
        ctx.moveTo(( Math.sin(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300), Math.cos(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300);
        ctx.arc(( Math.sin(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300), Math.cos(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300, 0.6, 0, 2 * Math.PI);

        ctx.moveTo((-Math.sin(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300), Math.cos(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300);
        ctx.arc(( -Math.sin(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300), Math.cos(((i) * 4 / 3) / 180 * Math.PI) * (198 - value) + 300, 0.6, 0, 2 * Math.PI);
    }
},

/**
 * 绘制外圈 bar
 */
drawOuter(array, i, ctx) {
    if (i > 130 && i < 271) {
        var value = (array[i]) * 120 / 256;
        if (value > 20) {
            value = (value - 20) * 120 / 100;
        } else {
            value = 0;
        }
        ctx.moveTo(( Math.sin((i * 4 / 3) / 180 * Math.PI) * 200 + 300), Math.cos((i * 4 / 3) / 180 * Math.PI) * 200 + 300);
        ctx.lineTo(( Math.sin((i * 4 / 3) / 180 * Math.PI) * (200 + value) + 300), Math.cos((i * 4 / 3) / 180 * Math.PI) * (200 + value) + 300);

        ctx.moveTo(( -Math.sin((i * 4 / 3) / 180 * Math.PI) * 200 + 300), Math.cos((i * 4 / 3) / 180 * Math.PI) * 200 + 300);
        ctx.lineTo(( -Math.sin((i * 4 / 3) / 180 * Math.PI) * (200 + value) + 300), Math.cos((i * 4 / 3) / 180 * Math.PI) * (200 + value) + 300);

    }
}
```

最后，本项目仅是本人处于兴趣与学习的目的搞出来的小玩具，很多功能不完善，以后有时间会考虑再优化美化一下~
