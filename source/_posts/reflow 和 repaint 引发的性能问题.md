---
title: reflow 和 repaint 引发的性能问题
date: 2018-06-11 10:54:21
type: "tags"
tags:
- 面试题
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://juejin.im/post/5a9372895188257a6b06132e

> reflow 和 repaint 在 pc 端只要不是怀有明知山有虎，偏向虎山行的心态写代码，这两货几乎不会引发性能问题， 但是移动端的渲染能力和 pc 端差了不止一个大截，一个不小心 reflow 和 repaint 就成了移动端的 “性能杀手”。所以了解 reflow 和 repaint 也是很有必要的，在考量页面性能的时候分析 reflow 和 repaint 也算是一个切入点。
<!-- more -->
## 是什么

* * *

`reflow` 回流，或者叫重排都可以。回流 (reflow) 这个名词指的是浏览器为了重新渲染部分或全部的文档而重新计算文档中元素的位置和几何结构的过程。

简单来说就是当页面布局或者几何属性改变时就需要 reflow。

在一个页面中至少在页面刚加载的时候有一次 reflow，在 reflow 的过程中浏览器会将 render tree 中受影响的节点失效，再重新构建 render tree，有时候，即使仅仅回流一个单一的元素，也可能要求它的父元素以及任何跟随它的元素也产生回流

`repaint`重绘，当页面中的元素只需要更新样式风格不影响布局，比如更换背景色 background-color，这个过程就是重绘。

## 如何触发

* * *

### reflow

从 reflow 的定义中就可以听出一些来，元素的布局和几何属性改变时就会触发 reflow。主要有这些属性：

*   **盒模型**相关的属性: width，height，margin，display，border，etc

*   **定位属性及浮动**相关的属性: top,position,float，etc

*   改变节点内部**文字结构**也会触发回流: text-align, overflow, font-size, line-height, vertival-align，etc

除开这三大类的属性变动会触发 reflow，以下情况也会触发：

*   调整窗口大小
*   样式表变动
*   元素内容变化，尤其是输入控件
*   dom 操作
*   css 伪类激活
*   计算元素的 offsetWidth、offsetHeight、clientWidth、clientHeight、width、height、scrollTop、scrollHeight

### repaint

页面中的元素更新样式风格相关的属性时就会触发重绘，如 background，color，cursor，visibility，etc

**注意**：由页面的渲染过程可知，reflow 必将会引起 repaint，而 repaint 不一定会引起 reflow

了解有哪些属性值改变会触发回流或者重绘点击[这里](https://link.juejin.im?target=https%3A%2F%2Fcsstriggers.com%2F)

## 聪明的浏览器

* * *

设想一个这样的场景，我们需要在一个循环中不断修改一个 dom 节点的位置或者是内容

```
   document.addEventListener('DOMContentLoaded', function () {
    var date = new Date();
    for (var i = 0; i < 70000; i++) {
        var tmpNode = document.createElement("div");
        tmpNode.innerHTML = "test" + i;
        document.body.appendChild(tmpNode);
    }
    console.log(new Date() - date);
});

```

这里多次测量消耗时间大概在 500ms（运行环境均为 pc 端，小霸王笔记本）。看到这个结果可能就有疑问了，这里有 **70000** 次内容的修改，就有 **70000**reflow 操作，也就用了 500ms 的时间（归功于迟缓的 dom 操作），说好的 reflow 消耗性能呢。

![](https://user-gold-cdn.xitu.io/2018/2/26/161d0d3ec84bf83e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

其实在这个过程中，浏览器为了防止我们犯二把多次 reflow 操作放在循环中而引发浏览器假死，做了一个聪明的小动作。它会收集 reflow 操作到缓存队列中直到一定的规模或者过了特定的时间，再一次性地 flush 队列，反馈到 render tree 中，这样就将多次的 reflow 操作减少为少量的 reflow。但是这样的小动作带来了另外一个问题，如果我们想要在一次 reflow 过后就获取元素变动过后的值呢？这个时候浏览器为了获取真实的值就不得不立即 flush 缓存的队列。这些值或方法包括：

*   offsetTop/Left/Width/Height
*   scrollTop/Left/Width/Height
*   clientTop/Left/Width/Height
*   getComputedStyle(), or currentStyle in IE

犯二代码如下：

```
        document.addEventListener('DOMContentLoaded', function () {
            var date = new Date();
            for (var i = 0; i < 70000; i++) {
                var tmpNode = document.createElement("div");
                tmpNode.innerHTML = "test" + i;
                document.body.offsetHeight; // 获取body的真实值
                document.body.appendChild(tmpNode);
            }
            console.log("speed time", new Date() - date);
        });

```

一般人应该不会去运行这种代码，如果你运行了的话，恭喜你的电脑 - 1s。但是如果没有衡量指标，优化性能也就无从谈起。

> “If you cannot measure it, you cannot improve it.” -Lord Kelvin

为了防止浏览器假死，把循环次数都改为 7000 次，得出的结果是（多次平均）：

*   获取了真实值的样例用时约 **18000**ms
*   没有获取真实值的样例用时约 **50**ms

通过这两个样例印证了浏览器确实有优化 reflow 的小动作，聪明的程序员不会依赖浏览器的优化策略，在日常开发中遇到 for 循环就应该慎重编写循环体内部的代码。

## 减少 reflow 和 repaint

* * *

如何减少 reflow 和 repaint 呢？回到定义去，`reflow在页面布局或者定位发生变化时才会发生`，从定义中我们至少可以得出两个优化思路

*   减少 reflow 操作
*   替代会触发回流的属性

### 减少 reflow 操作

其本质上为减少对 render tree 的操作。render tree 也就是渲染树，它的每个节点都是可见，且包含该节点的内容和对应的规则样式，这也是 render tree 和 dom 数最大的区别所在, 减少 reflow 操作，主旨是合并多个 reflow，最后再反馈到 render tree 中，诸如：

#### 1, 直接更改 classname

```
    // 不好的写法
    var left = 1;
    var top = 2;
    ele.style.left = left + "px";
    ele.style.top = top + "px";
    // 比较好的写法
    ele.className += " className1";

```

或者直接修改 cssText：

```
    ele.style.cssText += ";
    left: " + left + "px;
    top: " + top + "px;";

```

#### 2\. 让频繁 reflow 的元素 “离线”

*   使用 DocumentFragment 进行缓存操作, 引发一次回流和重绘；
*   使用 display:none，只引发两次回流和重绘；
*   使用 cloneNode(true or false) 和 replaceChild 技术，引发一次回流和重绘；

Dom 规定文档片段（document fragment）是一种 “轻量级” 的文档，可以包含和控制节点，但不会想完整的文档那样占用额外的资源。虽然不能把文档片段直接添加到文档中，但是可以将它作为一个 “仓库” 来使用，即可以在里面保存将来可能会添加到文档中的节点。 比如最开始的样例结合 DocumentFragment 就可以这样写：

```
    document.addEventListener('DOMContentLoaded', function () {
        var date = new Date(),
            fragment = document.createDocumentFragment();
        for (var i = 0; i < 7000; i++) {
            var tmpNode = document.createElement("div");
            tmpNode.innerHTML = "test" + i;
            fragment.appendChild(tmpNode);
        }
        document.body.appendChild(fragment);
        console.log("speed time", new Date() - date);
    });

```

将多个修改结果收纳到了 documentFragment 这个 “仓库” 中，这个过程并不会影响到 render tree，待循环完毕再将这个 “仓库” 的“存货”添加到 dom 上，以此达到减少 reflow 的目的，使用 cloneNode 也是同理。 而使用 display：none 来降低 reflow 的性能开销的原理在于使节点从 render tree 中失效，等经过多个会触发 reflow 操作后再“上线”

#### 3\. 减少会 flush 缓存队列属性的访问次数，如果一定要访问，使用缓存

```
// 不好的写法
for(let i = 0; i < 20; i++ ) {
    el.style.left = el.offsetLeft + 5 + "px";
    el.style.top = el.offsetTop + 5 + "px";
}
// 比较好的写法
var left = el.offsetLeft,
top = el.offsetTop,
s = el.style;
for (let i = 0; i < 20; i++ ) {
    left += 5;
    top += 5;
    s.left = left + "px";
    s.top = top + "px";
}

```

### 替代会触发 reflow 和 repaint 的属性

我们可以将一些会触发回流的属性替换，来避免 reflow。比如用 translate 代替 top，用 opacity 替代 visibility

样例代码：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta >
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #react {
            position: relative;
            top: 0;
            width: 100px;
            height: 100px;
            background-color: red;
        }
    </style>
</head>

<body>
    <div id="react"></div>
    <script type="text/javascript">
        setTimeout(() => {
            document.getElementById("react").style.top = "100px"
        }, 2000);
    </script>
</body>
</html>

```

代码很简单，页面上有一个红色的方块，2 秒后它的 top 值将会变为 “100px”，为了方便体现替代的属性可以避免 reflow 这里我们使用 chrome 的开发者工具，部分截图如下

![](https://user-gold-cdn.xitu.io/2018/2/26/161d147145492045?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 如上图，在 top 值变为 “100px” 的过程中有上图五个阶段.

*   Recalculate Style，浏览器计算改变过后的样式
*   Layout，这个过程就是我们说得 reflow 回流过程
*   Update Layer Tree，更新 Layer Tree
*   Paint，图层的绘制过程
*   Composite Layers, 合并多个图层

我们把这五个过程用时记下：80 + Layout(73) + 72 + 20 + 69 = **316us**

再用 translate 替代 top：

```
-       position: relative;
-       top: 0;
+       transform: translateY(0);

-       document.getElementById("react").style.top = "100px"
+       document.getElementById("react").style.transform = "translateY(100px)"

```

Performace 截图：

![](https://user-gold-cdn.xitu.io/2018/2/26/161d151b998557a7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 可以看到用 translate 替换 top 后减少了原来的 Layout 也就是 reflow 的过程，用时：81 + 80 + 36 + 83 = **280us**。 结果非常明显 315us 减少到了 280us。有人说这个效果不明显呀，但是让我们设想这样一个业务场景，有许多网站都会有不停移动的飘窗，这种飘窗通常是用定时器实现，每隔 100ms 就去修改一次它的 top，如果用 translate 的话 1s 就可以减少 10 次 reflow，如果这个飘窗样式比较多，比较复杂，那么 1 秒钟减少的 10 次 reflow 就有可能**减少几百毫秒甚至几秒 Layout 的过程**

我们再用 opacity 去替代 visibility 试试看。

```
-            document.getElementById("react").style.transform = "translateY(100px)"
+            document.getElementById("react").style.visibility = "hidden"

```

Performace 截图：

![](https://user-gold-cdn.xitu.io/2018/2/26/161d16786911025c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) visibility 属性值改变只会触发 repaint，不会触发 reflow，所以只有四个阶段，其中第三个阶段 Paint 就是重绘的体现，用时：48 + 50 + Paint(14) + 71 = **183us**。我们再用 opacity 替代 visibility

```
+            opacity: 1;

-            document.getElementById("react").style.visibility = "hidden"
+            document.getElementById("react").style.opacity = "0"

```

按照上面的样例，应该得出用 opacity 替代 visibility 后重绘也就是 Paint 这个过程会消失从而达到性能提升的目的，既然这样我们来看 Performace 截图：

![](https://user-gold-cdn.xitu.io/2018/2/26/161d16f8733b450a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 对，你没有看错，我也没有截错图，这次不光是 Paint 过程没有消失，就连 Layout 都出现了，惊不惊喜！意不意外！ ![](https://user-gold-cdn.xitu.io/2018/2/26/161d17199702f730?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 我们再来重定义一下 repaint 重绘，它是重新绘制当前图层的内容，(什么是图层，[点击](https://link.juejin.im?target=http%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F50661553)查看这篇文章)

其实 opacity 变化并不能改变这个图层的内容，改变的只是当前图层的 alpha 通道的值，从而来决定这个图层显不显示。但是这 opacity 变化的元素并不是单独的图层，而是在 document 这个图层上的，如下 Layers 截图：

![](https://user-gold-cdn.xitu.io/2018/2/26/161d1c9ffb65423e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

就是说浏览器并不能因为图层里面有一个 opacity 为 0 的元素就让整个图层的 alpha 通道变为零，而让整个图层不显示，所以就有了 Layout 和 Paint 这两个过程。解决办法也很简单那就是直接让这个元素单独为一个图层

修改 css 新建图层有两种办法：

*   will-change：transform
*   transform：translateZ(0)

这里我们用下面一个

```
+   transform: translateZ(0);

```

Performace 截图：

![](https://user-gold-cdn.xitu.io/2018/2/26/161d1dabd4041a60?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 现在就和理想中的情况一样了，用 opacity 替代 visibility 可以避免 Paint 重绘的过程。再来看看用时: 66 + 53 + 52 = **171us**

这里由于我变动的元素非常简单，只有一个简单的 div，减少 Paint 过程带来的优化收益并不是很明显，如果是 Paint 过程是毫秒级别减少 Paint 过程的效果还是可观的。

由上述两个替代会触发 reflow 和 repaint 的属性取得性能优化收益的例子中可以看出，这个方法是可行的，除开第一点减少 reflow 操作和第二点替换属性以外还有一些方法可以减少 reflow 和 repaint

*   减少 table 的使用

*   动画实现的速度选择

*   对于动画新建图层

    table 自带的样式和一些非常方便的特性会方便我们的开发，但是 table 也有一些与生俱来的性能缺陷，如果想要修改表格里不管哪一个单元格，都会导致整张表格的重新 Layout，如果这个表格很大，性能的消耗会有一个上升成本的。

## 图层的运用

* * *

在上一个样例中我们新建了一个图层实现了 opacity 替代 visibility 去减少 repaint 的可行性，那么图层还有什么其他运用吗？答案是有的，我们可以将一些频繁重绘回流的 DOM 元素作为一个图层，那么这个 DOM 元素的重绘和回流的影响只会在这个图层中，当然如果你为每一个元素都创建一个图层那样肯定也会聪明反被聪明误，还记得上述的 Performance 截图中的过程吗，最后一个 Composite Layers 这个过程就是合并多个图层的，图层过多这个过程会非常耗时，其实这个过程本身也非常耗时，原则上是在必要的情况下才会新建图层来减少重绘和回流的影响范围，到底使不使用就需要开发人员在业务情景中 balance. 在 Chrome 浏览器下可以这样创建图层:

*   3D 或透视变换 CSS 属性 (perspective transform)
*   使用加速视屏解码的 video 标签
*   拥有 3D(WebGL) 上下文或加速的 2D 上下文的 canvas
*   混合插件如（如 Flash）
*   对自己的 opacity 做 CSS 动画或使用一个动画 webkit 变换的元素
*   拥有加速 CSS 过滤器的元素（GPU 加速）
*   元素有一个包含复合层的后代节点
*   元素有一个 z-index 较低且包含一个复合层的兄弟元素
*   will-change: transform;

大体思路就是我们把频繁重绘回流的 DOM 元素作为一个图层，那么这个 DOM 元素的重绘和回流的影响只会在这个图层中，来提升性能。举个栗子，我们打开 chrome 开发者工具中的 Layers, 然后打开某网站

![](https://user-gold-cdn.xitu.io/2018/2/26/161d20d422e9910e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 从红框中可以看出这个网站已经被分为了很多图层，当前选中的的这个 baner 图层在视图区域已经标注出来，由图可知，将一个经常触发回流和重绘的元素新开图层也算一个优化性能的做法。我们再勾选这个选项![](https://user-gold-cdn.xitu.io/2018/2/26/161d2125bf63f458?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 浏览器会用绿色高亮出当前正在 repaint 的元素，勾选上过后我们打开一个视频：![](https://user-gold-cdn.xitu.io/2018/2/26/161d213fafb463d9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 可以看到视频在播放过程中一直处于高亮状态，这个不难理解，video 为单独一个图层，在整个视频播放过程中 video 接受到发送过来的每一帧，都会将触发 video 所在图层的重绘。

## 结语

简单回顾一下本文，我们最开始聊了一下 reflow 和 repaint 是什么，如何触发它们，接下来谈了一下浏览器在处理它们所采取的策略，最后就是如何避免 reflow 和 repaint 带来的性能开销，还补充了一下图层的存在意义和简单运用。 其实在优化 reflow 和 repaint 上就是两点：

*   避免使用触发 reflow、repaint 的 css 属性
*   将 reflow、repaint 的影响范围限制在单独的图层之内

### 参考资料

https://csstriggers.com

http://blog.csdn.net/luoshengyang/article/details/50661553
