### 注释的写法

```js
// 这是是注释
/** 这是注释内容 **/
```

### 向控制台输出内容

```js
console.log('输出的内容');
```

### 如何定义变量

```js
// var定义变量，var是es5的语法
var name = '小王';
// let定义变量
let name = '小王';
// const定义变量，常量
const name = '小王';
```

### var和let的区别

let只在块级作用域有效

var存在变量提升，let不存在变量提升

let变量不能重复声明

### js数据类型

```js
// js变量类型分为基本数据类型和引用数据类型
// 基本数据类型有 - 六种
// number => 数字
var age = 18;
// string => 字符串 单引号或者双引号包裹
var name = 'xiaowang';
// bool => 布类 true 或者 false
var isTrue = true;
// null => 空
var data = null;
// undefined => 未定义的，一种js独有的特殊的数据类型
var data = undefined;
// Symbol 不常用 唯一标识符号
// 引用数据类型有 -
// 对象
var obj = {
  name: '小王',
  age: 18
} // 字面量的方式去定义一个对象
var obj = new ClassName(); // new 关键字实例化一个对象
// 数组
var arr = ['小王', '小张'];
// 函数
function eat() {
  // 函数内容
}
var eat = function() {
  // 函数内容
}
```

### 运算符

```js
// 以下是比较运算符
// == 判断值是否相等
// === 全等于，值和类型都相同
// != 不等于
// !== 全不等于，值和类型有一个不相等，或者俩个都不想等
// > 大于
// < 小于
// >= 大于并且等于
// <= 小于并且等于
// 以下是逻辑运算符
// && 并且 俩个值都为true，返回true
// || 或 俩个值只要满足一个为true，返回true
// ! 非 原来的true，返回false，原来为false，返回true
// 以下为条件运算符
// age > 18 ? '成年人' : '未成年' 三目运算符
```

### for循环实现一个乘法口诀

```js
for (var i = 1; i <= 9; i++) {
  for (var j = i; j <= 9; j++) {
    console.log(i + '*' + j + '=' + (i * j));
  }
}
```

### typeof 判断变量的数据类型

```js
typeof "John"                // 返回 string
typeof 3.14                  // 返回 number
typeof false                 // 返回 boolean
typeof [1,2,3,4]             // 返回 object
typeof {name:'John', age:34} // 返回 object
```

### 函数

```js
// 定义一个函数，形参可以设置一个默认值
// 函数可以设置返回值
// argument 可以在函数内部使用，输出本次函数调用的参数
function run(name, sudu = '50km/h') {
  console.log(name + '正在跑步, 速度是' + sudu);
  console.log(arguments);
  return name;
}

var result = run('小王');
console.log(result);
```

### 类和对象

##### ES6使用class关键字创建类

```js
//方法和函数执行的时候要用小括号()
// 定义一个类
class Student {
	// 属性
  classes = '';
	// 静态属性 静态属性需要用类名.属性名调用，属于类的属性，实例化的对象没有这个静态属性
  static school = '渤海大学';
	// 静态方法，与静态属性相似
	static schoolAge() {
    return 100;
  }
	// 方法，实例化的对象会存在这个方法
  getGrade() {
    return 88;
  }
  // 构造函数 new对象时会调用这个构造方法，可以传入参数
  constructor(name, age) {
    this.name = name;
    this.age = 18;
  }
}
// 实例化一个对象
var xiaowang = new Student('小王', 18);
// 通过对象.属性名调用
xiaowang.classes = '1班';
xiaowang.age = 25;
// 也可以通过对象.属性名访问这个属性值
console.log(xiaowang.name);
console.log(xiaowang);
console.log(xiaowang.getGrade());
console.log(Student.school);
console.log(Student.schoolAge());

// 这是也是一种创建对象的方法，叫做字面量创建对象
var xiaowang2 = {
  name: '小王',
  age: 20
}
console.log(xiaowang2);
```

##### 使用构造函数定义类

```js
// 使用构造函数定义类
function Student(name, age) {
  this.name = name;
  this.age = age;

  this.getGrade = function () {
    return 88;
  }
}
var xiaochi = new Student('小迟', 20)
console.log(xiaochi);
console.log(xiaochi.getGrade());
```

##### 使用原型模式定义类

