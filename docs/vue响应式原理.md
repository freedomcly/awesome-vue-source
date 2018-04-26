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

## 数据劫持

    function initData (vm: Component) {
      let data = vm.$options.data
      data = vm._data = typeof data === 'function' ? getData(data, vm) : data || {}
      ...

      // proxy data on instance
      const keys = Object.keys(data)
      let i = keys.length
      while (i--) {
        const key = keys[i]
        proxy(vm, `_data`, key)
      }

      // observe data
      observe(data, true /* asRootData */)
    }
    
* 首先，initData中会检查选项options中的data的类型是对象还是函数，对此进行不同的处理。应用vue时data有两种写法：data: {key1: value1, key2: value2}; data(){return {key1: value1, key2: value2}}，可以参考vue的api文档：[data](https://cn.vuejs.org/v2/api/#data)
* initData做了什么呢？初始化$data。在initData之前，$data是undefined，initData之后$data是:


    {
      key1: value1,
      key2: value2,
      ...,
      __ob__: {...},
      get key1: ...,
      set key1: ...,
      get key2: ...,
      set key2: ...,
      ...,
      __proto__: {...}
    }

* proxy会把$data中的数据代理到vm上，也就是vm.$data.key1可以直接用vm.key1来访问，具体实现机制可以后面再看


    function observe(value, asRootData) {
      ...
      let ob = new Observer(value)
      ...
      return ob
    }
    
    export class Observer {
      value: any;
      dep: Dep;
      vmCount: number; // number of vms that has this object as root $data

      constructor (value: any) {
        this.value = value
        this.dep = new Dep()
        this.vmCount = 0
        def(value, '__ob__', this)
        if (Array.isArray(value)) {
          const augment = hasProto ? protoAugment : copyAugment
          augment(value, arrayMethods, arrayKeys)
          this.observeArray(value)
        } else {
          this.walk(value)
        }
      }

      /**
        * Walk through each property and convert them into
        * getter/setters. This method should only be called when
        * value type is Object.
        */
      walk (obj: Object) {
        const keys = Object.keys(obj)
        for (let i = 0; i < keys.length; i++) {
          defineReactive(obj, keys[i], obj[keys[i]])
        }
      }

      /**
        * Observe a list of Array items.
        */
      observeArray (items: Array<any>) {
        for (let i = 0, l = items.length; i < l; i++) {
          observe(items[i])
        }
      }
    }
    
* 在Observer中，会把Observer实例赋值给$data.\_\_ob\_\_
* 另外，对data是数组和键值对的情况进行分别处理，数组时对每个值再进行observe(item)，键值对时，defineReactive(obj, key, value)


    export function defineReactive (
      obj: Object,
      key: string,
      val: any,
      customSetter?: ?Function,
      shallow?: boolean
    ) {
      const dep = new Dep()

      const property = Object.getOwnPropertyDescriptor(obj, key)
      if (property && property.configurable === false) {
        return
      }

      // cater for pre-defined getter/setters
      const getter = property && property.get
      const setter = property && property.set

      let childOb = !shallow && observe(val)
      Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: function reactiveGetter () {
          const value = getter ? getter.call(obj) : val
          if (Dep.target) {
            dep.depend()
            if (childOb) {
              childOb.dep.depend()
              if (Array.isArray(value)) {
                dependArray(value)
              }
            }
          }
          // console.log(dep)
          return value
        },
        set: function reactiveSetter (newVal) {
          const value = getter ? getter.call(obj) : val
          /* eslint-disable no-self-compare */
          if (newVal === value || (newVal !== newVal && value !== value)) {
            return
          }
          /* eslint-enable no-self-compare */
          if (process.env.NODE_ENV !== 'production' && customSetter) {
            customSetter()
          }
          if (setter) {
            setter.call(obj, newVal)
          } else {
            val = newVal
          }
          childOb = !shallow && observe(newVal)
          // console.log(dep)
          dep.notify()
        }
      })
    }


* defineReactive就是vue响应式的核心代码，把data中所有值改写成get和set。
* defineReactive get中涉及到两个类：Dep和Watcher。
