---
title: 水平垂直居中
date: 2018-09-10 14:50:32
categories: [CSS]
---

### 高度位置的盒子在父元素水平垂直居中的方法
##### 1. table(td text-align:center;)
- 使用表格单元格特性
- 不推荐使用
```CSS
td {width:200px;height:100px;border: 1px solid #ddd;text-align:center;}
```
<!-- more -->
##### 2. 仿table(父元素display:table;子元素display:table-cell;text-align:center;vertical-align:middle;)
- 使用表格单元格特性
- IE6/7不支持
- 不推荐使用
```CSS
.parent {display:table;list-style:none;margin:0;padding:0;}
.child {display:table-cell;width:200px;height:100px;border: 1px solid #ddd;text-align:center;vertical-align:middle;}
```
##### 3. Line-height(设置height和line-height的值相等，若为img再设置vertical-align:middle;)
- 单行文本或者图片
- IE6不支持图片居中
- 视场景使用
```CSS
.parent {overflow:hidden;list-style:none;margin:0;padding:0;}
.child {float:left;width:200px;height:100px;border: 1px solid red;text-align:center;line-height:100px;}
img {vertical-align:middle;}
```
##### 4. Button(button标签内部的元素或者文字本身就能够居中，若子元素img则vertical-align:center;)
- 单行文本或者图片
- 纯玩票性质
- 不推荐使用
##### 5. Absolute拉伸(父元素position:relative;子元素position:absolute;top:0;left:0;right:0;bottom:0;margin:auto;)
- 图片（即置换元素）
- 不支持IE6/7
- 不支持非置换元素
- 不推荐使用
```CSS
.parent {float:left;position:relative;width:200px;height:100px;margin:10px;border: 1px solid #ddd;background-color:#fff;}
img {vertical-align:middle;}
.child {position:absolute;top:0;right:0;bottom:0;left:0;margin:auto;}
```
##### 6. 伪元素(父元素text-align:center;伪类/真实元素 子元素display:inline-block;vertical-align:middle;伪类/真实元素 width:0;height:100%;content:"\0020";)
- 利用行内级元素vertical-align特性
- IE6/7需真实元素替代伪元素
- 推荐使用
```CSS
.parent {float:left;width:200px;height:100px;margin:10px;border: 1px solid red;background-color:#fff;font:0 arial;text-align:center;}
img {vertical-align:middle;}
.parent::after,
.parent .after,
.parent .child {display:inline-block;font-size:16px;vertical-align:middle;}
.parent::after,
.parent .after {width:0;height:100%;content:"\0020";}
```
##### 7. transform（父元素position:relative;子元素position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);）
- 使用宽度高度减半
- IE6/7/8不支持
- 视场景使用
```CSS
.parent {float:left;position:relative;width:200px;height:100px;margin:10px;border: 1px solid #ddd;background-color:#fff;}
img {vertical-align:middle;}
.child {position:absolute;top:50%;left:50%;transform:translate(-50%, -50%);}
```
##### 8. flex(父元素display:flex/inline-flex;justify-content:center;align-items:center;)
- IE6/7/8/9不支持
- 移动端推荐使用，PC视场景使用
```CSS
.demo {display:inline-flex;width:200px;height:100px;margin:10px;border: 1px solid #ddd;background-color:#fff;vertical-align:top;justify-content:center;align-items:center;}
img {vertical-align:middle;}
```
##### 9. 代表性的页码对齐(bfc  text-align:center;子父容器display:inline-block;)
- 页码子项需要可以定义宽高等外观
- 页码需要可以各方向对齐
```CSS
nav {text-align: center;}
ul {display: inline-block;text-align: center;}
```
