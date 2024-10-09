---
title: this指向
date: 2018-06-20 18:15:50
categories: 技术
tags: [前端]
---

学习 js 的过程中，this 的指向是绕不过去的坎，因为有时候会让人感到有点乱，比如说：

```javascript
var name = 1;
var people = {
  name: 2,
  getName: function(){
    return this.name;
  },
  son: {
    name: 3,
    getName: function(){
      console.log(this);
      return this.name
    }
  }
}
var a = people.getName;
var b = people.son;
console.log(a())
console.log(people.getName());
```

<!--more-->

会输出什么呢？

```javascript
1
2
```

因为 a 的 this 为 window，所以为 window.name，而 b 的 this 是 people。

继续看

```javascript
console.log(b.getName()); // 3      // b的this为son，son.name = 3
console.log(people.son.getName()); // 3   //getName为son的直接调用，所以也为son.name
```

在实例化构造函数时，如果返回值是一个对象，那么 this 指向的就是那个返回的对象，如果返回值不是一个对象那么 this 还是指向函数的实例，例子如下：

```javascript
function GetName(){
    this.name = 1;
    return {}; // 返回对象
}
var a = new getName;
console.log(a.name); //undefined

function GetName(){
    this.name = 1;
    return 2; // 返回非对象
}
var d = new getName;
console.log(d.name); //1
```

回调函数也会导致 this 丢失：

```javascript
window.name = 'window jack'
var obj = {
    name: 'obj jack',
    show: function(){
    console.log(this.name)
    }
}

obj.show() //obj jack

function fun(fn){
    fn()
}
fun(obj.show) //window jack
```
