---
layout: post
title: "CSS三列布局8种实现方法"
date: 2015-09-26 
description: "CSS三列布局使用多种方法实现"
tag: CSS
---


CSS三列布局, 是我们在做PC页面时比较常见的布局需求, 指的是左侧右侧宽度固定中间内容宽度自适应. 我们今天来梳理下实现3列布局的N种方法

如下效果:
![三列布局效果](/images/css-layout.png)

> 三列布局是一个需求也是一个面试题, 你应该设法用最多的方法来实现它, 并能说出每种方案的优缺点和需要注意的地方


#### 规约:
* 需求: 两侧宽度假设都为200px, 中间宽度自适应容器宽度(方法1除外)
* 两侧的都是用div,且宽度都设置为200px, 这个适用于下面所有的两侧div
* 高度无具体要求
* 如果有设置margin, 则margin的宽度都为200px
* 请使用最新版本chrome/firefox测试

### 方法1. 双飞翼布局
* 解决方案: 左中右浮动, 内容外层嵌套容器
* 注意事项: 中间的内容区域在元素外嵌套100%宽度div且放在左右两侧元素的前面, 
* 实现原理: 100%宽度浮动元素, 把左右侧留白出来
* 参考代码:

```
<!DOCTYPE html>
<head>
  <meta charset="utf-8">
  <title>三列布局8-双飞翼布局</title>
  <style type="text/css">
    * {
      margin: 0;
      padding: 0;
    }
    body{ 
      min-width: 700px;
    }
    .left,
    .main,
    .right{
      float: left;
      min-height: 130px;
    }
    .left{
      margin-left: -100%;
      width: 200px;
      background: red;
    }
    .right{
      margin-left: -220px;
      width: 220px;
      background: blue;
    }
    .main{
      width: 100%;
    }
    .main-inner{
      margin-left: 200px;
      margin-right: 220px;
      min-height: 130px;
      background: green;
      word-break: break-all;
    }
  </style>
</head>
<body>

<div class="main">
  <div class="main-inner">
    <h4>中间内容</h4>
    <p>HHHHHHHHHHHHHHHHHHHHHH
      hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
      HHHHHHHHHHHHHHHHHHHHHH
      hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
    </p>
  </div>
</div>
<div class="left">
  <h4>左边的</h4>
  <p>oooooooooooooo
    00000000000000000
    ooooooooooooooo
    ooooooooooooooo
    000000000000000</p>
</div>

<div class="right">
  <h4>右边的</h4>
  <p>BBBBBBBBBBBBBB
    BBBBBBBBBBBBBBBBBB
    88888888888888888888</p>
</div>

</body>
</html>
```


### 方法2. 两侧浮动 + 中间块元素自适应
* 解决方案: 左侧div左浮动, 右侧右浮动, 中间div设置左右margin自适应
* 注意事项: 中间的div由于块元素的特性, 默认会挡住后面的浮动元素而换行, 所以把中间的元素的代码位置放到右浮动代码之后
* 实现原理: 块元素宽度自适应, 设置margin后宽度改变, float之后脱离普通流
* 参考代码:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>三列布局实现方法1-两侧浮动</title>
  <style>
    html, body {
      height: 100%;
    }
    * {
      margin:0;
      padding:0;
    }
    .page-wrapper {
      height: 100%;
    }
    .left {
      float: left;
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .right {
      float: right;
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .main {
      margin: 0 200px;
      background: #adfcdd;
      height:200px;
    }
  </style>
</head>
<body>
  <div class="page-wrapper">
    <div class="left">哈哈, 左边的Left</div>
    <div class="right">右边的, right</div>
    <div class="main">中间的</div>
  </div>
</body>
</html>
```

### 方法3. 两侧绝对定位 + 中间块元素自适应
* 解决方案: 左侧div绝对定位, 右侧绝对定位, 中间div设置左右margin自适应
* 注意事项: 左中右元素顺序不用变换
* 实现原理: 块元素宽度自适应, 设置margin后宽度改变, 绝对定位之后脱离普通流
* 参考代码: 

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>三列布局实现方法2-两侧绝对定位</title>
  <style>
    html, body {
      height: 100%;
    }
    * {
      margin:0;
      padding:0;
    }
    .page-wrapper {
      position: relative;
      height: 100%;
    }
    .left {
      position: absolute;
      top: 0;
      left: 0;
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .right {
      position: absolute;
      top: 0;
      right: 0;
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .main {
      margin: 0 200px;
      background: #adfcdd;
      height:200px;
    }
  </style>
</head>
<body>
  <div class="page-wrapper">
    <div class="left">哈哈, 左边的Left</div>
    <div class="right">右边的, right</div>
    <div class="main">中间的</div>
  </div>
</body>
</html>
```

### 方法4. 两侧绝对定位 + 中间块元素也绝对定位, 拉伸宽度
* 解决方案: 左侧div绝对定位, 右侧绝对定位, 中间div设置 top right bottom left 四个方向的值
* 注意事项: 左中右三个元素都需要参考合适的父级定位, 中间元素上下左右距离都要设置, 且左右距离为200px
* 实现原理: 绝对定位之后脱离普通流, 同时设置对立方向的距离会拉伸这个方向的宽/高, 此特性IE8+支持
* 参考代码: 

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>三列布局实现方法3-两侧绝对定位-中间也绝对定位</title>
  <style>
    html, body {
      height: 100%;
    }
    * {
      margin:0;
      padding:0;
    }
    .page-wrapper {
      position: relative;
      height: 100%;
    }
    .left {
      position: absolute;
      top: 0;
      left: 0;
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .right {
      position: absolute;
      top: 0;
      right: 0;
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .main {
      position: absolute;
      top: 0;
      right: 200px;
      bottom: 0;
      left: 200px;
      background: #adfcdd;
      height:200px;
    }
  </style>
</head>
<body>
  <div class="page-wrapper">
    <div class="left">哈哈, 左边的Left</div>
    <div class="main">中间的</div>
    <div class="right">右边的, right</div>
  </div>
</body>
</html> 
```

### 方法5. inline-block结合 calc 计算函数
* 解决方案: 所有容器设置为 inline-block类型, 中间的宽度是 `calc` 动态计算的
* 注意事项: 由于是inline-block就需要注意 空白间隙问题, 垂直对齐问题, `calc` 兼容 IE9+, 要注意相对单位的参考者
* 实现原理: inline-block使得div横向排列, `calc` 可以实现动态的宽度值
* 参考代码: 

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>三列布局实现方法4-inline-block + calc</title>
  <style>
    html, body {
      height: 100%;
    }
    * {
      margin:0;
      padding:0;
    }
    .page-wrapper {
      position: relative;
      height: 100%;
      font-size: 0;
    }
    .page-wrapper > div {
      display: inline-block;
      font-size: 20px;
    }
    .left {
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .right {
      width: 200px;
      height: 200px;
      background: #f00;
    }
    .main {
      width: calc(100% - 400px);
      background: #adfcdd;
    }
  </style>
</head>
<body>
  <div class="page-wrapper">
    <div class="left">哈哈, 左边的Left</div>
    <div class="main">中间的</div>
    <div class="right">右边的, right</div>
  </div>
</body>
</html>

```

### 方法6. flex弹性布局
* 解决方案: 容器设置display: flex, 子元素中间的设置 flex: 1拉伸占据剩余空间
* 注意事项: flex有多个版本, 注意兼容性, 使用一些工具加上前缀 , IE10+支持; 默认方向为 row
* 实现原理: flex可以伸缩长度
* 参考代码: 

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>三列布局实现方法5- flex弹性布局</title>
  <style>
    html, body {
      height: 100%;
    }
    * {
      margin:0;
      padding:0;
    }
    .page-wrapper {
      display: flex;
      height: 100%;
    }
    .page-wrapper > div {

    }
    .left {
      width: 200px;
      background: #f00;
    }
    .right {
      width: 200px;
      background: #f00;
    }
    .main {
      flex: 1;
      background: #adfcdd;
    }
  </style>
</head>
<body>
  <div class="page-wrapper">
    <div class="left">哈哈, 左边的Left</div>
    <div class="main">中间的</div>
    <div class="right">右边的, right</div>
  </div>
</body>
</html>
```

### 方法7. 表格布局
* 解决方案: 容器设置display: table, 子容器设置 display: table-cell
* 注意事项: 需要有两层容器嵌套, 如果需要垂直居中可以使用 vertical-align ,最外层table容器默认会包裹内容, 宽度需要设置100%
* 实现原理: 显示为表格模式, 子容器类似于 td, 宽度可以自适应
* 参考代码: 

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>三列布局实现方法6-表格布局</title>
  <style>
    html, body {
      height: 100%;
    }
    * {
      margin:0;
      padding:0;
    }
    .page-wrapper {
      display: table;
      width: 100%;
      height: 100%;
    }
    .page-wrapper > div {
      display: table-cell;
    }
    .left {
      width: 200px;
      background: #f00;
    }
    .right {
      width: 200px;
      background: #f00;
    }
    .main {
      background: #adfcdd;
    }
  </style>
</head>
<body>
  <div class="page-wrapper">
    <div class="left">哈哈, 左边的Left</div>
    <div class="main">中间的</div>
    <div class="right">右边的, right</div>
  </div>
</body>
</html>
```

### 方法8. grid网格布局
* 解决方案: 容器设置display: grid, 同时设置 grid-template-columns/rows
* 注意事项: 浏览器支持力度不够, 需要兼容
* 实现原理: 为实现栅格化布局增加的CSS功能
* 参考代码: 

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>三列布局实现方法7-grid网格布局</title>
  <style>
    html, body {
      height: 100%;
    }
    * {
      margin:0;
      padding:0;
    }
    .page-wrapper {
      display: grid;
      grid-template-columns: 200px auto 200px;
      grid-template-rows: auto;
      height: 100%;
    }
    .page-wrapper > div {

    }
    .left {
      /*width: 300px;*/
      background: #0f0;
    }
    .main {
      background: #00f;
    }
    .right {
      background: #0f0;
    }

  </style>
</head>
<body>
  <div class="page-wrapper">
    <div class="left">哈哈, 左边的Left</div>
    <div class="main">中间的</div>
    <div class="right">右边的, right</div>
  </div>
</body>
</html>
```

### 其他方法
可以使用 calc 函数和 float/定位组合的形式, 使用更多方法实现它


### 总结
小小的布局题 蕴藏着大大滴智慧呢, 你必须对CSS的各种布局都很熟悉才能快速的写出多种解决方案

( 完 )

<br>

转载请注明：[blog.nodejs.tech](http://blog.nodejs.tech) 