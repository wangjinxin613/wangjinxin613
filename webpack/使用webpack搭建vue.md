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
    filename: "boundle.js",
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
"build": "webpack --config webpack.config.js"
```

<img src=".\img\image-20200813134734963.png" alt="image-20200813134734963" style="zoom: 80%;float:left" />

### 执行打包

```shell
npm run build
```

打包后会在dist目录生成打包好后的文件。

