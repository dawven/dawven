---
title: React 中 setState 使用详解
date: 2018-07-21 10:54:21
tags: react
---

React 中 setState 使用的[注意事项](http://www.php.cn/code/10182.html)有哪些，下面就是实战案例，一起来看一下。

## 抛出问题

```javascript
class Example extends Component {
  contructor () {
    super()
    this.state = {
      value: 0,
      index: 0
    }
  }
  componentDidMount () {
    this.setState({value: this.state.value + 1})
    console.log(this.state.value) // 第一次输出
    this.setState({value: this.state.value + 1})
    console.log(this.state.value) // 第二次输出
    setTimeout(() => {
      this.setState({value: this.state.value + 1})
      console.log(this.state.value) // 第三次输出
      this.setState({value: this.state.value + 1})
      console.log(this.state.value) // 第四次输出
    }, 0)；
        this.refs.button.addEventListener('click', this.click)
  }
  click = () => {
    this.setState({value: this.state.index + 1})
    this.setState({value: this.state.index + 1})
  }
  render () {
    return (
      <p><span>value: {this.state.value}index: {this.props.index}</span>
        <button ref="button" onClick={this.click}>点击</button>
      </p>
    )
  }
}
```
<!-- more -->
*   这四次输出，按常理来说分别是: 1，2，3，4。但是，实际输出为: 0, 0, 2, 3

## setState 的注意点

1.  setState 不会立刻改变 React 组件中 state 的值（即 setState 是异步更新）

*   setState 通过一个队列机制实现 state 更新；

*   当执行 setState 时，会将需要更新的 state 合并后放入状态队列，而不会立即更新，队列可以高效的批量更新 state；

*   通过 this.state 直接修改的值，state 不会放入状态队列，当下次调用 setState 并对状态队列进行合并时，会忽略之前直接被修改的 state.

3.  setState 通过引发一次组件的更新过程来引发重新绘制

*   此处重绘指的就是引起 React 的更新[生命周期](http://www.php.cn/php/php-tp-being.html)函数 4 个函数：

*   shouldComponentUpdate（被调用时 this.state 没有更新；如果返回了 false，生命周期被中断，虽然不调用之后的函数了，但是 state 仍然会被更新）

*   componentWillUpdate（被调用时 this.state 没有更新）

*   render（被调用时 this.state 得到更新）

*   componentDidUpdate

5.  多个相邻的 state 的修改可能会合并到一起一次执行

<pre> this.setState({name: 'Pororo'})
 this.setState({age: 20})</pre>

*   等同于

<pre> this.setState({name: 'Pororo'，age: 20})</pre>

*   上面两块代码的效果是一样的。如果每次调用都引发一次生命周期更新，那性能就会消耗很大了。所以，React 会将多个 this.setState 产生的修改放进一个队列里，等差不多的时候就会引发一次生命周期更新。

## 问题分析

*   **对于前两次 setState：**

<pre>this.setState({value: this.state.val + 1});
console.log(this.state.value); // 第一次输出
this.setState({value: this.state.val + 1});
console.log(this.state.value); // 第二次输出</pre>

*   由于 setState 不会立即改变 React 组件中 state 的值，所以两次 setState 中 this.state.value 都是同一个值 0，故而，这两次输出都是 0。因而 value 只被加 1。

*   既然这样，那么是不是可以直接操作 this.state 呢？比如：`this.state.value=this.state.value+1;`

*   这样的确可以修改 this.state.value 的状态但是却不可以引发重复渲染。

*   所以，就必须通过 React 设定的 setState 函数去改变 this.state，从而引发重新渲染。

*   **setTimeout 里面的两次 setState：**

<pre>setTimeout(() => {
  this.setState({value: this.state.value + 1})
  console.log(this.state.value) // 第三次输出
  this.setState({value: this.state.value + 1})
  console.log(this.state.value) // 第四次输出
}, 0)；</pre>

*   这两次 this.state 的值同步更新了;

*   **同步更新：**是由 React 引发的[事件处理](http://www.php.cn/code/5688.html)（比如：onClick 引发的事件处理），调用 setState 会异步更新 this.state；

*   **异步更新：**除此之外的 setState 调用会同步执行 this.setState。 “除此之外” 指的是：绕过 React 通过 addEventListener 直接添加的事件处理函数和 setTimeout/setInterval 产生的异步调用。

*   this.setState 更新机制图解：

![](http://img.php.cn/upload/article/000/061/021/e98f921b8f164094719418c36ddc190f-0.png)

*   每次 setState 产生新的 state 会依次被存入一个队列，然后会根据 isBathingUpdates 变量判断是直接更新 this.state 还是放进 dirtyComponent 里回头再说。

*   isBatchingUpdates 默认是 false，也就表示 setState 会同步更新 this.state。

*   但是，当 React 在调用事件处理函数之前就会调用 batchedUpdates，这个函数会把 isBatchingUpdates 修改为 true，造成的后果就是由 React 控制的事件处理过程 setState 不会同步更新 this.state。

## 同步更新（函数式 setState）

1.  如果 this.setState 的参数不是一个[对象](http://www.php.cn/wiki/60.html)而是一个函数时，这个函数会接收到两个参数，第一个是当前的 state 值，第二个是当前的 props，这个函数应该返回一个对象，这个对象代表想要对 this.state 的更改；

2.  换句话说，之前你想给 this.setState 传递什么对象参数，在这种函数里就返回什么对象。不过，计算这个对象的方法有些改变，不再依赖于 this.state，而是依赖于输入参数 state。

<pre>function increment(state, props) {
  return {count: state.count + 1};
}
function incrementMultiple() {
  this.setState(increment);
  this.setState(increment);
  this.setState(increment);
}</pre>

*   假如当前 this.state.count 的值是 0，第一次调用 this.setState(increment)，传给 increment 的 state 参数是 0，第二调用时，state 参数是 1，第三次调用是，参数是 2，最终 incrementMultiple 让 this.state.count 变成了 3。

*   对于多次调用函数式 setState 的情况，React 会保证调用每次 increment 时，state 都已经合并了之前的状态修改结果。

> 要注意的是，在 increment 函数被调用时，this.state 并没有被改变，依然，要等到 render 函数被重新执行时（或者 shouldComponentUpdate 函数返回 false 之后）才被改变。

## 同步异步 setState 的用法混合

<pre>function incrementMultiple() {
  this.setState(increment);
  this.setState(increment);
  this.setState({count: this.state.count + 1});
  this.setState(increment);
}</pre>

*   在几个函数式 setState 调用中插入一个传统式 setState 调用，最后得到的结果是让 this.state.count 增加了 2，而不是增加 4。

*   这是因为 React 会依次合并所有 setState 产生的效果，虽然前两个函数式 setState 调用产生的效果是 count 加 2，但是中间出现一个传统式 setState 调用，一下子强行把积攒的效果清空，用 count 加 1 取代。

*   所以，传统式 setState 与函数式 setState 一定不要混用。

以上就是 React 中 setState 使用详解的详细内容，更多请关注 php 中文网其它相关文章！
本文原创发布 php 中文网 ，转载请注明出处，感谢您的尊重！