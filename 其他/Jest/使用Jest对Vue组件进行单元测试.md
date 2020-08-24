# 使用Jest对Vue组件进行单元测试

安装依赖

```shell
npm install --save-dev @vue/test-utils vue-jest jest-transform-stub jest-serializer-vue
```

然后在`components`目录下创建`Counter.vue`，代码如下：

```vue
<template>
  <div>
    <div>{{ computedCount }}</div>
    <button @click="inc">加</button>
    <button @click="dec">减</button>
    <button @click="reset">重置</button>
  </div>
</template>

<script>
export default {
  props: {
    factor: { type: Number, default: 1 }
  },
  data() {
    return {
      count: 0
    };
  },
  methods: {
    inc() {
      this.count++;
    },
    dec() {
      this.count--;
    },
    reset() {
      this.count = 0;
    }
  },
  computed: {
    computedCount: function() {
      return this.count * this.factor;
    }
  }
};
</script>
```

在`unit`目录下创建`Counter.spec.js`，代码如下：

```javascript
import { mount } from '@vue/test-utils'
import Counter from '@/components/Counter.vue'

describe("Counter.vue", () => {
    it("渲染Counter组件", () => {
        const wrapper = mount(Counter)
        expect(wrapper.element).toMatchSnapshot()
    })

    it("初始化之为0", () => {
        const wrapper = mount(Counter)
        expect(wrapper.vm.count).toEqual(0)
    })

    it("加1", () => {
        const wrapper = mount(Counter)
        wrapper.vm.inc()
        expect(wrapper.vm.count).toEqual(1)
    })

    it("减1", () => {
        const wrapper = mount(Counter)
        wrapper.vm.dec()
        expect(wrapper.vm.count).toEqual(-1)
    })

    it("重置", () => {
        const wrapper = mount(Counter)
        wrapper.vm.reset()
        expect(wrapper.vm.count).toEqual(0)
    })

    it("因数为10加1操作", () => {
        const wrapper = mount(Counter, { propsData: { factor: 10 } })
        wrapper.vm.inc()
        expect(wrapper.vm.computedCount).toEqual(10)
    })
})
```

打开控制台，进入项目根目录，运行`yarn test:unit`，如图1所示：

![img](https:////upload-images.jianshu.io/upload_images/5402876-7ee2e8edb485c7dd.png?imageMogr2/auto-orient/strip|imageView2/2/w/383/format/webp)

看代码很容易理解如何在`unit test`中初始化组件，初始化组件的属性，访问组件的数据，调用组件的方法等等。