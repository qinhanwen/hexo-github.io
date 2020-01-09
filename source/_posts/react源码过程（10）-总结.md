---
title: react源码过程（10）- 总结
date: 2020-01-09 12:31:47
tags:
  - react源码过程
categories:
  - react源码过程
---

## 总结

##### 同步情况

**首次渲染**

- createElement 生成 reactComponent
- 调用 render 方法，传入 reactComponent 与 挂载点的实例
- 生成 FiberRoot ，主要的属性是 containerInfo （挂载点的实例），current （RootFiber），以及一大堆的 time
- 调度阶段，把 reactComponent 添加到 update 里，最后放到 RootFiber 的 updateQueue 里
- 建立 FIber 树关联阶段，如果碰到 component 类型的组件，就创建实例，之后调用 render 方法，得到 ReactComponent
- 建立完 Fiber 树关联完之后。创建 DOMElement 实例，并且挂载到 FiberNode 的 stateNode 上，并且添加到父 DOMElement 实例里。也在这个阶段添加事件，绑定到 document 上
- 提交阶段，将 DOMELement 实例添加到 挂载点 下



**触发更新和DOM Diff**

- 点击事件触发，拿到 target 来获得 FiberNode。之后更新它的 updateQueue，如果多次 setState，会有个合并的阶段（每次更新的新的对象都在 updateQueue 链表里，会循环合并）
- 

