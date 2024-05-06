---
title: vue-property-decorator使用手册
date: 2024-05-05 14:52:32
tags:
	- vue
---
## 一，安装

```shell
npm i -s vue-property-decorator vue-class-component
```

## **二，用法**

**1，@Component(options:ComponentOptions = {})**

`@Component` 装饰器可以接收一个对象作为参数，可以在对象中声明 `components ，filters，directives`等未提供装饰器的选项，也可以声明`computed，watch`等

```typescript
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
```

**2，@Prop(options: (PropOptions | Constructor[] | Constructor) = {})**

`@Prop`装饰器接收一个参数，这个参数可以有三种写法：

- `Constructor`，例如`String，Number，Boolean`等，指定 `prop` 的类型；
- `Constructor[]`，指定 `prop` 的可选类型；
- `PropOptions`，可以使用以下选项：`type，default，required，validator`。

```typescript
import { Vue, Component, Prop } from 'vue-property-decorator'

export default class MyComponent extends Vue {
  @Prop(String) public propA: string | undefined
  @Prop([String, Number]) public propB!: string | number
  @Prop({
    type: String,
    default: 'abc'
  })
  public propC!: string
}
```

等同于下面的`js`写法

```javascript
export default {
  props: {
    propA: {
      type: String
    },
    propB: {
      type: [String, Number]
    },
    propC: {
      type: String,
      defalut: 'abc'
    }
  }
}
```

注意：

- 属性的ts类型后面需要加上`undefined`类型；或者在属性名后面加上!，表示`非null` 和 `非undefined`
  的断言，否则编译器会给出错误提示；
- 指定默认值必须使用上面例子中的写法，如果直接在属性名后面赋值，会重写这个属性，并且会报错。

**3，@PropSync(propName: string, options: (PropOptions | Constructor[] | Constructor) = {})**

`@PropSync`装饰器与`@prop`用法类似，二者的区别在于：

- `@PropSync` 装饰器接收两个参数：
  `propName: string` 表示父组件传递过来的属性名；
  `options: Constructor | Constructor[] | PropOptions` 与`@Prop`的第一个参数一致；
- `@PropSync` 会生成一个新的计算属性。

```typescript
import { Vue, Component, PropSync } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  @PropSync('propA', { type: String, default: 'abc' }) public syncedPropA!: string
}
```

等同于下面的`js`写法

```javascript
export default {
  props: {
    propA: {
      type: String,
      default: 'abc'
    }
  },
  computed: {
    syncedPropA: {
      get() {
        return this.propA
      },
      set(value) {
        this.$emit('update:propA', value)
      }
    }
  }
}
```

**注意：`@PropSync`需要配合父组件的`.sync`修饰符使用**

**4，@Model(event?: string, options: (PropOptions | Constructor[] | Constructor) = {})**

`@Model`装饰器允许我们在一个组件上自定义`v-model`，接收两个参数：

- `event: string` 事件名。
- `options: Constructor | Constructor[] | PropOptions` 与`@Prop`的第一个参数一致。

```typescript
import { Vue, Component, Model } from 'vue-property-decorator'

@Component
export default class MyInput extends Vue {
  @Model('change', { type: String, default: '123' }) public value!: string
}
```

等同于下面的`js`写法

```javascript
export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      type: String,
      default: '123'
    }
  }
}
```

上面例子中指定的是`change`事件，所以我们还需要在`template`中加上相应的事件：

```html
<template>
  <input
    type="text"
    :value="value"
    @change="$emit('change', $event.target.value)"
  />
</template>
```

**对`自定义v-model`不太理解的同学，可以查看[自定义事件](https://cn.vuejs.org/v2/guide/components-custom-events.html)**

**5，@Watch(path: string, options: WatchOptions = {})**

`@Watch` 装饰器接收两个参数：

- `path: string` 被侦听的属性名；

  options可以包含两个属性 ：

  `immediate?:boolean` 侦听开始之后是否立即调用该回调函数；
  `deep?:boolean` 被侦听的对象的属性被改变时，是否调用该回调函数；

**侦听开始，发生在`beforeCreate`勾子之后，`created`勾子之前**

```typescript
import { Vue, Component, Watch } from 'vue-property-decorator'

@Component
export default class MyInput extends Vue {
  @Watch('msg')
  public onMsgChanged(newValue: string, oldValue: string) {}

  @Watch('arr', { immediate: true, deep: true })
  public onArrChanged1(newValue: number[], oldValue: number[]) {}

  @Watch('arr')
  public onArrChanged2(newValue: number[], oldValue: number[]) {}
}
```

等同于下面的`js`写法

```javascript
export default {
  watch: {
    msg: [
      {
        handler: 'onMsgChanged',
        immediate: false,
        deep: false
      }
    ],
    arr: [
      {
        handler: 'onArrChanged1',
        immediate: true,
        deep: true
      },
      {
        handler: 'onArrChanged2',
        immediate: false,
        deep: false
      }
    ]
  },
  methods: {
    onMsgVhanged(newValue, oldValue) {},
    onArrChange1(newValue, oldValue) {},
    onArrChange2(newValue, oldValue) {}
  }
}
```

**6，@Emit(event?: string)**

- `@Emit` 装饰器接收一个可选参数，该参数是`$Emit`的第一个参数，充当事件名。如果没有提供这个参数，`$Emit`会将回调函数名的`camelCase`转为`kebab-case`，并将其作为事件名；
- `@Emit`会将回调函数的返回值作为第二个参数，如果返回值是一个`Promise`对象，`$emit`会在`Promise`对象被标记为`resolved`之后触发；
- `@Emit`的回调函数的参数，会放在其返回值之后，一起被`$emit`当做参数使用。

```typescript
import { Vue, Component, Emit } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  count = 0
  @Emit()
  public addToCount(n: number) {
    this.count += n
  }
  @Emit('reset')
  public resetCount() {
    this.count = 0
  }
  @Emit()
  public returnValue() {
    return 10
  }
  @Emit()
  public onInputChange(e) {
    return e.target.value
  }
  @Emit()
  public promise() {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}
```

等同于下面的`js`写法

```javascript
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    addToCount(n) {
      this.count += n
      this.$emit('add-to-count', n)
    },
    resetCount() {
      this.count = 0
      this.$emit('reset')
    },
    returnValue() {
      this.$emit('return-value', 10)
    },
    onInputChange(e) {
      this.$emit('on-input-change', e.target.value, e)
    },
    promise() {
      const promise = new Promise(resolve => {
        setTimeout(() => {
          resolve(20)
        }, 0)
      })
      promise.then(value => {
        this.$emit('promise', value)
      })
    }
  }
}
```

**7，@Ref(refKey?: string)**

`@Ref` 装饰器接收一个可选参数，用来指向元素或子组件的引用信息。如果没有提供这个参数，会使用装饰器后面的属性名充当参数

```typescript
import { Vue, Component, Ref } from 'vue-property-decorator'
import { Form } from 'element-ui'

@Componentexport default class MyComponent extends Vue {
  @Ref() readonly loginForm!: Form
  @Ref('changePasswordForm') readonly passwordForm!: Form

  public handleLogin() {
    this.loginForm.validate(valide => {
      if (valide) {
        // login...
      } else {
        // error tips
      }
    })
  }
}
```

等同于下面的`js`写法

```javascript
export default {
  computed: {
    loginForm: {
      cache: false,
      get() {
        return this.$refs.loginForm
      }
    },
    passwordForm: {
      cache: false,
      get() {
        return this.$refs.changePasswordForm
      }
    }
  }
}
```

**@Provide/@Inject 和 @ProvideReactive/@InhectReactive**

由于平时基本不用到provide/inject选项，暂时先放着，以后有时间再研究

参考：[https://github.com/kaorun343/...](https://github.com/kaorun343/vue-property-decorator)