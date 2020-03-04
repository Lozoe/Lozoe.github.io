---
title: 错误监控
date: 2017-09-10 16:04:04
categories: [JavaScript]
---

### 前端错误分类

- 即时运行错误： 代码错误
- 资源加载错误

### 错误的捕获方式

即时运行错误捕获：

```js
1) try...cache
try {

} catch (error) {

}
2) window.onerror
window.addEventListener('error', funtion() {}, false)
```

<!-- more -->
资源加载错误捕获：

1)object.onerror(script错误不冒泡到window)
2)performance.getEnties() 获取所有已经加载资源加载时长

```js
for(let [key, value] of performance.getEntries().entries()) {
    console.log(value)
}
document.getElementsByTagName('img')  两者差集
```

3)Error时间捕获

```js
window.addEventListener('error', function (e) {
  console.log('捕获', e);
}, true); // 捕获阶段
<script src="//xxx.com/test.js" charset="utf-8"></script>  

```

#### ps: 跨域js错误如何处理

跨域js只可以捕获到Script Error信息，可以通过以下形式来捕获跨域错误。

1. 客户端在`script`标签中添加crossorigin
2. 服务端在资源响应头添加 Access-Control-Allow-Origin: *

### 上报错误的基本原理

1. 利用Ajax通信上报
2. 利用Image对象上报(推荐)

```js
(new Image()).src = 'http://xxx.com/aaa?r=www';
```

使用decorator实现埋点上报的抽取
```js
{
  /**
   * 埋点系统抽离，形成可复用模块
   */
  class Report {
    click (options) {
      this.send(`http://xxx.com?${this.serialize(options)}`)
      console.info(`ad clicked type: ${JSON.stringify(options)}`)
    }

    send (url) {
      let img = new Image;
      img.onload = img.onerror = function() {
          img = img.onload = img.onerror = null
      }
      img.src = url
    }

    serialize (data) {
      let temp = []
      for(let [key, value] of this.entries(data)) {
        let type = this.getType(value)
        if(type === 'function') {
          if(value.toString().match(/native code/g)) continue
          value = value()
        } else if (type === 'object'){ 
          value = JSON.stringify(value);
        }
        try {
          temp.push(key + "=" + encodeURIComponent(decodeURIComponent(value === undefined ? '' : value)))
        } catch(e) {
          temp.push(key + "=" + encodeURIComponent(value === undefined ? "" : ""))
        }
        console.log(key + ': ' + data[key]);
      }
      return temp.join("&")
    }

    * entries(obj) {
      for(let key of Object.keys(obj)) {
          yield [key, obj[key]];
      }
    }

    getType (obj) {
      let typeName;
      return ((typeName = typeof(obj)) == "object" ? obj == null && "null" || Object.prototype.toString.call(obj).slice(8, -1) : typeName).toLowerCase()
    }
  }
  let report = new Report

  let log = (type) => {
    return function (target, name, descriptor) {
      let srcMethod = descriptor.value
      descriptor.value = (...args) => {
        srcMethod.apply(target, args)
        report.click({type})
      }
    }

  }

  class AD {
    @log('show')
    show (msg) {
      console.info(`ad showed msg: ${msg}`)
    }

    @log('click')
    click (type) {
      console.info(`ad clicked type: ${type}`)
    }
  }

  let ad = new AD()
  ad.show('advertisement')
  ad.click('click')
}
```
