---
title: generator生成器函数笔记
date: 2019-06-11 11:37:19
categories: 技术
tags: [前端]
---

#### `yield*` 表达式

<!--more-->

````javascript
function* foo() {
  yield "a";
  yield "b";
}
function* bar() {
  yield "x";
  yield* foo();
  yield "y";
}

// 等同于
function* bar() {
  yield "x";
  yield "a";
  yield "b";
  yield "y";
}

// 等同于
function* bar() {
  yield "x";
  for (let v of foo()) {
    yield v;
  }
  yield "y";
}

for (let v of bar()) {
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"


let delegatedIterator = (function* () {
  yield 'Hello!';
  yield 'Bye!';
}());

let delegatingIterator = (function* () {
  yield 'Greetings!';
  yield* delegatedIterator;
  yield 'Ok, bye.';
}());

for(let value of delegatingIterator) {
  console.log(value);
}
// "Greetings!
// "Hello!"
// "Bye!"
// "Ok, bye."
````

参考：[阮一峰 ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/generator)