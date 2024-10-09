---
title: javascript技巧1
date: 2019-04-26 14:35:44
categories: 技术
tags: 前端
---

最近发现 js 真的有很多奇淫技巧。

### 给函数添加属性

因为 js 一切皆对象，函数其实也是对象，是 Function 的实例。所以也可以给它添加属性。

```javascript
function people() {}
people.age = 30;
people.say = function() {
  console.log("say");
};
console.log(people.age); //30
people.say(); //say
```

<!--more-->

每个函数其实还有几个自带的属性：
name: 函数名称
length: 函数参数个数
prototype: 函数原型
利用这个特性，可以写出一些不太常见的代码：

```javascript
function Person() {
  function man() {
    man.say();
  }

  Object.setPrototypeOf(man, proto); //设置man的原型
  return man;
}

function proto() {}

proto.say = function() {
  console.log("say");
};

proto.eat = function() {
  console.log("eat");
};

var chinese = new Person();
chinese(); //say
chinese.eat(); //eat
```

构造函数一般都返回自身实例，但这里 Person 构造函数返回一个函数 man，此函数 man 设置了 proto 为原型，所以 man 可以调用 proto 的方法，因为函数也是对象的一种，所以实例化后的 chinese 既可以作为函数执行，也可以调用自己或原型链的方法。

### 把对象转化为函数

有这么一个对象：

```javascript
var app = {};
app.nickname = "jack";
//app.name = 'jack'
app.handle = function() {
  console.log("hello " + this.nickname);
};
```

假设要把这个对象传给一个函数，这个函数只能接受一个函数作为对象：

```javascript
function create(fn) {
  if (typeof fn !== "function") {
    throw TypeError("expected fn is a funtion");
  }

  console.log("create a app!");
  fn();
}
```

这个时候可以把 app 改造成一个函数：

```javascript
var application = function() {
  application.handle();
};
Object.assign(application, app);
```

这样就可以传入 create 函数了：

```javascript
create(application);
//create a app!
//hello jack
```

注意，当前面的 app 设置了名为 name 的属性时会报错："Cannot assign to read only property 'name' of function"。

这些提取于 express 源码。
