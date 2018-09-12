---
layout: post
title: NodeJs 中的 http、https 和 http2
categories: Node Http
---

## 一、 HTTP

### 1. http.server

- 创建服务器

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
    res.end();
}).listen(8780);
```

- 设置超时时间（默认是120s）

```javascript
server.setTimeout(60 * 1000, () => {
    console.log('超时回调');
});
```

- 服务器错误处理（实现端口被占用时自动更换端口）

```javascript
import http from 'http';

let port = 80;
const createServer = () => {

    const server = http.createServer((req, res) => {
        res.end("hello");
    }).listen(port);

    console.log("Server running on port:" + port);

    server.on('error', (e) => {
        if (e.code === 'EADDRINUSE') {
            console.log(`端口:${e.port} 被占用`);
            port++;
            createServer();
        }
    });
};
createServer();
```

- 关闭服务器

```javascript
server.close(); // 服务器停止接受新的连接
server.on('close', () => {
    // 服务器最后一个连接关闭时调用
});
```

### 2. http.IncomingMessage

- 获取请求头

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
    console.log(req.headers);// 返回的是header中属性组成的对象
    res.end();
}).listen(8080);
```

- 获取请求方法

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
    if (req.method == 'GET') {}
    if (req.method == 'POST') {}
    res.end();
}).listen(8080);
```

- 获取请求url与GET、POST参数

```javascript
import http from 'http';
import URL from 'url';
import querystring from 'querystring';

const server = http.createServer((req, res) => {
    const url = URL.parse(req.url, true);
    console.log(url); // 获取URL
    if (req.method == "GET") {
        console.log(url.query);// 获取GET参数
    }
    if (req.method == "POST") {
        let body = "";
        req.on('data', function (chunk) {
            body += chunk;
        });
        req.on('end', function () {
            const postQuery = querystring.parse(decodeURI(body));// 获取POST参数
        });
    }
    res.end();
}).listen(8080);
```

### 3. http.ServerResponse

- 写入响应头

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
    res.setHeader('Content-Type', 'text/html');
    res.setHeader('Set-Cookie', ['type=ninja', 'language=javascript']);
    res.end();
}).listen(8780);
```

- 写入响应体

```javascript
import http from 'http';

const server = http.createServer((req, res) => {
    res.write('{name:"liuhuihao"}');
    res.end();
}).listen(8780);
```

## 二、 HTTPS

![http2-009.png](https://geminate.github.io/assets/images/2018/http2-009.png)

### 1. 创建公钥、私钥及证书

- 创建私钥

```shell
openssl genrsa -out privatekey.pem 1024
```

- 创建证书签名请求

```shell
openssl req -new -key privatekey.pem -out certrequest.csr
```

- 获取证书，线上证书需要经过证书授证中心签名的文件；下面只创建一个学习使用证书

```shell
openssl x509 -req -in certrequest.csr -signkey privatekey.pem -out certificate.pem
```

- 创建pfx文件

```shell
openssl pkcs12 -export -in certificate.pem -inkey privatekey.pem -out certificate.pfx
```

### 2. http.server

- 创建服务器

```javascript
import https from 'https';
import fs from 'fs';

const options = {
    key: fs.readFileSync('dist/privatekey.pem'), // 私钥
    cert: fs.readFileSync('dist/certificate.pem') // 公钥
};

https.createServer(options, (req, res) => {
    res.end();
}).listen(8000);
```

## 三、 HTTP2

### 1. 优势

#### (1)多路复用 - 雪碧图、多域名CDN、接口合并

- 官方演示 - [https://http2.akamai.com/demo](https://http2.akamai.com/demo)

这是一个HTTP2的演示地址，分别用HTTP/1.1和HTTP/2请求379张图片，对比出HTTP/2在速度上的优势

![http2-001.png](https://geminate.github.io/assets/images/2018/http2-001.png)
打开控制台查看网络请求，我们可以发现HTTP/2和HTTP/1.1的明显区别

HTTP/1.1：
![http2-003.png](https://geminate.github.io/assets/images/2018/http2-003.png)

HTTP/2：
![http2-002.png](https://geminate.github.io/assets/images/2018/http2-002.png)

由上图可以看出，**多路复用允许同时通过单一的 HTTP/2 连接发起多重的请求-响应消息；而HTTP/1.1协议中，浏览器客户端在同一时间，针对同一域名下的请求有一定数量限制。超过限制数目的请求会被阻塞**

![http2-004.jpg](https://geminate.github.io/assets/images/2018/http2-004.jpg)

#### 首部压缩
- http/1.x 的 header 由于 cookie 和 user agent很容易膨胀，而且每次都要重复发送。http/2使用 encoder 来减少需要传输的 header 大小，通讯双方各自 cache一份 header fields 表，既避免了重复 header 的传输，又减小了需要传输的大小。高效的压缩算法可以很大的压缩 header，减少发送包的数量从而降低延迟

#### 服务端推送
- 服务端推送是一种在客户端请求之前发送数据的机制。在 HTTP/2 中，服务器可以对客户端的一个请求发送多个响应。举个例子，如果一个请求请求的是index.html，服务器很可能会同时响应index.html、logo.jpg 以及 css 和 js 文件，因为它知道客户端会用到这些东西。这相当于在一个 HTML 文档内集合了所有的资源。

![http2-005.png](https://geminate.github.io/assets/images/2018/http2-005.png)

### 2. nodeJS http/2 实践

- 搭建http/2 服务器

```javascript
import http2 from 'http2';
import fs from 'fs';

const options = {
    key: fs.readFileSync('dist/privatekey.pem'), // 私钥
    cert: fs.readFileSync('dist/certificate.pem') // 公钥
};

http2.createSecureServer(options, (req, res) => {
    res.end();
}).listen(8078);

```

- 验证 http/2 多路复用

```javascript
import http2 from 'http2';
import fs from 'fs';
import URL from 'url';
import mime from 'mime';

const {HTTP2_HEADER_PATH} = http2.constants;// :path

const options = {
    key: fs.readFileSync('dist/privatekey.pem'), // 私钥
    cert: fs.readFileSync('dist/certificate.pem') // 公钥
};

http2.createSecureServer(options, (req, res) => {
    const url = URL.parse(req.url, true);
    if (url.pathname != "/" && url.pathname != "/favicon.ico") {
        const indexObj = getFdAndHeader(url.pathname);
        res.stream.respondWithFD(indexObj.fd, indexObj.headers);
    } else {
        const indexObj = getFdAndHeader('more.html');
        res.stream.respondWithFD(indexObj.fd, indexObj.headers);
    }
}).listen(8078);

const getFdAndHeader = (fileName) => {
    const filePath = "public/" + fileName;
    const fd = fs.openSync(filePath, 'r');
    const stat = fs.fstatSync(fd);
    const headers = {
        'content-length': stat.size,
        'last-modified': stat.mtime.toUTCString(),
        'content-type': mime.lookup(filePath)
    };
    return {fd, headers};
};
```

效果：
![http2-008.png](https://geminate.github.io/assets/images/2018/http2-008.png)

- 实现 http/2 服务端推送

```javascript
import http2 from 'http2';
import fs from 'fs';
import mime from 'mime';

const {HTTP2_HEADER_PATH} = http2.constants;// :path

const options = {
    key: fs.readFileSync('dist/privatekey.pem'), // 私钥
    cert: fs.readFileSync('dist/certificate.pem') // 公钥
};

http2.createSecureServer(options, (req, res) => {
    const indexObj = getFdAndHeader('index.html');
    const jsObj = getFdAndHeader('someJs.js');
    const cssObj = getFdAndHeader('someCss.css');

    push(res.stream, jsObj);
    push(res.stream, cssObj);

    res.stream.respondWithFD(indexObj.fd, indexObj.headers);

}).listen(8078);

function push(stream, obj) {
    stream.pushStream({[HTTP2_HEADER_PATH]: obj.urlPath}, (pushStream) => {
        pushStream.respondWithFD(obj.fd, obj.headers)
    })
}

const getFdAndHeader = (fileName) => {
    const filePath = "public/" + fileName;
    const fd = fs.openSync(filePath, 'r');
    const stat = fs.fstatSync(fd);
    const urlPath = "/" + fileName;
    const headers = {
        'content-length': stat.size,
        'last-modified': stat.mtime.toUTCString(),
        'content-type': mime.lookup(filePath)
    };
    return {fd, headers, urlPath};
};
```

推送效果：
![http2-006.png](https://geminate.github.io/assets/images/2018/http2-006.png)
无推送效果：
![http2-007.png](https://geminate.github.io/assets/images/2018/http2-007.png)