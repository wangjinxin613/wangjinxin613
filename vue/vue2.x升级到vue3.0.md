### 开始升级

直接使用脚手架，如果本地没有安装的可以执行脚手架安装命令：

```shell
npm install -g @vue/cli
```

如果本地安装过的，可以尝试更新一下：

```shell
npm update -g @vue/cli
```

测试 vue-cli 版本：

```shell
vue -V
@vue/cli 4.5.4
```

将vue-cli更新到最新版本后，升级到vue-next (vue3.0)

```shell
vue add vue-next
```

执行完上述命令后，会自动安装 vue-cli-plugin-vue-next 插件，它会将项目升级为 vue3.0 的依赖环境，包括 vue-router 和 vuex 都会升级为 4.x 的版本。

antd-vue升级

```
npm i --save ant-design-vue@next
```

