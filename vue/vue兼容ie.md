# vue兼容ie浏览器

### [babel-polyfill](https://www.cnblogs.com/princesong/p/6728250.html)

babel-polyfill的作用是是让ie浏览器支持es6语法

安装

```shell
  npm install --save-dev babel-polyfill
```

引用方式有三种：

1. `require("babel-polyfill");`

2. `import "babel-polyfill";`

3. webpack.config.js

```
module.exports = {
   entry: ["babel-polyfill", "./app/js"]
};
```

注：第三种方法适用于使用webpack构建的同学，加入到webpack配置文件(webpack.config.js)entry项中

#### axios在安卓低版本兼容性处理

在较低版本的安卓手机中发现发现封装的axios请求无效，主要原因还是低版本的安卓手机无法使用promise

**注意：**安卓4.3以下的手机不支持axios的使用，无法使用promise，加上 polyfill就可以了。

**解决方案：** （1）、项目中安装 `es6-promise`

```undefined
npm install es6-promise -s
```

（2）、引入 `es6-promise`

```jsx
import promise from 'es6-promise'
```

（3）、注册 `es6-promise` **(一定要在axios之前注册)**

```jsx
// 注意： es6-promise   一定要在 axios 之前注册

promise.polyfill()

或者

require('es6-promise').polyfill();
```

