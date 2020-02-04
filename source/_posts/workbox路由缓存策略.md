---
title: workbox路由缓存策略
date: 2019-07-19 16:00:28
categories: [JavaScript]
tags:
---

下面介绍一下 workbox 默认提供的几种缓存策略 API，这些 API 可以被当成 handler 使用。

## Stale While Revalidate

这种策略的意思是当请求的路由有对应的 Cache 缓存结果就直接返回，在返回 Cache 缓存结果的同时会在后台发起网络请求拿到请求结果并更新 Cache 缓存，如果本来就没有 Cache 缓存的话，直接就发起网络请求并返回结果，这对用户来说是一种非常安全的策略，能保证用户最快速的拿到请求的结果，但是也有一定的缺点，就是还是会有网络请求占用了用户的网络带宽。可以像如下的方式使用 State While Revalidate 策略：

<!-- more -->

```js
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.staleWhileRevalidate()
);
```

## Network First

这种策略就是当请求路由是被匹配的，就采用网络优先的策略，也就是优先尝试拿到网络请求的返回结果，如果拿到网络请求的结果，就将结果返回给客户端并且写入 Cache 缓存，如果网络请求失败，那最后被缓存的 Cache 缓存结果就会被返回到客户端，这种策略一般适用于返回结果不太固定或对实时性有要求的请求，为网络请求失败进行兜底。可以像如下方式使用 Network First 策略：

```js
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.networkFirst()
);
```

## Cache First

这个策略的意思就是当匹配到请求之后直接从 Cache 缓存中取得结果，如果 Cache 缓存中没有结果，那就会发起网络请求，拿到网络请求结果并将结果更新至 Cache 缓存，并将结果返回给客户端。这种策略比较适合结果不怎么变动且对实时性要求不高的请求。可以像如下方式使用 Cache First 策略：

```js
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.cacheFirst()
);
```

## Network Only

比较直接的策略，直接强制使用正常的网络请求，并将结果返回给客户端，这种策略比较适合对实时性要求非常高的请求。可以像如下方式使用 Network Only 策略：

```js
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.networkOnly()
);
```

## Cache Only

这个策略也比较直接，直接使用 Cache 缓存的结果，并将结果返回给客户端，这种策略比较适合一上线就不会变的静态资源请求。可以像如下方式使用 Cache Only 策略：

```js
workbox.routing.registerRoute(
    match, // 匹配的路由
    workbox.strategies.networkOnly()
);
```

无论使用何种策略，你都可以通过自定义一个缓存来使用或添加插件（后面我们会介绍 workbox 插件）来定制路由的行为（以何种方式返回结果）。

```js
workbox.strategies.staleWhileRevalidate({
    // 使用用户自定义的缓存名称
    cacheName: 'my-cache-name',

    // 使用 workbox 插件
    plugins: [
        // ...
    ]
});
```

当然，这些配置通常需要在缓存请求时更安全，也就是说，需要限制缓存的时间或者确保设备上用的数据是被限制的。

自定义策略
如果以上的那些策略都不太能满足你的请求的缓存需求，那就得想想办法自己定制一个合适的策略，甚至是不同情况下返回不同的请求结果，workbox 也考虑到了这种场景（这也是为什么我会极力推荐 workbox 的原因），当然，最简单的方法是直接在 Service Worker 文件里通过最原始的 fetch 事件控制缓存策略。也可以使用 workbox 提供的另一种方式：传入一个带有对象参数的回调函数，对象中包含匹配的 url 以及请求的 fetchEvent 参数，回调函数返回的就是一个 response 对象，具体用法如下所示：

```js
workbox.routing.registerRoute(
    ({url, event}) => {
        return {
            name: 'workbox',
            type: 'guide',
        };
    },
    ({url, event, params}) => {
        // 返回的结果是：A guide on workbox
        return new Response(
            `A ${params.type} on ${params.name}`
        );
    }
);
```
