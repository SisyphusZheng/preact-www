---
title: Preact X, a story of stability
date: 2024-05-24
authors:
  - Jovi De Croock
---

# Preact X，一个关于稳定性的故事

很多人一直在等待[Preact 11](https://github.com/preactjs/preact/issues/2621)，这个版本早在2020年7月就在一个议题中宣布了，说实话，我是最期待v11的人之一。
当我们开始考虑Preact 11时，我们认为在不引入破坏性变更的情况下，无法在Preact X中引入我们心目中的变化，我们当时考虑的一些内容包括：

- 使用一个背景VNode结构来减少垃圾回收，通过这种方式，我们只会使用`h()`的结果来更新我们的背景节点。
- 协调器性能，允许为挂载/卸载等提供优化路径。
- 一些变更，如移除`px`后缀、`forwardRef`，以及终止IE11支持。
- 在props中保留ref。
- 解决事件/子元素差异对比的bug。

这些是我们对v11的初始目标，但在进行的过程中，我们意识到许多变更实际上并不是破坏性的，可以以非破坏性的方式直接在v10中发布。只有第三点，移除`px`后缀并直接在props中传递`ref`以及放弃IE11支持，属于破坏性变更类别。我们选择在稳定的v10版本线中发布其他功能，这允许任何Preact用户立即受益，而不必更改他们的代码。

与我们最初计划v11时相比，Preact现在拥有更大的用户群。它在许多大小公司的关键任务软件中得到广泛使用。我们真的希望确保我们可能引入的任何破坏性变更绝对值得将整个生态系统迁移到它的成本。

当我们在[实验](https://github.com/preactjs/preact/tree/v11)一种新型的差异对比方法，称为[斜对比(skew based diffing)](https://github.com/preactjs/preact/pull/3388)时，我们看到了实际的性能改进，同时它也修复了许多长期存在的bug。随着时间的推移，我们在Preact 11的这些实验上投入了更多时间，我们开始注意到，我们正在实现的改进不需要是Preact 11的专属。

## 发布

自上述Preact 11议题以来，已经有18个(!!)Preact X的次要版本。
其中许多直接受到了Preact 11工作的启发。让我们回顾一些并看看其影响。

### 10.5.0

引入了[恢复式水合(resumed hydration)](https://github.com/preactjs/preact/pull/2754)——这个功能基本上允许在组件树的重新水合过程中进行暂停。这意味着，例如在以下组件树中，我们将重新水合并使`Header`变得可交互，而`LazyArticleHeader`会暂停，同时服务器渲染的DOM将保留在屏幕上。当懒加载完成后，我们将继续重新水合，我们的`Header`和`LazyArticleHeader`可以交互，而我们的`LazyContents`解析。这是一个相当强大的功能，可以使你最重要的内容变得可交互，同时不会使初始包的大小/下载大小过载。

```jsx
const App = () => {
  return (
    <>
      <Header>
      <main>
        <Suspense>
          <LazyArticleHeader />
          <Suspense>
            <article>
              <LazyContents />
            </article>
          </Suspense>
        </Suspense>
      </main>
    </>
  )
}
```

### 10.8.0

在10.8.0中，我们引入了[状态沉降(state settling)](https://github.com/preactjs/preact/pull/3553)，这确保了如果组件在渲染期间更新钩子状态，我们会捕获到这一点，取消之前的效果并继续渲染。我们当然必须确保这不会循环，这个功能减少了因渲染中状态调用而排队的渲染数量，这个功能还增加了我们与React生态系统的兼容性，因为很多库依赖于效果不会因渲染中的状态更新而被多次调用。

### 10.11.0

经过大量研究，我们找到了一种将[useId](https://github.com/preactjs/preact/pull/3583)引入Preact的方法，这需要大量研究，来找出如何为给定的树结构添加唯一值。我们的一位维护者当时写了关于[我们的研究](https://www.jovidecroock.com/blog/preact-use-id)的文章，从那时起我们一直在迭代，尽量使其尽可能避免冲突...

### 10.15.0

我们发现，通过重新渲染导致多个新组件重新渲染可能会导致我们的`rerenderQueue`顺序错乱，这可能导致我们的(上下文)更新传播到后来会再次用过时值渲染的组件，你可以查看[提交信息](https://github.com/preactjs/preact/commit/672782adbf9ccefa7a4d7c175f0adf8580f73c92)获取更详细的解释！这样做既批处理了这些更新，也增加了我们与React库的一致性。

### 10.16.0

在我们对v11的研究中，我们深入研究了子节点差异，因为我们知道当前算法在某些情况下会有不足之处，列举一些这些问题：

- [在另一个元素之前移除一个元素会导致重新插入](https://github.com/preactjs/preact/issues/3973)
- [移除多个子节点时重新插入](https://github.com/preactjs/preact/issues/2622)
- [有键节点不必要的卸载](https://github.com/preactjs/preact/issues/2783)

这些并非全都导致错误状态，有些只是意味着性能下降...当我们发现可以将基于斜对比的差异算法移植到Preact X时，我们非常兴奋，不仅可以修复许多情况，还可以看到这个算法在实际环境中的表现！回想起来，它表现得很好，有时我希望我们有好的测试环境先运行这些，而不是由我们的社区来报告问题。在这里我想借此机会感谢大家，通过总是提交周到的问题和复现案例来帮助我们，你们都是最棒的！

### 10.19.0

在10.19.0中，Marvin将他在[fresh](https://fresh.deno.dev/)中的研究应用，添加了[预编译JSX函数](https://github.com/preactjs/preact/pull/4177)，这基本上允许你在转译过程中预编译你的组件，当render-to-string运行时，我们只需要连接字符串，而不必为整个VNode树分配内存。目前这个转换是Deno独有的，但总体概念已经存在于Preact中！

### 10.20.2

我们遇到了一些问题，事件可能会冒泡到新插入的VNode，这会导致不希望的行为，这是通过[添加事件时钟](https://github.com/preactjs/preact/pull/4322)解决的。在以下场景中，你点击按钮设置状态，浏览器将事件冒泡与微任务穿插，这也是Preact用来调度更新的方式。这种组合意味着Preact将更新UI，意味着`<div>`将获得那个`onClick`处理器，我们将冒泡到它并再次调用`click`，立即再次切换这个状态。

```jsx
const App = () => {
  const [toggled, setToggled] = useState(false);

  return toggled ? (
    <div onClick={() => setToggled(false)}>
      <span>clear</span>
    </div> 
  ) : (
    <div>
      <button
        onClick={() => setToggled(true)}
      >toggle on</button>
    </div>
  )
}
```

## 稳定性

上面是我们社区_在不引入破坏性变更的情况下_接收到的一些精选发布，但还有更多...添加一个新的主要版本总是会让部分社区落后，我们不想这样做。如果我们看看Preact 8发布线，我们可以看到过去一周仍有10万次下载，最后一个8.x版本是5年前的，这只是表明部分社区会被落下。

稳定性很重要，作为Preact团队，我们喜欢稳定性。我们实际上在其他生态系统项目上发布了多个主要功能：

- [Signals](https://github.com/preactjs/signals)
- [异步渲染](https://github.com/preactjs/preact-render-to-string/pull/333)
- [流式渲染](https://github.com/preactjs/preact-render-to-string/pull/354)
- [Prefresh](https://github.com/preactjs/prefresh)
- [带有预渲染的vite预设](https://github.com/preactjs/preset-vite#prerendering-configuration)
- [新的异步路由器](https://github.com/preactjs/preact-iso)
- [Create Preact](https://github.com/preactjs/create-preact)

我们重视我们的生态系统，我们重视通过[`options API`](https://marvinh.dev/blog/preact-options/)构建的扩展，这是我们不想引入这些破坏性变更的主要驱动力之一，而是允许你们所有人从我们的研究中受益，而不需要痛苦的迁移路径。

这并不意味着Preact 11不会发生，但它可能不是我们最初认为的那样。相反，我们可能只是放弃IE11支持并给你那些性能改进，同时给你Preact X的稳定性。还有许多其他想法，我们对更广泛的Preact体验非常感兴趣，特别是在提供内置路由等功能的元框架的上下文中。我们正在我们的vite预设以及[Fresh](https://fresh.deno.dev/)中探索这个角度，以便对Preact优先的元框架应该是什么样子有一个很好的感觉。 