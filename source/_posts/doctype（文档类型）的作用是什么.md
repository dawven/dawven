---
title: doctype（文档类型）的作用是什么
date: 2018-07-20 10:54:21
tags: 面试题
---

2000 年 1 月 5 日，微软发表声明要发布 「IE5 for Mac」。

我们先不要惊叹 IE 居然开发过 Mac 版本，也不要惊讶 Mac 版本的 IE2 到 IE5 存在了长达十年，更不要惊呼 IE for Mac 作为 Mac 的默认浏览器存在了五六年的时间。

以当时的眼光来说，那个时候的 IE5 非常先进，它完全实现了当时最新的 html 标准「HTML 4.0」。但是随之而来的问题是，对于一些旧的网页却不能正确的呈现（或者说，他们是被正确渲染了），那些网页是按照当时占统治地位的浏览器的渲染模式来渲染的，IE5 肯定不能就这样发布。

微软想到了一个解决方案，没错，这就是 「DOCTYPE」。使用新标准的页面可以在 `<html>` 之前加上 doctype 来激活新的标准模式。很快，各个浏览器都采用了这个方案，使用「混杂模式（Quirks）」和「标准模式（Standard Mode）」来进行渲染，但是实际上很多网页在使用标准模式时，却是基于某种混杂模式来写的，于是 Mozilla 1.1 创造了「准标准模式（Almost Standard Mode）」。
<!-- more -->
总结起来，混杂模式是不符合 Web 标准的模式，准标准模式是几乎要符合标准的模式，标准模式是符合标准的模式。

我们可以使用不同的 doctype 来激活不同的模式。

```
<!DOCTYPE html>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

No doctype

```

除了上面这些还有非常多的 doctype 类型，而他们在不同的浏览器中又会激活不同的模式，以我的经验而言，现在最常用的就是这两个：

第一种是 html5 的 doctype，激活标准模式

```
<!DOCTYPE html>

```

第二种是 XHTML 1.0 用于过渡的 doctype，它会激活准标准模式

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

```

可以说，这两种 doctype 占了整个互联网页面的绝大部分，现在的网页除了第一种就是第二种。这两种 doctype 的背后也有很多故事可以讲。

#### 什么是 XHTML

首先我们先来明确一点，什么是 XHTML？

W3C 发布了 HTML4 之后转向了 XHTML，「XHTML is the future」。XHTML 1.0 并没有在 HTML4 的基础上增加新的特性，而是用 XML 将 HTML4 进行了重写。这次重写对 HTML 进行了严格的限制，比如要使用 `<br/>` 而不是 `<br>`，要在属性值上使用双引号。而一旦遇到一个错误就要立刻停止解析，并显示错误信息。而当时的页面绝大多数都不可能满足这种严格的标准，所以开发者们也使用了这种新的语法，但是却是以 text/html 的形式来发布的。

声明自己使用的是 XHTML 1.0 的方式就是使用 doctype，

而随后发布的 XHTML 1.1 却没有得到广泛的支持，因为它只允许以 application/xhtml+xml 的形式发布，所以 WEB 开发者们就直接忽略掉了。

#### HTML5 的 doctype

为什么 html5 的 doctype 这么简单呢？因为 HTML5 不再是基于 SGML 的语言，而 doctype 只是用来激活模式的。

HTML5 之前，HTML 都是用 SGML 来书写的，DOCTYPE 则用来声明文档类型，它可以告诉 SGML parser 使用什么 DTD 来解析文档。所以到了 HTML5，根本就没有对应的 DTD，也就没有后面一串异常复杂的表述了。

那么，为什么 HTML5 不再是 SGML 了呢？

我想它的原因可能是这样：SGML 需要在 DTD 中定义好标签和属性，但是 HTML5 中要允许自定的标签和属性的，原来的框架太过束缚，它需要更加广大的范围来放飞自己。

#### 什么是 SGML

它是可以定义标签语言的元语言，比如 HTML5 之前的版本都是拿 SGML 来写的。

#### 总结

回到主题，doctype 有什么作用，它的作用就是用来激活各种渲染模式啊。