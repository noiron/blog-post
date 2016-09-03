---
layout: post
title: 用 React.js 写一个最简单的 To-do List 应用
date:   2016-06-19
tags: [javascript, react.js]
---

前段时间忙着找工作去了，有段时间没有写博客了，上一篇还是三月份的，今天来更新一篇。

---

最近在学 React.js，也写了一些练习的项目，之前参考网上的一些代码写了一个很简单的 to-do list。对于初学者来说，写个基本的 to-do list 对于理解 React 中的一些概念及语法倒是挺有帮助的。

现在很多的 React 项目中已经开始使用 ES6 来写了，不过因为我在学习 React 的时候看的教程大多都是用 ES5 写的，我这里还是用的还是更熟悉的 ES5 写法，虽然有点落伍了，但若想改成 ES6 版本倒也挺方便的。

> [GitHub 上的项目地址](https://github.com/noiron/simplest-react-todolist)
> [在线 demo](http://www.wukai.me/simplest-react-todolist)

<!-- more -->

## 文件目录

在正式的生产项目中，使用 webpack 可以很方便地对我们的文件进行打包，这里因为程序比较简单，直接用 `<script>` 标签将 React 组件引入了。

首先新建一个 `index.html` 文件，引入相关的资源文件。

再新建一个 `js` 文件夹，我们使用 React 需要这样的两个文件： `react.js` 和 `react-dom.js`，你可以使用 cdn 引入，这里直接将文件下载放在了 `js` 文件夹内。

`js` 文件夹内还有一个 `script.jsx` 文件，我们程序的主要内容就放在这个文件中。注意这里的后缀名是 `jsx`，表示它是使用 React 的 `jsx` 语法来写的，引入它的时候按如下写法：

```html
    <script type="text/babel" src="js/script.jsx"></script>
```

同时还需要一个 `browser.js` 文件，它可以让 `jsx` 语法的文件在浏览器中运行。

最后我们再建立一个 `css` 文件夹，存放样式文件，我的项目中使用了 Bootstrap 的样式，所以需要下载 Bootstrap 的样式文件。

## 程序功能

作为一个最简单的 to-do list，这个程序没有过多的功能。可以从 [demo](http://www.wukai.me/simplest-react-todolist/) 里看出，它的功能如下：

- 显示每一个任务

- 可以将任务标记为已完成，以区分未完成的任务

- 加入任务 / 删除任务

- 统计任务总数和完成的任务数量

作为一个示例程序，以上就是它的功能了。


## 组件结构

我们可以使用 React 开发出各种组件（component），利用不同组件的功能来实现一个应用。我们这里创建的组件有:

    TodoBox
	    -TodoList
		    -TodoItem
		    -TodoFooter
	    -TodoForm

- TodoBox 是最外层的组件，其余的都是它的子组件

- TodoList 是各个单独的待办任务的集合

- TodoItem 即为一条单独的待办事项

- TodoFooter 对上述的事项进行统计

- TodoForm 用于加入新的项目


## 属性的传递

React 的组件有两种不同的属性，`state` 和 `props`。可以用一种简单的方法来区分它们：如果这个属性是其父组件传给它的，那么就是 `props`，反之如果一个属性是组件自己的，那么就是 `state`。

具体什么时候用 `state`，什么时候用 `props`，可以参考这几条：

> - Is it passed in from a parent via props? If so, it probably isn't state.

> - Does it change over time? If not, it probably isn't state.

> - Can you compute it based on any other state or props in your component? If so, it's not state.

> [参考来源：Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html#step-3-identify-the-minimal-but-complete-representation-of-ui-state)

这里我们从代码来看看，属性是如何从父组件传递到子组件的。

每一条待办事项有这样的几个属性:

- id: 任务的编号
- task: 任务的具体内容
- complete: 任务是否已经完成

我们看看属性的传递过程。

首先在 `TodoBox` 组件的 `state` 中有一个 `data` 对象：

```javascript
  data: [
    {"id": "0001", "task":"吃饭", "complete": "false"},
    {"id": "0002", "task":"睡觉", "complete": "true"},
    ...
  ]
```
`TodoBox` 组件的 render 函数中会有 `TodoList` 组件：

```javascript
  <TodoList data={this.state.data}
    // 其他的属性及方法写在这里
  />
```

这样 `TodoBox` 组件的 `data` 属性就传递给了子组件 `TodoBox`。在 `TodoBox` 中通过 `this.props` 来引用这一属性，即 `this.props.data`。

`TodoBox` 组件还有子组件 `TodoItem`，可以将属性继续传递下去。在 `TodoList` 组件的 `render` 函数中这样写：

```javascript
  var taskList = this.props.data.map(function(listItem) {
      return (
        <TodoItem
          taskId={listItem.id}
          key={listItem.id}
          task={listItem.task}
          complete={listItem.complete}
          // 其他的属性及方法
      )
    }, this);
```

在 `TodoItem` 组件中就可以用 `this.props.taskId` 获得任务的 id 了。

## 函数的传递

我们的程序中需要的函数有这几个：

- handleTaskDelete: 根据id删除一项任务
- handleToggleComplete: 切换一项任务的完成状态
- handleSubmit: 新增一项任务
- generateId: 给新增的任务一个随机的id

在 React 的组件中传递方法与传递属性类似，现在 `TodoBox` 组件中有一个 `handleToggleComplete` 函数，将它传递给 `TodoList` 组件：

```javascript
  <TodoList toggleComplete={this.handleToggleComplete}
    // 其他的属性及方法写在这里
  />
```

这样你就可以在 `TodoList` 组件中通过 `this.props.toggleComplete` 来调用这一方法了，你也可以将这一方法继续向下一层的组件传递。

## 程序的运行

你可以下载 [GitHub 上的项目文件](https://github.com/noiron/simplest-react-todolist)，再用 python 开启一个 HTTP 服务器，就可以打开 http://localhost:8000/ 查看运行结果了。

    git clone https://github.com/noiron/simplest-react-todolist.git
    cd simplest-react-todolist
    python -m SimpleHTTP server // open a server with python

这篇博客里没有对整个项目所有的代码进行分析，更多内容还是[直接看代码](https://github.com/noiron/simplest-react-todolist)更清楚。
