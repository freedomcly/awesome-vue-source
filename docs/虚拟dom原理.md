
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
        ...
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
        ...
      }

`vm._update(vnode)`的作用是把虚拟DOM树渲染为真实DOM树。其中`vm.__patch__(prevVnode, vnode)`的作用是把当前vnode和之前已经渲染过的prevVnode做对比，以便更高效地更新DOM树。

# patch算法


# 参考资料

* [《React’s diff algorithm》](https://calendar.perfplanet.com/2013/diff/)
* [《解析vue2.0的diff算法》](https://github.com/aooy/blog/issues/2)
* [《VirtualDOM与diff(Vue实现)》](https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown)