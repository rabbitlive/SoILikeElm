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


## BEST

[Famous框架]()介绍了一个行为-时间-状态-树的MVC变种，Controller被分割成两个单向元素：行为和时间。

**组成部分：**

* **State**： 初始状态JSON形式的声明
* **Tree**：声明了组件的层次组成
* **Event**： 事件来监听tree状态的改变
* **Behavior**：动态属性（树）依赖于state

**特点：**

* **多范式**。state和tree完全是声明式的。event是必要的。behavior是函数式的。一些部分是响应式的，另外一些是被动的（例如）。

* **Behavior**。这篇文章中没有看到任何其他的结构，behavior将UI渲染从动态属性中分离出来。这些据说是不同的，tree组合成HTML，behavior组合成css。

* **用户事件从渲染处声明**。BEST是不在render中附加用户事件处理的少数几个单向数据流框架之一。用户事件处理程序属于event，而不是tree。

这个架构中的内容，view是一个tree，一个组件是一个behavior-event-tree-state元组。组建是UI程序，BEST是分形结构的。


## MODEL-VIEW-UPDATE

又被称为[Elm架构]()，Model-View-Update与Redux很相似，主要是因为后者是受此架构启发。这是一个纯函数式架构，因为他的主要语言是[Elm]()，一个web的函数式编程语言。


**组成部分：**

* **Model**： 定义state结构的类型
* **View**：纯函数，用来渲染state
* **Actions**：定义发送给mailboxes用户事件的类型
* **Update**：纯函数，由前一个状态和action来产生一个新的状态

**特点：**

* **组合层次结构无处不在**。前面的架构只在view层可以组合层次，但是MVU架构中，组合也发生在Model，Update，每个Actions都可以嵌套Actions。

* **组件可导出为组成部分**。因为组合结构无处不在，Elm架构中的一个组件包括了Model type，一个初始Model，一个View函数，一个Action type和一个Update函数。整个体系结构不存在偏离这种结构的组件。每个组件都是一个UI程序，并且他是分形的。


