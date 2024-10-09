---
title: react笔记
date: 2020-04-08 17:05:41
categories: 技术
tags: 前端
---
- 尽量不要使用props来派生出state，这样会导致props更新但state不更新。如实在要但又要保持两者同步更新，在静态方法`getDerivedStateFromProps`更新state
- 保持单一数据量，例如不要使用props来派生出state，然后组件内部又setState来修改state。
- getDerivedStateFromProps和componentWillReceiveProps不只是在props发生变化时才执行，只要父组件重新渲染，就会执行，可以使用shouldComponentUpdate来减少重复渲染