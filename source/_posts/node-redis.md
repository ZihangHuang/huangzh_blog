---
title: node_redis的一个例子
date: 2018-06-13 19:43:02
categories: 技术
tags: node
---

最近在看了不起的 node.js，看得有点仓促，只是涉猎了一遍。但 redis 这一章的代码我觉得写得很巧妙，对我这种 node 新手来说，很值得学习。
这是一个记录粉丝关注的例子，话不多说，直接看代码。

<!--more-->

首先创建一个 model.js 文件，当然这里是已经安装好 redis 这一模块。

连接 redis

```javascript
var redis = require("redis");

client = redis.createClient(6379, "127.0.0.1", {});
```

创建一个 User 类，并设置一些方法。

```javascript
function User(id, data) {
    this.id = id;
    this.data = data;
}

//查找用户
User.find = function(id, fn) {
    client.hgetall("user:" + id + ":data", function(err, obj) {
        if (err) {
            console.log(err);
        }
        fn(null, new User(id, obj)); //把实例对象传给回调函数，才能通过这个实例调用下面的实例方法
    });
};

//存储用户
User.prototype.save = function(fn) {
    if (!this.id) {
        this.id = String(Math.random()).substr(3);
    }
    client.hmset("user:" + this.id + ":data", this.data, fn);
};

//关注
User.prototype.follow = function(user_id, fn) {
    //console.log("user_id:", "user:" + user_id + ":followers", this.id);

    //multi()表示事务开始，exec(fn)表示开始执行sadd等，fn第一个参数是err,第二个是执行结果如[1,1]，[0,0]（1成功，0失败）
    client
        .multi()
        .sadd("user:" + user_id + ":followers", this.id)
        .sadd("user:" + this.id + ":follows", user_id)
        .exec(fn);
};

//取消关注
User.prototype.unfollow = function(user_id, fn) {
    client
        .multi()
        .srem("user:" + user_id + ":followers", this.id)
        .srem("user:" + this.id + ":follows", user_id)
        .exec(fn);
};

//获取粉丝
User.prototype.getFollowers = function(fn) {
    client.smembers("user:" + this.id + ":followers", fn);
};

//获取关注的人
User.prototype.getFollows = function(fn) {
    client.smembers("user:" + this.id + ":follows", fn);
};

//获取朋友，即互相关注的人
User.prototype.getFriends = function(fn) {
    client.sinter("user:" + this.id + ":follows", "user:" + this.id + ":followers", fn);
};

//输出
module.exports = User;
```

接下来创建一个 server.js 文件，设置一个初始对象，并创建两个函数。

```javascript
var User = require("./model");

var testUsers = {
    "mark@qq.com": { name: "Mark Zuck" },
    "bill@gmail.com": { name: "Bill Gates" },
    "jeff@163.com": { name: "Jeff Bezos" },
    "fred@fox.com": { name: "Fred Smith" }
};

//创建用户
function create(users, fn) {
    var total = Object.keys(users).length;

    for (var i in users) {
        (function(email, data) {
            var user = new User(email, data);
            user.save(function(err) {
                if (err) throw err;
                --total || fn();
            });
        })(i, users[i]);
    }
}

//检索用户
function hydrate(users, fn) {
    var total = Object.keys(users).length;

    for (var i in users) {
        (function(email) {
            User.find(email, function(err, user) {
                if (err) throw err;
                // console.log(user)
                users[email] = user;
                --total || fn();
            });
        })(i);
    }
}
```

create 函数里面，获取 users 的属性长度，然后循环执行 save，为了确保 save 的值，使用立即执行函数。
--total|fn()使循环结束后才执行 fn()。

好了，该执行函数了。bill 先关注 jeff，然后打印出 jeff 的粉丝，再打印出 jeff 的朋友。等 jeff 也关注 bill 后，再打印出 jeff 的朋友，看看有什么变化（异步回调真有点烦）。

```javascript
create(testUsers, function() {
    hydrate(testUsers, function() {
        console.log("testUsers: ", testUsers);

        testUsers["bill@gmail.com"].follow("jeff@163.com", function(err, reply) {
            if (err) throw err;
            console.log("+ bill followed jeff");

            testUsers["jeff@163.com"].getFollowers(function(err, users) {
                if (err) throw err;
                console.log("jeff's followers:", users);

                testUsers["jeff@163.com"].getFriends(function(err, users) {
                    if (err) throw err;
                    console.log("jeff's friends:", users);

                    testUsers["jeff@163.com"].follow("bill@gmail.com", function(err, reply) {
                        if (err) throw err;
                        console.log("+ jeff followed bill");

                        testUsers["jeff@163.com"].getFriends(function(err, users) {
                            if (err) throw err;
                            console.log("jeff's friends:", users);

                            process.exit(0);
                        });
                    });
                });
            });
        });
    });
});
```

结果如下：

```
user_id: user:jeff@163.com:followers bill@gmail.com
+ bill followed jeff
jeff's followers [ 'bill@gmail.com' ]
jeff's friends: []
user_id: user:bill@gmail.com:followers jeff@163.com
+ jeff followed bill
jeff's friends: [ 'bill@gmail.com' ]
```
