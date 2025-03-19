# Node.JS

## Node.JS是什么？

Node.JS是一种为JavaScript提供运行环境的程序，它拥有JS在浏览器上运行所需要的V8引擎，使得JS脱离浏览器的运行成为可能

***Node.js is an open-source, cross-platform JavaScript runtime environment.***

Node.JS可以用来开发服务器类，工具类（Webpack，Vite等）和桌面端应用（Electron）

## 命令行工具

命令的组成：**命令名称 + 命令参数**

常用命令：

cd（Change Directory）切换文件夹

c:或d: 切换盘符

dir 查看目录文件

## 主要知识

### Buffer数据流

1. 概念：

   Buffer是一个类似**数组**的**对象**，用于表示固定长度的**字节序列**，本质上是一段连续的**内存空间**，底层用来处理**二进制数据**，在终端中用**十六进制**表示；Node中作为一个预先提供好的模块供使用

2. 特点：
   Buffer的大小固定且无法调整；它的性能较好，可以直接对计算机内存进行操作；每个Buffer元素的大小为**1字节**

3. 创建：三种方法

   **alloc  allocUnsafe（分配的内存可能含有以前的数据，但速度更快）  from（将一个字符串，数字或数组转换成buffer）**

   buffer与字符串的转换：toString( )方法

   ```js
   const buf = Buffer.from([105, 108, 111, 118, 101, 121, 111, 117])
   console.log(buf.toString()) // i love you
   ```


### FS模块

FS（File System）模块可以实现与**硬盘**的交互，例如文件的创建，删除，重命名，移动，还有文件的写入和读取以及文件夹相关操作

#### 写入

1. FS异步和同步写入：
   异步：` fs.writeFile('文件路径', '写入内容', err => { } )`
   同步：` fs.writeFileSync('文件路径', '写入内容')`

   **异步**写入中，当执行写入操作时，回调函数中的操作会被放入I/O子线程等待执行，当主线程执行完，回调函数会被I/O线程压入任务队列，JS会再去检索任务队列内的内容并执行

2. FS追加写入：
   ` fs.appendFile('文件路径', '写入内容', err => { } )`
   其中，写入内容可以跟上'/n'之类的符号控制格式

3. FS流式写入：

   ```js
   import fs from 'fs'
   // 创建写入流对象
   const ws = fs.createWriteStream('./test.txt') 
   // 写入
   ws.write('我们不会停止阅读\r\n')
   ws.write('即使每本书都有读完的时候\r\n')
   // 关闭通道
   ws.close()
   ```

   为什么我们需要流式写入？**流式写入适用于写入频率较高或大文件写入的场景**。程序打开一个文件是需要消耗资源的，流式写入可以减少打开关闭文件的次数

4. 文件写入的场景：下载文件；安装软件；保存程序日志，如Git；编辑器保存文件等。**总之，当需要持久化保存数据时，应该想到文件写入**

#### 读取

1. FS异步和同步读取：
   异步：` fs.readFile('文件路径', (err, data) => { } )`
   同步：` const data = fs.readFileSync('文件路径')`
   **data均是buffer数据流，需要toString()方法格式转换**

2. FS流式读取：
   流式读取每次读入一整块的数据（具体大小为**64kb**），当每次数据块读完时，做一次回调函数的内容（chunk）

   ```js
   import fs from 'fs'
   // 创建读取流对象
   const rs = fs.createReadStream('./test.txt') 
   // 绑定 data 事件
   rs.on('data', chunk => {
       console.log(chunk.toString())
   })
   // end 可选事件
   rs.on('end', () => {
       console.log('读入完成')
   })
   ```

#### 重命名

通过rename函数，我们可以实现文件的重命名和位置的移动，本质上是对文件路径的更改

```js
fs.rename('旧文件路径', '新文件路径', err => { })
fs.renameSync('旧文件路径', '新文件路径')
```

#### 删除

通过unlink或rm函数，我们可以实现文件的删除

```js
fs.unlink('文件路径', err => { })
fs.rm('文件路径', err => { })
// 两种函数均有对应的Sync函数
```

#### 文件夹操作

通过mkdir函数，我们可以创建文件夹

```js
fs.mkdir('文件夹路径', err => { }) // 单个创建
fs.mkdir('./a/b/c', { recursive: true }, err => { }) // 递归创建
```

通过readdir，我们可以读取文件夹内容

```js
fs.readdir('文件夹路径', (err, data) => {
    if (err) console.log(err)
    else console.log(data)
})
```

通过rm，我们可以删除文件夹

```js
fs.rm('文件夹路径', err => { }) // 单个删除
fs.rm('文件夹路径', { recursive: true }, err => { }) // 递归删除
```

#### 查看资源信息

通过stat，我们可以查看资源信息

```js
fs.stat('文件路径', (err, data) => {
    if (err) console.log(err)
    else console.log(data)
    data.isFile() // 查看data是否为文件
    data.isDirectory() // 查看data是否为文件夹
})
```

#### 相对路径问题

在FS模块中，相对路径的参照物是**终端命令行的工作目录**；如何解决？**采用绝对路径**

**__dirname** 此变量保存了所在文件的所在目录的绝对路径**（此变量只在CJS可用，ES6不可使用）**

### Path模块

Path模块提供了操作路径的功能

path.resolve：**拼接规范的绝对路径（常用）**

```js
console.log(path.resolve(__dirname, '文件路径'))
```

### HTTP协议

HTTP协议即Hypertext Transfer Protocol，是对浏览器和服务器进行约束的一组**约定**，浏览器向服务器发送数据的行为称为**请求**，发送的数据叫做**请求报文**；服务器向浏览器返回数据的行为称为**响应**，返回的数据叫做**响应报文**

#### 请求报文

基本组成：请求行 请求头 请求体

1. 请求行
   ```http
   GET https://www.baidu.com/ HTTP/1.1
   ```

   请求行包括请求方法（如GET），URL（如https://www.baidu.com/）和HTTP版本号（如HTTP/1.1）

   **常见的请求方法**：
   GET => 用于获取数据	PUT/PATCH => 用于更新数据
   POST => 用于新增数据      DELETE => 用于删除数据

   **URL**：即Uniform Resource Locator统一资源定位符，本身也是一个字符串，用来在服务器中定位某一个资源
   URL由主要由**协议名称（https）**和**主机名（可以是域名也可以是IP地址）**构成，协议名称和主机名中间必须要有`://`，其后还有**端口号，路径和查询字符串**

2. 请求头
   使用键值对的方式记录

3. 请求体 一般为JSON格式数据

#### 响应报文

1. 响应行：
   由**HTTP版本号 响应状态码 响应状态的描述**组成

   常见状态码：
   200 => 请求成功；403 => 禁止请求

   404 => 找不到资源；500 => 服务器内部错误
   描述常常与状态码对应

2. 响应头

3. 响应体 常见格式有HTML，CSS，JS，图片视频，JSON

### HTTP模块

**特殊IP**：127.0.0.1 永远指向本机的IP地址

端口是应用程序的数字标识；一台计算机可以有多个端口；一个应用程序也可以使用多个端口
**作用：实现不同主机应用程序之间的通信**

```js
import http from 'http'
// request和response是请求报文和响应报文的两个封装对象
const server = http.createServer((request, response) => {
    response.end('Hello Http Server') // 设置响应体
})
server.listen(9000, () => {
    console.log('服务已启动')
})
```

若想解决中文乱码的问题，添加一行

```js
response.setHeader('content-type', 'text/html;charset=utf-8')

```

#### 获取请求报文

1. 获取请求行和请求头
   ```js
   const server = http.createServer((request, response) => {
       console.log(request.method) // 获取请求方法
       console.log(request.url) // 获取 url 中的路径和查询字符串
       console.log(request.httpVersion) // 获取版本号
       console.log(request.headers.host) // 获取IP和端口号
       response.end('Http Server') // 设置响应体
   })
   ```

2. 获取路径和查询字符串

   方法一 通过模块

   ```js
   import http from 'http'
   import url  from 'url'
   const server = http.createServer((request, response) => {
       const res = url.parse(request.url, true) // 这里true参数的作用是将query的内容从字符串转为对象方便使用
       console.log(res)
       response.end('url')
   })
   server.listen(9000, () => {
       console.log('服务已启动')
   })
   ```

   方法二 通过实例

   ```js
   import http from 'http'
   const server = http.createServer((request, response) => {
       const url = new URL(request.url, 'http://localhost:9000')
       console.log(url.pathname)
       console.log(url.searchParams.get('keyword'))
       console.log(url.searchParams.get('num'))
       response.end('url')
   })
   server.listen(9000, () => {
       console.log('服务已启动')
   })
   ```

#### 设置响应报文

```js
response.statusCode // 设置响应状态码
response.setHeader('头名', '头值') // 设置响应头信息
response.write('') // 设置响应体
response.end('') 
```

#### 搭建静态资源服务

静态资源指HTML，CSS，JS，图片，视频等资源；HTTP服务在哪个文件夹中寻找静态资源，那个文件夹就是**静态资源目录**，也称之为**网站根目录**

#### 设置资源类型（MIME类型）

MIME类型（媒体类型）是一种标准，用来表示文档，文件或字节流的性质和格式
结构：` [type]/[subType]`，如`text/css, text/html, image/png`，HTTP服务可以设置响应体content-type来表明响应体的MIME类型

```js
import path from 'path'
import http from 'http'
const server = http.createServer((request, response) => {
	const ext = request.extname
    console.log(ext)
})
server.listen(9000, () => {
    console.log('服务已启动')
})
```

**通过此方式获取发送请求的文件后缀名**

### 模块化

**详细知识见JS模块化笔记**

### 包管理工具

#### 一些概念

**包**的英文单词是package，代表了一组特定功能的源码集合；**包管理器**可以对包进行下载安装，更新，删除，上传等功能；包管理器是一个通用的概念，许多语言中都存在
`package.json`是包的配置文件，每一个包都需要有它的`package.json`

生产依赖：**开发和生产环境都需要的依赖包**，相关信息保存在package.json的dependencies属性下	命令`npm i -S 包名`，安装包默认为生产依赖
开发依赖：**只在开发环境下需要的依赖包**，相关信息保存在package.json的devDependencies属性下	命令`npm i -D 包名`

**环境变量Path**

#### npm常见操作

1. 创建：新建npm文件夹，命令行输入`npm init`进行初始化创建

2. 搜索：在npm.js官网搜索

3. 安装并使用：`npm install`；下载完成后，我们所下载的包的内容会存放在node_modules文件夹下，而package-lock.json用来固定安装的包的版本信息；我们在要用包的文件中通过**模块化**的方式导包并使用，安装包后，此包就是我们当前这个包的一个**依赖**

4. 全局安装：`npm i -g nodemon`	全局安装命令不受工作目录的影响，我们可以在命令行任何位置执行`nodemon`命令，它的作用是`自动重启运行node应用程序`，我们可以通过`npm root -g`查看全局安装的包的位置

5. 我们在从Git或别的地方下载使用别人的包是，输入`npm i`可以安装此包需要的所有依赖

6. 安装指定版本的包：`npm i 包名@版本号`

7. 删除：
   局部删除：`npm r 包名`或`npm remove 包名`
   全局删除：`npm remove -g 包名`

8. 配置命令别名简化操作：在package.json下的scripts属性中配置，使用
   `"命令": "路径"`的方式；在命令中输入`npm run 命令`执行；**注意：npm run有自动向上级目录查找的功能，避免包的重复安装**

   ```json
   "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "dev": "node ./test.js"
   },
   ```

#### NVM

NVM 全称 Node Version Manager 是用来对Node版本进行管理的工具，方便切换不同版本的Node.js

## Express.JS

Express.JS是一个基于Node.JS的Web应用开发框架（本质是一个封装好的工具包），便于我们开发WEB应用（HTTP服务）

### Express路由

官方定义：**路由确定了应用程序如何响应客户端对特定端点的请求**

#### 路由的使用

一个路由的组成有**请求方法，路径和回调函数**组成

```js
import express from 'express'

const router = express()

router.get('/home', (req, res) => {
    res.end('Hello Express.JS, This is a HomePage')
})

router.all('*', (req, res) => {
    res.end('<h1>404 Not Found</h1>')
}) // all指发请求的所有方法均可以；'*'路径表示全部路径均可以，通常用在末尾

router.listen(9000, () => { console.log('服务已启动...')})
```

对于请求方法，它有两个主要参数：

1. **路径（Path）**：字符串，表示要匹配的 URL 路径。
2. **处理函数（Handler Function）**：一个或多个中间件函数和最终的路由处理函数，这些函数会按顺序执行。处理函数中均有request和response两个对象做参数

#### 获取报文参数

```js
// 原生操作
console.log(req.method)
console.log(req.url)
console.log(req.httpVersion)
console.log(req.headers)

// express操作
console.log(req.path) // 获取路径
console.log(req.query) // 获取查询参数，以数组形式返回
console.log(req.ip) // 获取IP
console.log(req.get('host')) // 获取请求头,get参数写需要获取哪一个请求头
```



#### 设置路由参数

```js
router.get('/:id', (req, res) => {
    console.log(req.params.id)
    res.end('Details')
})
```

我们在路径处使用`'/:参数名'`的方式设置路由参数，通过`req.params.id`获取参数值

#### 设置响应内容

```js
router.get('/:id', (req, res) => {
    // 原生操作
    res.statusCode = 404
    res.statusMessage = 'Hello'
    res.setHeader('x', 'y')
    res.write('Hello Express')
    res.end('Response')

    // express操作
    res.status() // 设置响应码
    res.set() // 设置响应头
    res.send() // 设置响应体，此方法不会导致中文乱码
})
```

#### 其它响应设置

```js
// 跳转响应
res.redirect('文件路径')
// 下载响应
res.download('文件路径')
// JSON响应
res.json({
	name: 'Hasono',
    age: 18,
})
// 响应文件内容
res.sendFile('文件路径')
```

#### Express中间件

**中间件（Middleware）本质是一个回调函数**，它的作用是**使用函数封装公共操作，简化代码**

路由中间件：当匹配到对应的路由时，路由中间件执行，没匹配则不执行
全局中间件：只要一发请求，就执行全局中间件，再执行后续路由中间件
（用火车检票的进站口和检票口来理解全局中间件和路由中间件）

1. 全局中间件的使用
   ```js
   import express from 'express'
   import fs from 'fs'
   import path from 'path'
   import { fileURLToPath } from 'url'
   
   const router = express()
   const __filename = fileURLToPath(import.meta.url)
   const __dirname = path.dirname(__filename)
   
   // 声明中间件函数
   function recordMiddleware(req, res, next) {
       const { url, ip } = req
       fs.appendFileSync(path.resolve(__dirname, './access.log'), 
       `${ url }  ${ ip }\r\n`)
       next()
   }
   // 使用中间件函数
   router.use(recordMiddleware)
   
   router.get('/home', (req, res) => {
       res.end('HomePage') 
   })
   router.get('/about', (req, res) => {
       res.end('AboutPage')
   })
   
   router.listen(9000, () => { console.log('服务已启动...')})
   ```

2. 路由中间件的使用
   ```js
   import express from 'express'
   const router = express()
   
   const checkCodeMiddleware = (req, res, next) => {
       if (req.query.code === '521') {
           next() // next() 函数在 Express 中用于将控制权传递给下一个中间件函数或路由处理函数
       } else {
           res.send('Code错误')
       }
   }
   
   router.get('/home', checkCodeMiddleware, (req, res) => {
       res.send('HomePage') 
   })
   router.get('/about', checkCodeMiddleware, (req, res) => {
       res.send('AboutPage')
   })
   
   router.listen(9000, () => { console.log('服务已启动...')})
   ```

3. 静态资源中间件的使用
   ```js
   router.use(express.static('静态资源文件路径'))
   ```

4. 获取请求体：使用urlencoded函数，通过req.body获取
   ```js
   const express = require('express')
   
   const app = express()
   const urlencodedParser = express.urlencoded({ extended: false })
   
   app.get('/login', (req, res) => {
       res.sendFile(__dirname + '/index.html')
   }) // index.html为一个表单
   app.post('/login', urlencodedParser, (req, res) => {
       console.log(req.body)
       res.send('Login Successful')
   })
   
   app.listen(9000, () => { console.log('server is running...') })
   ```

#### 防盗链

#### 路由模块化

将相同功能的路由放入单独的一个文件中管理，作为模块导出和导入，在主文件中使用app.use()方法进行使用

