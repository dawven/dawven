---
title: 30 分钟精通 React 新特性 React Hooks
date: 2019-02-15 13:54:21
type: "tags"
tags:
- react
---
你还在为该使用无状态组件（Function）还是有状态组件（Class）而烦恼吗？——拥有了 hooks，你再也不需要写 Class 了，你的所有组件都将是 Function。

你还在为搞不清使用哪个生命周期钩子函数而日夜难眠吗？——拥有了 Hooks，生命周期钩子函数可以先丢一边了。

你在还在为组件中的 this 指向而晕头转向吗？——既然 Class 都丢掉了，哪里还有 this？你的人生第一次不再需要面对 this。

这样看来，说 React Hooks 是今年最劲爆的新特性真的毫不夸张。如果你也对 react 感兴趣，或者正在使用 react 进行项目开发，答应我，请一定抽出至少 30 分钟的时间来阅读本文好吗？所有你需要了解的 React Hooks 的知识点，本文都涉及到了，相信完整读完后你一定会有所收获。
<!-- more -->

**一个最简单的 Hooks**

首先让我们看一下一个简单的有状态组件：

![](https://pic4.zhimg.com/v2-84729d0d4f171dec2c1153eaa948b1a7_b.jpg)![](https://pic4.zhimg.com/v2-84729d0d4f171dec2c1153eaa948b1a7_r.jpg)

我们再来看一下使用 hooks 后的版本：

![](https://pic2.zhimg.com/v2-af0547ea65a4647e35a69345e868f4e1_b.jpg)![](https://pic2.zhimg.com/v2-af0547ea65a4647e35a69345e868f4e1_r.jpg)

是不是简单多了！可以看到， Example 变成了一个函数，但这个函数却有自己的状态（count），同时它还可以更新自己的状态（setCount）。这个函数之所以这么了不得，就是因为它注入了一个 hook-- useState，就是这个 hook 让我们的函数变成了一个有状态的函数。

除了 useState 这个 hook 外，还有很多别的 hook，比如 useEffect 提供了类似于 componentDidMount 等生命周期钩子的功能， useContext 提供了上下文（context）的功能等等。

Hooks 本质上就是一类特殊的函数，它们可以为你的函数型组件（function component）注入一些特殊的功能。咦？这听起来有点像被诟病的 Mixins 啊？难道是 Mixins 要在 react 中死灰复燃了吗？当然不会了，等会我们再来谈两者的区别。总而言之，这些 hooks 的目标就是让你不再写 class，让 function 一统江湖。

**React 为什么要搞一个 Hooks？**

**想要复用一个有状态的组件太麻烦了！**

我们都知道 react 都核心思想就是，将一个页面拆成一堆独立的，可复用的组件，并且用自上而下的单向数据流的形式将这些组件串联起来。但假如你在大型的工作项目中用 react，你会发现你的项目中实际上很多 react 组件冗长且难以复用。尤其是那些写成 class 的组件，它们本身包含了状态（state），所以复用这类组件就变得很麻烦。

那之前，官方推荐怎么解决这个问题呢？答案是：渲染属性（Render Props）和高阶组件（Higher-Order Components）。我们可以稍微跑下题简单看一下这两种模式。

渲染属性指的是使用一个值为函数的 prop 来传递需要动态渲染的 nodes 或组件。如下面的代码可以看到我们的 DataProvider 组件包含了所有跟状态相关的代码，而 Cat 组件则可以是一个单纯的展示型组件，这样一来 DataProvider 就可以单独复用了。

![](https://pic2.zhimg.com/v2-9e7a14496cb9ddf811ca86f7e458cdfd_b.jpg)![](https://pic2.zhimg.com/v2-9e7a14496cb9ddf811ca86f7e458cdfd_r.jpg)

虽然这个模式叫 Render Props，但不是说非用一个叫 render 的 props 不可，习惯上大家更常写成下面这种：

![](https://pic1.zhimg.com/v2-979d7089545e3e8d50da851e3346e9c8_b.jpg)![](https://pic1.zhimg.com/v2-979d7089545e3e8d50da851e3346e9c8_r.jpg)

高阶组件这个概念就更好理解了，说白了就是一个函数接受一个组件作为参数，经过一系列加工后，最后返回一个新的组件。看下面的代码示例， withUser 函数就是一个高阶组件，它返回了一个新的组件，这个组件具有了它提供的获取用户信息的功能。

![](https://pic3.zhimg.com/v2-14889fe5d382f69d24e6bb308517698a_b.jpg)![](https://pic3.zhimg.com/80/v2-14889fe5d382f69d24e6bb308517698a_hd.jpg)

以上这两种模式看上去都挺不错的，很多库也运用了这种模式，比如我们常用的 React Router。但我们仔细看这两种模式，会发现它们会增加我们代码的层级关系。最直观的体现，打开 devtool 看看你的组件层级嵌套是不是很夸张吧。这时候再回过头看我们上一节给出的 hooks 例子，是不是简洁多了，没有多余的层级嵌套。把各种想要的功能写成一个一个可复用的自定义 hook，当你的组件想用什么功能时，直接在组件里调用这个 hook 即可。

![](https://pic2.zhimg.com/v2-cb35c443ea400e01884b23a61c0cb095_b.jpg)![](https://pic2.zhimg.com/80/v2-cb35c443ea400e01884b23a61c0cb095_hd.jpg)

**生命周期钩子函数里的逻辑太乱了吧！**

我们通常希望一个函数只做一件事情，但我们的生命周期钩子函数里通常同时做了很多事情。比如我们需要在 componentDidMount 中发起 ajax 请求获取数据，绑定一些事件监听等等。同时，有时候我们还需要在 componentDidUpdate 做一遍同样的事情。当项目变复杂后，这一块的代码也变得不那么直观。

**classes 真的太让人困惑了！**

我们用 class 来创建 react 组件时，还有一件很麻烦的事情，就是 this 的指向问题。为了保证 this 的指向正确，我们要经常写这样的代码： this.handleClick=this.handleClick.bind(this)，或者是这样的代码： <buttononClick={()=>this.handleClick(e)}>。一旦我们不小心忘了绑定 this，各种 bug 就随之而来，很麻烦。

还有一件让我很苦恼的事情。我在之前的 react 系列文章当中曾经说过，尽可能把你的组件写成无状态组件的形式，因为它们更方便复用，可独立测试。然而很多时候，我们用 function 写了一个简洁完美的无状态组件，后来因为需求变动这个组件必须得有自己的 state，我们又得很麻烦的把 function 改成 class。

在这样的背景下，Hooks 便横空出世了！

**什么是 State Hooks？**

回到一开始我们用的例子，我们分解来看到底 state hooks 做了什么：

![](https://pic1.zhimg.com/v2-127b897e02ea1a4f9b38a7ffc19bd2d4_b.jpg)![](https://pic1.zhimg.com/80/v2-127b897e02ea1a4f9b38a7ffc19bd2d4_hd.jpg)

**声明一个状态变量**

![](https://pic2.zhimg.com/v2-df70e8eee3645dc183638305e6774e99_b.jpg)![](https://pic2.zhimg.com/80/v2-df70e8eee3645dc183638305e6774e99_hd.jpg)

useState 是 react 自带的一个 hook 函数，它的作用就是用来声明状态变量。 useState 这个函数接收的参数是我们的状态初始值（initial state），它返回了一个数组，这个数组的第 [0] 项是当前当前的状态值，第 [1] 项是可以改变状态值的方法函数。

所以我们做的事情其实就是，声明了一个状态变量 count，把它的初始值设为 0，同时提供了一个可以更改 count 的函数 setCount。

上面这种表达形式，是借用了 es6 的数组解构（array destructuring），它可以让我们的代码看起来更简洁。不清楚这种用法的可以先去看下我的这篇文章：30 分钟掌握 ES6/ES2015 核心内容（上）。

如果不用数组解构的话，可以写成下面这样。实际上数组解构是一件开销很大的事情，用下面这种写法，或者改用对象解构，性能会有很大的提升。具体可以去这篇文章的分析：Array destructuring for multi-value returns (in light of React hooks)，这里不详细展开，我们就按照官方推荐使用数组解构就好。

![](https://pic4.zhimg.com/v2-78cce52f042a2b33fcb15a371a2369c7_b.jpg)![](https://pic4.zhimg.com/80/v2-78cce52f042a2b33fcb15a371a2369c7_hd.jpg)

**读取状态值**

![](https://pic2.zhimg.com/v2-5b6cb263680c77038a660c4dd414b00d_b.jpg)![](https://pic2.zhimg.com/80/v2-5b6cb263680c77038a660c4dd414b00d_hd.jpg)

是不是超简单？因为我们的状态 count 就是一个单纯的变量而已，我们再也不需要写成 {this.state.count} 这样了。

**更新状态**

![](https://pic4.zhimg.com/v2-71701789d4e408c732f5cfa6621bcf2f_b.jpg)![](https://pic4.zhimg.com/80/v2-71701789d4e408c732f5cfa6621bcf2f_hd.jpg)

当用户点击按钮时，我们调用 setCount 函数，这个函数接收的参数是修改过的新状态值。接下来的事情就交给 react 了，react 将会重新渲染我们的 Example 组件，并且使用的是更新后的新的状态，即 count=1。这里我们要停下来思考一下，Example 本质上也是一个普通的函数，为什么它可以记住之前的状态？

**一个至关重要的问题**

这里我们就发现了问题，通常来说我们在一个函数中声明的变量，当函数运行完成后，这个变量也就销毁了（这里我们先不考虑闭包等情况），比如考虑下面的例子：

![](https://pic3.zhimg.com/v2-881704b6abed6a57eca5b4afe394b512_b.jpg)![](https://pic3.zhimg.com/80/v2-881704b6abed6a57eca5b4afe394b512_hd.jpg)

不管我们反复调用 add 函数多少次，结果都是 1。因为每一次我们调用 add 时，result 变量都是从初始值 0 开始的。那为什么上面的 Example 函数每次执行的时候，都是拿的上一次执行完的状态值作为初始值？答案是：是 react 帮我们记住的。至于 react 是用什么机制记住的，我们可以再思考一下。

**假如一个组件有多个状态值怎么办？**

首先，useState 是可以多次调用的，所以我们完全可以这样写：

![](https://pic4.zhimg.com/v2-39f11eafc60a053b2ced6dc6da58d14f_b.jpg)![](https://pic4.zhimg.com/80/v2-39f11eafc60a053b2ced6dc6da58d14f_hd.jpg)

其次，useState 接收的初始值没有规定一定要是 string/number/boolean 这种简单数据类型，它完全可以接收对象或者数组作为参数。唯一需要注意的点是，之前我们的 this.setState 做的是合并状态后返回一个新状态，而 useState 是直接替换老状态后返回新状态。最后，react 也给我们提供了一个 useReducer 的 hook，如果你更喜欢 redux 式的状态管理方案的话。

从 ExampleWithManyStates 函数我们可以看到，useState 无论调用多少次，相互之间是独立的。这一点至关重要。为什么这么说呢？

其实我们看 hook 的 “形态”，有点类似之前被官方否定掉的 Mixins 这种方案，都是提供一种“插拔式的功能注入” 的能力。而 mixins 之所以被否定，是因为 Mixins 机制是让多个 Mixins 共享一个对象的数据空间，这样就很难确保不同 Mixins 依赖的状态不发生冲突。

而现在我们的 hook，一方面它是直接用在 function 当中，而不是 class；另一方面每一个 hook 都是相互独立的，不同组件调用同一个 hook 也能保证各自状态的独立性。这就是两者的本质区别了。

**react 是怎么保证多个 useState 的相互独立的？**

还是看上面给出的 ExampleWithManyStates 例子，我们调用了三次 useState，每次我们传的参数只是一个值（如 42，‘banana’），我们根本没有告诉 react 这些值对应的 key 是哪个，那 react 是怎么保证这三个 useState 找到它对应的 state 呢？

答案是，react 是根据 useState 出现的顺序来定的。我们具体来看一下：

![](https://pic2.zhimg.com/v2-fbbaeb729e5cd97f7f38e91ad051dab5_b.jpg)![](https://pic2.zhimg.com/80/v2-fbbaeb729e5cd97f7f38e91ad051dab5_hd.jpg)

假如我们改一下代码：

![](https://pic2.zhimg.com/v2-90bd69323beee6dcd34863eea4d6a259_b.jpg)![](https://pic2.zhimg.com/80/v2-90bd69323beee6dcd34863eea4d6a259_hd.jpg)

这样一来，

![](https://pic2.zhimg.com/v2-fe5cb53c67c27cfd66ef5321419164f1_b.jpg)![](https://pic2.zhimg.com/80/v2-fe5cb53c67c27cfd66ef5321419164f1_hd.jpg)

鉴于此，react 规定我们必须把 hooks 写在函数的最外层，不能写在 ifelse 等条件语句当中，来确保 hooks 的执行顺序一致。

**什么是 Effect Hooks?**

我们在上一节的例子中增加一个新功能：

![](https://pic1.zhimg.com/v2-362f97c81dc6b4716cafe97aa83683d8_b.jpg)![](https://pic1.zhimg.com/80/v2-362f97c81dc6b4716cafe97aa83683d8_hd.jpg)

我们对比着看一下，如果没有 hooks，我们会怎么写？

![](https://pic1.zhimg.com/v2-8a45d062cfbad77129c56fa664bd2548_b.jpg)![](https://pic1.zhimg.com/80/v2-8a45d062cfbad77129c56fa664bd2548_hd.jpg)

我们写的有状态组件，通常会产生很多的副作用（side effect），比如发起 ajax 请求获取数据，添加一些监听的注册和取消注册，手动修改 dom 等等。我们之前都把这些副作用的函数写在生命周期函数钩子里，比如 componentDidMount，componentDidUpdate 和 componentWillUnmount。而现在的 useEffect 就相当与这些声明周期函数钩子的集合体。它以一抵三。

同时，由于前文所说 hooks 可以反复多次使用，相互独立。所以我们合理的做法是，给每一个副作用一个单独的 useEffect 钩子。这样一来，这些副作用不再一股脑堆在生命周期钩子里，代码变得更加清晰。

useEffect 做了什么？

我们再梳理一遍下面代码的逻辑：

![](https://pic1.zhimg.com/v2-dfd03c7dc16f93de8b3927bc9c39e6bc_b.jpg)![](https://pic1.zhimg.com/80/v2-dfd03c7dc16f93de8b3927bc9c39e6bc_hd.jpg)

首先，我们声明了一个状态变量 count，将它的初始值设为 0。然后我们告诉 react，我们的这个组件有一个副作用。我们给 useEffecthook 传了一个匿名函数，这个匿名函数就是我们的副作用。在这个例子里，我们的副作用是调用 browser API 来修改文档标题。当 react 要渲染我们的组件时，它会先记住我们用到的副作用。等 react 更新了 DOM 之后，它再依次执行我们定义的副作用函数。

这里要注意几点：

第一，react 首次渲染和之后的每次渲染都会调用一遍传给 useEffect 的函数。而之前我们要用两个声明周期函数来分别表示首次渲染（componentDidMount），和之后的更新导致的重新渲染（componentDidUpdate）。

第二，useEffect 中定义的副作用函数的执行不会阻碍浏览器更新视图，也就是说这些函数是异步执行的，而之前的 componentDidMount 或 componentDidUpdate 中的代码则是同步执行的。这种安排对大多数副作用说都是合理的，但有的情况除外，比如我们有时候需要先根据 DOM 计算出某个元素的尺寸再重新渲染，这时候我们希望这次重新渲染是同步发生的，也就是说它会在浏览器真的去绘制这个页面前发生。

useEffect 怎么解绑一些副作用

这种场景很常见，当我们在 componentDidMount 里添加了一个注册，我们得马上在 componentWillUnmount 中，也就是组件被注销之前清除掉我们添加的注册，否则内存泄漏的问题就出现了。

怎么清除呢？让我们传给 useEffect 的副作用函数返回一个新的函数即可。这个新的函数将会在组件下一次重新渲染之后执行。这种模式在一些 pubsub 模式的实现中很常见。看下面的例子：

![](https://pic4.zhimg.com/v2-4554ca1100a0f4c58eff1c2e7984cfa3_b.jpg)![](https://pic4.zhimg.com/80/v2-4554ca1100a0f4c58eff1c2e7984cfa3_hd.jpg)

这里有一个点需要重视！这种解绑的模式跟 componentWillUnmount 不一样。componentWillUnmount 只会在组件被销毁前执行一次而已，而 useEffect 里的函数，每次组件渲染后都会执行一遍，包括副作用函数返回的这个清理函数也会重新执行一遍。所以我们一起来看一下下面这个问题。

为什么要让副作用函数每次组件更新都执行一遍？

我们先看以前的模式：

![](https://pic2.zhimg.com/v2-e9a49b6cfdb730dd8106feb1586161fd_b.jpg)![](https://pic2.zhimg.com/80/v2-e9a49b6cfdb730dd8106feb1586161fd_hd.jpg)

很清除，我们在 componentDidMount 注册，再在 componentWillUnmount 清除注册。但假如这时候 props.friend.id 变了怎么办？我们不得不再添加一个 componentDidUpdate 来处理这种情况：

![](https://pic1.zhimg.com/v2-b857588311aa470ff47a27d87aa89efc_b.jpg)![](https://pic1.zhimg.com/80/v2-b857588311aa470ff47a27d87aa89efc_hd.jpg)

看到了吗？很繁琐，而我们但 useEffect 则没这个问题，因为它在每次组件更新后都会重新执行一遍。所以代码的执行顺序是这样的：

*   页面首次渲染
*   替 friend.id=1 的朋友注册
*   突然 friend.id 变成了 2
*   页面重新渲染
*   清除 friend.id=1 的绑定
*   替 friend.id=2 的朋友注册
*   ...

怎么跳过一些不必要的副作用函数

按照上一节的思路，每次重新渲染都要执行一遍这些副作用函数，显然是不经济的。怎么跳过一些不必要的计算呢？我们只需要给 useEffect 传第二个参数即可。用第二个参数来告诉 react 只有当这个参数的值发生改变时，才执行我们传的副作用函数（第一个参数）。

![](https://pic3.zhimg.com/v2-fc01410792d0217d9f3a07c7b2bcf686_b.jpg)![](https://pic3.zhimg.com/80/v2-fc01410792d0217d9f3a07c7b2bcf686_hd.jpg)

当我们第二个参数传一个空数组 [] 时，其实就相当于只在首次渲染的时候执行。也就是 componentDidMount 加 componentWillUnmount 的模式。不过这种用法可能带来 bug，少用。

**还有哪些自带的 Effect Hooks?**

除了上文重点介绍的 useState 和 useEffect，react 还给我们提供来很多有用的 hooks：

*   useContext
*   useReducer
*   useCallback
*   useMemo
*   useRef
*   useImperativeMethods
*   useMutationEffect
*   useLayoutEffect

我不再一一介绍，大家自行去查阅官方文档。

怎么写自定义的 Effect Hooks?

为什么要自己去写一个 Effect Hooks? 这样我们才能把可以复用的逻辑抽离出来，变成一个个可以随意插拔的 “插销”，哪个组件要用来，我就插进哪个组件里，so easy！看一个完整的例子，你就明白了。

比如我们可以把上面写的 FriendStatus 组件中判断朋友是否在线的功能抽出来，新建一个 useFriendStatus 的 hook 专门用来判断某个 id 是否在线。

![](https://pic3.zhimg.com/v2-0d010c52670f62e6a0c442ce1d9b27f6_b.jpg)![](https://pic3.zhimg.com/80/v2-0d010c52670f62e6a0c442ce1d9b27f6_hd.jpg)

这时候 FriendStatus 组件就可以简写为：

![](https://pic3.zhimg.com/v2-8a1a1c7ba219d546a5f9dd67c7151406_b.jpg)![](https://pic3.zhimg.com/80/v2-8a1a1c7ba219d546a5f9dd67c7151406_hd.jpg)

简直 Perfect！假如这个时候我们又有一个朋友列表也需要显示是否在线的信息：

![](https://pic4.zhimg.com/v2-363ae531bb66b1affc8331174d91b0b3_b.jpg)![](https://pic4.zhimg.com/80/v2-363ae531bb66b1affc8331174d91b0b3_hd.jpg)

简直 Fabulous!

作者：zach5078

[http://segmentfault.com/a/1190000016950339](http://link.zhihu.com/?target=http%3A//segmentfault.com/a/1190000016950339)

写下你的评论...

那状态管理的那些东西究竟还用不用啊？redux…
肯定还要用呀，useReducer 只能管理自己的状态。两个组件之间的状态还是无能为力，可以试试 unstate 这个状态库
做了几个内部系统，感觉没多少可封装可复用的无状态组件，毕竟现在的三方 UI 库很成熟了。

简单易懂 hook 出了那么我楞是没看懂是什么.。
