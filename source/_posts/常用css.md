---
title: 常用css
date: 2020-05-21 14:39:26
categories: 技术
tags: [前端]
---

记录下常用 css

<!--more-->

### 常用 css

```css
body,
div,
ul,
li,
h1,
h2,
h3,
h4,
h5,
input,
form,
a,
p,
textarea {
    margin: 0;
    padding: 0;
}
ol ul li {
    list-style: none;
}
a {
    text-decoration: none;
    display: block;
    color: #fff;
}
img {
    border: 0;
    display: block;
}

/* 清除浮动 */
.clearfloat {
    zoom: 1;
}
.clearfloat:after {
    display: block;
    clear: both;
    content: "";
    visibility: hidden;
    height: 0;
}

/*修改placeholder样式*/
:-moz-placeholder {
    /* Mozilla Firefox 4 to 18 */
    color: hsla(40, 70%, 100%, 0.3) !important;
    font-size: 14px;
    opacity: 1; /*火狐浏览器默认opacity小=小于1*/
}
::-moz-placeholder {
    /* Mozilla Firefox 19+ */
    color: hsla(40, 70%, 100%, 0.3) !important;
    font-size: 14px;
    opacity: 1;
}
:-ms-input-placeholder {
    color: #fff !important;
    font-size: 14px;
}
::-webkit-input-placeholder {
    color: #fff !important;
    font-size: 14px;
    opacity: 0.3;
}

/*单行省略*/
.ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

/*多行省略溢出省略号*/
p {
    position: relative;
    line-height: 1.4em;
    /* 3 times the line-height to show 3 lines */
    height: 4.2em;
    overflow: hidden;
}
p::after {
    content: "...";
    font-weight: bold;
    position: absolute;
    bottom: 0;
    right: 0;
    padding: 0 20px 1px 45px;
    background: url(http://css88.b0.upaiyun.com/css88/2014/09/ellipsis_bg.png) repeat-y;
}
/*注意：
height高度真好是line-height的3倍；
结束的省略好用了半透明的png做了减淡的效果，或者设置背景颜色；
IE6-7不显示content内容，所以要兼容IE6-7可以是在内容中加入一个标签，比如用<span class="line-clamp">...</span>去模拟；
要支持IE8，需要将::after替换成:after；*/

/*webkit内核多行省略*/
.ellipsis-webkit {
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2; /* 限制在一个块元素显示的文本的行数 */
    -webkit-box-orient: vertical;
}

/*多行文字水平垂直居中（一般是尾部）*/
footer {
    display: table;
    width: 100%;
    height: 80px;
    text-align: center;
}
footer > div {
    display: table-cell;
    vertical-align: middle;
}
footer > div > p {
}

/*绝对定位的div相对父类居中*/
div｛position: absolute;
top: 50%;
left: 50%;
transform: translate(-50%, -50%);
｝

/*去掉谷歌浏览器报错Unable to preventDefault inside passive event listener due to target being treated as passive. */
* {
    touch-action: pan-y;
}

/*实现0.5px的线*/
.line1 {
    transform: scaleY(0.5);
    transform-origin: 50% 100%;
    width: 200px;
    height: 1px;
    background: #000;
}
.line2 {
    width: 200px;
    height: 1px;
    background: linear-gradient(0deg, #fff, #000);
}
.line3 {
    width: 200px;
    height: 1px;
    background: none;
    box-shadow: 0 0.5px 0 #000;
}

/*下面每两个一组是相等的*/
/*当flex取值为一个非负数字，则该数字为flex-grow 值，flex-shrink为1，flex-basis为0% ，使子元素平分空间*/
.item {
    flex: 1;
}
.item {
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: 0%;
}

/*元素会根据自身宽高来设置尺寸。它是完全非弹性的：既不会缩短，也不会伸长来适应 flex 容器。*/
.item {
    flex: none;
}
.item {
    flex-grow: 0;
    flex-shrink: 0;
    flex-basis: auto;
}

/*元素会根据自身的宽度与高度来确定尺寸，但是会伸长并吸收 flex 容器中额外的自由空间，也会缩短自身来适应 flex 容器。*/
.item {
    flex: auto;
}
.item {
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: auto;
}

/*针对全面屏的底部安全区域做适配*/
/*constant()和env()位置不能换*/
/*env()和constant()函数有个必要的使用前提，H5网页设置viewport-fit=cover的时候才生效，小程序里的viewport-fit默认是cover。*/
@supports (bottom: constant(safe-area-inset-bottom)) or (bottom: env(safe-area-inset-bottom)) {
    .body {
        padding-bottom: constant(safe-area-inset-bottom); /* 兼容 iOS < 11.2 */
        padding-bottom: env(safe-area-inset-bottom); /* 兼容 iOS >= 11.2 */
       /* padding-bottom: calc(60px(假设值) + constant(safe-area-inset-bottom));
        padding-bottom: calc(60px(假设值) + env(safe-area-inset-bottom));*/
    }
}

/*三角形*/
.triangle-up {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-bottom: 100px solid red;
}

.triangle-down {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-top: 100px solid red;
}

.triangle-left {
    width: 0;
    height: 0;
    border-top: 50px solid transparent;
    border-right: 100px solid red;
    border-bottom: 50px solid transparent;
}

.triangle-right {
    width: 0;
    height: 0;
    border-top: 50px solid transparent;
    border-left: 100px solid red;
    border-bottom: 50px solid transparent;
}

.triangle-topleft {
    width: 0;
    height: 0;
    border-top: 100px solid red;
    border-right: 100px solid transparent;
}

.triangle-topright {
    width: 0;
    height: 0;
    border-top: 100px solid red;
    border-left: 100px solid transparent;
}

.triangle-bottomleft {
    width: 0;
    height: 0;
    border-bottom: 100px solid red;
    border-right: 100px solid transparent;
}

/*钝角箭头*/
.right-arrow {
    width: 44px;
    height: 44px;
    border-top: 1px solid #ccc;
    border-right: 1px solid #ccc;
    transform: rotate(45deg) skew(30deg, 30deg);
}

/*纯css实现loading*/
.loading {
  width: 20px;
  height: 20px;
  border: 2px solid #7d329c;
  border-bottom: 2px solid transparent;
  border-radius: 50%;
  animation: load 1.5s linear infinite;
}

@keyframes load {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
```

### 兼容性问题

-   IE6 中，第一个浮动到父元素边上的元素，如果含有该方向的 margin 值，那么 margin 会以双倍显示。即：浮动元素的左边距在 IE6 上为所设定的左边距的两倍。这个问题只会发生在浮动行的第一个浮动元素上。准确的说：应该是每一行的第一个元素都会受此影响。
    为了解决该问题，需要给浮动元素添加属性 display: inline，即可解决。

-   iOS11 中 position:fixed 弹出框中的 input 光标错位的问题  
    在弹框出现的时候给 body 添加 fixed:

```css
body {
    position: fixed;
    width: 100%;
}
```

    当弹框消失的时候
    ```css
    $("body").css("position","relative")
    ```
