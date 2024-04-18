##### 问题1: 小程序默认作用域插槽在使用自定义组件时必须指定v-slot="slotProps"才能显示（https://github.com/dcloudio/uni-app/issues/4822）

问题原因：所有作用域插槽都会被编译成具名插槽，但是默认作用域插槽的自定义组件会被编译成默认插槽（无slot="d"），然作用域插槽编译成具名插槽是为了解决插槽的遍历，不可改成默认插槽，只能将默认作用域插槽编译成具名插槽。


```js
// uni-mp-compiler/src/transforms/vSlot.ts
import {
  createDirectiveNode,
  createBindDirectiveNode,
  isUserComponent,
} from '@dcloudio/uni-cli-shared'
...
let onComponentSlot = findDir(node, 'slot', true)
...
if (
    !isTemplateNode(slotElement) ||
    !(slotDir = findDir(slotElement, 'slot', true))
  ) {
    // not a <template v-slot>, skip.
    if (slotElement.type !== NodeTypes.COMMENT) {
      implicitDefaultChildren.push(slotElement)
    }
  	// ++++
    if (!onComponentSlot) { // 此处需要判断是作用域插槽
      onComponentSlot = createDirectiveNode(
        'slot',
        SLOT_DEFAULT_NAME,
        'slotProps'
      )
    }
    // +++
    continue
  }
```

问题难点是如何在 VSlot 中判断 <slot></slot> 是否是作用域插槽（是否存在除name以外的属性），此处方法主要是解析 v-slot，而  <slot></slot> 在子组件，难以判断是否是作用域插槽

该问题暂存，等想到好的解决方案再解决

##### 问题2: 小程序分包引用分包内图片资源，路径会变成根目录/static/ (https://ask.dcloud.net.cn/question/167662)

![image-20240412140941372](../img/image-20240412140941372.png)

已定为具体位置，难点是如何判断是否是分包，或者找到 options.base 源头位置看是否可以更好判断是否是分包

##### 问题3: 百度小程序 web-view 组件无法触发 message 事件（https://github.com/dcloudio/uni-app/issues/4681）

##### 问题4: 抖音小程序created前就加载了组件，导致组件数据可能不对（https://ask.dcloud.net.cn/question/189702?item_id=270366&rf=false）
