---
title: 使用webpack搭建vue
date: 2024-05-05 14:52:32
tags:
	- webpack
---
### 生成package.json

```shell
npm init
```

### 安装依赖

```shell
npm i webpack vue vue-loader css-loader vue-template-compiler
```

### 新建文件夹src，创建文件app.vue

```javascript
<template>
    <div class="text">{{text}}</div>
</template>

<script>
export default {
    data(){
        return{
            text:'abc'
        }
    }
}
</script>

<style scoped>
    .text{
        color:red;
    }
</style>
```

### 创建webpack.config.js （帮助打包前端资源）

```javascript
// 打包前端资源
const path = require('path')
const VueLoaderPlugin = require("vue-loader/lib/plugin");

module.exports = {
  entry: path.join(__dirname, "src/index.js"),
  output: {
    filename: "app.js",
    path: path.join(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /.vue$/,
        loader: "vue-loader",
      },
      {
        test: /\.css$/,
        loader: "css-loader",
      }
    ],
  },
  plugins: [
    // 请确保引入这个插件！
    new VueLoaderPlugin(),
  ],
};
```

### 在src目录中创建index.js （入口文件）

```javascript
// 入口文件
import Vue from 'vue'
import App from './app.vue'

const root=document.createElement("div")
document.body.appendChild(root)

new Vue({
    render:(h)=>h(App)
}).$mount(root)
```

### 在package.json中的scripts里添加一个命令

```
...
"build": "webpack --config webpack.config.js"
...
```

### 执行打包

```shell
npm run build
```

打包后会在dist目录生成打包好后的文件。

### webpack插件 - plugin

plugin是用于扩展webpack的功能，各种各样的plugin几乎可以让webpack做任何与构建相关的事情。
plugin的配置很简单，plugins配置项接收一个数组，数组里的每一项都是一个要使用的plugin的实例，plugin需要的参数通过构造函数传入。

举个例子

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
  plugins: [
    new HtmlWebpackPlugin({ // 打包输出HTML
      title: 'Hello World app',
      minify: { // 压缩HTML文件
        removeComments: true, // 移除HTML中的注释
        collapseWhitespace: true, // 删除空白符与换行符
        minifyCSS: true// 压缩内联css
      },
      filename: 'index.html',
      template: 'index.html'
    }),
  ]
```

使用plugin的难点在于plugin本身的配置项，而不是如何在webpack中引入plugin，几乎所有webpack无法直接实现的功能，都能找到开源的plugin去解决，我们要做的就是去找更据自己的需要找出相应的plugin。

### HTMLPlugin

虽然现在可以打包了，但是只生成了js文件，并没有生成html文件，我们需要引入一个插件 HTMLPlugin。

在webpack.config.js中加入以下代码：

```javascript
const HTMLPlugin = require('html-webpack-plugin');
...
// 在插件列表中声明引用了HTMLPlugin插件
plugins: [
    new VueLoaderPlugin(),
    new HTMLPlugin({
        title: 'Hello World app',
    }) // 处理html模版
],
...
```

现在执行npm run build可以打包出index.html了，双击这个文件是可以直接在浏览器中运行的。

了解html-webpack-plugin的更多内容，详见  [HTMLPlugin.md](./HTMLPlugin.md)

### webpack-dev-server

每次打包一次再运行实在是太麻烦了，我们希望能够在一个服务端口上运行这个项目，并且代码有改动时能够热更新，这个时候就需要webpack-dev-server。

安装webpack-dev-server

```
npm install webpack-dev-server --save
```

修改 webpack.config.js

```javascript
// 打包前端资源
const path = require('path')
const VueLoaderPlugin = require("vue-loader/lib/plugin");
const HTMLPlugin = require('html-webpack-plugin');
const isDev = process.env.NODE_ENV === 'development';  // 判断当前环境是否为开发环境

const config = {
  entry: path.join(__dirname, "src/index.js"),
  output: {
    filename: "app.js",
    path: path.join(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /.vue$/,
        loader: "vue-loader",
      },
      {
        test: /\.css$/,
        loader: "css-loader",
      }
    ],
  },
  plugins: [
    new VueLoaderPlugin(),
    new HTMLPlugin({
      title: '使用webpack搭建一个vue应用'
    })
  ],
};

if(isDev) {
  Object.assign(config, {
    // 调试代码时可以看到自己原本的代码，而不是编译后的
    devtool: '#cheap-module-eval-source-map',
    devServer: {
      port: 8000,
      host: '0.0.0.0',
      overlay: {
          errors: true // 将webpack编译的错误显示在网页上面
      },
      open: true, // 在启用webpack-dev-server时，自动打开浏览器
      hot: true 
    }
  })
}

module.exports = config;
```

package.json中的scripts里添加一个命令

```javascript
...
"scripts": {
    ...
    "dev": "webpack-dev-server --config webpack.config.js"
}
...
```

执行命令 `npm run dev` 就会启动一个8080端口，就可以在浏览器`http://localhost:8080` 访问到这个项目了。

### 图片的加载

目前的配置是不支持加载图片的，接下来我们让它支持加载图片。

使用`file-loader`处理文件的导入

```shell
npm install file-loader --save-dev
```
在webpack.config.js中修改
```javascript
module: {
    rules: [
    	...
        {
            test: /\.(gif|jpg|jpeg|png|svg)$/, // 处理图片文件
            use: {
              loader: 'file-loader',
              options: {
                name: '[name]_[hash].[ext]',
              }
            }
        }
        ...
      ]
}
```

经测试使用`file-loader`仅支持 `import img from "./assets/logo.jpg";` 和 ` background: url('./assets/logo.jpg');` 的引用，并不能支持 ` <img src="./assets/logo.jpg" >`，会将图片地址解析成`[object module]` ，查了资料后发现：

默认情况下，file-loader生成使用ES模块语法的JS模块。在某些情况下，使用ES模块是有益的，比如在模块连接和树抖动的情况下。

简而言之就是file-loader新版本默认使用了esModule语法，造成了引用图片文件时的方式和以前的版本不一样，通过查看其仓库的release会发现：在v4.3.0版本就引入这一新特性。

解决方法是在webpack的file-loader配置项里，设置esModule为false

```javascript
module: {
    rules: [
    	...
        {
            test: /\.(gif|jpg|jpeg|png|svg)$/, // 处理图片文件
            use: {
              loader: 'file-loader',
              options: {
                name: '[name]_[hash].[ext]',
                esModule: false  
              }
            }
        }
       ...
      ]
}
```

除了file-loader还可以使用url-loader，url-loader是在file-loader的基础上做了一层封装，能够让较小的图片打包成base64

```shell
npm install url-loader --save-dev
```

```javascript
// 将file-loader换成url-loader，并加入	limit 属性
use: {
    loader: 'url-loader',
    options: {
        name: '[name]_[hash].[ext]',
        esModule: false,
        limit: 4096088  // 表示小于这个大小的图片的地址都会转化成base64
  	}
}
```

### 支持less

less是css的预处理语言，它扩展了css语言，下面让我们的webpack支持less

安装less相关依赖

```shell
npm i less-loader less --save-dev
```

在webpack.config.js中加入less的对应规则

```javascript
{
  test: /\.less$/,
    use: [
      {
        loader: 'style-loader'
      },
      {
        loader: 'css-loader'
      },
      {
        loader: 'less-loader'
      }
    ]
}
```

至此一个简单的vue脚手架就搭建好啦，如果你想更完善一点，后续可以添加更多的功能。

### 支持typescript

typescript是[JavaScript](https://baike.baidu.com/item/JavaScript)的一个超集，而且本质上向这个语言添加了可选的静态类型和基于类的[面向对象编程](https://baike.baidu.com/item/面向对象编程)。

安装依赖

```shell
npm install typescript ts-loader --save-dev
```

将入口文件文件名index.js改成index.ts

修改webpack配置

```javascript
...
// 入口文件地址改一下
entry: path.join(__dirname, "src/index.ts"),
...

// 加入对ts文件的处理规则
...
{
  test: /\.tsx?$/,
  loader: 'ts-loader',
  exclude: /node_modules/,
  options: {
  	appendTsSuffixTo: [/\.vue$/]
  }
}
...
```

创建typescript配置文件，tsconfig.json，这里的配置可以根据实际要求进行配置

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "strict": true,
    "jsx": "preserve",
    "importHelpers": true,
    "moduleResolution": "node",
    "experimentalDecorators": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "sourceMap": true,
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "src/*"
      ]
    },
    "lib": [
      "esnext",
      "dom",
      "dom.iterable",
      "scripthost"
    ]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "tests/**/*.ts",
    "tests/**/*.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

添加对vue的声明，在src目录下创建文件`shims-vue.d.ts`

```typescript
declare module "*.vue" {
  import Vue from "vue"
  export default Vue
}
```

需要注意的是如果在Vue原型当中添加了属性，也需要添加对应的声明，但是由于vue-router和vuex这种是作为Vue插件来使用的 (也就是按`Vue.use(Router)`这种方式来使用)，并且都已经支持TypeScript，所以并不需要单独来编写声明了。但是像axios这种并不是按Vue插件使用，需要用下面的方式添加到Vue原型当中

```javascript
Vue.prototype.$http = axios
```

那么使用了TypeScript之后，就需要对该属性进行声明
否则ts编译的过程就会认为该属性不存在

在src目录下创建`index.d.ts`

```typescript
import { AxiosInstance } from 'axios'

declare module 'vue/types/vue' {
  interface Vue {
    $http: AxiosInstance
  }
}
```

### vue-class-component和vue-property-decorator

[vue-class-component](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-class-component%5D(https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-class-component)) 是官方推出的vue对typescript支持的装饰器(库)，可以将Vue中的组件用类的方式编写，**vue-property-decorator**是[vue-class-component](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-class-component%5D(https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-class-component)) 的超集，下面我们来学习一下**vue-property-decorator**。

安装依赖包

```shell
npm install vue-property-decorator vue-class-component --save
```

组件就可以这么写了

```typescript
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'
@Component({
  filters: {
    toFixed: (num: number, fix: number = 2) => {
      return num.toFixed(fix)
    }
  }
})
export default class MyComponent extends Vue {
  public list: number[] = [0, 1, 2, 3, 4]
  get evenList() {
    return this.list.filter((item: number) => item % 2 === 0)
  }
}
</script>
```

### 集成eslint	

安装

```shell
npm install --save-dev eslint-loader eslint
```

在 `webpack.config.js` 中添加如下代码：

```css
{
    test: /\.js$/,
    loader: 'eslint-loader',
    enforce: "pre",
    include: [path.resolve(__dirname, 'src')], // 指定检查的目录
    options: { // 这里的配置项参数将会被传递到 eslint 的 CLIEngine 
        formatter: require('eslint-friendly-formatter') // 指定错误报告的格式规范
    }
}
```

最后，项目想要使用那些eslin规则，可以创建一个配置项文件 '.eslintrc.js'，代码如下:

```java
module.exports = {
    root: true, 
    parserOptions: {
        sourceType: 'module'
    },
    env: {
        browser: true,
    },
    rules: {
        "indent": ["error", 2],
        "quotes": ["error", "double"],
        "semi": ["error", "always"],
        "no-console": "error",
        "arrow-parens": 0
    }
}
```