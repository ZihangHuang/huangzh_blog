---
title: Promise的基础实现
date: 2020-02-15 11:21:38
categories: 技术
tags: 前端
---

实现基础的 Promise：

<!--more-->

```typescript
type MyPromiseCb = (resolve?: Function, reject?: Function) => any;

class MyPromise {
  private status: "pending" | "fullfilled" | "rejected" = "pending";
  private value: any = ""; // 保存resolve或者reject的值
  private handlers: Function[] = [];
  private errorHandlers: Function[] = [];

  constructor(func: MyPromiseCb) {
    func.call(this, this.resolve.bind(this), this.reject.bind(this));
  }

  resolve(...args) {
    this.value = args;
    let handler: Function;
    while ((handler = this.handlers.shift())) {
      handler(...args);
    }

    this.status = "fullfilled";
  }

  reject(...args) {
    this.value = args;
    let handler: Function;
    while ((handler = this.errorHandlers.shift())) {
      handler(...args);
    }
    this.status = "rejected";
  }

  then(onFulfilled?: any, onRejected?: any) {
    const t = this;

    return new MyPromise(function(onFulfilledNext, onRejectedNext) {
      let fulfilled = val => {
        try {
          if (typeof onFulfilled !== "function") {
            onFulfilledNext(val);
          } else {
            let res = onFulfilled(val);

            if (res instanceof MyPromise) {
              // 如果回调函数返回MyPromise对象，则由它来执行下一个回调
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              // 否则会将返回结果直接作为参数，传入新Promise的resolve函数
              onFulfilledNext(res);
            }
          }
        } catch (err) {
          onRejectedNext(err); // 传入的回调发生错误，则新promise失败
        }
      };

      let rejected = err => {
        try {
          if (typeof onRejected !== "function") {
            onRejectedNext(err);
          } else {
            let res = onRejected(err);

            if (res instanceof MyPromise) {
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              // 捕获错误后将返回结果传入新Promise的resolve函数
              onFulfilledNext(res);
            }
          }
        } catch (err) {
          onRejectedNext(err);
        }
      };

      switch (t.status) {
        case "pending":
          t.handlers.push(fulfilled);
          t.errorHandlers.push(rejected);

          break;

        case "fullfilled":
          onFulfilled(...t.value);
          break;

        case "rejected":
          onRejected(...t.value);
          break;
      }
    });
  }

  catch(onRejected?: any) {
    return this.then(undefined, onRejected);
  }

  finally(callback?: any) {
    return this.then(
      value => {
        callback();
        return value; // 和es6的Promise一样，返回resolve的参数
      },
      reason => {
        callback();
        throw reason; // 抛出错误，让catch可以捕获
      }
    );
  }

  static race(promises: MyPromise[]) {
    return new MyPromise((resolve, reject) => {
      promises.forEach(promise => {
        promise.then(resolve, reject).catch(err => reject(err));
      });
    });
  }

  static all(promises: MyPromise[]) {
    return new MyPromise((resolve, reject) => {
      let len = promises.length;
      let res = [];
      let index = 0;
      promises.forEach((p, i) => {
        p.then(r => {
          res[i] = r;
          index++
          if (index === len) {
            resolve(res);
          }
        }, reject).catch(err => reject(err));
      });
    });
  }
}

// 测试：
const p1 = new MyPromise(resolve => setTimeout(resolve.bind(null, "resolved"), 2000));
p1.then(res => res + " then").then((...args) => console.log("second", ...args));
// second resolved then

// then返回MyPromise
const p1_p = new MyPromise(resolve => setTimeout(resolve.bind(null, "resolved"), 2000));
p1_p.then(res => new MyPromise((resolve) => {
    resolve(res + " then")
})).then((...args) => console.log("p1_p second", ...args));
// p1_p second resolved then

const p2 = new MyPromise((resolve, reject) => setTimeout(reject.bind(null, "rejected"), 2000));
p2.then(res => res + " then")
  .catch((...args) => {
    console.log("fail", ...args);
    return "fail";
  })
  .then(res => console.log(res + " then2"));
// fail rejected
// fail then2

const p3 = new MyPromise(resolve => {
  setTimeout(resolve.bind(null, "p3"), 1000);
});
const p4 = new MyPromise(resolve => {
  setTimeout(resolve.bind(null, "p4"), 3000);
});
MyPromise.race([p3, p4]).then(res => {
  console.log("promise race:", res);
});
// promise race: p3

const p5 = new MyPromise(resolve => {
  setTimeout(resolve.bind(null, "p5"), 1000);
});
const p6 = new MyPromise(resolve => {
  setTimeout(resolve.bind(null, "p6"), 3000);
});
MyPromise.all([p5, p6])
  .then(res => {
    console.log("promise all:", res);
  })
  .catch(err => console.log("promise all error:", err));
// promise all: ["p5", "p6"]

const p7 = new MyPromise((resolve, reject) => {
  setTimeout(resolve.bind(null, "p7"), 1000);
});
p7.finally(() => {
  console.log("p7-finally"); // p7-finally
}).then(res => {
  console.log(res);
});
// p7-finally
// p7

const p8 = new MyPromise((resolve, reject) => {
  setTimeout(reject.bind(null, "p8-err"), 1000);
});
p8.finally(() => {
  console.log("p8-finally");
}).catch(res => {
  console.log(res);
});
// p8-finally
// p8-err

// 这里实现的能捕获到，但原生的不能捕获到，会吃掉错误，所以原生的promise要写catch
try {
    new MyPromise((resolve, reject) => {
        throw new Error('err')
    })
} catch (e) {
    console.error('捕获到了:',e) // 捕获到了: Error: err
}

try {
    new Promise((resolve, reject) => {
        throw new Error('err') // 报错：Uncaught (in promise)
    })
} catch (e) {
    console.error('捕获到了:',e)
}
```
