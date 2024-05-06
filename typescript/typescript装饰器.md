---
title: typescript装饰器
date: 2024-05-05 14:52:32
tags:
	- typescript
---
# 了解typescript装饰器

> 装饰器是一种特殊类型的声明，它能够附加到类、类的函数、类属性、类函数的参数上，以达到修改类的行为

### 一、装饰器的种类

- 根据装饰器的位置

1. 类装饰器
2. 类函数装饰器
3. 类属性装饰器
4. 类函数参数装饰器

- 根据装饰器是否有参数

1. 无参数装饰器（一般装饰器）
2. 有参数装饰器（装饰器工厂）

> 装饰器是一项实验性特性，在未来的版本中可能会发生改变

若要启用实验性的装饰器特性，你必须在命令行或`tsconfig.json`里启用`experimentalDecorators`编译器选项：

**命令行**:

```
tsc --target ES5 --experimentalDecorators
```

**tsconfig.json**:

```
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

## 二、类的装饰器

- 1、类装饰器的写法

```
    function desc(target) {
      console.log('---------------类的装饰器参数 start------------------');
      console.log(target); // 输出 [Function: Person]表示当前装饰的类
      console.log('---------------类的装饰器参数 end------------------');
    }

    @desc // 使用装饰器
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }
    }

    let p = new Person('哈哈', 20);
```

- 2、使用类的装饰器扩展类的属性和方法

```
    function desc(target) {
      console.log('---------------类的装饰器参数 start------------------');
      console.log(target);
      console.log('---------------类的装饰器参数 end------------------');
      return class extends target{ // 在react高阶组件中经常看到这种写法
        gender = '男';
        say() {
          console.log(this.name, this.age, this.gender);
        }
      }
    }

    @desc
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }
    }

    let p = new Person('哈哈', 20);
    console.log(p);
    p.say();

    /*
    ---------------类的装饰器参数 start------------------
    [Function: Person]
    ---------------类的装饰器参数 end------------------
    class_1 { name: '哈哈', age: 20, gender: '男' }
    哈哈 20 男
    */
```

- 3、使用装饰器修改类的构造函数(构造函数的重载、方法重载)

```
    function desc(target) {
      return class extends target{
        name = '我是重载后的';
        sayHell() {
          console.log('我是重载后的', this.name);
        }
      }
    }

    @desc
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor() {
        this.name = '哈哈';
        this.age = 20;
      }

      sayHell() {
        console.log('hello word', this.name);
      }
    }

    let p = new Person();
    console.log(p);
    p.sayHell();
```

- 4、装饰器工厂的写法

```
    function desc(params: string) {
      return function (targe: any) {
        console.log('---------------参数说明 start------------------');
        console.log('params', params);
        console.log('target', targe);
        console.log('---------------参数说明 end------------------');
        // 直接在原型上扩展一个属性
        targe.prototype.apiUrl = params;
      }
    }

    @desc('http://www.baidu.com')
    class P {
      say() {
        console.log('说话')
      }
    }

    let p:any = new P();
    console.log(p.apiUrl);
```

## 三、类函数装饰器

> 它应用到方法上，可以用来监视、修改、替换该方法

- 1、定义方式

```
    function desc(target, key, descriptor) {
      console.log('---------------类的装饰器参数 start------------------');
      console.log('target', target); // Person { say: [Function] } 表示类的原型
      console.log('key', key); // 被装饰的函数名
      console.log('descriptor', descriptor); // 被装饰的函数的对象属性
      console.log('---------------类的装饰器参数 end------------------');
    }
```

- 2、使用

```
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }

      @desc
      say() {
        console.log('说的方法')
      }
    }
```

- 3、在装饰器中添加类的原型属性和原型方法

```
    function desc(target, key, descriptor) {
      target.gender = '男';
      target.foo = function () {
        console.log('我是原型上的方法')
      }
    }

    // 测试代码
    let p = new Person('哈哈', 20);
    console.log(p);
    console.log(Person.prototype);
    p.say();
    console.log(p.gender); // 使用p原型链上的属性
    p.foo() // 调用了p原型链上的方法
```

- 4、使用装饰器拦截函数的调用(替换)

```
    function desc(params: string) {
      return function (target: any, key: string, descriptor: {[propsName: string]: any}) {
        // 修改被装饰的函数的
        let method = descriptor.value;
        descriptor.value = function (...args: Array<any>) {
          args = args.map(it => String(it));
          console.log(args);
          // method.apply(this, args);
        }
      }
    }
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }

      @desc('装饰器上的参数')
      say() {
        console.log('说的方法')
      }
    }


    let p = new Person('哈哈', 20);
    console.log(p);
    p.say(123, 23, '你好');
```

- 5、使用装饰器拦截函数的调用(附加新的功能)

```
    function desc(params: string) {
      return function (target: any, key: string, descriptor: {[propsName: string]: any}) {
        // 修改被装饰的函数的
        let method = descriptor.value;
        descriptor.value = function (...args: Array<any>) {
          args = args.map(it => String(it));
          console.log(args);
          method.apply(this, args);
        }
      }
    }
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }

      @desc('装饰器上的参数')
      say(...args) {
        console.log('说的方法', args)
      }
    }


    let p = new Person('哈哈', 20);
    console.log(p);
    p.say(123, 23, '你好');
```

## 四、类属性装饰器

- 1、定义方式

```
    function desc(target, name) {
      console.log('---------------类属性装饰器的参数 start------------------');
      console.log('target', target, target.constructor); // 表示类的原型
      console.log('name', name); // 表示被装饰属性名
      console.log('---------------类属性装饰器的参数 end------------------');
    }


    class Person {
      public name: string | undefined;
      public age: number | 0;

      @desc
      private gender: string | undefined;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }
    }

    let p = new Person('哈哈', 20);
    console.log(p);
```

- 2、在装饰器中修改属性值

```
    function desc(target, name) {
      target[name] = '女';
    }


    class Person {
      public name: string | undefined;
      public age: number | 0;

      @desc
      public gender: string | undefined;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }

      say() {
        console.log(this.name, this.age, this.gender);
      }
    }

    let p = new Person('哈哈', 20);
    console.log(p);
    p.say();
```

## 五、类函数参数的装饰器

> 参数装饰器表达式会在运行时候当做函数被调用，以使用参数装饰器为类的原型上附加一些元数据

- 1、使用方式

```
    function desc(params: string) {
      return function (target: any, key, index) {
        console.log('---------------参数装饰器 start------------------');
        console.log(target); // 类的原型
        console.log(key); // 被装饰的名字
        console.log(index); // 序列化
        console.log('---------------参数装饰器 end------------------');
      } 
    }
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }

      say(@desc('参数装饰器') age: number) {
        console.log('说的方法')
      }
    }

    let p = new Person('哈哈', 20);
    console.log(p);
    p.say(20);
```

- 2、为类的原型上添加一些东西

```
    function desc(params: string) {
      return function (target: any, key, index) {
        console.log('---------------参数装饰器 start------------------');
        console.log(target); // 类的原型
        console.log(key); // 被装饰的名字
        console.log(index); // 序列化
        target.message = params;
        console.log('---------------参数装饰器 end------------------');
      } 
    }
    class Person {
      public name: string | undefined;
      public age: number | 0;

      constructor(name, age) {
        this.name = name;
        this.age = age;
      }

      say(@desc('参数装饰器') age: number) {
        console.log('说的方法')
      }
    }


    let p: any = new Person('哈哈', 20);
    console.log(p);
    p.say(20);
    console.log(p.message)
```

## 六、几种装饰器的执行顺序

- 1、测试代码

```
    function logCls(params: string) {
      return function (target: any) {
        console.log('4.类的装饰器');
      }
    }

    function logMehod(params: string) {
      return function (target: any, key: string, descriptor: {[propsName: string]: any}) {
        console.log('3.类的函数装饰器');
      }
    }

    function logParams(params: string) {
      return function (target: any, name: string) {
        console.log('1.类属性装饰器');
      }
    }

    function logQuery(params: string) {
      return function (target: any, key: string, index: number) {
        console.log('2.函数参数装饰器');
      }
    }

    @logCls('类的装饰器')
    class Person{
      @logParams('属性装饰器')
      public name: string | undefined;

      @logMehod('函数装饰器')
      getData(@logQuery('函数参数装饰器') age: number, @logQuery('函数参数装饰器') gender: string) {
        console.log('----');
      }
    }
```

- 2、运行结果

```
    1.类属性装饰器
    2.函数参数装饰器
    3.类的函数装饰器
    4.类的装饰器
```