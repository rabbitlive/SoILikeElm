翻自[Unidirectional User Interface Architectures](http://staltz.com/unidirectional-user-interface-architectures.html)

# 单项UI架构

这篇文章是对单向数据流架构的简单的快速浏览。主要用来比较各架构之间的差异和特殊性，可能不适合作为一份初学者教程。在最后我将介绍一个新的架构，它与其他的架构有着明显的差异。这篇文章只针对客户端的 Web UI。

## 术语

如果没有一些同一的术语来讨论这些架构是很困惑的，所以我们先来假设以下：

> **用户事件**是指由用户直接操作输入设备的事件，如：鼠标点击，滚动，键盘按下，触摸触摸屏等。

架构使用的术语“视图”可能有很大的不同。相反，我们用“渲染”来代指视图的一般解释：

> **用户界面渲染**指的是屏幕上的图形输出，通常表示为 HTML 或一些高级代码如JSX。

> **UI 程序**是一个把用户事件作为输入并输出渲染的程序，是一个持续性的过程，而不是一次性的。

DOM 和其他框架层和类库都假定存在于用户和架构之间。

模块间箭头的所有权。`A--> B`是不同于`A -->B`的。前者是被动程序，而后者是响应式程序。更多的阅读[这里](http://cycle.js.org/observables.html#reactive-programming)

一个单项架构可以被称为分形，如果子组建的结构与整体的相同。

在分形架构中，所有的东西可以打包成一个组建用于大型应用程序。

非分形架构，非重复的部分是说有层次的构成部分参与者。

## Flux

首先有必要提及的就是[Flux]()。虽说他并不是第一个单向数据流框架，但他确实是第一个被普及为好多人使用的框架。

**组成部分：**

* **Stores**： 管理数据和状态
* **View**：由React组件组成的层次结构
* **Action**： 创建一个由view触发的事件
* **Dispatcher**： 事件总线，用于集中管理所有的action

**特点：**

* **Dispatcher**。因为这一个事件总线，他是单例的。所以许多Flux框架的变形直接将dispatcher去掉了，或者其他单向流框架没有一个等同与dispatcher的东西。

* **仅仅view可以组合组件**。层次结构组合只发生在React组件，不是store，也不是action。一个React组件就是一个UI程序，通常不会写为Flux架构内部的一部分。因此，Flux不是分形结构，他的参与者是dispatcher和store。

* **用户事件声明在render中**。换句话说，由React组件的`render()`函数处理与用户的交互：渲染和用户事件（如，`onClick={this.clickHandler}`）

## Redux

Redux是Flux的一个变形，在Redux中，单例dispatcher变成了单例store。store并不是从零构建出的，相反，他是由一个给定的reducer函数创建出的store工厂。


**组成部分：**

* **单例Stores**： 管理状态并且有一个`dispatch(action)`函数
* **Provider**：一个Store的订阅者，接口是像React或Angular之类的视图框架
* **Actions**： 创建一个由Provider上用户触发的事件
* **Reducers**： 纯函数，将前一个状态和一个action得到一个新状态

**特点：**

* **stores工厂**。一个store是由`createStore()`工厂函数创建，作为参数传递给组合的reducer函数。他也是传递给一个元工厂`applyMiddleware()`中间件函数的参数。中间件是非常重要的`dispatch()`函数，用于链式调用。

* **Providers**。Redux并不关心用什么view框架去做UI程序。他能够使用React，Angular或其他。在这个架构中，view就是一个UI程序。像Flux一样，Redux也不是分形结构，store是一个协调者。

用户事件处理可以不放在render中声明，这取决与Provider。
