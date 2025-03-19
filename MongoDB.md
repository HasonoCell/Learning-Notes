# MongoDB数据库

## 三个主要概念

1. 数据库（database）数据库是一个数据仓库，数据库服务（如MongoDB服务）下可以创建很多数据库，数据库中可以存放很多集合
2. 集合（collection）集合类似于JS中的数组，在集合中可以存放很多文档
3. 文档（document）文档是数据库中的最小单位，类似于JS中的对象

我们可以通过JSON文件来理解MongoDB中的概念

一个**JSON文件**好比是一个**数据库**，一个MongoDB服务下可以有N个数据库，JSON文件中的**一级属性的数组值**好比是**集合**，数组中的对象好比是**文档**，对象中的属性有时也称之为**字段**

```js
{
    "accounts": [
        {
            "id": "3-YLju5f3",
            "title": "买电脑",
            "time": "2023-02-08",
            "type": "-1",
            "account": "5500",
            "remarks": "为了上网课"
        },
        {
            "id": "3-YLju5f4",
            "title": "请女朋友吃饭",
            "time": "2023-02-08",
            "type": "-1",
            "account": "214",
            "remarks": "情人节聚餐"
        },
    ],
        "users":[
            {
                "id": 1,
                "name": "zhangsan",
                "age": 18
            },
            {
                "id": 2,
                "name": "lisi",
                "age": 20
            },
        ]
}
```

## 数据库和集合命令

命令行输入 `mongod` 打开服务器，输入 `mongo` 链接服务，在链接服务的命令行窗口下输入数据库和集合命令 
```shell
数据库指令
show dbs // 显示所有数据库，只有数据库中有集合时才会显示
use 数据库名 // 切换到指定数据库，若不存在会自动创建数据库
db // 显示当前所在数据库
db.dropDatabase() // 删除当前数据库（需先切换到当前数据库）

集合指令
db.createCollection('集合名称') // 创建集合
show collections // 显示当前数据库中的所有集合
db.集合名.drop() // 删除某个集合
db.集合名.renameCollection('新名字')

文档指令
db.集合名.insert(文档对象) // 增
db.集合名.find(查询条件) // 查
db.集合名.update(查询条件,新的文档) // 改
db.集合名.update({name: 'Hasono'}, {$set: {age: 19}})// 只改某一属性
db.集合名.remove(查询条件) // 删
```

## Mongose

**Mongose**是一个对象文档模型库，是一个可以使我们**通过代码而非命令行来操作Mongodb数据库**的包

**27017**是mongodb的服务器默认端口

```js
const mongoose = require('mongoose');

// 定义文档结构对象
const BookSchema = new mongoose.Schema({
    name: String,
    author: String,
    price: Number
});

// 创建模型对象
const BookModel = mongoose.model('books', BookSchema);

// 连接 MongoDB 服务
async function connectAndCreateDocument() {
    try {
        await mongoose.connect('mongodb://127.0.0.1:27017/data', { useNewUrlParser: true, useUnifiedTopology: true });
        console.log('Connected to MongoDB');

        // 新增文档
        const result = await BookModel.create({
            name: '红楼梦',
            author: '曹雪芹',
            price: 40
        });
        console.log('Document created:', result);
    } catch (err) {
        console.error('Error:', err);
    } finally {
        // 关闭连接
        mongoose.connection.close();
        console.log('Connection closed');
    }
}

connectAndCreateDocument();

// 监听连接事件
mongoose.connection.on('error', () => {
    console.log('Error!!');
});

mongoose.connection.on('close', () => {
    console.log('Close Successfully');
});
```

