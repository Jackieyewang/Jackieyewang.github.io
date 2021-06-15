---
layout: post
title: "React Learning -- Day 1"
date: 2021-06-16
description: ""
tag: React
---   
# React.JS

React 起源于Facebook内部项目，由于对市场上JavaScript MVC项目不满意，用于建设Instagram，13年开源

- 设计过程无断层
- 组件化

React 不仅仅是 js 框架本身，更是一套完整的前端开发生态体系，这套体系包括：

1. React.js
2. ReactRenders: ReactDOM / ReactServer / ReactCanvas
3. Flux 模式及其实现（Redux , Fluxxor）
4. React 开源组件
5. React Native
6. GraphQl + Relay

```
框架与库之间最本质区别在于控制权：you call libs, frameworks call you（控制反转）
- lib 库：小而巧
  - 提供特定的API
  - 一个封装好的特定的集合
- framework（框架）：大而全
  - 一整套解决方案
```

![v2-af3f30c3fbcb7a8b63571ac5410364c6_1440w](https://pic3.zhimg.com/80/v2-af3f30c3fbcb7a8b63571ac5410364c6_1440w.png)

## 三大框架

**Angular**：较早，NG2开始支持组件化开发，包含模板、资料双向繫结、路由、模组化、服务、过滤器、依赖注入等所有功能，AngularJS完全基于HTML和JavaScript，使用 TypeScript能够提高程式码可维护性，有利于后期重构

[**React**](https://zhuanlan.zhihu.com/p/21108312#:~:text=%E7%8B%AD%E4%B9%89%E6%9D%A5%E8%AE%B2React%20%E6%98%AF,ReactRenders%3A%20ReactDOM%20%2F%20ReactServer%20%2F%20ReactCanvas):组件化，符合现代web发展趋势，技术成熟，适合大型web项目，生态系统健全，使用方式简单，性能非常高，支持服务端渲染

**Vue.js**:简单易上手，目前最常用，轻量级的框架，不够成熟

