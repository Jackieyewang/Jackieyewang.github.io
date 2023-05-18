---
layout: post
title: "React（1）"
date: 2021-06-15
description: "React 1"
tag: 语言学习
---   
# [Nervos](https://github.com/nervosnetwork/rfcs)



# React.JS

[toc]

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

**Vue.js**:简单易上手，轻量级的框架，不够成熟



## React 核心概念

### [虚拟DOM](https://juejin.cn/post/6844903870229905422)

**DOM的本质：**document object model，浏览器的概念，用JS对象表示页面上的元素，并提供了操作DOM对象的API

**Virtual DOM**：框架的概念，通过原生`JS`的Object对象模拟DOM中的节点，然后再通过特定的render方法将其渲染成真实的DOM节点 --- 使用`JavaScript`对象来表示`DOM`树的信息和结构

#### 虚拟DOM本质

- `Vue`中的虚拟`DOM`是使用`template`完成，过`vue-loader`对其进行编译处理最后形成虚拟`DOM`

- `React`使用的是`JSX`对进行编译，最后产生虚拟DOM

- 虚拟DOM就是一个普通的JavaScript对象，包含了`tag`、`props`、`children`三个属性（标签名，各种属性，孩子节点）。

  ```
  <div id="app">
    <p class="text">hello world!!!</p>
  </div>
  
  ====== 》》》》
  
  {
    tag: 'div',
    props: {
      id: 'app'
    },
    chidren: [
      {
        tag: 'p',
        props: {
          className: 'text'
        },
        chidren: [
          'hello world!!!'
        ]
      }
    ]
  }
  ```

  - createElement方法创建虚拟DOM

    createElement 接受`type`, `props`,`children`三个参数创建一个虚拟标签元素DOM的方法。

    ```
    function createElement(type, props, children) {
        return new Element(type, props, children);
    }
    ```

  - render方法将虚拟DOM转化成真实DOM

    ```
    function render(eleObj) {
        let el = document.createElement(eleObj.type); // 创建元素
        for(let key in eleObj.props) {
            // 设置属性的方法
            setAttr(el, key, eleObj.props[key])
        }
        eleObj.children.forEach(child => {
            // 判断子元素是否是Element类型，是则递归，不是则创建文本节点
            child = (child instanceof Element) ? render(child) : document.createTextNode(child);
            el.appendChild(child);
        });
        return el;
    }
    ```

网页呈现过程：

 	1. 浏览器请求服务器获取页面HTML代码
 	2. 浏览器在内存中解析DOM结构，并在内存中渲染DOM树
 	3. 将DOM树呈现在页面上

#### [Diff算法](https://blog.csdn.net/qq_39414417/article/details/104763824)

 	1.用JavaScript模拟DOM树，并渲染这个DOM树。
     2.比较新老DOM树，得到比较的差异对象。
     3.把差异对象应用到渲染的DOM树。

**diff过程整体策略：深度优先，同层比较**

![20200310082333622](https://img-blog.csdnimg.cn/20200310082333622.png)



## 创建webpack项目

