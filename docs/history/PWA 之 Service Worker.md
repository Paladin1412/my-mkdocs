> Progressive Web Apps ，简称 PWA， 号称下一代 Web 应用模型，兀自风靡。作为前端爱好者，也想一窥究竟。说起 PWA ，必绕不开 Service Worker 和离线 Cache ，本文简述其原理，并通过一个简单 DEMO 验证一下。


<br />

# 1. 概念

Service Worker 是一段**独立于当前页面，运行于浏览器后台进程**的脚本。

**注意**：

- Service Worker 异步运行，且独立于全局上下文。
- Service Worker 没有操作DOM的权限。但可通过 `postMessage` 方法与Web页面通信，让页面操作DOM。
- Service Worker 是事件驱动。处于 Idle 状态时，随时会被浏览器回收，以节省内存。

<br />



# 2. 生命周期

![sw-lifecycle](http://47.94.165.69:3031/upload/aaa/474a05cc53dc7.png)  

由图可见，一个 Service Worker 一生能经历如下过程：

1. 安装；
2. 激活，可通过 `chrome://inspect/#service-workers` 查看到当前运行的service worker；
3. 空闲，监听 fetch 和 message 事件；
4. 销毁，若长期不使用或内存有限，浏览器会销毁之，以回收内存。



### fetch 事件

向页面发送 http 请求时，Service Worker 可以通过 fetch 事件拦截请求，并且给出自己的响应。



### message 事件

页面和 serviceWorker 之间可以通过 postMessage() 方法发送消息，发送的消息可以通过 message 事件接收到。

<br />



# 3. 环境和调试



## 3.1 HTTPS

服务器需要支持 HTTPS。因为通过 Service Worker 可以劫持连接，伪造和过滤响应；只能在 HTTPS 的网页上注册 Service Workers，防止以上恶意篡改行为。



## 3.2 调试工具

- `chrome://inspect/#service-workers` : 查看 Service workers 列表。


- `chrome://serviceworker-internals/` : 查看 Service Worker 详细信息和状态，且调试时可执行 unregister、stop、start 等操作。
- `ChromeDevTools` : 选择 Application 选项卡 > Service Workers, 可查看 Service Worker 脚本，设置网络状态，包括 offline， update on reload 等。



## 3.3 离线存储数据建议

对 **URL寻址资源** (URL addressable resources)，使用 [Cache API](https://davidwalsh.name/cache)（这是 [Service Worker](https://developers.google.com/web/fundamentals/primers/service-worker/) 的一部分）。对其他数据，使用 **IndexedDB**（包裹进 [Promises](http://www.html5rocks.com/en/tutorials/es6/promises/)）。

<br />



# 4. Service Worker 原理

以下是通过 Service Worker 和 Cache API 实现的一个URL寻址资源缓存的 DEMO。原文参阅 [service-workers](https://developers.google.com/web/fundamentals/getting-started/primers/service-workers)。 



## 4.1 Register a service worker

首先在页面里注册 service worker ，这就告诉了浏览器你的 service worker JavaScript file 在哪儿。

```
if ('serviceWorker' in navigator) {
	window.addEventListener('load', function() {
		navigator.serviceWorker.register('/sw.js').then(function(registration) {
      		// Registration was successful
      		console.log('ServiceWorker registration successful with scope: ', registration.scope);
    	}, function(err) {
      		// registration failed :(
      		console.log('ServiceWorker registration failed: ', err);
    	});
    });
}
```

You can call `register()` every time a page loads without concern; the browser will figure out if the service worker is already registered or not and handle it accordingly.

**注意**： service worker file 在域的根目录下，表示 the service worker's scope will be the entire origin. 若注册为`/example/sw.js`，则表示 service worker 只可见 `fetch` events for pages whose URL starts with `/example/` 。



## 4.2 Install a service worker

注册完成之后，在 service worker script 里处理 `install` 事件。在 `install` 事件的 callback 里执行以下步骤：

1. Open a cache.
2. Cache our files.
3. Confirm whether all the required assets are cached or not.

```
var CACHE_NAME = 'my-site-cache-v1';
var urlsToCache = [
  '/',
  '/styles/main.css',
  '/script/main.js'
];

self.addEventListener('install', function(event) {
  // Perform install steps
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function(cache) {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});
```

`event.waitUntil()` method 的参数为 a chain of promises (`caches.open()` and `cache.addAll()`). 打开名为 CACHE_NAME 的 cache ，然后缓存 urlsToCache 数组中所列的文件。

**注意** ： If **any** of the files fail to download, then the install step will fail. This allows you to rely on having all the assets that you defined, but does mean you need to be careful with the list of files you decide to cache in the install step.



## 4.3 Cache and return requests

安装成功之后，当刷新或访问另一个页面时，service worker 将开始接收 `fetch` 事件。

拦截网络请求， `event.respondWith()` method 的参数为 `caches.match()` ， 用来匹配请求。若缓存命中，则返回缓存资源，否则，返回网络请求 `fetch` 的结果。

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Cache hit - return response
        if (response) {
          return response;
        }
        return fetch(event.request);
      }
    )
  );
});
```

若想要 cache new requests cumulatively， 我们可以通过缓存 response of the fetch request 来实现。

步骤如下：

1. Add a callback to `.then()` on the `fetch` request.
2. Once we get a response, we perform the following checks:
   1. Ensure the response is valid.
   2. Check the status is `200` on the response.
   3. Make sure the response type is **basic**, which indicates that it's a request from our origin. This means that requests to third party assets aren't cached as well.
3. If we pass the checks, we [clone](https://fetch.spec.whatwg.org/#dom-response-clone) the response. 

**注意**： request 和 response 都是 stream, 只能消费一次。因此将它们克隆，一份传给 cache， 一份传给 browser。

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Cache hit - return response
        if (response) {
          return response;
        }

        // IMPORTANT: Clone the request. A request is a stream and
        // can only be consumed once. Since we are consuming this
        // once by cache and once by the browser for fetch, we need
        // to clone the response.
        var fetchRequest = event.request.clone();

        return fetch(fetchRequest).then(
          function(response) {
            // Check if we received a valid response
            if(!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }

            // IMPORTANT: Clone the response. A response is a stream
            // and because we want the browser to consume the response
            // as well as the cache consuming the response, we need
            // to clone it so we have two streams.
            var responseToCache = response.clone();

            caches.open(CACHE_NAME)
              .then(function(cache) {
                cache.put(event.request, responseToCache);
              });

            return response;
          }
        );
      })
    );
});
```



## 4.4 Update a service worker

service worker 何时更新，更新步骤又如何？

1. 跳转至本页时，浏览器尝试重新下载 service worker JavaScript file，若与当前不同，则认为它是新的。
2. 新 service worker  启动，`install` event 被触发。
3. 此时，老 service worker 依旧控制页面，新 service worker 进入 `waiting` 状态。
4. 此页面关闭时，老 service worker 被杀死，新 service worker 才掌握控制权。
5. 新 service worker 的 `activate` event 被触发。



`activate` 事件使用的一个场景就是， if you were to wipe out any old caches in the install step。

举个栗子，比如我们想将 cache 一分为二 `'pages-cache-v1'` 和  `'blog-posts-cache-v1'`。意味着，我们在 install step 创建这两个 cache， 在 activate step 删除 `'my-site-cache-v1'`。

```
self.addEventListener('activate', function(event) {

  var cacheWhitelist = ['pages-cache-v1', 'blog-posts-cache-v1'];

  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```



## 4.5 验证步骤

1.  `ChromeDevTools` 中 Application 选项卡 > Cache Storage 查看缓存成功与否。

2.  若缓存成功，Application 选项卡 > Service Workers 将网络置为 offline。

3.  重新访问页面，server worker 拦截请求，缓存命中。

    ​

DEMO 地址，见 https://silverbulllet.github.io/

<br />



# 5. My First Progressive Web App

- 下载 [The sample code](https://github.com/googlecodelabs/your-first-pwapp/archive/master.zip), 包含 WeatherApp 的代码，不用赘述。




## 5.1 Cache the site assets

service worker 注册后，install event 触发， 在 callback 内 cache 应用需要的所有资源。

```
var cacheName = 'my-first-pwapp-assets-v1';

var filesToCache = [
  '/my-first-pwapp/',
  '/my-first-pwapp/index.html',
  '/my-first-pwapp/scripts/app.js',
  '/my-first-pwapp/styles/inline.css',
  '/my-first-pwapp/images/clear.png',
  '/my-first-pwapp/images/cloudy-scattered-showers.png',
  '/my-first-pwapp/images/cloudy.png',
  '/my-first-pwapp/images/fog.png',
  '/my-first-pwapp/images/ic_add_white_24px.svg',
  '/my-first-pwapp/images/ic_refresh_white_24px.svg',
  '/my-first-pwapp/images/partly-cloudy.png',
  '/my-first-pwapp/images/rain.png',
  '/my-first-pwapp/images/scattered-showers.png',
  '/my-first-pwapp/images/sleet.png',
  '/my-first-pwapp/images/snow.png',
  '/my-first-pwapp/images/thunderstorm.png',
  '/my-first-pwapp/images/wind.png'
];

self.addEventListener('install', function(e) {
  console.log('[ServiceWorker] Install');
  e.waitUntil(
    caches.open(cacheName).then(function(cache) {
      console.log('[ServiceWorker] Caching app shell');
      return cache.addAll(filesToCache);
    })
  );
});
```



## 5.2 Serve the app shell from the cache

Service workers 可拦截网络请求，并在 service worker 内部处理。

```
self.addEventListener('fetch', function(e) {
  console.log('[ServiceWorker] Fetch', e.request.url);
  e.respondWith(
    caches.match(e.request).then(function(response) {
      return response || fetch(e.request);
    })
  );
});
```



## 5.3 Use service workers to cache the forecast data

使用 service worker 来 cache 天气预报的数据。

选择正确的**缓存策略**很重要，The [cache-first-then-network](https://jakearchibald.com/2014/offline-cookbook/#cache-network-race) strategy is ideal for our app. 这样能尽可能快的显示获取的天气数据，而且能网络返回最新数据时，第一时间更新。

Cache-first-then-network 将触发两个异步请求，一个到缓存，一个到网络。



### 5.3.1 拦截请求，缓存响应

 检查拦截到的请求，若请求的 URL 以 weather API 地址开头，则缓存响应的克隆，同时返回响应。

```
var dataCacheName = 'weatherData-v1';

// 修改 fetch event handler 如下, 将拦截到的请求分别处理
self.addEventListener('fetch', function(e) {
  console.log('[Service Worker] Fetch', e.request.url);
  var dataUrl = 'https://query.yahooapis.com/v1/public/yql';
  if (e.request.url.indexOf(dataUrl) > -1) {
    /*
     * When the request URL contains dataUrl, the app is asking for fresh
     * weather data. In this case, the service worker always goes to the
     * network and then caches the response. This is called the "Cache then
     * network" strategy:
     * https://jakearchibald.com/2014/offline-cookbook/#cache-then-network
     */
    e.respondWith(
      caches.open(dataCacheName).then(function(cache) {
        return fetch(e.request).then(function(response){
          cache.put(e.request.url, response.clone());
          return response;
        });
      })
    );
  } else {
    /*
     * The app is asking for app shell files. In this scenario the app uses the
     * "Cache, falling back to the network" offline strategy:
     * https://jakearchibald.com/2014/offline-cookbook/#cache-falling-back-to-network
     */
    e.respondWith(
      caches.match(e.request).then(function(response) {
        return response || fetch(e.request);
      })
    );
  }
});
```



### 5.3.2 从缓存中获取数据

app 发出俩请求，一个来自 cache，一个通过 XHR。若 cache 内有数据，则直接渲染；若 XHR 仍在继续，则渲染数据被更新为 weather API 直接获取的数据。

在 `app.getForecast()` 内增加如下代码：

```
    // TODO add cache logic here
    if ('caches' in window) {
      /*
       * Check if the service worker has already cached this city's weather
       * data. If the service worker has the data, then display the cached
       * data while the app fetches the latest data.
       */
      caches.match(url).then(function(response) {
        if (response) {
          response.json().then(function updateFromCache(json) {
            var results = json.query.results;
            results.key = key;
            results.label = label;
            results.created = json.query.created;
            app.updateForecastCard(results);
          });
        }
      });
    }
```



### 5.3.4 测试验证

自此，PWApp 已能完全离线地址，地址见 https://silverbulllet.github.io/my-first-pwapp/ 。

保存若干城市，点击刷新获取最新数据，置为离线，重载页面。

