---
title: HTMLPlugin
date: 2024-05-05 14:52:32
tags:
	- webpack
---
### HtmlWebpackPlugin

这个plugin曝光率很高，他主要有两个作用

- 为html文件中引入的外部资源如script、link动态添加每次compile后的hash，防止引用缓存的外部文件问题
- 可以生成创建html入口文件，比如单页面可以生成一个html文件入口，配置N个html-webpack-plugin可以生成N个页面入口
   [github](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fjantimon%2Fhtml-webpack-plugin)上有些关于htmlwebpackplugin的属性介绍

#### title

生成html文件的标题

#### filename

输出的html的文件名称

#### template

html模板所在的文件路径
 根据自己的指定的模板文件来生成特定的 html 文件。这里的模板类型可以是任意你喜欢的模板，可以是 html, jade, ejs, hbs, 等等，但是要注意的是，使用自定义的模板文件时，需要提前安装对应的 loader， 否则webpack不能正确解析。
 如果你设置的 title 和 filename于模板中发生了冲突，那么以你的title 和 filename 的配置值为准。

#### inject

注入选项。有四个选项值 true, body, head, false.

1. true：默认值，script标签位于html文件的 body 底部
2. body：script标签位于html文件的 body 底部（同 true）
3. head：script 标签位于 head 标签内
4. false：不插入生成的 js 文件，只是单纯的生成一个 html 文件

#### favicon

给生成的 html 文件生成一个 favicon。属性值为 favicon 文件所在的路径名

#### minify

minify 的作用是对 html 文件进行压缩，minify 的属性值是一个压缩选项或者 false 。默认值为false, 不对生成的 html 文件进行压缩。
 下面罗列了一些常用的配置：



```javascript
plugins:[
        new HtmlWebpackPlugin({
//部分省略，具体看minify的配置
minify: {
     //是否对大小写敏感，默认false
    caseSensitive: true,
    
    //是否简写boolean格式的属性如：disabled="disabled" 简写为disabled  默认false
    collapseBooleanAttributes: true,
    
    //是否去除空格，默认false
    collapseWhitespace: true,
    
    //是否压缩html里的css（使用clean-css进行的压缩） 默认值false；
    minifyCSS: true,
    
    //是否压缩html里的js（使用uglify-js进行的压缩）
    minifyJS: true,
    
    //Prevents the escaping of the values of attributes
    preventAttributesEscaping: true,
    
    //是否移除属性的引号 默认false
    removeAttributeQuotes: true,
    
    //是否移除注释 默认false
    removeComments: true,
    
    //从脚本和样式删除的注释 默认false
    removeCommentsFromCDATA: true,
    
    //是否删除空属性，默认false
    removeEmptyAttributes: true,
    
    //  若开启此项，生成的html中没有 body 和 head，html也未闭合
    removeOptionalTags: false, 
    
    //删除多余的属性
    removeRedundantAttributes: true, 
    
    //删除script的类型属性，在h5下面script的type默认值：text/javascript 默认值false
    removeScriptTypeAttributes: true,
    
    //删除style的类型属性， type="text/css" 同上
    removeStyleLinkTypeAttributes: true,
    
    //使用短的文档类型，默认false
    useShortDoctype: true,
    }
    }),
]
```

#### hash

hash选项的作用是 给生成的 js 文件一个独特的 hash 值，该 hash 值是该次 webpack 编译的 hash 值。默认值为 false 。同样看一个例子。



```javascript
plugins: [
    new HtmlWebpackPlugin({
        hash: true
    })
]
```

编译打包后



```html
<script type=text/javascript src=bundle.js?22b9692e22e7be37b57e></script>
```

执行 webpack 命令后，你会看到你的生成的 html 文件的 script 标签内引用的 js 文件，是不是有点变化了。
 bundle.js 文件后跟的一串 hash 值就是此次 webpack 编译对应的 hash 值。

#### cache

默认是true的，表示内容变化的时候生成一个新的文件。

#### showErrors

这个我们自运行项目的时候经常会用到，showErrors 的作用是，如果 webpack 编译出现错误，webpack会将错误信息包裹在一个 pre 标签内，属性的默认值为 true ，也就是显示错误信息。
 开启这个，方便定位错误

#### chunks

chunks主要用于多入口文件，当你有多个入口文件，那就回编译后生成多个打包后的文件，那么chunks 就能选择你要使用那些js文件



```javascript
entry: {
    index: path.resolve(__dirname, './src/index.js'),
    devor: path.resolve(__dirname, './src/devor.js'),
    main: path.resolve(__dirname, './src/main.js')
}

plugins: [
    new httpWebpackPlugin({
        chunks: ['index','main']
    })
]
```

那么编译后：



```javascript
<script type=text/javascript src="index.js"></script>
<script type=text/javascript src="main.js"></script>
```

而如果没有指定 chunks 选项，默认会全部引用。

#### excludeChunks

排除掉一些js,



```javascript
entry: {
    index: path.resolve(__dirname, './src/index.js'),
    devor: path.resolve(__dirname, './src/devor.js'),
    main: path.resolve(__dirname, './src/main.js')
}

plugins: [
    new httpWebpackPlugin({
     excludeChunks: ['devor.js']//和的等等效
    })
]
```

那么编译后：



```javascript
<script type=text/javascript src="index.js"></script>
<script type=text/javascript src="main.js"></script>
```

### webpack4与html-webpack-plugin

用最新版本的的 html-webpack-plugin你可能还会遇到如下的错误：
 throw new Error('Cyclic dependency' + nodeRep)

产生这个 bug 的原因是循环引用依赖，如果你没有这个问题可以忽略。
 目前解决方案可以使用 Alpha 版本，npm i --save-dev html-webpack-plugin@next
 或者加入chunksSortMode: 'none'就可以了。
 但仔细查看文档发现设置成chunksSortMode: 'none'这样是会有问题的。
 这属性会决定你 chunks 的加载顺序，如果设置为none，你的 chunk 加载在页面中加载的顺序就不能够保证了，可能会出现样式被覆盖的情况。比如我在app.css里面修改了一个第三方库element-ui的样式，通过加载顺序的先后来覆盖它，但由于设置为了none，打包出来的结果变成了这样：

```html
<link href="/app.8945fbfc.css" rel="stylesheet">
<link href="/chunk-elementUI.2db88087.css" rel="stylesheet">
```

app.css被先加载了，之前写的样式覆盖就失效了，除非你使用important或者其它 css 权重的方式覆盖它，但这明显是不太合理的。
 vue-cli正好也有这个相关 issue，尤雨溪也在不使用@next版本的基础上 hack 了它，有兴趣的可以自己研究一下，

其它 html-webpack-plugin 的配置和之前使用没有什么区别。

参考链接
 [html-webpack-plugin用法全解](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000007294861)
 [插件 html-webpack-plugin 的详解](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000013883242)
 [手摸手，带你用合理的姿势使用webpack4（上）](https://www.jianshu.com/p/1fcc2f8ed8bb)

