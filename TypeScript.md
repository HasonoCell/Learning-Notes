# TypeScript使用指南

## 初始操作

1. 命令行`npm install -g typescript`安装typescript包
2. 输入`tsc --init`创建初始`tsconfig.json`文件
3. 输入`tsc --watch`对当前文件夹内的所有ts文件进行监听转译

## 类型声明与限制

```ts
let a: string
let b: number
let c: boolean

a = 'Hello'
b = 18
c = true

function plus(a: number, b: number): number {
    return a + b
}
```

```ts
let b: 'Hello'
b = 'Hello' // 字面量类型，只能存字面量写的值
```

### 关于any和unknown

1. unknown可被看作是类型安全的any，适用于不确定具体数据类型的情况，unknown会强制在使用前进行安全检查
   ```ts
   let a: unknown
   let x: string
   
   a = false
   a = 'Hello'
   
   x = a // 报错：不能将类型“unknown”分配给类型“string”。
   ```

   可以用如下方式转换

   ```ts
   x = a as string // 断言操作
   x = <string> a // 同样是断言操作
   ```

   

2. 读取`any`类型数据的任何属性都会报错，而`unknown`与之相反

### 关于void

1. `void`常用于函数返回值声明

   ```ts
   function logMessage(msg: string): void {
       console.log(msg)
   }
   logMessage('Hello')
   ```

   `logMessage`函数是没有显式返回值的，但是具有一个隐式返回值`undefined`，所以对于如下代码都是接受的

   ```ts
   function logMessage(msg: string): void {
       console.log(msg)
       return
   }
   logMessage('Hello')
   
   // 或
   
   function logMessage(msg: string): void {
       console.log(msg)
       return undefined
   }
   logMessage('Hello')
   ```

   若函数返回类型为void，则：
   从语法上讲：函数只能接受返回`undefined`，显示返回或隐式返回均可
   从语义上讲：函数调用者不应该关心函数的返回值也不该对其进行操作，即使返回了`undefined`

### 关于对象

1. 我们一般不通过`let person: object`或`let person: Object`的方式声明一个变量是对象，因为数据的类型太宽泛

   常采用以下操作

   ```ts
   let person: { name: string, age?: number } // age为可选属性
   
   //也可以写作
   let person: {
   	name: string
       age?: number
   }
   ```

2. **索引签名**：允许定义对象具有任意数量的属性

   ```ts
   let person: {
       name: string
       age: number
       [key: string]: any
   }
   
   // 这样我们就可以对person进行自由赋值，如
   
   person = {
   	name: 'Hasono',
       age: 18,
       gender: '男'
   }
   ```

### 关于函数

​	声明函数类型
```ts
let count: (a: number, b: number) => number // TS中的 => 在函数类型声明时表示这是一个函数类型，用来描述参数类型和返回类型，与JS中的 => 用法不同

count = function(a, b) {
	return a + b
}
```

### 关于数组

​	声明数组类型
```ts
let arr1: string[]
let arr2: Array<number> // 泛型

arr1 = ['a', 'b']
arr2 = [100, 200]
```

### 关于元组（Tuple）

​	元组是一种特殊的**数组类型**，可以储存**固定数量的**元素，且每个元素的**数据类型已知且可以不同**

```ts
let arr1: [string, number]
let arr2: [number, boolean?]
let arr3: [number, ...string[]] // ...string[] 表示arr3可以拥有任意多个字符串类型的元素，string可以换为其它基本类型

arr1 = ['a', 999]
arr2 = [100, true]
arr2 = [200]
arr3 = [300, 'a', 'b', 'c']
```

### 关于枚举

1. 数字枚举：枚举成员的值是数字，具有**连续递增，反向映射**的特点
   ```ts
   enum Direction {
       Up,
       Down,
       Right,
       Left,
   } 
   /* 打印 Direction 会看见如下内容
   {
     '0': 'Up',
     '1': 'Down',
     '2': 'Right',
     '3': 'Left',
     Up: 0,
     Down: 1,
     Right: 2,
     Left: 3
   } */
   
   function walk(data: Direction) {
       if (data === Direction.Up) {
           console.log('Go Up')
       } else if (data === Direction.Down) {
           console.log('Go Down')
       } else if (data === Direction.Left) {
           console.log('Go Left')
       } else if (data === Direction.Right) {
           console.log('Go Right')
       } else {
           console.log('No Direction')
       }
   }
   
   walk(Direction.Up)
   walk(Direction.Left)
   ```

2. 字符串枚举：枚举成员的值是字符串，没有**反向映射**的特点 
   ```ts
   enum Direction {
       Up = 'up',
       Down = 'down',
       Right = 'right',
       Left = 'left',
   } 
   ```

3. 常量枚举：使用const定义，在被编译时避免产生额外代码

   ```ts
   const enum Direction {
       Up = 'up',
       Down = 'down',
       Right = 'right',
       Left = 'left',
   }
   console.log(Direction.Up)
   
   // 编译后的JS文件内容 
   // console.log("up" /* Direction.Up */)
   ```

### 关于Type

​	`type`可以为任意类型创建别名

1. 联合类型（或）

   ```ts
   type Status = number | string
   type Gender = '男' | '女' // 字面量类型
   
   function printStatus(data: Status) {
       console.log(data)
   }
   
   function printGender(data: Gender) {
       console.log(data)
   }
   
   printStatus(404)
   printStatus('404')
   printGender('男')
   printGender('女')
   ```

2. 交叉类型（与）
   ```ts
   type Area = {
       height: number
       width: number
   }
   
   type Address = {
       num: number // 楼号
       cell: number // 单元号
       room: string // 房间号
   }
   
   type House = Area & Address
   
   const house: House = {
       height: 200,
       width: 400,
       num: 2,
       cell: 3,
       room: '702'
   }
   ```

   **特殊情况**

   使用**类型声明**限制函数返回值是`void`时，**TypeScript不会严格要求函数返回空**

   ```ts
   type LogFunc = () => void
   
   const func1: LogFunc = function() {
       return 1000 // 允许返回非 undefined 值
   }
   const func2: LogFunc = () => true // 允许返回非 undefined 值
   ```

   关注以下代码：
   ```ts
   type LogFunc = () => void
   const func1: LogFunc = function() {
       return 1000
   }
   
   const x = func1()
   console.log(x)
   
   if (x < 2000) // 报错：x是一个void类型
   ```

## 类相关

1. 各种修饰符

   1. `public`修饰符：公开的，可以被`类内部，子类，类外部`访问

      ```ts
      class Person {
          name: string
          age: number
          constructor(name: string, age: number) {
              this.name = name
              this.age = age
          }
      }
      
      // 简写形式
      class Person {
          constructor(
              public name: string,
              public age: number
          ) { }
      }
      ```

   2. `protected`修饰符：受保护的，可以被`类内部，子类`访问

   3. `private`修饰符：私有的，可以被`类内部`访问

   4. `readonly`修饰符：只读的，不可被修改

2. 抽象类：我们用快递包裹的例子来理解，包裹是一个通用的类，它又包含标准包裹和特快包裹两个子类；我们就可以将最初的包裹设置为一个**抽象类**
   ```ts
   abstract class Package {
       // 构造方法
       constructor( public weight: number ) { }
       // 抽象方法
       abstract calculate(): number
       // 具体方法
       printPackage() {
           console.log(`包裹重量为${ this.weight }kg 
           运费为${ this.calculate() }元`)
       }
   } // 这是 包裹 这一最大的抽象类
   
   class StandardPackage extends Package {
       constructor(
           weight: number,
           public unitPrice: number
       ) { super(weight) }
       calculate(): number {
           return this.weight * this.unitPrice
       }
   } // 这是继承自 包裹 这一抽象类的 标准包裹 这一子类
   
   const s1 = new StandardPackage(10, 5)
   ```

   **一些注意**：**抽象类**不能直接实例化，其中既可以写**具体方法**也可以写**抽象方法**；抽象类主要用来为其派生类提供一个**基础结构**，里面的**抽象方法**必须被派生类实现；**一个类只能继承一个抽象类**

   **何时使用抽象类**：

   1. 定义**通用接口**：为一组相关的类定义通用的方法或属性
   2. 提供**基础实现**：派生类可以继承抽象类的基础实现
   3. 确保**关键实现**：强制派生类对抽象类的关键方法或属性的实现
   4. **共享**代码逻辑：当多个类需要共享部分代码时，抽象类避免重复

## 接口（Interface）

接口是一种用来**定义结构**的方式，主要作用是为**类，对象，函数等**规定一种**约束**，`interface`只能定义**格式**，不能**包含任何实现**

1. 定义类结构

   ```ts
   interface PersonInterface {
       name: string
       age: number
       speak(n: number): void
   }
   
   class Person implements PersonInterface {
       constructor(
           public name: string,
           public age: number
       ) { }
       speak(n: number): void {
           for (let i = 0; i < n; i++) {
               console.log(`我是${ this.name }，今年${ this.age }岁`)
           }
       }
   }
   ```

   

2. 定义对象结构

   ```ts
   const person: PersonInterface = {
       // 内容相关
   }
   ```

3. 定义函数结构
   ```ts
   interface PlusInterface {
       (a: number, b: number): void // 函数类型签名
   }
   
   const plusFunc: PlusInterface = function(x, y) {
       return x + y
   }
   ```

4. 接口间的继承

   ```ts
   interface PersonInterface {
       name: string
       age: number
   }
   
   interface StudentInterface extends PersonInterface {
       grade: string
   }
   ```

5. 接口的可重复定义
   ```ts
   interface PersonInterface {
       name: string
       age: number
   }
   
   interface PersonInterface {
       grade: string
   } 
   
   // PersonInterface具有name，age，grade三个属性
   ```

## 泛型

**泛型**允许我们在定义函数，类或接口时，使用类型参数来表示未指定的类型。这些参数在被**具体使用**时，才会被指定**具体的类型**。泛型可以让同一段代码适用于多种数据类型

1. 泛型函数
   ```ts
   function logData<T> (data: T) {
       console.log(data)
   }
   
   logData<number> (10)
   logData<string> ('Hello')
   
   // 泛型可以有多个 logData<T, U> (a: T, b: U)
   ```

   

2. 泛型接口
   ```ts
   interface PersonInterface<T> {
   	name: string,
       age: number,
       extraInfo: T
   }
   ```

   