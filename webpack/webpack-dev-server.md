# devServer

devServer是用来提高开发效率的，不是用devServer来做打包的，它提供了一些配置项，可以用于改变devServer的默认行为，要配置devServer，除了可以在配置文件里通过devServer传入参数，还可以通过命令行传入参数。
 ⚠️注意！！！只有在通过devServer启动webpack时，配置文件里的devServer才会生效，因为这些参数所对应的功能都是devServer提供的，webpack本事并不认识devServer的配置项。
 devServer的配置项也很多，我们老规矩，介绍几个常用的，感兴趣的可以戳这里（[开发中 Server(devServer)](https://www.webpackjs.com/configuration/dev-server/)）去官网查看。

### hot

#### hot介绍

hot配置是否启用模块的热替换功能，devServer的默认行为是在发现源代码被变更后，通过自动刷新整个页面来做到事实预览，开启hot后，将在不刷新整个页面的情况下通过新模块替换老模块来做到实时预览。

#### 使用方法

hot 有两种使用方法，

###### 1、通过配置文件，具体如下

```javascript
const webpack = require('webpack');
const path = require('path');
let env = process.env.NODE_ENV == "development" ? "development" : "production";
const config = {
  mode: env,
 devServer: {
     hot:true
 }
}
  plugins: [
     new webpack.HotModuleReplacementPlugin(), //热加载插件
  ],
module.exports = config;
```

需要引入一个plugins:webpack.HotModuleReplacementPlugin(),
 看看官网怎么说

> Note that webpack.HotModuleReplacementPlugin is required to fully enable HMR. If webpack or webpack-dev-server are launched with the --hot option, this plugin will be added automatically, so you may not need to add this to your webpack.config.js. See the HMR concepts page for more information.

大致意思是说，你如果通过--hot配置，就会自动引用webpack.HotModuleReplacementPlugin这个插件，所以我们不建议这么配置hot，麻烦
 用第二种
 ###### 2、通过命令行
 在package.json中的script中处理

```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "NODE_ENV=development  webpack-dev-server --config  webpack.develop.config.js --hot",
  },
```

如上在start中添加--hot即可，省去了繁琐的配置

### host

配置devServer服务监听的地址，例如如果想让局域网内的其他用户访问自己的设备，可以将host配置为自己本机的IP地址，
 分享一个可以自动获取本地ip地址的方法

```javascript
const webpack = require('webpack');
const path = require('path');
let env = process.env.NODE_ENV == "development" ? "development" : "production";
const os = require('os')
// 动态获取 host || 也可在package.json中配置 HOST （--host 0.0.0.0)
let arr = []
let HOST
for (let key in os.networkInterfaces()) {
  os.networkInterfaces()[key].forEach((item) => {
    if (item.family === 'IPv4' && item.address.indexOf('192.168.') !== -1) {
      arr.push(item.address)
    }
  })
}
HOST = arr[0]

const config = {
  mode: env,
 devServer: {
     hot:true,
     hsot:HOST
 }
}
  plugins: [
     new webpack.HotModuleReplacementPlugin(), //热加载插件
  ],
module.exports = config;
```

通过命令行也是可以的

```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "NODE_ENV=development  webpack-dev-server --config  webpack.develop.config.js --hot --host 0.0.0.0",
  },
```

host的默认地址是127.0.0.1

### port

这个没什么说的了就是端口号，默认是8080，如果占用就换成8081或者1234，都可以

### open

open 在devServer启动且第一次构建完成时，自动用我们的系统的默认浏览器去打开要开发的网页，
 使用方法
 可在配置文件中使用

```javascript
 devServer: {
     open:true
 }
}
```

也可以在命令行中使用

```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "NODE_ENV=development  webpack-dev-server --config  webpack.develop.config.js --hot --open --host 0.0.0.0",
  },
```

还有类似的openPage，用来打开指定网页，具体可去官网查看

### 其他

还有很多其他常用的配置，例如
 [contentBase](https://www.webpackjs.com/configuration/dev-server/#devserver-contentbase)配置devServer，Http服务器的文件根目录，
 [proxy](https://www.webpackjs.com/configuration/dev-server/#devserver-proxy)配置代理，处理本地跨域，
 这些官网介绍的都很详细，就不一一介绍了
 今天就到这里，明天我们不见不散

[原文链接](https://www.qdtalk.com/2018/11/13/从零搭建webpack4之必须要懂的devserver/)