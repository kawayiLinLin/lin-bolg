---
title: webpack相关知识点梳理
date: 2022-07-03 14:58:25
tags:
  - webpack
category:
  - 技术
  - 分享
---

## 与 Webpack 类似的工具有哪些？为什么选择和弃用 Webpack

- [grunt](https://gruntjs.net)

  自动化。对于需要反复重复的任务，例如压缩、编译、单元测试等。在 `Gruntfile` 文件配置任务

  最老牌的打包工具，用配置的思想来写打包脚本，一切皆可配置

  - 优点

    早期出现

  - 缺点

    配置项太多
    不同插件配置项的扩展字段不同
    学习成本高，需要学习不同插件的组合配合方式

  - ES6 -> ES5 demo

    1. 安装

       ```shell
       npm install grunt grunt-babel @babel/core @babel/preset-env -D
       ```

    2. Gruntfile.js

       ```js
       module.exports = function (grunt) {
         // 加载 babel 任务
         grunt.loadNpmTask("grunt-babel")
         // 初始化配置文件
         grunt.initConfig({
           // babel 配置
           babel: {
             options: {
               sourceMap: true,
               // babel预设 ES6 -> ES5
               persets: ["@babel/perset-env"],
             },
             dist: {
               // 把 src/app.js 打包到 dist/app.js
               files: {
                 "dist/app.js": "src/app.js",
               },
             },
           },
         })
         // default 默认/入口任务
         grunt.registerTask("default", ["babel"])
       }
       ```

- gulp

  <!-- more -->

  基于 `nodejs` 的 `stream` 的打包工具
  定位是基于任务流的自动化构建工具
  通过 task 对整个开发过程进行构建

  - 优点

  流式写法，清晰看到资源的流转过程
  API 简单
  易于学习
  适合多页面

  - 缺点

  异常处理麻烦
  工作流顺序难以精细控制
  不太适合单页或自定义模块的开发

  - ES6 - ES5 demo

    1. 安装

       ```shell
       npm install -S -D glup-cli gulp gulp-babel @babel/core @babel/preset-env
       ```

    2. gulpfile.js

       ```js
       const gulp = require("gulp")
       const babel = require("gulp-babel")
       function defaultTask(callback) {
         gulp
           .src("src/app.js") // 读取文件
           .pipe(
             babel({
               // 传给 babel 任务
               presets: ["@babel/preset-env"],
             })
           )
           .pipe(gulp.dest("dist")) // 写到 dist 里
         callback()
       }
       module.exports = defaultTask
       ```

- Webpack

  模块化的打包和管理工具。通过 loader 转换，任何形式的资源都能视为模块
  将按需加载的模块进行代码分割，等到需要的时候再异步加载
  定位是模块打包器，而 Gulp/Grunt 属于构建工具

  - 优点

  可以模块化的打包任何资源
  适合任何模块化系统
  适合 SPA 应用的开发

  - 缺点

  学习成本高，配置复杂
  通过 babel 编译后的 js 体积过大

  - ES6 - ES5 demo

    1. 安装

       ```shell
       npm install webpack
       ```

    2. webpack.config.js

       ```js
       const path = require('path')
       module.exports = {
           mode: 'development', // 开发模式
           devtools: false, // 不生成sourcemap
           entry: "./src/app.js", //入口
           output: {
               path: path.resolve(__dirname, 'dist'),
               filename: 'bundle.js'
           },
           module: {
               rules: [
                   test: /\.jsx?$/,
                   use: {
                       loader: 'babel-loader',
                       options: {
                           presets: ['@babel/preset-env']
                       }
                   },
                   include: path.join(__dirname, 'src'),
                   exclude: /node_modules/
               ]
           },
           plugins: [],
           devServer: {}
       }
       ```

- Rollup

  下一代 ES6 模块化工具，利用 ES6，tree-shaking 生成简洁更简单的代码
  一般而言，类库使用 Rollup，应用使用 Webpack
  需要拆分资源，或者静态资源较多，需要引入较多的 CommonJS 模块时，使用 Webpack
  代码库是基于 ES6 模块，希望代码能被其他人直接使用，使用 Rollup

  - 优点

  用 ES6 写代码，通过减少 dead code 来缩小包的体积

  - 缺点

  代码拆分、静态资源、CommonJS 支持不好

  - ES6 - ES5 demo

    1. 安装

       ```shell
       npm install rollup
       ```

    2. rollup.config.js

       ```js
       import resolve from 'rollup-plugins-node-resolve'
       import babel from 'rollup-plugins-babel'

       export default {
       input: 'src/main.js' // webpack 的 entry
       output: {
           file: 'dist/bundle.js',
           format: 'cjs',
           exports: 'default'
       },
       plugins: [
           resolve(),
           babel({
               "presets": ['@babel/preset-env'],
               exclude: "node_modules/**"
           })
       ]
       }
       ```

- parcel

  快速，零配置的 web 应用程序打包器
  目前只能用于构建在浏览器运行的网页

  - 优点

  内置了常见场景的构建方案及其依赖，无需再安装依赖
  内置开发服务器
  能以 html 为入口，自动检测和打包依赖资源
  支持模块热替换，开箱即用

  - 缺点

  不支持 sourcemap
  不支持 tree-shaking
  配置不灵活

  - ES6 -> ES5 demo

    1. 安装

       ```shell
       npm install -g parcel-bundler

       parcel src/index.html -p 3000
       ```

## Loader 和 Plugins 的不同

- Loader -> 加载器。Webpack 将一切视为模块，但是 Webpack 原生只支持解析 JS 文件，Loader 让 Webpack 具有了加载和解析非 JS 文件的能力

- Plugin -> 插件。Plugin 可以扩展 Webpack 的能力，让 Webpack 更灵活。Webpack 在运行的生命周期中会广播很多事件，插件监听合适的事件，在合适的时机通过 Webpack 提供的 API 改变输出结果

## webpack 基本的工作流

```js
const webpack = require("webpack")
const config = require("./webpack.config.js")

debugger
const compiler = webpack(config)
function compilerCallback(err, stats) {
  const statsString = stats.toString()
  console.log(statsString)
}

compiler.run((err, stats) => {
  compilerCallback(err, stats)
})
```

1. 初始化参数

从配置文件和 shell 文件中读取和合并参数，得出最终的参数

2. 开始编译

用上一步得到的参数初始化 Compiler 对象
加载所有配置的插件，执行对象的 run 方法开始执行编译
根据配置中的 entry 确定入口文件

3. 编译模块

从入口文件触发，调用所有配置的 loader 对模块进行编译
再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理

4. 完成模块编译

使用 loader 翻译完所有的模块后，得到每个模块的资源和对应的依赖关系

5. 输出资源

根据入口和模块的依赖关系，组装成一个个包含多个模块的 Chunk
把每个 Chunk 转换成一个单独的文件加入到输出列表（可以修改输出内容的最后机会）

6. 输出完成

确定好输出内容后，根据配置的输出路径和路径名，把文件写到文件系统（可以指定文件系统 compiler.inputFileSystem compiler.outputFileSystem compiler.watchFileSystem）

伪代码模拟

```js
const { SyncHook } = require("tapable")
const fs = require("fs")

class Compiler {
  constructor(options) {
    this.options = options
    this.hooks = {
      run: new SyncHook(),
      done: new SyncHook(),
    }
  }
  run() {
    const modules = []
    const chunks = []
    const files = []
    this.hooks.run.call()
    // 2.3根据配置中的entry确定入口文件
    const entry = path.join(this.options.context, this.options.entry)
    // 3.1 读取模块内容
    const entryContent = fs.readFileSync(entry, "utf-8")
    // 3.2 使用 loader 解析
    const entrySource = babelLoader(entryContent)
    // module 模块
    // chunk 代码块
    // file 文件
    // bundle
    const entryModule = {
      id: entry,
      source: entrySource,
    }

    modules.push(entryModule)
    // 把入口模块的代码转换成 AST，分析里面的 import 和 require 依赖
    // 继续解析入口引入的模块
    const subEntry = path.join(this.options.context, "subEntryPath")
    // 3.1 读取模块内容
    const subEntryContent = fs.readFileSync(subEntry, "utf-8")
    // 3.2 使用 loader 解析
    const subEntrySource = babelLoader(subEntryContent)

    const subModule = {
      id: subEntry,
      source: subEntrySource,
    }

    modules.push(subModule)
    // 5.1根据入口和模块的依赖关系，组装成一个个包含多个模块的 Chunk
    const chunk = {
      name: "main",
      modules: modules,
    }
    chunks.push(chunk)
    // 5.2把每个 Chunk 转换成一个单独的文件加入到输出列表
    const file = {
      file: this.options.output.filename,
      source: `输出的文件内容`,
    }
    files.push(file)
    const outputPath = path.join(
      this.options.output.path,
      this.options.output.filename
    )
    fs.writeFileSync(outputPath, file.source, "utf-8")
    this.hooks.done.call()
  }
}
// 1. 从配置文件和 shell 文件中读取和合并参数，得出最终的参数
const options = require("./webpack.config.js")
// 2.1用上一步得到的参数初始化 Compiler 对象
const compiler = new Compiler(options)
// 2.2加载所有配置的插件，执行对象的run方法开始执行编译
if (options.plugins && Array.isArray(options.plugins)) {
  for (const plugin of options.plugin) {
    plugin.apply(compiler)
  }
}
// 2.3根据配置中的entry确定入口文件
compiler.run()

// es6 - es5
function babelLoader(source) {
  return "返回解析后的新内容，如 ES6 转 ES5"
}
```

## 常见的 loader 和 plugin

可以通过 create-react-app 创建一个项目，然后 `npm run eject`，暴露配置文件

![](changjiandeloader.jpg)

![](changyongchajian.jpg)

## sourcemap 是什么，生产环境怎么用？

- sourcemap 是为了解决开发代码和实际运行代码不一致时，帮助我们 debug 到原始开发代码的技术
- webpack 是通过配置可以自动给我们 sourcemap 文件，map 文件是一种对应编译文件和源文件的方法

sourcemap 的类型

![](sourcemaps.jpg)

[sourcemap 原理](http://www.zhufengpeixun.com/strong/html/103.14.webpack-sourcemap.html#t152.%20sourcemap)

## 代码分割

1. 入口点分割

在 entry 配置多个入口

可能会导致重复打包 chunk 到多个 bundle 中

2. 动态导入或懒加载

2.1 按需加载 如 React.lazy(() => import('path/file.js'))
2.2 预加载 preload （必须要用到的资源）
`<link href="xxx" rel="preload" as="script" />`

```js
import(
  "file.js"
  /* webpackPreload: true */
  /* webpackChunkName: "file" */
)
```

2.3 预先拉取 prefetch (可能要用到的资源，告诉浏览器可能在未来会用到某个资源，在闲时加载)
`<link href="xxx" rel="prefetch" as="script" />`

```js
import(
  "file.js"
  /* webpackPrefetch: true */
  /* webpackChunkName: "file" */
)
```

module：就是 js 的模块化，webpack 支持 commonJS，ES6 等模块规范，简单说就是可以用 import 引入的代码
chunk：webpack 根据功能拆出来的，包含三种情况

1. 项目入口
2. import 动态引入的
3. 通过 splitChunks 拆分的代码
   
bundle：是 webpack 打包后的各个文件，一般和 chunk 是一对一的关系，是对 chunk 进行编译压缩打包处理之后的产物

好处：缓存、CDN、懒加载

```js
{
  splitChunks: {
    chunks: 'all', // 默认作用于异步chunk，值为all/initial/async
    minSize: 0, // 默认值30kb，代码块的最小尺寸
    minChunks: 1, // 被多少模块共享，在分割之前模块的被引用次数
    maxAsyncRequests: 2, // 限制异步模块内部并行最大请求数
    maxInitialRequests: 4, // 限制入口的拆分数量
    name: true, // 打包后的名称，默认是 chunk 的名字通过分割符分割开，如vendor~
    automaticNameDelimiter: "~", // 默认 webpack 会使用入口名和代码块的名称生成命名，如 vendor~main.js
    cacheGroup: { // 满足不同条件的缓存组
      chunks: "all",
      test: /node_modules/, // 条件
      priority: -10, // 优先级，一个 chunk 可能满足多个缓存组，会被抽取到优先级高的缓存组中，为了能够让自定义缓存组有更高的
    },
    common: {
      chunks: "all",
      minSize: 0, // 最小提取字节数
      minChunks: 2, // 最少被几个 chunk 引用
      priority: -20,
      reuseExistingChunk: true, // 如果该 chunk 中引用了已经被抽取的 chunk，直接引用该 chunk，不会重复打包代码
    }
  }
}
```

## 如何通过 webpack 来优化前端性能

1. 压缩JS TerserPlugin
2. 压缩CSS OptimizeCssAssetsPlugin
3. 压缩图片 image-webpack-loader
4. 单独提取CSS MiniCssExtractPlugin
5. 清除无用的CSS purgecss-webpack-plugin
6. Tree Shaking 
7. Scope hoisting 在生产环境默认开启，开发环境需要 webpack.optimize.ModuleConcatenationPlugin 插件，把导出直接编译成函数
8. 代码分割
9. 入口点分割
10. 动态导入和懒加载 import(xxx)
11. preload(资源的权重提高) & prefetch(可能用到，闲时加载)

## CDN

首页不缓存

第三方库强缓存

其他 chunk 变了，缓存失效（hash）

```js
{
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[hash].js',
    chunkFileName: '[name].[hash].chunk.js',
    publicPath: 'https://img.yzl.xyz'
  }
}

new MiniCssExtractPlugin({
  filename: '[name].[chunk].css'
})
```

## hash、chunkhash、conenthash 的区别

文件指纹

`[name].[hash].js` 占位符

ext 资源后缀
name 文件名称
path 相对路径
folder 文件所在的文件夹
hash 每次webpack构建生成的唯一的hash值
chunkhash 根据chunk生产的hash值，来源于同一个chunk，则hash值就一样
contenthash 根据内容生产hash值，文件内容相同，hash值就相同

## 对 bundle 体积进行监控和分析 BundleAnalyzerPlugin

```js
new BundleAnalyzerPlugin({
  analyzerMode: "disabled", // 不启动展示打包的服务和页面
  generateStatsFile: true // 生成 stats.json 文件
})
```

```shell
webpack --profile --json -> stats.json # 生成文件
webpack-bundle-analyzer --port 8888 ./dist/stats.json # 基于已有的文件查看依赖图
```

## 如何提高 webpack 的构建速度

**费时分析**

```js
const smw = new SpeedMeasureWebpackPlugin()

module.export = swm.wrap({
  // webpack 配置
})
```

**缩小范围**

1. 指定 `extensions: ['.js', '.jsx']`，不用在 require 或 import 时，添加扩展名
2. alias 别名，加快查找模块的速度, `alias: { "bootstrap": path.resolve(__dirname, 'node_modules/_bootstrap@3.3.7@bootstrap/dist/css/bootstrap.css') }`，写死部分模块的路径
3. 配置 resolve.modules: `[path.resolve(__dirname, 'node_modules')]`（原来会一级一级向上找）
4. resolve.mainFields 查找包时，要导入的文件，从该包的 package.json 文件中查找 （target 为 web 或 webworker，`["browser", "module", "main"]` ，否则为 `["module", "main"]`）
5. `resolve.mainFiles: ["index"]`，如果没有 package.json 时，可以直接查找文件
6. resolve.resolveLoader (modules, extensions, mainFields等字段)，用于查找loader
7. module.noParse 不需要解析依赖的第三方库，支持配置正则或函数
8. ignore-plugin 指定模块不打包 `new Webpack.IgnorePlugin(/文件夹/, /模块/)`
9. oneOf 只匹配到一个 loader 就不继续匹配了

**日志优化**

friendly-errors-webpack-plugin

**利用缓存**

```js
{
  use: [
    "cache-loader"
    {
      loader: "babel-loader"
      options: {
        presets: ['@babel/preset-env'],
        cacheDirectory: true // 开启babel缓存
      }
    }
  ]
}
```

hard-source-webpack-plugin
为模块提供中间缓存，webpack5内置

**多进程处理**

thread-loader 线程loader
放在这个 loader 后面的 loader 会在一个单独的 work pool 中运行

**动态链接库**

.dll 后缀，包含给其他模块调用的函数和数据

dll-plugin 打包动态链接库
dll-reference-plugin 在配置文件中引入动态链接库

webpack.dll.config.js

```js
const path = require("path")
const DllPlugin = require("webpack/lib/DllPlugin")

module.exports = {
  mode: "development",
  entry: {
    react: ["react", "react-dom"],
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].dll.js", // react.dll.js
    library: "_dll_[name]" // window._dll_react
  },
  plugins: [
    new DllPlugin({
      name: "_dll_[name]",
      path: path.resolve(__dirname, "dist", "[name].manifest.json") // react.manifest.json 全局变量里有哪些模块的清单
    })
  ]
}
```

```shell
webpack --config webpack.dll.config.js -mode=development
```

使用动态链接库文件

```js
new DllReferencePlugin({
  manifest: require("./dist/react.manifest.json")
})
```

`import React from 'react'` 时，如果上面清单有，就不打包了，直接去全局变量里找了（要引入打包后的 react.dll.js）

## 如何编写 loader

loader 的叠加顺序

post 后置、inline 内联、normal 正常、pre 前置

从右往左

```js
function loader(source) {
  return source + '// 这是注释'
}
```

pitch 是从左往右的，和 loader 执行顺序相反

```js
loader.pitch = function () {
  return 'pitch'
}
```

如果一个 pitch 有返回值，则将结果给前一个 loader，并从右往左执行

## 写loader的思路

```js
const babel = require('babel/core')
function loader(source, inputSourceMap, data) {
  const options = {
    presets: ['@babel/preset-env'],
    inputSourceMap: inputSourceMap,
    filename: this.request.split('!')[1].split('/').pop()
  }

  const { code, map, ast } = babel.transform(source, options)

  return this.callback(null, code, map, ast)
}

module.exports = loader
```

```js
resolveLoader: {
  alias: {
    // 配置别名
    "babel-loader": resolve("xxxx/babel-loader.js")
  },
  // 或者配置loader加载目录
  modules: [path.resolve('./loaders'), 'node_modules']
}

// 然后
{
  test: /\.js$/,
  use: ['babel-loader']
}
```

## 写 plugins 的思路

1. 看webpack源码，照着它内置的插件去实现

## webpack 打包的原理，babel 抽象语法树

实现一个非常简单的 webpack

```js
const fs = require('fs') // 读文件
const path = require('path') // 处理路径
const types = require('babel-types') // 处理各种节点类型，把节点类型处理成对象
const parser = require('@babel/parser') // 把源代码处理成抽象语法树
const traverse = require('@babel/traverse').default // 遍历语法树
const generate = require('@babel/generate').default // 语法树 -> 源代码

const baseDir = process.cwd().replace(/\\/g, path.posix.sep) // 拿到根目录（path.posix.sep === '/'）
const entry = path.posix.join(baseDir, 'src/index.js') // 拿到入口文件

const modules = [] // 本次编译的所有模块
const entryModule = buildModule(entry)
const content = `
(function (modules) {
  function __webpack_require(moduleId) {
    var module = {
      i: moduleId,
      exports: {}
    }
    module[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    )

    return module.exports
  }
  return __webpack_require__("${entryModule.id}")
})({
  ${
    modules.map(
      module => `"${module.id}": function (module, exports, __webpack_require__) { ${module._source} }`
    ).join(",")
  }
})
`
fs.writeFileSync('./dist/bundle.js', content)


function buildModule(absolutePath) {
  const body = fs.readFileSync(absolutePath, 'utf-8') // 读取文件内容
  const ast = parser.parse(body, {
    sourceType: 'module'
  }) // 源代码转成抽象语法树
  const moduleId = './' + path.posix.relative(baseDir, absolutePath) // 拿到相对路径，作为 moduleId，比如 ./src/index.js，在 webpack 里面，模块 id 都是相对（相对根目录）路径
  const module = { id: moduleId, deps: [] } // 声明一个模块对象
  
  traverse(ast, {
    // 处理函数调用的节点
    CallExpression({ node }) {
      if (node.callee.name === 'require') {
        node.callee.name = '__webpack_require__'
        const moduleName = node.arguments[0].value // ./title.js

        const dirname = path.posix.dirname (absolutePath) // 当前模块的父目录的绝对路径 c://xxx/xxx/src
        const depPath = path.posix.join(dirname, moduleName) // 模块的依赖的路径 c://xxx/xxx/src/title.js
        const depModuleId = './' + path.posix.relative(baseDir, depPath) // 依赖的模块 id
        node.arguments = [types.stringLiteral(depModuleId)] // 把 __webpack_require__ 的参数替换为“依赖模块id”
        module.deps.push(buildModule(depPath))
      }
    }
  })

  const { code } = generate(ast) // 重新生成后的代码
  module._source = code
  modules.push(module)

  return module
}
```

## tree-shaking

```js
import { flatten, concat } from 'lodash'
// 转换为
import { flatten } from 'lodash/flatten'
import { concat } from 'lodash/concat'
```

```js
{
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ['@babel/preset-env'],
            plugins: [["import" /* babel-plugin-import */, { libraryName: "lodash" }]]
          }
        }
      }
    ]
  }
}
```

babel-plugin-import 实现原理

```js
const types = require('babel-types')
const visitor = {
  ImportDeclartion: {
    enter(path, state) {
      const specifiers = path.node.specifiers
      const source = path.node.source // lodash

      if (state.opts.libraryName === source.value && !types.isImportDefaultSpecifier(specifiers[0])) {
        const declarations = specifiers.map(specifier => {
          return types.ImportDeclartion(
            [types.ImportDefaultSpecifier(specofier.local)],
            types.stringLiteral(`${source.value}/${specifier.local.name}`)
          )
        })
        path.replaceWithMultiple(declarations)
      }
    }
  }
}

// babel 插件需要返回的东西
module.exports = function (babel) {
  return {
    visitor
  }
}
```

## 热更新 HMR

不刷新页面的情况下更新页面，保持应用状态，提升开发效率

1. 创建 webpack 实例，得到 compiler
2. 创建一个 webpack-dev-server（server.listen(1234)）
3. webpack-dev-middleware（express中间件），用于提供编译后的静态文件服务，如果路径是 `/`，返回 index.html，如果访问文件，（没有文件404），把 mimetype 写到 content-type 响应头，响应文件
4. socket 服务器
5. 启动服务器，compiler.watch({}, (err) => {})
6. `compiler.hooks.done.tap('webpack-dev-server', (stats) => { lastHash = stats.hash; sockets.forEach(socket = > (socket.emit('hash', stats.hash), socket.emit('ok'))) })`，sockets 是 socket 连接时添加的一个个 websocket 连接实例

7. 客户端 currenthash， hotcurrenthash（上一次的hash，下文用lasthash替代）， socket连接，监听hash事件，修改currenthash，监听ok事件，执行hotCheck
8. 如果没有lastHash 或者 lastHash 和 currenthash 一样，则return
9. 否则调用hotDownloadManifest请求hot-update.json（/ + lasthash + './hot-update.json'），`hotDownloadManifest().then(res => { const chunkIds = []; /* [{main: true}] */ chunkIds.forEach(chunkId => hotDownloadUpdateChunk(chunkId)) /* 通过jsonp获取到最新的模块代码 */ })`
10. `function hotDownloadUpdateChunk() { script.src = '/' + chunkId + '.' + lashhash + '.hot-update.json' }`，然后这个 script 里会调用 webpackHotUpdate

```js
// 注入
import '../webpackHotDevClient'
if (module.hot) {
  module.hot.accpet(['./title.js'], () => { /* 触发 */ })
}
// webpackHotUpdate('main', { [`${chunkId}`]: function(module, exports) { eval(xxx 新代码) } })
function webpackHotUpdate(chunkId, moreModules) {
  for (const moduleId in moreModules) {
    const oldModules = __webpack_require__.c[moduleId] // 获取老模块
    const { parent, children } = oldModules // 谁引用了它，就是它的 parents
    const module = (__webpack_require__.c[moduleId] = {
      i: moduleId,
      exports: {},
      parents,
      children,
      hot: window.hotCreateModule()
    })
    moreModules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    )
    parents.forEach(parent => {
      const parentModule = __webpack_require__.c[parent]
      parentModule.hot && parentModule.hot._accpetedDependencies[moduleId] && parentModule.hot._accpetedDependencies[moduleId]
    })
    lastHash = currenthash
  }
}

window.hotCreateModule = function() {
  const hot = {
    _accpetedDependencies: {},
    accpet: function(deps, callback) {
      for (let i = 0; i < deps.length, i++) {
        hot._accpetedDependencies[deps[i]] = callback
      }
    }
  }
  return hot
}
```