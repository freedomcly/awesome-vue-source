# vue生命周期

[vue生命周期图](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)

## 生命周期钩子函数

可以看到，vue生命周期钩子函数有：
* beforeCreate
* created
* beforeMount
* mounted
* beforeUpdate
* updated
* beforeDestroy
* destroyed

create即创建vue实例。从源码可以看到，钩子函数beforeCreate和created之间进行了initState，也就是初始化data、props、computed、methods、watch等Observer数据。**因此，created钩子函数中可以做与data相关的操作，如ajax请求。ajax请求放在created中的好处是：如果初始页面过于复杂，mount过程时间长，让ajax异步请求和mount并行，提升首页渲染速度。**

mount。从源码可以看到，钩子函数beforeMount和mounted之间进行了render和第一次update（虚拟dom的创建和patch等）。**因此，mounted钩子函数中可以认为DOM已经就绪了，可以进行依赖于DOM的操作。**

## 各生命周期中el、data的状态

![](/assets/life-cycle.png)
