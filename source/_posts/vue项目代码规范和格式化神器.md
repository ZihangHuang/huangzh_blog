---
title: vue项目代码规范和格式化神器
date: 2018-12-15 19:08:24
categories: 技术
tags: [工具]
---

&emsp;&emsp;最近用 vue 做了几个很小的活动页。越来越觉得不错，文档写得非常好，生态环境现状也很不错了。
&emsp;&emsp;对于团队间的代码规范，vue 也有一个插件 vuter，配合 eslint、prettier，从此妈妈再也不用担心代码写的乱七八糟了。
&emsp;&emsp;配置也不麻烦，我搭配的编辑器是 vs code，也是一大神器。下面记录下配置详情。

<!--more-->

&emsp;&emsp;首先在 vue 项目初始化的时候选择手动配置，然后在 Linter/Formatter 选择 ESlint + Prettier。
&emsp;&emsp;然后在 vscode 安装 eslint、prettier、vuter。vuter 是 vue 配合 vscode 的一个插件。虽然 eslint 可以处理 vue 文件了，但却跟 prettier 的格式化冲突，因为 prettier 不知道这是什么东西。安装了 vetur 插件后，prettier 知道.vue 原来是一个 html 格式文件的，但依然没办法很好的格式化。
&emsp;&emsp;prettier 覆盖 vscode 默认格式化快捷键，但没有针对 eslint 配置进行格式化，所以需要单独配置用户设置开启。
&emsp;&emsp;找到设置菜单，在右侧用户配置中添加 "prettier.eslintIntegration": true 开启 eslint 支持。
&emsp;&emsp;具体配置如下：

```javascript
"prettier.eslintIntegration": true,
"eslint.autoFixOnSave": true, //保存时使用自动格式化
    "eslint.validate": [
        "javascript",
        "javascriptreact",
         {
          "language": "vue", //添加vue文件校验
          "autoFix": true //vue文件保存时使用自动格式化
        },
        "html"
    ]
```

&emsp;&emsp;然后添加.prettierrc 配置文件，设置你自己的规则，比如我是这样设置：

```javascript
{
  "trailingComma": "none", //对象最后不带,
  "tabWidth": 4,
  "semi": false, //不分号
  "singleQuote": true
}
```

&emsp;&emsp;然后就可以保存时自动格式化成你自己的规则。
&emsp;&emsp;不过还有问题，刚才开启的"prettier.eslintIntegration": true 只是针对.js 文件的，而不是针对.vue 文件。所以你的 vue 文件的 html 部分没有被格式化。所以要配置 vetur 对 html 的格式化。vetur 就是把.vue 文件中的 html+script+style 3 部分拆分，然后交给对应的语言处理器去处理。我们设置 html 部分使用 js-beautify-html 插件格式化。

```javascript
// 使用 js-beautify-html 插件格式化 html
"vetur.format.defaultFormatter.html": "js-beautify-html",
// 格式化插件的配置
"vetur.format.defaultFormatterOptions": {
  "js-beautify-html": {
    // 属性强制折行对齐
    "wrap_attributes": "force-aligned"
    //或者设置为'auto'，自动分行
    //"wrap_line_length": 120,
    //"wrap_attributes": "auto" //自动检测这行到了120字符就换行
  }
}
"editor.formatOnSave": true //每次保存的时候自动格式化(这个设置为true才起作用，但网上一些教程好像没说要)
```

&emsp;&emsp;好了，可以愉快地工作了。
