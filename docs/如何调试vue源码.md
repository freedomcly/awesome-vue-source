# 如何调试vue源码

## npm scripts

在package.json中可以看到所有的npm scripts，与调试相关的有

```
"scripts": {
    "dev": "rollup -w -c build/config.js --environment TARGET:web-full-dev",
    "dev:cjs": "rollup -w -c build/config.js --environment TARGET:web-runtime-cjs",
    "dev:esm": "rollup -w -c build/config.js --environment TARGET:web-runtime-esm",
    "dev:test": "karma start test/unit/karma.dev.config.js",
    "dev:ssr": "rollup -w -c build/config.js --environment TARGET:web-server-renderer",
    "dev:compiler": "rollup -w -c build/config.js --environment TARGET:web-compiler ",
    "dev:weex": "rollup -w -c build/config.js --environment TARGET:weex-framework",
    "dev:weex:factory": "rollup -w -c build/config.js --environment TARGET:weex-factory",
    "dev:weex:compiler": "rollup -w -c build/config.js --environment TARGET:weex-compiler ",
    ......
}
```

这些脚本有什么区别？

TARGET不同。build/config.js文件里记录了所有vue的编译场景和对应入口。

## vue的编译场景和对应入口

| **场景** | **入口** | **备注** |
| :--- | :--- | :--- |
| web-full-cjs | web/entry-runtime-with-compiler.js | cjs: CommonJS |
| web-full-esm | web/entry-runtime-with-compiler.js | esm: ES module |
| web-full-dev | web/entry-runtime-with-compiler.js |  |
| web-full-prod | web/entry-runtime-with-compiler.js |  |
| web-runtime-cjs | web/entry-runtime.js |  |
| web-runtime-esm | web/entry-runtime.js |  |
| web-runtime-dev | web/entry-runtime.js |  |
| web-runtime-prod | web/entry-runtime.j |  |
| 其他场景是关于服务器端渲染和weex |  |  |

build/config.js可以和[官网对于vue不同版本的解释](https://vuejs.org/v2/guide/installation.html#Explanation-of-Different-Builds)对应起来。

entry-runtime-with-compiler.js和entry-runtime.js的区别是什么呢？

* entry-runtime-with-compiler.js：编译出来是完整版vue，包含runtime和compiler。

* entry-runtime.js：只包含runtime版本。

compiler用来做模板字符串编译，也就是只有在基于vue的业务项目中包含template选项才会用到。

```
new Vue({template: ''})
```

**大多数情况下，基于vue的业务项目应该使用only runtime版本，它比完整版体积小30%，gzip后大约20kb多一点。**

## 调试

```
npm run dev // 编译模式为web-full-dev，入口为entry-runtime-with-compiler
```

运行起来后，rollup（类似于webpack的打包工具）会监听所有源文件的变化，即时地生成dist/vue.js。

打开examples文件夹，把html文件中对vue的引用指向dist/vue.js（一般之前是dist/vue.min.js）。

再打开html文件，就可以调试了。

## 从入口文件开始

上面说到有两个入口文件，`entry-runtime-with-compiler.js`和`entry-runtime.js`。以`entry-runtime.js`为例，

    import Vue from './runtime/index'
    export default Vue

而`entry-runtime-with-compiler.js`在`./runtime/index`的基础上做了一些compiler相关的处理，也就是template选项相关的处理。在`/runtime/index.js`文件中，

    import Vue from 'core/index'

在`core/index.js`文件中，
    
    import Vue from './instance/index'

在`instance/index.js`文件中，Vue构造函数的定义如下：

    import { initMixin } from './init'
    function Vue (options) {
      this._init(options)
    }
    initMixin(Vue)
    export default Vue

`init.js`文件中，

    export function initMixin (Vue) {
      Vue.prototype._init = function(options) {
        const vm = this
        ...
        // 对Vue实例初始化
      }
    }