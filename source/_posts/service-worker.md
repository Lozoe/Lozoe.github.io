---
title: service-worker
date: 2019-08-19 11:14:10
categories: [Browser]
tags:
---

## 一、什么是Service Worker

service worker是一个浏览器背后运行的脚步，独立于web页面，为无需一个页面或用户交互的功能打开了大门。今日，它包含了推送通知和背景异步（push notifications and background sync）的功能。将来，service worker将支持包括periodic sync or geofencing的功能。 本教程中讨论的核心功能是拦截和处理网络请求的能力，包括以编程方式管理响应缓存。

<!-- more -->

## 二、PWA和Service Worker的关系

PWA (Progressive Web Apps) 不是一项技术，也不是一个框架，我们可以把她理解为一种模式，一种通过应用一些技术将 Web App 在安全、性能和体验等方面带来渐进式的提升的一种 Web App的模式。对于 webview 来说，Service Worker 是一个独立于js主线程的一种 Web Worker 线程， 一个独立于主线程的 Context，但是面向开发者来说 Service Worker 的形态其实就是一个需要开发者自己维护的文件，我们假设这个文件叫做 sw.js。通过 service worker 我们可以代理 webview 的请求相当于是一个正向代理的线程，fiddler也是干这些事情），在特定路径注册 service worker 后，可以拦截并处理该路径下所有的网络请求，进而实现页面资源的可编程式缓存，在弱网和无网情况下带来流畅的产品体验，所以 service worker 可以看做是实现pwa模式的一项技术实现。

三、Service Worker注意事项

- service worker 是一种JS工作线程，无法直接访问DOM, 该线程通过postMessage接口消息形式来与其控制的页面进行通信;

- service worker 广泛使用了Promise，这些在接下来代码示例中将会看到;

- 目前并不是所有主流浏览器支持 service worker, 可以通过 navigator && navigator.serviceWorker 来进行特性探测;

- 在开发过程中，可以通过 localhost 使用服务工作线程，如若上线部署，必须要通过https来访问注册服务工作线程的页面，但有种场景是我们的测试环境可能并不支持https，这时就要通过更改host文件将localhost指向测试环境ip来巧妙绕过该问题（例如：192.168.22.144 localhost）;

## 四、Service Worker生命周期

### 注册

这里需要注意的是register方法注册服务工作线程文件的位置，该path就是默认的 serviceworker 的作用域，例如注册path为/a/b/service-worker.js,则默认scope为/a/b/,当然也可以通过传入{scope: ‘/a/b/c/‘}来指定自己的scope，但这里要特别注意的是，传入的scope参数一定是在默认作用域范围内再自定义（例如/a/b/c/），反之自定义为/d/e/就不行；
通俗来讲，上面提到的scope就是 service worker 能够控制和发挥作用的范围；
注意注册是在自己的业务代码中进行，后面会有具体通过插件来实现注册的代码示例

```js
if(navigator && navigator.serviceWorker) {
  navigator.serviceWorker.register('/service-worker.js').then(function(registration) {
    console.log(registration)
  }).catch(function(err) {
    console.log(err)
  })
}
```

### 安装

下面代码就是前面注册的service-worker.js文件内容；

我们通过install事件来定义安装步骤，通过缓存名称调用caches.open(), 之后再调用cache.addAll()并传入具体缓存文件清单数组，这是一个Promise链式event.waitUntil()方法带有Promise参数并使用它来判断花费耗时以及安装是否成功；

正如前面提到，安装过程中如果所有清单中文件成功缓存，则安装结束，否则安装过程视为失败，所以在实践中我们尽可能缓存核心资源以避免服务工作线程未能安装;

```js
var cacheVersion = 'test_01';
// 安装服务工作线程
self.addEventListener('install', function(event) {
  // 需要缓存的资源
  var cacheFiles = [
    '/dist/index.html',
    '/dist/js/bundle.js'
  ];
  console.log('service worker: run into install');
  event.waitUntil(caches.open(cacheVersion).then(function(cache) {
      return cache.addAll(cacheFiles);
  }));
});
```

### 激活

- 在某个时间点服务工程线程需要更新（例如：service-worker.js文件发生更改并上线）,用户访问页面时浏览器会尝试在后台重新下载service-worker.js,如果服务工程线程文件与当前所用文件存在字节差异，则将其视为“新服务工作线程”;
- 新服务工作线程将会启动，且将会触发 install 事件;
- 此时旧的服务工作线程仍将控制着当前页面，因此新服务工作线程将会进入waiting状态;
- 当网站当前页面关闭时，旧服务工作线程将会终止，新服务工作线程将会取得控权;
- 新服务工作线程取得控制权后，将会触发 activate 事件;
- 监听 activate 事件的回调函数中常见的任务是管理缓存，前面我也提到过这是管理旧缓存的绝佳时机，因为如果在安装步骤中清理了旧缓存，由于旧的服务工作线程仍旧控制着页面，将无法从缓存中提取文件，但是在 activate 时旧服务工作线程已经终止了页面控制权，所在在这里清理旧缓存再合适不过;

```js
// 新的service worker线程被激活（其实和离线包一样存在"二次生效"的机理）
self.addEventListener('activate', function(event) {
  console.log('service worker: run into activate');
  event.waitUntil(caches.keys().then(function(cacheNames) {
    return Promise.all(cacheNames.map(function(cacheName) {
      // 注意这里cacheVersion也可以是一个数组
      if(cacheName !== cacheVersion){
        console.log('service worker: clear cache' + cacheName);
        return caches.delete(cacheName);
      }
    }));
  }));
});
```

### 监听

- 这里通过监听fetch事件来代理响应，进而实现自定义前端资源缓存;
- 在event.respondWith()中我们传入来自caches.match()的一个promise,此方法拦截请求并从服务工作线程所创建的任何缓存中查找缓存结果，如若发现匹配的响应则返回缓存的值，否则，将会调用fetch以代理发出网络请求，并将从网络中检索的数据作为结果返回;
- 如果希望连续性缓存新的请求，则注意注释的代码部分，其通过cache.put来将请求的响应添加到缓存来实现;
- 在fetch请求中添加对then()的回调，获得响应后执行检查，并clone响应，注意这样处理的原因是该响应是stream,主体只能使用一次，我们需要返回能被浏览器使用的响应，还要传递到缓存以供使用，因此需要克隆一份副本;

```js
// 拦截请求并响应
self.addEventListener('fetch', function (event) {
  console.log('service worker: run into fetch');
  event.respondWith(caches.match(event.request).then(function (response) {
    // 发现匹配的响应缓存
    if(response){
      console.log('service worker 匹配并读取缓存：' + event.request.url);
      return response;
    }
    console.log('没有匹配上：' + event.request.url);
    return fetch(event.request);
    /*var fetchRequest = event.request.clone();
    return fetch(fetchRequest).then(function(response){
        if(!response || response.status !== 200 || response.type !== 'basic'){
            return response;
        }
        var responseToCache = response.clone();
        caches.open(cacheVersion).then(function (cache) {
            console.log(cache);
            cache.put(fetchRequest, responseToCache);
        });
        return response;
    });*/
  }));
});
```

![life-cycle.png](life-cycle.png)

![life-cycle-detail.png](life-cycle-detail.png)

## 五、项目如何快速接入Service Worker

方案一： workbox方案（我在项目中采用这种方式）
方案二： webpack插件

- 在接入前有两个问题摆在我们面前，service worker可以帮助我们解决资源缓存问题，有缓存就必须要有更新的机制，service-worker.js本身也会被浏览器缓存，后续产品迭代过程中如何解决该文件自身的更新问题，否则其他资源的缓存更新也就无从谈起（旧的服务工作线程将一直控制页面），无可厚非每次构建部署时service-worker.js需要携带版本号（例如?v=201811201146）,当然也可以在服务器运维层控制该文件的cache-control: no-cache从而规避浏览器缓存问题，但这样太麻烦；
- 我们是在业务代码中通过register的方式引入service-worker.js, 那问题就变为如何在注册服务工作线程的位置引入版本号呢，我们可以通过 sw-register-webpack-plugin 来解决该问题，其思路是将服务工作线程的注册放在一个单独的文件中（sw-register.js），然后自动在页面入口（例如index.html）写入一段JS脚本来动态加载sw-register.js文件，这里sw-register.js的加载路径是带有实时时间戳的，而生成的sw-register.js文件内容中注册service-worker.js的位置自动携带构建版本号参数（默认是当前构建时间），该插件配置如下(基于webpack构建的项目)：

```js
let SwRegisterWebpackPlugin = require('sw-register-webpack-plugin')
...
plugins: [
  new SwRegisterWebpackPlugin({
    filePath: path.resolve(__dirname, '../src/sw-register.js')
  })
]
```

构建后html新增部分如图:

![sw-register-webpack-plugin](sw-register-webpack-plugin.png)

构建后生成的sw-register.js文件变化如图：

![result.png](result.png)

这样处理后，sw-register.js文件就不会被浏览器缓存，也即每次刷新会多一次sw-register.js的文件请求，由于它只是用来做注册的工作，体量不会太大，可以接受，关键是前端可以自行控制;

已缓存资源文件如何更新呢？上述插件只是解决了service-worker.js文件本身的更新的问题（保证每次构建部署后会新启一个服务工作线程），但对于service-worker.js文件中定义的cacheFiles而言，当我们修改了已缓存文件后如何来更新缓存呢，我的项目是基于vue.js + webpack，打包后的JS文件是[name].[hash].[ext]格式，从前面的介绍可知资源的缓存也是基于url（作为key）来的,不可能每次构建后都手动去调整service-worker.js文件内容中cacheFiles的路径值吧，应该是将构建后的文件名（包括路径）直接放到service-worker.js内容中，看到这里你应该想到了有webpack插件已经帮我们做好了，那就是 sw-precache-webpack-plugin ,该插件会自动在dist目录下生成service-worker.js文件，供给service worker运行，也就是说service-worker.js文件本身不需要我们手动添加了，但问题是我们如何自定义需要缓存的文件呢，该插件的配置参数会告诉你

```js
// 生成service-worker.js和配置缓存清单
new SwPrecacheWebpackPlugin({
  cacheId: 'attendance-mobile-cache',
  filename: 'service-worker.js',
  minify: true,
  dontCacheBustUrlsMatching: false,
  staticFileGlobs: [
    'dist/static/js/manifest.**.*',
    'dist/static/js/vendor.**.*',
    'dist/static/js/app.**.*'
  ],
  stripPrefix: 'dist/'
})
```

由上可知，我们能够通过正则来匹配需要缓存的文件。

## 六、如何调试Service Worker

- 通过上述两个插件，我们的service-worker接入工作基本完成，那接下来就是验证服务工作线程运行是否ok,通过chrome devTools（Application项）我们可以很方面的查看当前服务工作线程的运行情况和已缓存了哪些文件，具体如何查看这里不再介绍;

- 当首次运行 service worker 时我们会发现要缓存的文件还是走正常的网络请求，cache storage 下也看不到我们的缓存项，因为服务工程线程也存在“二次生效”的机制（即使需要缓存的资源延迟加载），具体如下图所示：

![首次请求](首次请求.png)

![首次请求后台](首次请求后台.png)

通过刷新访问我们可以看到，service worker 缓存文件已经生效，在network面板下自定义的缓存文件size项都显示为“from ServiceWorker”, 耗时也明显很低。在cache storage下面也可以看到已经缓存的文件列表，具体如下图所示：

![刷新](刷新.png)

![刷新后台](刷新后台.png)

- 接下来我们更新service-worker.js文件来看下新服务工作线程如何工作,正如前面所讲新服务工作线程将会启动安装，但由于旧服务工作线程控制着页面，所以新服务工作线程将进入waiting状态，当当前打开的页面关闭时，旧服务工作线程将会被终止，新服务工作线程会得的控制权并触发activate事件，在开发过程中我们需要通过Chrome Devtools的skipWaiting或者勾选Updated on reload来强制激活新服务工作线程，具体如下图所示：

![更新](更新.png)

- 在开发过程中我们可以通过上述来了解新服务工作线程的更新流程，但在实际项目中我们可以通过self.skipWaiting()跳过等待过程安装后直接激活，一般我们在install事件中调用,具体可参见sw-precache-webpack-plugin生成的service-worker源代码。这会导致新服务工作线程将当前活动的工作线程逐出，skipWaiting()意味着新服务工作线程可能会控制使用较旧工作线程加载的页面，也就是页面获取的部分数据由旧工作线程处理，而新服务工作线程处理后来获取的数据，如果有问题就不要使用skipWaiting();

- 手动清理service worker缓存后刷新页面，在 Network 面板中，我们会看到本应缓存文件的一组初始请求。之后是前面带有齿轮图标的第二轮请求，这些请求似乎要获取相同的资源，“齿轮”图标代表这些请求来自服务工作线程，如果不unregsiter该服务工作线程，我们会发现即使多次刷新页面，Network 面板依然如此，其实也就是说资源没有再次缓存（因为服务工作线程已经安装且控制当前页面，刷新操作不会重新触发install事件，也就不会再次添加资源到缓存，除非unregister或者更新service-worker.js文件），具体如下图所示：

![delete](delete.png)

![network](network.png)

## 七、Service Worker的异常回滚（注销）

某些场景下如果service worker使用出现异常，比如不同页面间 service worker 控制的scope存在“重叠污染”的问题，那么我们就需要紧急回滚（撤销）当前 service worker,在开发环境很好解决，我们依然可以通过Chrome Devtools来进行unregister, 那么在线上环境已经有服务工作线程在运行的情况下呢，我们需要在新上线版本的service worker注册前将被污染或者异常的service worker注销掉，具体代码如下：

```js
if (navigator.serviceWorker) {
  navigator.serviceWorker.getRegistrations().then(function(registrations) {
    for (var item of registrations) {
      if (item.scope === 'http://localhost/attendance-mobile/dist/') {
        item.unregister();
      }
    }
    // 注销掉污染 Service Worker 之后再重新注册...
  });
}
```

七、代码

1）通过service worker直接返回 Network-only

```js
self.addEventListener('fetch', event => {
  event.respondWith(fetch(event.request));
  );
});
```

2）stale-while-revalite

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.open('my-cache').then(cache => {
      return caches.match(event.request).then(cacheResponse => {
        const fetchPromise = fetch(event.request).then(netWorkResponse => {
          cache.put(event.request, netWorkResponse.clone());
          return netWorkResponse;
        })
        return cacheResponse || fetchPromise;
      })
    });
  );
});

// workbox实现
workbox.strategies.staleWhileRevalidate({
  cacheName: 'prefix' + ':js',
  plugins: [
    new workbox.expiration.Plugin({
      maxEntries: 20, // 限制缓存条目数
      maxAgeSeconds: 7 * 24 * 60 * 60  // 7天后过期
    })
  ]
});
```

3）cache-first

```js
// workbox实现 service worker可以告诉web app这是一个导航请求，从fetch event事件去获取信息，service worker 知道这个url被请求过，使用先前已经预缓存的通用应用程序，并且最终从缓存中返回到Web应用程序，完全避免网络请求。

// 对于不同路径（单页应用）需要返回同样的html，配置实现

// 先检查cache,如果可用，则直接返回，否则去请求网络。

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response) => {
      return response || fetch(event.request);
    })
  );
  );
});

workbox.router.registerNavigationRoute({
  blackList: [
    new RegExp('/about'),
    new RegExp('/email/(remove|add)')
  ]
});
```

3）network-first

```js
// workbox实现 service worker可以告诉web app这是一个导航请求，从fetch event事件去获取信息，service worker 知道这个url被请求过，使用先前已经预缓存的通用应用程序，并且最终从缓存中返回到Web应用程序，完全避免网络请求。

// 对于不同路径（单页应用）需要返回同样的html，配置实现

// 先检查cache,如果可用，则直接返回，否则去请求网络。

self.addEventListener('fetch', event => {
  event.respondWith(
    fetch(event.request).cache(() => {
      return caches.match(event.request);
    })
  );
  );
});
```

参考：

https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers
https://jakearchibald.com/2014/offline-cookbook/#cache-then-network
https://developers.google.com/web/fundamentals/codelabs/your-first-pwapp/?hl=zh-cn
https://blog.caowencheng.cn/2018/11/20/service-worker/
http://taobaofed.org/blog/2018/08/08/workbox3/
https://codelabs.developers.google.com/codelabs/workbox-lab-cn/index.html?index=..%2F..gddchina#0
https://zoumiaojiang.com/article/amazing-workbox-3/
https://huangxuan.me/2017/07/12/upgrading-eleme-to-pwa/
https://www.youtube.com/watch?v=q4k-GOmI8Tg
