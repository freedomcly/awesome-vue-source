# 开始

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
* 如果没有以`new Vue({})`的方式运行，this指向不是vm，`this instanceof Vue`返回`false`，浏览器console中会提示：Vue is a constructor and should be called with the \`new\` keyword
* warn是vue项目中自定义的console方法，底层使用JavaScript`console.error`
* 为什么vue实例vm可以调用`this._init`方法呢？事实上，经过`initMixin(Vue)`后，Vue的原型对象增加了Vue.prototype._init。新建vm实例时，也就是`new Vue({})`运行时，vm实例可以调用构造函数Vue的原型对象上的方法

# _init中做了哪些事

源码中`process.env.NODE_ENV !== 'production'`

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

初始化的变量有：
* _uid
* _isVue
* $options
* _renderProxy
* _self


// initLifecycle
* $parent
* $root
* $children
* $refs
* _watcher
* _inactive
* _directInactive
* _isMounted
* _isDestroyed
* _isBeingDestroyed

// initEvents
* _event
* _hasHookEvent

// initRender
* _vnode
* $vnode
* $slots
* $scopedSlots
* _c
* $createElement
* $attrs
* $listeners
// 25
// initState
* _watchers
* _props
* [data name]
* [method name]
// 31


# 一些问题

* 为什么`_init`一进去，vue实例里已经有几个变量：$data/$isServer/$props/$ssrContext
* 前缀有什么区别：_/$/

