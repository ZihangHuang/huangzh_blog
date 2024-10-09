---
title: 常用js
date: 2021-08-02 16:59:54
categories: 技术
tags: [前端]
---

记录下常用 js

<!--more-->

### 常用 js

复制到剪贴板

```js
function copyToClipboard(text, cb) {
    if (navigator && navigator.clipboard) {
        navigator.clipboard
            .writeText(text)
            .then(() => {
                cb && cb();
            })
            .catch(err => {
                console.error("Could not copy text: ", err);
            });
    } else {
        const dummyElement = document.createElement("span");
        dummyElement.style.whiteSpace = "pre";
        dummyElement.textContent = text;
        document.body.appendChild(dummyElement);

        const selection = window.getSelection();
        selection.removeAllRanges();
        const range = document.createRange();
        range.selectNode(dummyElement);
        selection.addRange(range);

        const isSuccess = document.execCommand("copy");

        if (isSuccess) {
            cb && cb();
        }

        selection.removeAllRanges();
        document.body.removeChild(dummyElement);
    }
}
```

```js
//if value > max, value = max, if value < min, value = min, otherwise value = value
function validValue(value, max, min) {
    return Math.min(Math.max(value, min), max);
}

// getClass
function getClassName(parent, className) {
    var obj = parent.getElementsByTagName("*"); //获取 父级的所有子集
    var pinS = []; //创建一个数组 用于收集子元素
    for (var i = 0; i < obj.length; i++) {
        //遍历子元素、判断类别、压入数组
        if (obj[i].className == className) {
            pinS.push(obj[i]);
        }
    }
    return pinS;
}

// 将对象拼接成 key1=val1&key2=val2&key3=val3 的字符串形式
function objParams(obj) {
    var result = "";
    var item;
    for (item in obj) {
        result += "&" + item + "=" + encodeURIComponent(obj[item]);
    }

    if (result) {
        result = result.slice(1);
    }

    return result;
}

//把多维数组变扁平
var givenArr = [[1, 2, 2], [3, 4, 5, 5], [6, 7, 8, 9, [11, 12, [12, 13, [14, [99]]]]], 10];
function flatten(arr) {
    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr);
    }
    return arr;
}

flatten(givenArr);

// 数组去重合并
function combine(){
    let arr = [].concat.apply([], arguments);  //没有去重复的新数组
    return Array.from(new Set(arr));
}

var m = [1, 2, 2], n = [2,3,3];
console.log(combine(m,n));

/**
 * @param {Number} time 时间戳
 * @param {String} format 时间格式
 */
export function getFormatDate(
  time = new Date().getTime(),
  format = 'YY-MM-DD'
) {
  const date = new Date(time)

  const year = date.getFullYear(),
    month = date.getMonth() + 1,
    day = date.getDate(),
    hour = date.getHours(),
    minute = date.getMinutes(),
    second = date.getSeconds()

  const preArr = Array.apply(null, Array(10)).map(function (elem, index) {
    return '0' + index
  })

  const newTime = format
    .replace(/YY/g, year)
    .replace(/MM/g, preArr[month] || month)
    .replace(/DD/g, preArr[day] || day)
    .replace(/hh/g, preArr[hour] || hour)
    .replace(/mm/g, preArr[minute] || minute)
    .replace(/ss/g, preArr[second] || second)

  return newTime
}


/**
 * 时间段格式化（日时分秒）
 * @param remainTime 剩余时间
 */
export function remainTimeFormatDHMS(remainTime: number) {
    const d = Math.floor(remainTime / 864e5);
    const h = Math.floor((remainTime % 864e5) / 36e5);
    const m = Math.floor((remainTime % 36e5) / 6e4);
    const s = Math.floor(((remainTime % 36e5) % 6e4) / 1e3);
    return { d, h, m, s };
}

// 取小数后两位，不四舍五入
function getDecimal(n) {
    return Math.floor(n * 100) / 100;
}

/**
 * 把数据保存为csv文件
 */
function download(filename, text) {
  var element = document.createElement("a");
  element.setAttribute("href", "data:text/csv;charset=utf-8," + encodeURIComponent(text));
  element.setAttribute("download", filename);

  element.style.display = "none";
  document.body.appendChild(element);

  element.click();

  document.body.removeChild(element);
}

/**
 * 把canvas转化成图片图片下载下来
 */
function canvasDownLoad(id) {
  var canvas = document.getElementById(id) as HTMLCanvasElement;
  var image = new Image();
  image.src = canvas.toDataURL("image/png");

  var a = document.createElement("a");
  var event = new MouseEvent("click");
  a.download = "下载图片";
  a.href = image.src;
  a.dispatchEvent(event);
}

function getSearchParam(key, url) {
  return (new RegExp(`[?&]${key}=([^&]+)`).exec(url) || [])[1];
}

/**
 * 获取元素到页面顶部的距离
 */
function getElementTop(element: HTMLElement): number {
  let actualTop = element.offsetTop;
  let current: any = element.offsetParent;
  while (current !== null) {
    actualTop += current.offsetTop;
    current = current.offsetParent;
  }
  return actualTop;
}
```

### 常用工具

HSTS 是 HTTP 严格传输安全（HTTP Strict Transport Security） 的缩写。 这是一种网站用来声明他们只能使用安全连接（HTTPS）访问的方法。 如果一个网站声明了 HSTS 策略，浏览器必须拒绝所有的 HTTP 连接并阻止用户接受不安全的 SSL 证书。 目前大多数主流浏览器都支持 HSTS。

清除某个域名的 HSTS：

-   谷歌： chrome://net-internals/#hsts，在 Delete domain security policies 下的文本框中输入要删除的域名然后点击删除。
-   Safari：偏好设置->隐私->管理网站数据->删除对应域名
