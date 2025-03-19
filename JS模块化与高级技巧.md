# JS面向对象

## 关于构造函数

1. 概念：构造函数是专门用来构造对象的函数，和一般的函数没有本质区别，**首字母大写**用于区分普通函数，**使用构造函数创建对象的过程叫实例化**

2. 实例成员和静态成员：

   通过构造函数创建的对象称为**实例对象**，实例对象中的属性和方法称为**实例成员**。

   构造函数本身的属性和方法被称为**静态成员**，可以为构造函数动态添加属性和方法

3. JS中的主要的6种数据类型都有其对应的**内置构造函数**，也有一些静态的属性和方法，常见如下：

   1. 数组的**forEach，filter，map，join，find，every**
   2. 字符串的**length，split**

## 面向对象

1. 通过构造函数，我们可以实现JS中对象逻辑的**封装**

2. 每一个构造函数都有一个prototype属性指向另一个对象，称为**原型对象**，这个对象可以挂载函数，我们可以把一些不变的方法直接定义在prototype上，所有这个构造函数的实例都可以共享这些方法
   ```html
   <script>
     function Person() {
       // 此处未定义任何方法
     }
   
     // 为构造函数的原型对象添加方法
     Person.prototype.sayHi = function () {
       console.log('Hi~');
     }
   	
     // 实例化
     let p1 = new Person();
     p1.sayHi(); // 输出结果为 Hi~
   </script>
   ```

   **构造函数的属性或方法优先于原型对象的属性或方法**

3. 由于我们可以通过prototype来设置某一构造函数的原型对象，如
   ```js
   function Person() {
       this.head = 1
       this.eyes = 2
   }
   function Man() { }
   Man.prototype = new Person()
   ```

   我们可以通过prototype的**constructor**属性得知这个该原型对象的构造函数` Man.prototype.constructor 是 Person 函数`

4. 基于原型对象的继承使得不同构造函数的原型对象关联在一起，并且这种关联的关系是一种链状的结构，我们将原型对象的链状结构关系称为**原型链**，我们可以用**instanceof**运算符来检测构造函数的prototype属性是否出现在某个实例对象的原型链上

# JS高级技巧

## 深浅拷贝

对于简单数据类型，直接赋值操作复制的是值；对于引用数据类型（数组，对象），直接赋值操作复制的是地址，导致对副本的操作会影响原本，如
```js
const obj = {
    name: 'Hasono',
    age: 18
}
const obj2 = obj
obj2.age = 20 // obj中的age也被更改为20
```

因此，我们需要拷贝操作来复制引用数据类型

1. **浅拷贝**的两种操作：
   对数组：` const new = [ ...old ]`
   对对象：`const new = { ...old } `

   然而浅拷贝只会拷贝**最外层**的值，对里层的复杂数据类型仍然拷贝的是地址，如

   ```js
   const obj = {
       name: 'Hasono',
       age: 18，
       hobby: {
       	first: 'jog',
       	second: 'swim'
   	}
   }
   ```

   对obj的拷贝只拷贝了最外层的数据，hobby对象的拷贝仍然拷贝的是地址，这时候我们就需要深拷贝

2. **深拷贝**的操作：

   1. 自己书写函数
      ```js
      const obj1 = {
          name: 'Hasono',
          age: 18,
          hobby: ['jog', 'swim']
      }
      const obj2 = {}
      function deepCopy(newObj, oldObj) {
          for (let k in oldObj) {
              if (oldObj[k] instanceof Array) {
                  newObj[k] = []
                  deepCopy(newObj[k], oldObj[k])
              } else if (oldObj[k] instanceof Object) {
                  newObj[k] = {}
                  deepCopy(newObj[k], oldObj[k])
              } else {
                  newObj[k] = oldObj[k]
              }
          }
      }
      ```

   2. 通过lodash库使用cloneDeep函数

   3. 使用JSON
      ```js
      const obj2 = JSON.parse(JSON.stringfy(obj1))
      ```

## 异常处理

关键字：**throw，try，catch，finally**

```js
function func() {
	try {
    // 预估中可能出现问题的代码写到这里，如AJAX中请求数据的成功与否
    } catch(err) {
		console.log(err.message)
        throw new Error('出现错误') // throw抛出错误，中断执行
    } finally {
        // 无论try中代码是否有问题，都执行finally中代码
    }
}
```

## 防抖

含义：单位时间内，如果频繁触发事件，**只执行最后一次**

底层实现：

```js
let num = 0
const div = document.querySelector('div')

function mouseMove() {
    div.innerHTML = num++
}

function debounce(func, time) {
    let timer
    return function() {
        if (timer) clearTimeout(timer)
        timer = setTimeout(func, time)
    }
}

div.addEventListener('mousemove', debounce(mouseMove, 500))
```

## 节流

含义：单位时间内，如果频繁触发事件，**只执行一次**

```js
let num = 0
const div = document.querySelector('div')

function mouseMove() {
    div.innerHTML = num++
}

function throttle(func, time) {
    let timer
    return function() {
        if (!timer) timer = setTimeout(function() {
            func()
            timer = null
        }, time)
    }
}

div.addEventListener('mousemove', debounce(mouseMove, 500))
```

# JS模块化

## 概述

含义：将程序按照一定规则**拆分**成多个文件，每个拆分出来的文件就是**一个模块**，模块中的数据是**私有的**，模块之间相互**独立**，但是也可以通过一些手段将模块中的数据**暴露**出去，供其它模块使用

模块化规范：CommonJS -- 服务端应用广泛；ES6模块化 -- 浏览器端应用广泛

## CommonJS

**导出：**JS提供了一个**空对象**来实现数据的导出，我们将需要导出的数据作为属性和方法添加到空对象上，然后JS会自动实现空对象的导出，我们可以通过exports和module.exports访问到这个空对象

1. 直接通过exports导出
   ```js
   exports.name = name
   exports.age = 18
   ```

2. 通过module.exports导出
   ```js
   module.exports = { name, age } // 采用ES6新语法，若键和值同名，可缩写
   ```

   **注意：无论如何修改导出对象，最终导出的都是module.exports的值**

**导入：**我们通过**require函数**来导入数据，如

```js
const person = require('文件相对路径')
const { name, age } = require('文件相对路径')
const { name:p1Name, age:p1Age } = require('文件相对路径') // 重命名语法处理名字冲突
```

**如果导入的路径是一个文件夹，会首先检测该文件夹下面`package.json`文件中`main`属性对应的文件，若存在则导入，不存在则报错；**
**若`main`属性不存在或`package.json`不存在，则会尝试导入文件夹下的`index.js`和`index.json`，如果还是没找到，则会报错**
我们可以通过`console.log(arguments.callee.toString())`来查看这个函数

注意：CommonJS的底层运行机制是将模块中的语句作为一个Node环境提供的内置函数的函数体，通过调用这个内置函数来实现，**浏览器环境中无法复用**
我们可以用**browserify**来编译CJS文件使得浏览器能读懂

## ES6模块化（服务器端和浏览器端都支持）

无论是服务器端还是浏览器端，我们都需要先**将html页面的入口js文件的type改为module**

```html
<script type='module' src='./index.js'></script>
```



**Node中运行ES6模块：**两种方法

1. 将所有模块文件的后缀改为.mjs
2. 新建package.json文件，配置` { "type": "module" }`

**导出**：无论哪种导出方式，我们导出的都是一个**Module对象**

1. 分别导出：在要导出的数据前面加上**export**，如

   ```js
   export const name = 'Hasono'
   export const age = 18
   ```

2. 统一导出：在模块文件最后添加**export与一个类似对象的结构（不是对象）**，如

   ```js
   export { name, age }
   ```

3. 默认导出：为Module添加一个**default属性**，将要导出的数据作为该属性的属性和方法（若default属性是对象类型）

   ```js
   export default { name, age } // 与统一导出不同，这里是导出一个对象（使用了简写）
   ```

**导入：**

1. 全部导入：将导出的全部内容整合到**一个对象**

   ```js
   import * as 对象名 from '文件相对路径'
   ```

2. 命名导入：（对应导出方式：**默认导出不适用此导入**）

   ```js
   import { name as personName } from '文件相对路径'
   ```

   若名字相同，也可以写作
   ```js
   import { name } from '文件相对路径'
   ```

   

3. 默认导入：（对应到处方式：**默认导出**）直接采用以下写法来接收默认导出的值

   ```js
   import person from '文件相对路径'
   ```

4. 默认导入和命名导入可以混用

5. 动态导入：涉及异步知识，暂缓



**数据引用问题：**CJS中不存在数据引用问题，导出模块中的数据和导入模块的数据互不干涉影响；ES6**存在数据引用问题**，导出模块中的数据和导入模块的数据**共用同一内存空间**，在导入模块中数据的操作**会影响到**导入模块，所以我们推荐将**要导出的数据设置为常量**
