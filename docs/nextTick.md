# nextTick

nextTick中的回调函数在**Vue完成data update和DOM update之后，render之前**执行。

## 一个Demo

nextTick和setTimeout有什么不同？nextTick在render之前执行，setTimeout在render之后执行，见demo：

    <div id="app" ref="test">
      {{value}}
    </div>
    <script>
      var app = new Vue({
        el: '#app',
        data () {
          return {
            value: 0
          }
        },
        mounted () {
          this.value = 1
          console.log(this.$refs.test.innerText)
          setTimeout(() => {
            this.value = 3
          })
          this.$nextTick(() => {
            this.value = 2
          })
        }
      })
    </script>
    
* 打印`this.$refs.test.innerText`是0
* nextTick不会看到value从1变成2，setTimeout能看到value从1变成3。另外setTimeout回调函数总是在nextTick之后执行。

为什么呢？

## 异步流程

| **macro task** | **micro task** | **macro task** | ... |
| :--- | :--- | :--- | :--- |
| vue data update / vue DOM update | `Vue.nextTick` | vue render / `setTimeout` | ... |

由此可见，vue render在下一个macro task队列，`setTimeout`也会把回调函数放到下一个macro task队列，而`Vue.nextTick`会把回调函数放在下一个micro task队列。所以`Vue.nextTick`会在render之前执行，`setTimeout`在render之后执行。

在应用中，Vue.nextTick适合那些即时的data更新，在render前更新会避免闪烁，用户体验更好。**比如，data有一个初始值，也有一个从接口读取的值，Vue.nextTick能避免从初始值到更新值的闪烁。**

- [ ] vue render是不是总是在下一个macro task中？

# 参考资料

* [What is nextTick or what does it do in VueJs](https://stackoverflow.com/questions/47634258/what-is-nexttick-or-what-does-it-do-in-vuejs)