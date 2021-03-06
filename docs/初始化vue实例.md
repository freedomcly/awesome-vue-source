# 初始化vue实例

## 开始

这是`src/core/instance/index`文件的全部内容。

    import { initMixin } from './init'
    import { stateMixin } from './state'
    import { renderMixin } from './render'
    import { eventsMixin } from './events'
    import { lifecycleMixin } from './lifecycle'
    import { warn } from '../util/index'

    function Vue (options) {
      if (process.env.NODE_ENV !== 'production' &&
        !(this instanceof Vue)
      ) {
        warn('Vue is a constructor and should be called with the `new` keyword')
      }
      this._init(options)
    }

    initMixin(Vue)
    stateMixin(Vue)
    eventsMixin(Vue)
    lifecycleMixin(Vue)
    renderMixin(Vue)

    export default Vue

可以先关注一下其中的初始化部分：

    import { initMixin } from './init'
    ...

    function Vue (options) {
      if (process.env.NODE_ENV !== 'production' &&
        !(this instanceof Vue)
      ) {
        warn('Vue is a constructor and should be called with the `new` keyword')
      }
      this._init(options)
    }

    initMixin(Vue)
    ...

    export default Vue
    
需要注意：
* 如果以`new Vue()`的方式运行，Vue构造函数中this指向vue实例vm
* 如果没有以`new Vue({})`的方式运行，this指向不是vm，`this instanceof Vue`返回`false`，dev版本的vue会在浏览器console中提示：Vue is a constructor and should be called with the \`new\` keyword
* warn是vue项目中自定义的console方法，底层使用JavaScript`console.error`
* 为什么vue实例vm可以调用`this._init`方法呢？事实上，经过`initMixin(Vue)`后，Vue的原型对象增加了Vue.prototype._init。新建vm实例时，也就是`new Vue({})`运行时，vm实例可以调用构造函数Vue的原型对象上的方法
* `process.env.NODE_ENV !== 'production'`可以在`build/config.js`中配置

## _init中做了哪些事

    Vue.prototype._init = function (options?: Object) {
      const vm: Component = this
      // a uid
      vm._uid = uid++

      let startTag, endTag
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        startTag = `vue-perf-start:${vm._uid}`
        endTag = `vue-perf-end:${vm._uid}`
        mark(startTag)
      }

      // a flag to avoid this being observed
      vm._isVue = true
      // merge options
      if (options && options._isComponent) {
        // optimize internal component instantiation
        // since dynamic options merging is pretty slow, and none of the
        // internal component options needs special treatment.
        initInternalComponent(vm, options)
      } else {
        vm.$options = mergeOptions(
          resolveConstructorOptions(vm.constructor),
          options || {},
          vm
        )
      }
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production') {
        initProxy(vm)
      } else {
        vm._renderProxy = vm
      }
      // expose real self
      vm._self = vm
      initLifecycle(vm)
      initEvents(vm)
      initRender(vm)
      callHook(vm, 'beforeCreate')
      initInjections(vm) // resolve injections before data/props
      initState(vm)
      initProvide(vm) // resolve provide after data/props
      callHook(vm, 'created')
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        vm._name = formatComponentName(vm, false)
        mark(endTag)
        measure(`vue ${vm._name} init`, startTag, endTag)
      }

      if (vm.$options.el) {
        vm.$mount(vm.$options.el)
      }
    }

依次初始化的变量有：
* _uid                      // 同一个页面上vue实例的唯一标识，从0开始递增，在单页应用中是什么情况呢？
* _isVue
* $options                  // 合并选项，vue组件和vue页面情况不同？什么是vue组件，什么是vue页面？
* _renderProxy              // 生产环境是vm实例自身
* _self                     // vm实例自身

// initLifecycle
* $parent                   // 指向父组件
* $root                     // 指向根组件
* $children                 // []
* $refs                     // {}
* _watcher                  // null
* _inactive                 // null
* _directInactive           // false
* _isMounted                // false
* _isDestroyed              // false
* _isBeingDestroyed         // false

// initEvents
* _events                   // null
* _hasHookEvent             // false

// initRender
* _vnode                    // null
* $vnode                    // undefined
* $slots                    // {}
* $scopedSlots              // {}
* _c                        // ?
* $createElement            // ?
* $attrs                    // {}
* $listeners                // {}

// initState
* _watchers                 // ?
* _props                    // {}
* [data name]
* [method name]

* $el                       // 真实dom元素

## 问题们

* 为什么`_init`一进去，vue实例里已经有几个变量：$data/$isServer/$props/$ssrContext

这些变量是用`Object.defineProperty(Vue.prototype, '$data', dataDef)`这种形式定义的，因此所有Vue实例共享了这些变量。$data的getter函数返回\_data，$props的getter函数返回\_props。

* 前缀有什么区别：_/$/

比较粗糙的解释是\_data是供vue源码内部使用，$data供外部的业务代码使用。但真实情况是外部$data和\_data都可以用。

* $data、_data和vm实例下对应变量们[data name]的区别是什么，为什么要多次代理
* 多页应用和单页应用中_uid的情况
* vue组件和vue页面的区别是什么？_isComponent有什么用
* let a1 = {}和let a2 = Object.create(null)的区别是什么
* \_\_proto\_\_是什么

问题们在之后再慢慢回答。



