title: 使用 JWTs 做浏览器端 RESTful API 认证
date: 2017-09-09 01:32  
tag:
 - RESTful
 - API认证
 - JWTs

photos:
 - /img/2017/cfakepathKeycard.png 
---

梳理了一下做 RESTful API 认证的流程，使用 Node 结合 `passport.js`，最终实现：router 中间件做认证，通过认证后续流程可以获取到认证的用户信息。

<!--more-->

参考：

https://cnodejs.org/topic/53c652bfc9507b404446ee40

https://juejin.im/entry/58c3bfac570c35006d5b23dc

http://passportjs.org/docs/overview

https://github.com/auth0/node-jsonwebtoken

梳理了一下做 RESTful API 认证的流程，使用 Node 结合 `passport.js`，最终实现：router 中间件做认证，通过认证后续流程可以获取到认证的用户信息。
## 关于JWTs
JSON Web Tokens

- 可以在请求头发送 Authorization -> Bearer  + token 来验证 API
- 可以在请求体 body 中设置 access_token 字段来验证 API

## API设计中的token的思路
1. 设定一个密钥比如 `key = any%^Y&)words`
2. 这个 key 只有发送方(服务器)知道
3. 登录授权时，服务器组合各个参数，用用户名、密码、key、有效期按照一定的规则生成一个 access_key，然后返回给客户端(浏览器)。
4. 客户端请求需要授权的 API 时，需要附上前面的 access_key，服务器解析并验证 access_key 是否有效，然后做出对应的返回结果。

## 具体实现
依赖：

- node-jsonwebtoken (https://github.com/auth0/node-jsonwebtoken)
- passport.js
- passport-http-bearer (https://github.com/jaredhanson/passport-http-bearer)

### 1.入口文件 app.js
```js
const express = require('express');
const passport = require('passport'); // 用户认证模块passport
const configurePassport = require('./configure_passport');


const app = express();
configurePassport(passport); // 配置 passport
app.use(passport.initialize());
```

入口文件里要配置并初始化 passport

### 2.登录成功 auth.js
```js
const jwt = require('jsonwebtoken');

// 登录成功后
const token = jwt.sign({
        id: user.get('uid'),
      }, config.jwtSecret, { expiresIn: config.jwtExpiresIn });

// token返回给浏览器
return User.update(user, { logintoken: token })
	.then(() => res.send({ token, expiresIn: config.jwtExpiresIn }))
	.catch()

```

登录成功后，结合 **userId**, **自定义 Secret**, **有效期** 生成一个 token ，token 存到数据库，并返回给浏览器。

浏览器后续请求会附带这个 token，服务器拿到该 token 后进行验证，验证成功后，将得到：是否格式正确，是否在有效期内，以及加密前的 userId。如果 token 有效，再去数据库核对并取得对应 user。随后路由中间件将对验证的成功结果，user 信息进行处理(返回给浏览器看结果)。
### 3.配置文件 configure_passport.js
```js
const Strategy = require('passport-http-bearer').Strategy;
const User = require('../proxy/user'); // Model
const config = require('../config');
const jwt = require('jsonwebtoken');

// 这个配置就是告诉passport怎么来认证，什么情况认证通过，什么情况是认证失败
module.exports = (passportIns) => {
  passportIns.use(new Strategy((token, cb) => {
    // jwt 会验证是否有效, 是否过了有效期
    jwt.verify(token, config.jwtSecret, (err, decoded) => {
      if (err) return cb(err.name);
      console.log('verifyInfo:', decoded);

      // token有效
      return User.getUsersByQuery({ logintoken: token })
        .then((user) => {
          if (!user) return cb('userNotFound');
          return cb(null, user);
        })
        .catch(dbErr => cb(dbErr));
    });
  }));
};
```

### 4.路由配置 router.js
```js
// 提交登录，登录成功会进入到前面part2的代码处
router.post('/api/v1/signin', signController.signin);
// 获取用户信息前，进行登录验证。 authMiddleware:权限验证中间件
router.get('/api/v1/userinfo', authMiddleware, userController.userInfo);

function authMiddleware(req, res, next) {
  const fn = passport.authenticate('bearer', (err, user) => {
    if (err) return res.send(errResponse('认证失败', 401), 401);
    if (!user) return res.send(errResponse('未获取到用户', 401), 401);
    
    // 认证通过
    req.user = user;
    return next();
  });

  // r -> function(req, res, next) {...}
  fn(req, res, next);
}
```
这里最难理解。

访问到`/api/v1/userinfo`时，

首先进入 authMiddleware， 这里我是自定义回调函数（也就是自定义认证失败时返回给客户端的数据），if (err) 则返回“认证失败”，if (!user)则返回“未获取到用户”

认证成功时，req 对象上加上一个 user 属性，值为 configure_passport.js 中拿到的 user，next()

fn(req, res, next); 必须调用一次。因为 passport.authenticate 的返回值实际上是一个
```
function(req, res, next) {...}
```
这样子的函数，调用时才能把 `req, res, next` 传入进去。

认证通过才会进入到 userController.userInfo 方法中，方法中可以通过 req.user 拿到登录的用户信息

## 总结
- passport 的文档很难理解，用它来做 Restful API 相关的文档又很少。

- API 认证简单说来是：服务器加密--传给客户端--客户端请求时携带凭证--服务器解密验证

- 刚写 Node，可能会有一些理解不对的地方，欢迎指出。


