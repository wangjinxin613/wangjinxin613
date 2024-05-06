---
title: vue面试题复习
date: 2024-05-05 14:52:32
tags:
	- 面试题
---
1. Vue模版引擎

Vue会根据其规定的模板语法规则，将其解析成AST语法树（其实就是用一个树状的大对象来描述我们所谓的“HTML”）；然后会对这个大对象进行一些初步处理，比如标记没有动态绑定值的节点；最后，会把这个大对象编译成render函数，并将它绑定在组件的实例上。这样，我们所认为的“HTML”就变成了JavaScript代码，可以基于JavaScript模块规则进行导入导出，在需要渲染组件的地方，就调用render函数，根据组件当前的状态生成虚拟dom，然后就可以根据这个虚拟dom去更新视图了。
————————————————
版权声明：本文为CSDN博主「Yaalon Cui」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_42482733/article/details/111502669

2. Axios如何中断请求

XMLHttpRequest.abort()   方法用于终止 `XMLHttpRequest` 对象的请求，该方法没有参数，也没有返回值。

```js
// axios 0.22.0版本之前
// 创建一个CancelToken对象： 
const source = axios.CancelToken.source();
// 将CancelToken对象传递给请求的config中：
axios.get('/api/data', {
  cancelToken: source.token
}).then(response => {
  console.log(response.data);
}).catch(error => {
  if (axios.isCancel(error)) {
    console.log('请求已被取消：', error.message);
  } else {
    console.log('请求出错：', error.message);
  }
});
// 在需要中断请求的地方，调用cancel方法：
source.cancel('请求被用户取消');
————————————————
版权声明：本文为CSDN博主「ATalk机器人」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/liyananweb/article/details/130334371
```

```
// 0.22之后的版本
// 创建请求控制器 
this.controller = new AbortController();
console.log("初始声明的请求控制器------", this.controller);

// 第一种方法：绑定事件处理程序
this.controller.signal.addEventListener("abort", () => {
   console.log("请求已终止，触发了onabort事件");
   // 进行后续处理
});

// 第二种方法：try...catch
try {
    // 发送文件上传请求
    const res = await this.$axios.post(api.Upload, uploadData, {
     timeout: 0, // 设置超时时间为 0/null 表示永不超时
     signal: this.controller.signal, // 绑定取消请求的信号量
	});
} catch (error) {
    console.log("终止请求时catch的error---", error);
    // 判断是否为取消上传
    if (error.message == "canceled"){
        // 进行后续处理
    };
}

// 终止请求
this.controller.abort();
————————————————
版权声明：本文为CSDN博主「努力的小朱同学」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_45092437/article/details/131211952
```

3. vue组件的设计原则

单一职责原则、扩展性、通用性、可配置性、可测试的单元、幂等性

4. Vue中data和methods的方法名可以重复吗？

可以重复，但是重名的话只会执行data里的，不会执行methods的

5. slot是什么，有哪些类型

插槽、默认插槽、具名插槽、作用域插槽

6. vue如何检测路由改变

```
1. 组件内watch监听$router
2. router.beforeEach((to, from, next) => {}) 全局路由守卫
3. 路由独享的守卫 
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
4. 组件内的守卫(组件内钩子)
 beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当钩子执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
 
  }
————————————————
版权声明：本文为CSDN博主「笑魇轻轻」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_45259626/article/details/106076863
```

7. Vue-loader是什么

  简单的说，他就是基于webpack的一个的loader，解析和转换 .vue 文件，提取出其中的逻辑代码 script、样式代码 style、以及 HTML 模版 template，再分别把它们交给对应的 Loader 去处理，核心的作用，就是提取，划重点。