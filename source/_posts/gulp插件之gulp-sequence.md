---
title: gulp插件之gulp-sequence
date: 2017-08-24 15:06:29
tags:
  - gulp 
  - gulp-sequence
---

### gulp之 gulp-sequence 按顺序逐个同步地运行任务

我们在使用gulp的时候，有时候需要按顺序，有先后的同步的执行gulp任务，这时候就需要gulp-sequence这个插件了

用法都在下面啦~
<!-- more -->
```javascript
var gulp = require('gulp')     //首先是必备的gulp
var gulpSequence = require('gulp-sequence')    //然后主角登场

gulp.task('a', function (cb) {   //任务a
  return //... cb()  执行代码
})

gulp.task('b', function (cb) {   //任务b
  return //... cb()  执行代码
})

gulp.task('c', function (cb) {   //任务c
  return //... cb()  执行代码
})

gulp.task('d', function (cb) {   //任务d
  return //... cb()  执行代码
})

gulp.task('e', function (cb) {   //任务e
  return //... cb()  执行代码
})

// 用法 1, 推荐
// 下面的意思是 任务'a' 和 'b' 并行运行;
// 任务'c' 在 任务'a' 和 'b' 都执行完成之后执行 ;
// 并行运行任务 'd' 和 'e' 在任务 'c' 完成之后;
//总结就是中括 '[]'  表示并行执行的任务
gulp.task('sequence-1', gulpSequence(['a', 'b'], 'c', ['d', 'e']))


// 用法 2  cb 表示回调函数，在顺序执行完任务后执行的函数
gulp.task('sequence-2', function (cb) {
  gulpSequence(['a', 'b'], 'c', ['d', 'e'], cb)
})

// 用法 3
gulp.task('sequence-3', function (cb) {
  gulpSequence(['a', 'b'], 'c', ['d', 'e'])(cb)
})

//使用 gulp.watch 监听文件变化 执行同步任务

gulp.watch('src/**/*.js', function (event) {
  gulpSequence('a', 'b')(function (err) {
    if (err) console.log(err)
  })
})
```