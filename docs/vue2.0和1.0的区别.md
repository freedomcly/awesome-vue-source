# vue2.0和1.0的区别

* 引入轻量级的virtual dom
* 引入render函数
* 服务器端渲染
* weex native渲染

 
API：

* 生命周期api，如：ready替换为mounted
* v-for，如：track-by替换为key
* props，如：twoWay移除 单向数据流
* 事件，如：$dispatch和$broadcast被弃用，复杂的交互使用Vuex