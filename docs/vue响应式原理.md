# vue响应式原理

## 一些概念

什么是MVVM？[详见维基百科中的解释](https://en.wikipedia.org/wiki/Model–view–viewmodel)

MVVM是从MVC演化而来的软件架构模式。在现代vue项目中，可以这样理解：

* Model =&gt; state data
* View =&gt; template / render
* ViewModel =&gt; vue实例的剩余部分

---

什么是数据双向绑定？

数据双向绑定是在Angular时代经常被提起的概念，简单地说，

* Model层中的数据改变时，立即更新View
* View层有交互事件，立即更新Model层的数据，从而更新View层

第二部分很好解决，用JavaScript监听交互事件，再更新对应的Model数据就可以。**困难的是第一部分，Model层数据改变时，如何更新View层。这就是所谓的响应式。**

很多现代MVVM framework解决了这个问题，如backbone（发布订阅模式）、angularJS（数据脏检查）、react（View抽象的脏数据检查）。vue是如何解决的呢？

## Vue响应式

![](/assets/reactive.png)

可以结合官网的[《深入响应式原理》](https://cn.vuejs.org/v2/guide/reactivity.html)和vue源码来理解。

一言以蔽之，vue做了几件事：

* 用ES5新特性Object.defineProperty改写了Model层的所有data，为每个data添加了getter/setter函数，也就是所谓的数据劫持
* render View时进行依赖收集
* 更新Model层数据时，调用setter函数，如果有依赖就发出通知

## 源码






