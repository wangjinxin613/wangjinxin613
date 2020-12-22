## 一个新的全局 API：`createApp`

调用 `createApp` 返回一个应用实例，这是 Vue 3 中的新概念：

```js
import { createApp } from 'vue'

const app = createApp({})
```

应用实例暴露当前全局 API 的子集，经验法则是，任何全局改变 Vue 行为的 API 现在都会移动到应用实例上，以下是当前全局 API 及其相应实例 API 的表：

| 2.x 全局 API               | 3.x 实例 API (`app`)                                         |
| -------------------------- | ------------------------------------------------------------ |
| Vue.config                 | app.config                                                   |
| Vue.config.productionTip   | *removed* ([见下方](https://www.vue3js.cn/docs/zh/guide/migration/global-api.html#config-productiontip-removed)) |
| Vue.config.ignoredElements | app.config.isCustomElement ([见下方](https://www.vue3js.cn/docs/zh/guide/migration/global-api.html#config-ignoredelements-is-now-config-iscustomelement)) |
| Vue.component              | app.component                                                |
| Vue.directive              | app.directive                                                |
| Vue.mixin                  | app.mixin                                                    |
| Vue.use                    | app.use ([见下方](https://www.vue3js.cn/docs/zh/guide/migration/global-api.html#a-note-for-plugin-authors)) |

所有其他不全局改变行为的全局 API 现在被命名为 exports，文档见[全局 API Treeshaking](https://www.vue3js.cn/docs/zh/guide/migration/global-api-treeshaking.html)。

## provide

- **参数：**

  - `{string | Symbol} key`
  - `value`

- **返回值：**

  - 应用实例

- **用法：**

  设置一个可以被注入到应用范围内所有组件中的值。组件应该使用 `inject` 来接收提供的值。

  从 `provide`/`inject` 的角度来看，可以将应用程序视为根级别的祖先，而根组件是其唯一的子级。

  该方法不应该与 [provide 组件选项](https://www.vue3js.cn/docs/zh/api/options-composition.html#provide-inject)或组合式 API 中的 [provide 方法](https://www.vue3js.cn/docs/zh/api/composition-api.html#provide-inject)混淆。虽然它们也是相同的 `provide`/`inject` 机制的一部分，但是是用来配置组件提供的值而不是应用提供的值。

  通过应用提供值在写插件时尤其有用，因为插件一般不能使用组件提供值。这是使用 [globalProperties](https://www.vue3js.cn/docs/zh/api/application-config.html#globalProperties) 的替代选择。

  Note

  `provide` 和 `inject` 绑定不是响应式的。这是有意为之。不过，如果你向下传递一个响应式对象，这个对象上的 property 会保持响应式。

- **示例：**

  向根组件中注入一个 property，值由应用提供。

```js
import { createApp } from 'vue'

const app = createApp({
  inject: ['user'],
  template: `
    <div>
      {{ user }}
    </div>
  `
})

app.provide('user', 'administrator')
```

- 参考：
  - [Provide / Inject](https://www.vue3js.cn/docs/zh/guide/component-provide-inject.html)