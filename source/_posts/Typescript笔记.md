---
title: Typescript笔记
date: 2019-04-04 10:26:52
categories: 技术
tags: [前端]
---

#### Typescript 常用的映射类型：

-   ##### Recode(内置)

根据 K 中的所有可能值来设置 key 以及 value 的类型

```typescript
type RecodeProps = Record<"prop1" | "prop2" | "prop3", string>;
let per1: RecodeProps = {
    prop1: "abc",
    prop2: "abc",
    prop3: "bcd"
};
```

<!--more-->

它的定义:

```typescript
type Record<K extends string, T> = { [P in K]: T };
```

-   ##### Pick(内置)

把某个类型中的子属性挑选出来

```typescript
interface Person2 {
    name: string;
    age: number;
}
```

```typescript
//定义
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type PickPerson = Pick<Person2, "name">; // { name: string; }
let per2: PickPerson = { name: "123" };
```

-   ##### Readonly(内置)

每个属性成为 readonly 类型

```typescript
//定义
type Readonly<T> = { readonly [P in keyof T]: T[P] };
type ReadonlyPerson = Readonly<Person2>;
let per3: ReadonlyPerson = {
    name: "22",
    age: 123
};

//拓展，设置子元素也成为 readonly 类型
type DeepReadonly<T> = { readonly [P in keyof T]: DeepReadonly<T[P]> };
```

-   ##### Partial(内置)

每个属性成为可选

```typescript
//定义
type Partial<T> = { [P in keyof T]?: T[P] };
type PartialPerson = Partial<Person2>;
let per4: PartialPerson = {
    name: "fff" //name和age属性都可以不写
};
```

-   ##### 其他(摘自官网)

```
Exclude<T, U> -- 从T中剔除可以赋值给U的类型。
Extract<T, U> -- 提取T中可以赋值给U的类型。
ReturnType<T> -- 获取函数返回值类型。
InstanceType<T> -- 获取构造函数类型的实例类型。
NonNullable<T> -- 从T中剔除null和undefined。
```

示例：

```typescript
type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "b" | "d"
type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "a" | "c"

type T02 = Exclude<string | number | (() => void), Function>; // string | number
type T03 = Extract<string | number | (() => void), Function>; // () => void

type T04 = NonNullable<string | number | undefined>; // string | number
type T05 = NonNullable<(() => string) | string[] | null | undefined>; // (() => string) | string[]

function f1(s: string) {
    return { a: 1, b: s };
}

class C {
    x = 0;
    y = 0;
}

type T10 = ReturnType<() => string>; // string
type T11 = ReturnType<(s: string) => void>; // void
type T12 = ReturnType<<T>() => T>; // {}
type T13 = ReturnType<<T extends U, U extends number[]>() => T>; // number[]
type T14 = ReturnType<typeof f1>; // { a: number, b: string }
type T15 = ReturnType<any>; // any
type T16 = ReturnType<never>; // any
type T17 = ReturnType<string>; // Error
type T18 = ReturnType<Function>; // Error

type T20 = InstanceType<typeof C>; // C
type T21 = InstanceType<any>; // any
type T22 = InstanceType<never>; // any
type T23 = InstanceType<string>; // Error
type T24 = InstanceType<Function>; // Error
```

-   ##### 还有几个拓展的映射类型

```typescript
type Omit<T, U> = Pick<T, Exclude<keyof T, U>>;
type Overwrite<T, U> = Pick<T, Exclude<keyof T, keyof U>> & U;

////示例
type Item1 = { a: string; b: number; c: boolean };
type Item2 = { a: number; d: number };

type T2 = Omit<Item1, "a">; // { b: number, c: boolean };
type T3 = Overwrite<Item1, Item2>; // { a: number, b: number, c: boolean, d: number };

let t3: T3 = {
    a: 123,
    b: 123,
    c: true,
    d: 123
};

//获取类本身的类型
type ClassOf<T> = new (...args: any[]): T;

//示例
class Person{}

let per: Person = new Person()

let per2: InstanceType<typeof Person> = new Person()

//Person是类实例的类型，ClassOf<Person>才是类本身的类型，或者可以写成typeof Person
let setPerson = function (Per: ClassOf<Person>): Person {
    return new Per()
}
```

#### 装饰器(decorator)

-   ##### 类装饰器

```typescript
function classDecorator<T extends { new(...args: any[]): {} }>(target: T) {
    return class extends target {
        hello = "override";
    }
}

@classDecorator
class Greeter {
    property = "property";
    hello: string;
    constructor(m: string) {
        this.hello = m;
    }
}
console.log(new Greeter("world").hello); // override
```

-   ##### 方法装饰器

```typescript
function enumerable(value: boolean) {
    /**
     * @param {any}                target      [装饰的属性所属的类的原型，如Greeter2.prototype]
     * @param {string}             propertyKey [装饰的属性的key]
     * @param {PropertyDescriptor} descriptor  [装饰的属性的对象的描述符对象即descriptor，表示可枚举可写等的对象]
     */
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const fun = descriptor.value //即是装饰的属性的值
        descriptor.value = function(str: string){
            console.log('我在前面加上了这句')
            const res = fun.call(this, str) //执行原来的方法，注意有可能不是方法而是属性，这里设定是方法
            return res + ' china'
        }
        //可以返回一个新的描述符用来修改原描述符，如return {enumerable: false}
    };
}
class Greeter2 {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet(str) {
        return "Hello, " + this.greeting + str;
    }
}
let greet2 = new Greeter2('green')
console.log(greet2.greet(', welcome to'))
// 我在前面加上了这句
// Hello, green, welcome to china
```

#### 常用技巧

-   ##### 根据属性获取对象的某个值

```typescript
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}
var aa3 = { a: 1, b: 2 };
getProperty(aa3, "a");
```

-   ##### 限制传入特定的参数

```typescript
interface API {
    baidu: string;
    google: string;
}

function get<URL extends keyof API>(url: URL): API[URL] {
    return "https://www." + url + ".com";
}

get("baidu");
```

-   ##### Record 设置类型相同的类型

```typescript
enum AnimalType {
    CAT = "cat",
    DOG = "dog",
    FROG = "frog"
}

type Maps = Record<AnimalType, Function>;

let animal: Maps = {
    cat() {},
    dog() {},
    frog() {}
};
```

-   ##### `$('button')`是个 DOM 元素选择器，可是返回值的类型是运行时才能确定的，除了返回 any ，还可以

```typescript
function $<T extends HTMLElement>(id: string): T {
    return document.getElementById(id);
}

$<HTMLInputElement>("input").value;
```

-   ##### 在 window 对象上显式设置属性

```typescript
// 1
(window as any).MyNamespace = {};

// 2
declare interface Window {
  MyNamespace: any;
}
window.MyNamespace = window.MyNamespace
```

##### 常用
```typescript
// 导出从别的模块导入的类型
export type { AuditMessage } from '@/types/voice'

// 遍历enum
export enum MotifIntervention {
    Intrusion,
    Identification,
    AbsenceTest,
    Autre
}
// 1:
for (let item in MotifIntervention) {
    if (isNaN(Number(item))) {
        console.log(item);
    }
}
// 2:
Object.keys(MotifIntervention).filter(key => !isNaN(Number(MotifIntervention[key])));
```
