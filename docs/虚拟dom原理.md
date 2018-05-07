
# patch上下文
patch注入：

    import { patch } from './patch'
    Vue.prototype.__patch__ = patch

patch使用：

    export function lifecycleMixin (Vue: Class<Component>) {
      Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
        const vm: Component = this
        if (vm._isMounted) {
          callHook(vm, 'beforeUpdate')
        }
        const prevEl = vm.$el

        const prevVnode = vm._vnode
        const prevActiveInstance = activeInstance
        activeInstance = vm
        vm._vnode = vnode
        // Vue.prototype.__patch__ is injected in entry points
        // based on the rendering backend used.
        if (!prevVnode) {
          // initial render
          // console.log(vm.$el)
          vm.$el = vm.__patch__(
            vm.$el, vnode, hydrating, false /* removeOnly */,
            vm.$options._parentElm,
            vm.$options._refElm
          )
          // no need for the ref nodes after initial patch
          // this prevents keeping a detached DOM tree in memory (#5851)
          vm.$options._parentElm = vm.$options._refElm = null
        } else {
          // updates
          vm.$el = vm.__patch__(prevVnode, vnode)
        }
        activeInstance = prevActiveInstance
        // update __vue__ reference
        if (prevEl) {
          prevEl.__vue__ = null
        }
        if (vm.$el) {
          vm.$el.__vue__ = vm
        }
        // if parent is an HOC, update its $el as well
        if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
          vm.$parent.$el = vm.$el
        }
        // updated hook is called by the scheduler to ensure that children are
        // updated in a parent's updated hook.
      }


# 参考资料

* [《React’s diff algorithm》](https://calendar.perfplanet.com/2013/diff/)
* [《解析vue2.0的diff算法》](https://github.com/aooy/blog/issues/2)
* [《VirtualDOM与diff(Vue实现)》](https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown)