---
title: vue支持tsx
date: 2024-05-05 14:52:32
tags:
	- vue
---
使用vuecl3创建的项目，集成tsx

```
vue add tsx-support
```

## 导入和配置

安装 vue-tsx-support 包

```shell
npm install vue-tsx-support --save
```

导入TS声明，有两种方式

编辑tsconfig.js

```
"include": [
    "node_modules/vue-tsx-support/enable-check.d.ts", 
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "tests/**/*.ts",
    "tests/**/*.tsx"
  ],
  
 // 注意：将exclude内的 "node_modules" 删掉，不然永远也无法被引用到了
```

或者在main.js中 import

```
import "vue-tsx-support/enable-check";
```

## 创建视图组件

来创建个按钮组件：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 import { Component, Prop } from "vue-property-decorator";
 2 import * as tsx from "vue-tsx-support";
 3 
 4 export enum ButtonType {
 5   default = "default",
 6   primary = "primary"
 7 }
 8 
 9 export enum ButtonSize {
10   large = "large",
11   small = "small"
12 }
13 export interface IButtonProps {
14   type?: ButtonType;
15   size?: ButtonSize;
16   num: number;
17 }
18 
19 @Component
20 export default class Button extends tsx.Component<IButtonProps> {
21   @Prop() public type!: ButtonType;
22   @Prop() public size!: ButtonSize;
23   @Prop({ default: 0 }) public num!: number;
24 
25   protected render() {
26     return (
27       <div>
28         <p>id:{this.num}</p>
29         {this.type && <p>type:{this.type}</p>}
30         {this.size && <p>size:{this.size}</p>}
31       </div>
32     );
33   }
34 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

再创建Container 用TSX引用组件Button：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import { Component, Prop } from "vue-property-decorator";
import { Component as tsc } from "vue-tsx-support";
import Button, { ButtonType, ButtonSize } from "./button";

interface IContainerProps {
  name?: string;
}

@Component
export default class Container extends tsc<IContainerProps> {
  @Prop() public name!: string;

  protected render() {
    return (
      <div>
        <p>container Name:{this.name}</p>
        <p>{this.$slots.default}</p>
        <p>
          button:
          <Button num={9} type={ButtonType.primary} size={ButtonSize.large} />
        </p>
      </div>
    );
  }
}
```