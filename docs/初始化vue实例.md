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

# 





