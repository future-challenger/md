## GraphQL与认证

很多人会问GraphQL怎么认证和授权。最终的答案是GraphQL只是一个查询语言和认证之类的没什么关系，每一个应用都可以有自己的实现方法。但是，我们还是来深入聊聊这个问题。

### 大局上看
基本上，有三种情况会发生：
1. 已经登录的用户发出GraphQL查询，未登录的用户不可以。认证在非GraphQL节点完成。
2. 所有用户都可以发出GraphQL查询，未登录用户可以使用其中的一个子集。认证在非GraphQL节点完成。
3. 所有用户都可以发出GraphQL查询，认证就由GraphQL节点完成。

第一种情况可能是普遍存在的。你已经有一个REST或者RPC节点，只有认证成功的用户可以访问，添加*/graphql*非常的简单。不好的地方是，你的客户端不得不处理GraphQL和非GraphQL两种情况。这可能是一笔技术债。

第二种情况是第一种项目进化以后的结果。最终前端代码只使用GraphQL，只不过久经考验的认证节点还会留着。

第三种情况一般是一个全新的后端会有的形态，尽量避免处理非GraphQL节点。我（作者）还没有见过这样形态的服务端。

### 非GraphQL节点的处理机制
**非GraphQL节点**，我是指cookies、JSON和web tokens，或者HTTP基本认证。基本上无论何种方式，你的server都可以通过认证一个用户、一个请求并最终把数据传输给你的resolver。

这里是一个使用`express-graphql`和cookies是例子（从他们的例子里结果来的）：
```js
var session = require('express-session');
var graphqlHTTP = require('express-graphql');
var MySchema = require('./MySchema');
var app = express();

app.use(session({ secret: 'secret', cookie: { maxAge: 60000 }}));

app.use('/graphql', graphqlHTTP((request) => ({
  schema: MySchema,
  rootValue: { session: request.session },
  graphiql: truem
})));
```

在express里，请求在一个比GraphQL路由更早的中间件处理了。之后，请求才会到达GraphQL代码。我们知道请求是从哪里来的。我们甚至都可以在请求到达GraphQL代码以前，把请求重定向到登录页面。

下面的例子使用了`express-session`，但是处理的原则和`express-jwt`差不多。在GraphQL层面上，你的schema代码会是这样的：
```js
new GraphQLObjectType({
  name: 'Secrets',
  fields: {
    bigSecret: {
      type: GraphQLString,
      resolve(parentValue, _, { rootValue: { session } }) {
        return getBigSecret(session);
      }
    }
  }
});

```

只要能取到*session*，那么用户就可以访问其他相关的资源了。或者，如果*session*不存在，那么你按照你的设计抛出错误或者实现其他的处理。

关键是`rootValue`并没有在我们的GraphQL模式中定义为一个公开的字段或者参数，我们不信任客户端直接发送过来的数据，所以它是由server的其他代码注入的。

### 使用GraphQL时的实现机制
但是我们要完全的使用GraphQL呢？以上的方法可以在使用了`express-graphql`的时候使用。但是无法迁移到其他的实现里。

在少数的例子里，Facebook谈到了[ concept of a viewer field](https://gist.github.com/wincent/598fa75e22bdfa44cf47#how-does-graphql-know-what-type-of-data-is-being-requested-by-a-node-root-call)。主要的思想是你的应用的数据和谁访问相关，所以全部的其他字段的数据都嵌入到里面。实际情况是，不可能所有的数据都和访问者相关。但是这么做的话，你可以有改变的余地。
```js
{
  viewer {
    name
    friends {
      name
    }
    getProfile(id: String!) {
      name
    }
  }
}

```
注意即使和访问者无关的`getProfile`字段也放在了`viewer`字段里，为了以防万一哪天要限制访问者可以访问的数据的时候处理起来就简单了。

一个像Facebook一样的APP为了保护隐私，有很多什么人可以查看什么数据的逻辑处理。即使是一个简单的APP也不会让用户查看他没有创建的数据。一个常用的方法是修改URL里的userID来查看一些私有数据，如果server不检查用户所有权的话。使用一个单一的`viewer`字段就让所有权检查简单了很多。

上面的`schema`也可以和非GraphQL节点的认证方法一起使用。但是如果我们这么干的话呢：
```js
{
  viewer(token: String) {
    name
  }
}

```
如果不是用header或者查询参数（比如：JWT、OAuth、等），我们可以把它放在GraphQL的查询里。你的`schema`的代码可以使用JWT库等工具直接解析传过来的*token*。

	**注意**：永远使用HTTPS来传输敏感数据。

要发出新的token，mutation就可以使用了：
```
mutation {
  createToken(username: String!, password: String!) {
    token
    error
  }
}

```

我们可以认证放在mutation里，要么返回一个`token`要么返回一个错误。这样前端就可以把token存起来在之后的请求里使用了。






