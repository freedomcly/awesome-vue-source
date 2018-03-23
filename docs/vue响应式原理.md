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
* View层有IO事件，立即更新Model层的数据，从而更新View层



