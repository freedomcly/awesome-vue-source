# 如何调试vue源码

### npm scripts

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

这个表格可以和官网的解释对应起来 [https://vuejs.org/v2/guide/installation.html\#Explanation-of-Different-Builds](https://vuejs.org/v2/guide/installation.html#Explanation-of-Different-Builds)

表格里场景有很多，入口只有两个文件：entry-runtime-with-compiler.js和entry-runtime.js，他们的区别是什么呢？

entry-runtime-with-compiler.js：编译出来是完整版vue，包含runtime和compiler。

entry-runtime.js：只包含runtime版本。

compiler用来做模板字符串编译，也就是只有在基于vue的项目中需要包含template才会用到

```
new Vue({template: ''})
```

**大多数情况下，基于vue的业务项目应该使用only runtime版本，它比完整版体积要小30%。only runtime版本gzip后大约20kb多一点。**



### 所见即所得的调试

```
npm run dev // 编译模式为web-full-dev，入口为entry-runtime-with-compiler
```

运行起来后，rollup（类似于webpack的打包工具）会监听所有源文件的变化，即时地生成dist/vue.js。

打开examples文件夹，把html文件中对vue的引用指向dist/vue.js（一般之前是dist/vue.min.js）。

再打开html文件，就可以调试了。

