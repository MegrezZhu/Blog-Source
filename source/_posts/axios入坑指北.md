---
title: axios入坑指北
date: 2018-04-08 10:07:07
tags:
	- JavaScirpt
	- Node.js
---

[axios](https://github.com/axios/axios)是一个基于JavaScript的HTTP请求模块，为JavaScript的两个主要环境（Node.js与Browser）提供了一致的HTTP接口。

由于axios的[readme](https://github.com/axios/axios/blob/master/README.md)里面已经有十分详尽的API文档以及设置项说明，因此这篇博客将主要介绍一些文档里没有的东西（a.k.a 坑），使用部分将简单带过。

Note：本篇博客基于axios目前的最新版本`0.18.0`，由于axios近期可能会有大的更新(see [this issue](https://github.com/axios/axios/issues/1333))，届时本篇博客部分内容可能将过时。

<!-- More -->

## Features

以下来自readme：

- Make [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) from the browser
- Make [http](http://nodejs.org/api/http.html) requests from node.js
- Supports the [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
- Intercept request and response
- Transform request and response data
- Cancel requests
- Automatic transforms for JSON data
- Client side support for protecting against [XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

在一般的使用情景下，主要的痛点在于：

* Promise API
* 取消请求的能力（which is使用原生Promise比较难办的需求）
* 通过intercept扩展功能

## 基本使用

### 引入

对于Browser：

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

对于Node.js，以及使用了[Webpack](https://github.com/webpack/webpack)等构建工具的前端项目：

```javascript
import axios from 'axios'; // es module
```

or:

```javascript
const axios = require('axios'); // Node.js / CommonJS
```

### 配置

axios有许多配置项，它们的默认值在[这里](https://github.com/axios/axios#request-config)可以察看。在需要对默认值进行修改的时候（比如修改请求的默认baseURL），通过：

```javascript
const ax = require('axios').create({
    baseURL: 'http://my.custom-base-domain.com',
    timeout: 3000
});
```

对默认配置进行覆盖。

而在请求的时候，可以通过提供配置项来对上面的设置进行再次覆盖：

```javascript
ax.get('https://www.google.com', { timeout: 5000 });
```

总地来说，axios中的配置项的优先级为：请求中配置>用户默认配置>axios默认配置。

### 请求

由于axios提供了基于Promise的API，因此可以很优雅地写出链式的代码：

```javascript
axios
	.get('https://www.google.com')
    .then(res => {
    	console.log(res.data)
	})
	.catch(console.error);
axios
	.post('/api/user', { name: 'chu' })
    .then(({ status }) => {
    	console.log(`regist done with status code ${status}.`); 
	});
```

当然，一般更推荐Promise和Async/Await配合食用（需要注意执行环境的兼容性）：

```javascript
(async () => {
    try {
        const { data } = await axios.get('http://fake-domain/news-today');
        console.log(data);
    } catch (err) {
        console.error('error!');
    }
})()
	.catch(console.error);
```

## 入坑与爬坑

> Note again: 这里讲的是`0.18.0`中的情况，在新的axios大版本将有可能：
>
> * 修复这些问题
> * 原生提供解决方案

### Cookies

虽然说axios旨在为Node.js端与Browser端提供统一的HTTP API， 但在Cookie的支持上还是有一点不同。

在浏览器端，由于浏览器自带对Cookie的管理，因此只需要在axios的配置中加入`withCredentials: true`即可。

而在Node.js端，由于node环境原生并没有对cookie进行handle（而很神奇地，axios也并没有做处理），因此想要在node下获得和浏览器端一致的体验，需要引入下面两个额外的模块：

* [axios-cookiejar-support](https://www.npmjs.com/package/axios-cookiejar-support)
* [tough-cookie](https://www.npmjs.com/package/tough-cookie)

并进行如下的配置：

```javascript
const axios = require('axios').default;
const axiosCookieJarSupport = require('axios-cookiejar-support').default;
const tough = require('tough-cookie');
 
axiosCookieJarSupport(axios);
 
const cookieJar = new tough.CookieJar();
 
axios.get('https://google.com', {
  jar: cookieJar, // tough.CookieJar or boolean
  withCredentials: true // If true, send cookie stored in jar
})
.then(() => {
  console.log(cookieJar);
});
```

注意，引入模块之后仍然需要`withCredentials: true`的配置项来使用cookies。

### Proxy

> 这一项由于我不清楚是使用方式的原因还是axios本身的bug，因此只能提供一种workaround。如果是我自己使用方式不当导致的，请在评论区或者发邮件告诉我:)

由于一些不可名状的原因，在写需要进行HTTP请求的代码的时候（特别是各种爬虫），经常需要通过中间代理，而axios自身虽然在配置项中提供了`proxy`字段，但亲测它的可用性比较差（主要是无法通过http proxy访问https的链接），因此需要一些额外的工作。

需要用到的库：[caw](https://www.npmjs.com/package/caw)

> caw是一个用于提供http/https agent的库，具有相似功能的还有：tunnel, http-proxy(s)-agent等，但经测试似乎只有caw能与axios兼容:(

使用方式：

```javascript
const ax = require('axios');
const caw = require('caw');

ax.get(
	'https://google.com',
    {
        httpAgent: caw('http://127.0.0.1:1080'),
        httpsAgent: caw('https://127.0.0.1:1080', { protocol: })
    }
);
```

### Retry

> 这一点并不是axios的bug或者不足之处，只是我自己在写项目时候的一个痒点，在这里分享出来。

由于网络服务的不稳定性，在一些比较重要的、需要保证成功请求的情景下，需要通过多次请求来保证成功，于是经常会写出下面这样的代码：

```javascript
async function getSomething (url) {
    let maxRetry = 3;
    while (maxRetry--) {
        try {
            const { data } = await axios.get(url);
            return data;
        } catch (err) {}
    }
    throw new Error('failed too many times.');
}
```

当然，为了D.R.Y.，到处这样写当然是不优雅的，因此可以通过axios的interceptor来达成：

```javascript
axios.interceptors.response.use(
  res => res,
  err => {
    if (err.message && err.message.match(/timeout/)) { // timeout case
      const config = err.config;
      if (config.method !== 'get') return Promise.reject(err); // only retry GET methods
      if (config._leftRetry) config._leftRetry--;
      else config._leftRetry = 3; // default retry times

      if (config._leftRetry) {
        return axios(config);
      } else {
        return Promise.reject(err);
      }
    }
    return Promise.reject(err);
  }
);
```

当然上面代码还有许多修改的空间，比如可以根据具体需求修改重试的条件（在上面的代码中我只对超时的情景进行了重新请求），或者修改重试的method条件。

Note：重试时需要保证调用的API的幂等性[Idempotency](http://restcookbook.com/HTTP%20Methods/idempotency/)，否则重试可能会带来额外的副作用。

### Timeout

axios自身即对请求超时有支持（设置项`timeout: 3000`ms），但超时的计算仅限于从发出请求到服务端返回response header的时刻，也就是说从收到response header到完整接收response body的这段时间是没有计算在内的。

而在一些比较少见的情况下（特别是response body比较大，比如图片下载的情况），response body的接收速度会降为0，但由于连接仍未被断开，因此仍未算作请求失败——于是就出现了请求时间>>超时时限仍未结束的问题。

其实从严格意义上来讲，这并不是bug或者缺陷。不过有时候尽早结束没有速度的下载并重新开始一个新的请求比等待一个不定长的时间来得更加行之有效，于是可以使用Interceptor结合[cancelToken](http://restcookbook.com/HTTP%20Methods/idempotency/)对下载超时的情况增加timeout限制。

```javascript
axios.interceptors.request.use(
  config => {
    const cancelSource = CancelToken.source();
    const token = cancelSource.token;

    setTimeout(() => {
      cancelSource.cancel('custom timeout.');
    }, TIMEOUT);

    return {
      ...config,
      cancelToken: cancelSource.token
    };
  },
  Promise.reject
);
```

当然，这段代码只是一个不成熟的草稿，有一些额外的问题（比如会覆盖原请求的cancelToken）。

### Incomplete Body

与上一个类似，在response body较大、网络状态不佳的时候有可能出现。在传输过程中连接被中断，axios会认为是连接正常终止：

![incomplete file](1523163443871.png)

对于这个问题，没有什么很优雅的解决方法，可以通过设置response interceptor来加一层body size的判断。

## 最后

其实相比于另外一个HTTP库[request](https://github.com/request/request)，axios还是有比较多的坑的。但毕竟头比较铁，尝试着找到了一些workaround配合上也能用。

（毕竟相比request，个人觉得axios的名字更酷炫一点哈哈哈哈）