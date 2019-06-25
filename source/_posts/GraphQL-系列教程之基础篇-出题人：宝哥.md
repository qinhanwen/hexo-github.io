---
title: GraphQL 系列教程之基础篇-出题人：宝哥
date: 2019-04-20 23:14:52
tags: 
- GraphQL 系列教程之基础篇-出题人：宝哥
categories: 
- GraphQL 系列教程之基础篇-出题人：宝哥
---

> 出题人：[宝哥](<http://www.semlinker.com/>)
>
> GraphQL 系列教程之基础篇

<https://segmentfault.com/a/1190000011263214>
### 一、GraphQL 是什么，它有哪些特点？

#### GraphQL 的定义

GraphQL 是一种新的 API 标准，是一个用于 API 的查询语言，是一个使用基于类型系统来执行查询的服务端运行时（类型系统由你的数据定义）。GraphQL 并没有和任何特定数据库或者存储引擎绑定，而是依靠你现有的代码和数据支撑。

#### GraphQL 的特点

1. 没有和任何特定数据库或者存储引擎绑定		  	

2. 可以精确的获取需要的数据，合并多次请求

### 二、GraphQL vs REST API

#### REST API

1）通过不同的方法，请求不同路径，后端使用不同的方式对路径下的资源处理(比如：GET 查询，POST 新增，PUT 修改，DELETE 删除)。

2）比如说要得人员的 id，和人员的详细信息，可能就要发送 2 次的请求（先获取id，再拿着id去获取详细信息）。

#### GraphQL

1）只会返回你需要的数据

2）可以合并多次请求

比如查询：

```
{
  user(id:"100"){
    name,
    email
  }
}
```

只需一次查询，只返回你需要的

```json
{
  "user": {
    "name": "qinhanwen",
    "email": "xxx@qq.com" 
  }
}
```

### 三、在 GraphQL 中 Schema 有什么作用？

1）Schema 是关于数据的描述，能选择什么字段，服务器会返回哪种对象，这些对象下有哪些字段可用，都描述在 Schema 里。

2）每次查询的时候，服务器都会根据 Schema 验证并且执行查询。

```typescript
type Character {
  name: String!
  appearsIn: [Episode!]!
}
//Character是GraphQL对象类型
//String! 表示非空字符串
//[Episode!]! 非空的，每一项都是Episode类型的数组
```

### 四、在 GraphQL 中 Resolver 有什么作用？

#### Resolver 的作用

1）Resolver 是决定 Schemas 中的 field 该如何执行的函数；

2）Query（包括query、mutation、subscription）和与之对应的 Resolver 是同名的。

### 五、请利用 graphql-yoga 搭建 GraphQL 服务器，完成用户列表的查询，用户信息至少包含 id 和 name 字段

```javascript
const { GraphQLServer } = require('graphql-yoga');

const userList = [
    { id: 1, name: 'qinhanwen' },
    { id: 2, name: 'zengheiren' }
];

const typeDefs = `
    type User{
       id: ID!
       name: String!
    }

    type Query {
        users: [User!]!
    }
    `

const resolvers = {
    Query: {
        users: () => userList
    }
}

const server = new GraphQLServer({ typeDefs, resolvers })
server.start(() => console.log('Server is running on localhost:4000'))
```





提示：为了方便开发与测试，建议安装 nodemon 这个工具，然后通过配置 npm scripts 快速启动服务器。