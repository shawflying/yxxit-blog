---
layout: post
title: token
date: 2018-04-27 14:54:28
tags: api
---

JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。其JWT的组成一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。

## Token 相对于token Auth的优点

![img](https://ws4.sinaimg.cn/large/006tKfTcgy1fqr8iefeeij30w80gn422.jpg)

- 支持跨域访问: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.
- 无状态(也称：服务端可扩展行):Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.
- 更适用CDN: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可.
- 去耦: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.
- 更适用于移动应用: 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。
- CSRF:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。
- 性能: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多.
- 不需要为登录页面做特殊处理: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理.
- 基于标准化:你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）.


### 安装模块

```
npm i --save jsonwebtoken
```

### 生成token信息

```Js
const jwt = require('jsonwebtoken');

//1. 验证用户名，密码是否正确
//2. 从新生成token信息
//3. 将生成的token信息存储在mongo中
//4. 将生成的token信息响应出去
User.findOne({
	name: req.body.name
}, (err, user) => {
	if (err) throw err
	if (!user) return res.fail("认证失败,用户不存在!");
	p.log(user)
	//校验密码是否正确
	if (user.password != req.body.password) return res.fail("认证失败,密码错误!");
	//密码正确，生成token，根据用户名生成token
	//将信息存储在 生成sign  
	let token = jwt.sign({
		name: user.name
	}, config.token_secret, {
		expiresIn: 60 * 1
	}) //token 信息，秘钥，过期时间（以秒为单位）
	user.token = token
	user.save((err) => {
		if (err) return res.fail("认证失败");
		return res.success({
			token: token,
			name: user.name
		});
	});
})
```
载荷（Payload）

```json
    { "iss": "Online JWT Builder", 
      "iat": 1416797419, 
      "exp": 1448333419, 
      "aud": "www.example.com", 
      "sub": "jrocket@example.com", 
      "GivenName": "Johnny", 
      "Surname": "Rocket", 
      "Email": "jrocket@example.com", 
      "Role": [ "Manager", "Project Administrator" ] 
    }
```
- iss: 该JWT的签发者，是否使用是可选的；
- sub: 该JWT所面向的用户，是否使用是可选的；
- aud: 接收该JWT的一方，是否使用是可选的；
- exp(expires): 什么时候过期，这里是一个Unix时间戳，是否使用是可选的；
- iat(issued at): 在什么时候签发的(UNIX时间)，是否使用是可选的；
  其他还有：
- nbf (Not Before)：如果当前时间在nbf里的时间之前，则Token不被接受；一般都会留一些余地，比如几分钟；，是否使用是可选的；



### 第三方接入时，请求携带token

```Js
var headers = {
    'Content-Type': 'application/json',
    "token": "",
    "version": "1.0"
};
var options = {
	url: url,
	method: 'GET',
	headers: headers,
	json: true,
	timeout: config.timeout
};

request(options, function(error, response, content) {
	callback(error, response, content);
});
```



### 前端页面获取token，并存储记录

认证获取token后，将token存放位置

1. localStorage
2. sessionStorage
3. cookie
4. 针对单页面大应用，存放在某个标签上，或者公共的变量

```Js
var str = sessionStorage.getItem("userInfo");
var currUser = JSON.parse(str);
var userId = currUser["userID"];
var token = 'Bearer ' + currUser.token;
```

### 前端请求时携带token

* Url?token=xxxx
* Post  参数中携带token
* Header 参数中携带token，推荐使用这种方式

```Js
function GetInfo(userId) {
    $.ajax({
        url: 'http://*******/api/orders',
        data: {
            currUserId: userId,
            type: 1
        },
        beforeSend: function(request) {
            request.setRequestHeader("token", token);
        },
        dataType: 'JSON',
        type: 'GET',
        success: function (list) {
        },
        error: function () {
        }
    });
}
```



### 获取页面传入的token信息

```Js
var token = req.body.token || req.query.token || req.headers['x-access-token'];
var token = rq.body.token || rq.query.token || rq.headers["x-access-token"]; // 从body或query或者header中获取token
let token = req.body.token || req.query.token || req.headers.token;
if (_.isEmpty(token)) return res.fail("The token is lose")//token 信息丢失
req.headers.token = token

//最麻烦的
if (_.isEmpty(req.headers.token)) { //headers.token有值，直接跳转
	if (!_.isEmpty(req.body.token)) {
		req.headers.token = req.body.token;
	} else if (!_.isEmpty(req.query.token)) {
		req.headers.token = req.query.token;
	}

	if (_.isEmpty(req.headers.token)) return res.fail("The token is lose") //token 信息丢失
}
```



### Bearer 存在的意义?

```js
 config.headers.Authorization = `Bearer ${store.state.token}`
```



### token 中间件校验verify

```Js
 let token = req.body.token || req.query.token || req.headers.token; //根据需要可以调整顺序
 if (_.isEmpty(token)) return res.fail("The token is lose") //token 信息丢失

 //校验token中包含的信息
 jwt.verify(token, config.token_secret, function(err, data) {
 	if (err) return res.fail(err.message);
 	req.headers.token = token
 	next()
 })
```

### 是否可以使用session 进行存储？

1. 可以通过token获取session中的数据
2. 通过session判断token是否存在的方式进行校验
3. 但是，这样不安全，并且性能比较低

```Js
User.findOne({
	token: token
}, (err, data) => {
	console.log(data)
	if (err) return res.fail("token is 失效");
	if (!data) return res.fail("token is null")
	next()
})
```

 

## 关于Token，你应该知道的一些事

#### 一、Token 应该被保存起来（放到 `local / session stograge` 或者 `cookies`）

#### 二、Tokens 除了像 cookie 一样有`有效期`，而且你可以有更多的操作方法

1. cookies 可以在浏览器关闭后删除（session cookies）；
2. 另外你可以实现服务器端的检查（通常由你使用的 Web 框架完成），还有也可以实现绝对有效期或弹性有效期
3. Cookies 可以带有有效期地保存起来（浏览器关闭后也不删除）。

而在 tokens 的使用中，一旦 token 过期，只需要重新获取一个。你可以使用一个接口去刷新 token：

1. 让旧的 token 失效；
2. 检查这个用户是不是还存在，权限是否被取消或者任何对你的程序来说是有必要的；
3. 得到一个更新了有效期的 token。

你甚至可以把 token 原来的发布时间也保存起来，并且强制在两星期后重新登录什么的。

```Js
app.post('/refresh_token', function(req, res) {
	// verify the existing token 
	var profile = jwt.verify(req.body.token, secret);
	// if more than 14 days old, force login 
	if (profile.original_iat - new Date() > 14) {
		// iat == issued at return res.send(401); 
		// re-logging } 
		// check if the user still exists or if authorization hasn't been revoked 
		if (!valid) return res.send(401);
		// re-logging 
		// issue a new token 
		var refreshed_token = jwt.sign(profile, secret, {
			expiresInMinutes: 60 * 5
		});
		res.json({
			token: refreshed_token
		});
	}
});
```

如果你需要撤回 tokens（当 token 的生存期比较长的时候这很有必要）那么你需要一个 token 的生成管理器去作检查。

### 三、每个 CORS（跨域资源共享）请求都会带上预请求（Preflight request）

#### 四、XSS要更容易防范

#### 五、注意 token 的大小

Token 机制在每次请求 API 的时候，都需要带上一个 Authorization的 Http Header 。

Token 的大小其实由你储存在 token 中的信息量所决定，例如可能有 nickname，openid等开发者另外加上的信息。

#### 六、有需要的话，要加密并且签名 token

虽然 TLS/SSL 机制可以隔绝大多数中间人攻击，但是如果 token 中带有了用户的敏感信息，开发者也应该要加密这些信息。
使用 JWT（文中第 9 点） 可以加密 token，但是由于目前大多数 Web 框架还未支持 JWT，所以可以使用 AES-CBC 算法加密 token。

```
app.post('/authenticate', function(req, res) {
	// 校验用户； 
	// 加密 token； 
	var encrypted = {
		token: encryptAesSha256('shhhh', JSON.stringify(profile))
	};
	// 给加密后的 token 签名； 
	var token = jwt.sign(encrypted, secret, {
		expiresInMinutes: 60 * 5
	});
	res.json({
		token: token
	});
})

function encryptAesSha256(password, textToEncrypt) {
	var cipher = crypto.createCipher('aes-256-cbc', password);
	var crypted = cipher.update(textToEncrypt, 'utf8', 'hex');
	crypted += cipher.final('hex');
	return crypted;
}
// 上面就是 encrypt-then-MAC （加密后签名）做法。
```



## 参考目录：
https://stormpath.com/blog/build-secure-user-interfaces-using-jwts
https://auth0.com/blog/2014/01/27/ten-things-you-should-know-about-tokens-and-cookies/
https://www.quora.com/Is-JWT-JSON-Web-Token-insecure-by-design
https://github.com/auth0/node-jsonwebtoken/issues/36
http://christhorntonsf.com/secure-your-apis-with-jwt/
https://www.cnblogs.com/xiekeli/p/5607107.html
https://www.jianshu.com/p/f9faeac8bd5e