---
title: MobX 核心概念
sidebar_label: MobX 核心概念
hide_title: true
---

<script async type="text/javascript" src="//cdn.carbonads.com/carbon.js?serve=CEBD4KQ7&placement=mobxjsorg" id="_carbonads_js"></script>

# MobX 核心概念

## 概念

在您的应用程序中，MobX区分了以下三个概念:

1. State
2. Actions
3. Derivations

下面让我们仔细看看这些概念，或者，在[10分钟了解MobX和React](https://mobx.js.org/getting-started)中，你可以一步一步地深入研究这些概念，并创建一个简单的TodoList应用程序。

### 1. 定义 state 并使其 observable

_State_ 是指驱动应用程序的数据。
通常，有 _特定领域状态(domain specific state)_ ，如待办事项列表，还有 _视图状态(view state)_ ，如当前选中的元素。状态就像保存值的电子表格的单元格一样。

将状态存储在任何你喜欢的数据结构中:普通对象、数组、类、循环数据结构或引用。这与MobX的工作方式无关。只要确保所有你想要随时间改变的属性都被标记为 `observable`，这样MobX就可以跟踪它们(的变化)。

下面是一个简单的例子：

```javascript
import { makeObservable, observable, action } from "mobx"

class Todo {
    id = Math.random()
    title = ""
    finished = false

    constructor(title) {
        makeObservable(this, {
            title: observable,
            finished: observable,
            toggle: action
        })
        this.title = title
    }

    toggle() {
        this.finished = !this.finished
    }
}
```

**提示**: 这个例子可以用[`makeauoobservable`](observable-state.md)来简化，但是通过显式的声明，我们可以更详细地展示不同的概念。

使用 `observable` 就像是将对象的属性转换为电子表格的单元格。但与电子表格不同的是，这些值不仅可以是基本类型，还可以是引用、对象和数组。
但是被我们标记为 `action` 的 `toggle` 呢？

### 2. 使用 actions 更新 state

_action_ 是任何改变状态的代码段。用户事件、后端数据推送、定时任务等等。一个 action 就像一个用户在电子表格单元格中输入一个新值。

在上面的 `Todo` 模型中，您可以看到我们有一个 `toggle` 方法可以更改 `finished` 的值。`finished` 被标记为 `observable`。 建议您将任何可更改 `observable` 们的代码标记为[`action`](actions.md)。 这样，MobX可以自动应用事务以轻松实现最佳性能。

使用 actions 可以帮助您组织代码，并防止您无意中更改 state。在 MobX 术语中，修改状态的方法被称为 _actions_, 与 _views_ 不同，_actions_ 基于当前 state 计算新的状态。每一种方法最多只能服务于这两个目标中的一个。

### 3. 创建自动响应 state 变化的 derivations(派生)

_任何_ 可以从 state 中衍生出而不需要进一步交互的东西都是 derivation。
derivation 有多种形式：

-   _用户界面_
-   _派生数据_ ，例如未完成 _待办事项_ 的数量
-   _后台操作_ ，例如将更改发送给服务器

MobX 把派生区分为两种情况：

-   _Computed values(计算值)_，可以始终使用纯函数从当前 state 计算得出
-   _Reactions(副作用)_，状态改变时需要自动执行的副作用（命令式和反应式编程之间的桥梁）

当刚开始使用MobX时，人们往往会过度使用 Reactions。
黄金法则是，如果您想基于当前 state 创建一个值，总是使用 `computed`。
(译者注：此处原则和React[确定 UI state 的最小（且完整）表示](https://zh-hans.reactjs.org/docs/thinking-in-react.html#step-3-identify-the-minimal-but-complete-representation-of-ui-state)有异曲同工之处)

#### 3.1. 使用计算值派生的模型

要创建一个计算值，可以使用 JS 的 getter 函数 `get`定义一个属性，并用 `makeObservable` 将其标记为 `computed`。

```javascript
import { makeObservable, observable, computed } from "mobx"

class TodoList {
    todos = []
    get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length
    }
    constructor(todos) {
        makeObservable(this, {
            todos: observable,
            unfinishedTodoCount: computed
        })
        this.todos = todos
    }
}
```

当添加一个 todo 或修改某个 todo 的 `finished` 属性时，MobX 将确保自动更新 `unfinishedTodoCount`。

这些计算类似于 MS Excel 等电子表格程序中的公式。它们只在需要时自动更新。换句话说，有地方用到它们结果的时候才会更新。

#### 3.2. 使用副作用派生的模型

作为用户，您可以在屏幕上看到状态的变化或计算值的变化，需要一个 _reaction_ 来重绘 GUI 的一部分。

Reactions 类似于计算值，但它们不产生信息，而是产生副作用，如打印到控制台、发送网络请求、增量更新 React 组件树来更新 DOM 等等。

简而言之，reactions 架起了 [响应式](https://en.wikipedia.org/wiki/Reactive_programming) 和 [命令式](https://en.wikipedia.org/wiki/Imperative_programming) 编程的桥梁。

到目前为止，最常用的 reactions 形式是 UI 组件。
需要注意的是 actions 和 reactions 都有可能引发副作用。
具有清晰、明确触发源的副作用，比如在提交表单时发出网络请求，应该由相关事件处理函数显式触发。

#### 3.3. 响应式 React 组件

如果你正在使用 React，你可以通过[安装期间选择的](installation.md#installation)绑定包中的 [`observer`](react-integration.md) 函数来包装组件，使它们具有响应性。在这个例子中，我们将使用更轻量级的 `mobx-react-lite` 包。

```javascript
import * as React from "react"
import { render } from "react-dom"
import { observer } from "mobx-react-lite"

const TodoListView = observer(({ todoList }) => (
    <div>
        <ul>
            {todoList.todos.map(todo => (
                <TodoView todo={todo} key={todo.id} />
            ))}
        </ul>
        Tasks left: {todoList.unfinishedTodoCount}
    </div>
))

const TodoView = observer(({ todo }) => (
    <li>
        <input type="checkbox" checked={todo.finished} onClick={() => todo.toggle()} />
        {todo.title}
    </li>
))

const store = new TodoList([new Todo("Get Coffee"), new Todo("Write simpler code")])
render(<TodoListView todoList={store} />, document.getElementById("root"))
```

`observer` 将 React 组件转换为它们所呈现的数据的派生。当使用 MobX 时，没有容器组件和展示组件的区别，所有的组件都被智能地渲染，但是以展示组件的形式定义。MobX 会确保组件只在需要的时候被重新渲染，而不会超出此范围。

所以在上面例子中，如果执行了 `toggle` action，`onClick` 事件处理程序将强制重新 render 对应的 `TodoView` 组件，如果只有未完成的任务数量发生改变，则只会重新 render `TodoListView` 组件。如果你删除了 `Tasks left` 这一行(或者把它放到一个单独的组件中)，`TodoListView` 组件在勾选一个任务时将不再重新 render。

要了解更多关于 React 如何与 MobX 搭配使用的信息，请查看 [React integration](react-integration.md) 一节。

#### 3.4. 自定义 reactions(派生)

您可能需要它们，但可以使用 [`autorun`](reactions.md#autorun) 来创建它们,使用 [`reaction`](reactions.md#reaction) 或 [`when`](reactions.md#when) 函数来匹配特定的情形。
例如，下面的 `autorun` 在每次 `unfinishedTodoCount` 的数量发生变化时打印一条日志消息：

```javascript
// A function that automatically observes the state.
autorun(() => {
    console.log("Tasks left: " + todos.unfinishedTodoCount)
})
```

Why does a new message get printed every time the `unfinishedTodoCount` is changed? The answer is this rule of thumb:
为什么每次更改 `unfinishedTodoCount` 时都会打印一条新消息？答案就是这条经验法则：

_MobX reacts to any existing observable property that is read during the execution of a tracked function._
_在跟踪函数的执行过程中，MobX会对任何已存在的 observable 属性做出响应。_

To learn more about how MobX determines which observables need to be reacted to, check out the [Understanding reactivity](understanding-reactivity.md) section.
要了解更多关于 MobX 如何决定需要对哪些 bservable 对象做出响应的，请查看[理解响应性](understanding-reactivity.md)一节。

## 原则

MobX 使用单向数据流，其中 _actions_ 改变 _state_ ，进而更新所有受影响的 _views_ 。

![Action, State, View](assets/action-state-view.png)

1. 当 _state_ 更改时，所有 _derivations(派生)_ 都会**自动和原子地**更新。 所以，永远不可能观测到中间值。

2. 默认情况下，所有 _derivations(派生)_ 都是同步更新的。这意味着，例如， _actions_ 可以在改变 _state_ 后直接安全地使用计算值。

3. _computed values(计算值)_ 被懒惰地更新。除非有副作用（I/O）所需，否则不会更新任何未实际使用的计算值，如果一个视图不再使用它，它将被自动垃圾回收。

4. 所有 _computed values(计算值)_ 都应该是**纯**的。它们不应该改变 _state_。

想了解更多背景知识，请查看[the fundamental principles behind MobX](https://hackernoon.com/the-fundamental-principles-behind-mobx-7a725f71f3e8).

## 试一试!

你可以在 [CodeSandbox](https://codesandbox.io/s/concepts-principles-il8lt?file=/src/index.js:1161-1252) 页面尝试上面的例子。

## Linting

If you find it hard to adopt the mental model of MobX, configure it to be very strict and warn you at runtime whenever you deviate from these patterns. Check out the [linting MobX](configuration.md#linting-options) section.
如果你觉得难以适应 MobX 的思维模式，可以将其配置得非常严格，以便在运行时每当你偏离这些模式时警告你。请查看 [linting MobX](configuration.md#linting-options) 章节。
