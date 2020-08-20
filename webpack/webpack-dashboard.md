## webpack-dashboard

webpack-dashboard是用于改善开发人员使用[webpack](https://link.jianshu.com?t=http%3A%2F%2Fwebpack.github.io%2F)时控制台用户体验的一款工具。它摒弃了webpack（尤其是使用dev server时）在命令行内诸多杂乱的信息结构，为webpack在命令行上构建了一个一目了然的仪表盘(dashboard)，其中包括**构建过程**和**状态**、**日志**以及涉及的**模块列表**。有了它，你就可以更加优雅的使用webpack来构建你的代码。

#### 1.1 它是什么

简单地说，[webpack-dashboard](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FFormidableLabs%2Fwebpack-dashboard)就是把原先你使用webpack时（特别是使用webpack dev server时）命令行控制台打印的日志：

```csharp
Creating an optimized production build...
Compiled with warnings.

./src/App.js
  Line 10:  'One' is assigned a value but never used  no-unused-vars
  Line 16:  'Two' is assigned a value but never used  no-unused-vars

Search for the keywords to learn more about each warning.
To ignore, add // eslint-disable-next-line to the line before.

File sizes after gzip:

  97.39 KB  build/static/js/main.4348a7e3.js
  69.13 KB  build/static/js/0.f84fa404.chunk.js
  39.43 KB  build/static/js/2.81fe7f28.chunk.js
  39.42 KB  build/static/js/1.a2c322a6.chunk.js

The project was built assuming it is hosted at the server root.
To override this, specify the homepage in your package.json.
For example, add this to build it for GitHub Pages:
```

转换成了这样：

![img](https:////upload-images.jianshu.io/upload_images/1637794-41c9ce180f1a1ab1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

看到这里，是不是觉得整个人生都变美好了呢。仔细看，这个dashboard里面按日志(Log)、状态(Status)、运行(Operation)、过程(Progess)、模块(Modules)、产出(Assets)这6个部分将信息区分开来。用官方的话，这将会给你一种在NASA工作的即使感，哈哈。

#### 1.2 如何使用

其实安装和使用webpack-dashboard的过程非常简单，首先使用npm本地安装它，到你基于webpack的前端项目上：

```undefined
npm install webpack-dashboard --save-dev
```

如果你利用webpack-dev-server启动了一个server，而不是express的话，可以直接在webpack.config.js里面初始化dashboard。

首先，导入dashboard和其对应的插件，并创建一个dashboard的实例：

```jsx
const Dashboard = require('webpack-dashboard');
const DashboardPlugin = require('webpack-dashboard/plugin');
const dashboard = new Dashboard();
```

然后，在对应的plugins里面添加DashboardPlugin：

```cpp
plugins: [
  new DashboardPlugin(dashboard.setData)
]
```

最后，你需要让dev server在静默的状态中启动（主要是为了去掉多余的日志），要实现这一点，你可以像官方的做法那样，在WebpackDevServer的构造函数里添加 quiet: true。

```cpp
new WebpackDevServer(
  Webpack(settings),
  {
    publicPath: settings.output.publicPath,
    hot: true,
    quiet: true, // lets WebpackDashboard do its thing
    historyApiFallback: true,
  }
).listen
```

当然，你也可以在npm的script里面启动dev server时添加quiet选项（我在尝试的时候选择这种简单的方式）。

```bash
"scripts": {
  "start": "webpack-dev-server --quiet"
},
```

这样，你就可以运行诸如npm start这样的命令启动你的server。然后，你就可以休息一下，泡杯咖啡，假装自己就是一位宇航员，静静地看着你的dashboard。

如下图所示，为我尝试时的截图：

![img](https:////upload-images.jianshu.io/upload_images/1637794-1ad7222734321c33.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

这里显示的字符串“越界”，是因为没有使用quiet模式

#### 1.3 最后

本文只介绍了基于webpack-dev-server的这一种使用情况，其他启动server的方式（比如express）或者其他情况可以参考[webpack-dashboard github官方仓库](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FFormidableLabs%2Fwebpack-dashboard)。

webpack-dashboard目前还处于初期阶段，所以必然还有一些值得注意或者值得改进的地方。如果你使用的是OS X自带的终端(Terminal)，需要确认“View → Allow Mouse Reporting”是使能(Enable)状态，如果你的终端没有这个功能的话，你或许可以尝试一下[iTerm2](https://link.jianshu.com?t=https%3A%2F%2Fwww.iterm2.com%2Findex.html)。另外，如果你忘记使用quiet模式或者你的某句日志或者名字过长，可能会导致显示的字符串“越界”，跑到另一个区域，看起来没有那么直接美观了。

最后，如果你想简单的看一下webpack-dashboard启动起来的效果，你可以参考使用[本文示例代码](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FYaowenjie%2FReact-learning%2Ftree%2Fmaster%2Flesson1)。

# jarvis

参考[zouhir/jarvis](https://github.com/zouhir/jarvis)

## 效果图

![这里写图片描述](https://img-blog.csdn.net/20180630233127343?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FjaGVueXVhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
它是在浏览器里展示的，所以它的自适应效果做的好，看的清楚。同时，它统计的信息也比较多。
`jarvis`是`Just A Rather Very Intelligent System`首字母缩写，意思是它是一个非常聪明的系统。你可以通过浏览器展示结果，它能收集webpack编译或者运行阶段的信息。

## 优点

- 界面美观
- 运行在浏览器里
- 很方便查看资源大小
- 错误提示做的非常好

## 安装模块

```
npm i -D webpack-jarvis1
```

## 在webpack.config.js配置插件

```
//引入模块
const Jarvis = require("webpack-jarvis");

/* the rest of your webpack configs */
//添加插件，指定监听端口是1337
plugins: [
  new Jarvis({
    port: 1337 // optional: set a port
  })
];12345678910
```

## 查看效果

启动编译`npm start`或`npm run dev`（这些命令在package.json的scripts配置的）。
访问`http://localhost:1337/`即能看到好看的、美丽的界面了。

> 启动里，如果报`Error: listen EADDRINUSE xxx`错误，那是端口被占用了。打开任务管理器，然后杀死node.exe或nodex.js进程即可。

## 可选参数

jarvis提供了一些可选参数供自定义

- `port`:监听的端口，默认`1337`，监听面板将监听这个端口，通常像`http://localhost:port/`
- `host`:域名，默认localhost,也可以设为`0.0.0.0`，不限制域名。
- `watchOnly`:仅仅监听编译阶段。默认为true,如果高为false，jarvis不仅仅运行在编译阶段，在编译完成后还保持运行状态。

比如

```
 new Jarvis({
    host: 0.0.0.0
  })123
```

------

# 总结

这里介绍到这里。个人推荐jarvis方式，在浏览器打展示效果。