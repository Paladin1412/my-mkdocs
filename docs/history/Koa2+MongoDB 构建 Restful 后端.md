> Nodejs风靡以来，全栈、前后端分离、跨平台等关键词，早已让前端工程师耳朵起茧。不会完整独立的开发一个系统的前端，已经不能称其为合格。



# 0. 前言

事实上，本博客基于Nodejs采用前后端分离架构，分为三个独立的部分。

1. 前端： Dva+antd，博客展示；
2. 管理端：Dva+antd，博客，标签，图库，用户权限等数据的CRUD操作；
3. 后端： Koa2+MongoDB提供restful api供前端访问。

这里主要介绍第三部分，后端的架构思路和实现技术。



## Koa2 简介

Koa是一个极简的Web框架，较之Express更轻、更优雅、更健壮。

Koa通过组合不同的 `generator`，可以避免重复的回调函数嵌套，并极大地提升错误处理的效率。

Koa2又增加了`async / await`语法的支持。



## MongoDB 介绍

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富、最像关系数据库的。功能强大，便于扩展。

```
# db.js
const mongoose = require('mongoose');

mongoose.connect(''mongodb://db:27017/sb_db'', { useMongoClient: true });

// 连接成功 
mongoose.connection.on('connected', () => {
  console.log('Mongoose connection open to 27017');
});
```



# 1. 项目文件及运行

## 1.1 文件目录

![file-tree](http://47.94.165.69:3031/upload/aaa/8a7b8d95b24a4.png) 



## 1.2 安装

安装koa依赖包，架设HTTP服务并运行。

```
$ npm start
```

其中，`package.json` ，`app.js`， `www`，文件如下所列。

```
/* package.json */
{
  "name": "koa-demo-1",
  "version": "0.0.1",
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "koa": "^2.2.0",
    "koa-router": "^7.2.0",
    "mongoose": "^4.10.2",
  }
}
```

```
// # app.js
const Koa = require('koa');
const app = new Koa();
module.exports = app;
```

```
// # /bin/www
const http = require('http');
const app = require('../app');

// Create HTTP Server
const server = http.createServer(app.callback());

// Listen on this port
server.listen(3030);
server.on('listening', () => {
  console.log("Listening on 3030");
});
```



# 2. 核心架构

## 2.1 模型

数据模型Model，对数据进行抽象，使用`mongoose`进行建模。

- Schema  ：  一种以文件形式存储的数据库模型骨架，不具备数据库的操作能力

* Model   ：  由Schema发布生成的模型，具有抽象属性和行为的数据库操作对
* Document  ：  由Model创建的实体，它的操作也会影响数据库

```
// PostMoel.js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var postSchema = new Schema({
	title: { type: String, require: true }
});

postSchema.add({
	author: { type: Schema.Types.ObjectId, ref: 'User' },
	state: { type: String, default: 'draft' },
	tags: [{ type: Schema.Types.ObjectId, ref : 'PostTag'}],
	brief: { type: String },
	content: { type: String },
	created_at: { type: String },
	picture: { type: Schema.Types.ObjectId,  ref: 'Image' },
});

var Post = mongoose.model("Post", postSchema);
module.exports = Post; 
```



## 2.2 控制器

后端的逻辑层Controller，为抽象出来的类定义方法，完成对数据库内条目的CRUD操作。

```
// PostCtrl.js
const { PostModel } = require('../models');

class PostCtrl {

	static async articles(ctx){
		const {_page, _limit} = ctx.query;
		let limit = parseInt(_limit)||10;
		let skip = (parseInt(_page)-1)*limit||0;
		const total =  await PostModel.count();
		const list = await PostModel.find().populate("tags picture").sort({'created_at':-1}).skip(skip).limit(limit);
		if(!list) return ctx.error({ msg: 'not found' })
		return ctx.success({ data : { list, total } });
	}
	
	/*
	...
	*/
	static async delete(ctx){
		const { _id } = ctx.request.body;
		const result =  await PostModel.findByIdAndRemove(_id);
		if(result) 
			return ctx.success({ data: result})
		else
			return ctx.error({ message: 'delete fail' })
	}
}

module.exports = PostCtrl;
```



## 2.3 路由

路由Routes，Koa的核心中间件，采用官方提供的`koa-router`模块。对外开放的API，接收GET，PUT，post，PATCH等HTTP请求，并调用controllers里定义的各种方法。

```
// app.js
const apiRouter = require('./rest/routes');
const app = new Koa();
app
	.use(apiRouter.routes())
	.use(apiRouter.allowedMethods());
```

```
/* 
 routes/
 └── index.js
*/
const { PostCtrl } = require('../controllers');
const router = require('koa-router');
const route = new router();

route
	.get('/api/articles',PostCtrl.articles)
	/* 
	... 
	*/
	.del('/api/articles',isBearerAuthenticated, PostCtrl.delete)
	
module.exports = route;
```



# 3. 中间件

## 3.1 passport策略

passport是一个基于Nodejs的认证中间件，干净，易维护，也方便集成到其他的应用中。

本项目采用了两种不同的策略，LocalStrategy 和 BearerStrategy。LocalStrategy策略，用于匹配本地环境的用户名和密码，可以扩展这个策略，通过数据库的方式进行匹配。BearerStrategy策略，通过一个bearer token来认证用户的http请求，匹配请求header里的authorization。

文件目录如下所示。

```
auth/
├──	strategies
├	├── bearer.js
├	└── local.js
└── index.js
```

```
/**  local.js  **
* 配置local本地策略为LocalStrategy,通过 username，password 来验证用户
*/
const LocalStrategy = require('passport-local').Strategy;
const { UserModel } = require('../../models');

const registerLocal = (passport) => {
	passport.use('local', new LocalStrategy({
	  usernameField: 'username', // 'username' 为 ctx.request.body中提交的名称
	  passportField: 'password'  // 'password' 为 ctx.request.body中提交的名称
	}, async(username, password, done)=>{
		const user = await UserModel.findOne({name: username});
		if(!user) return done(null, false, { message: '用户不存在' });
		if(password !== user.password) 
			return done(null, false, { message: '密码不匹配' });
		 return done(null, user);
	}));
}

module.exports = registerLocal;
```

```
/** bearer.js **
*  配置passport策略为bearer Strategy, 通过 access token 来验证用户访问权限
*/
const BearerStrategy = require('passport-http-bearer').Strategy;
const { AccessTokenModel } = require('../../models');

const registerBearer = (passport) => {
	passport.use('bearer', new BearerStrategy( async(token, done) => {
		const accessToken = await AccessTokenModel.findOne({ token }).populate('user');
		if(!accessToken)  return done(null, false);
		const { name, role, password } = accessToken.user;
		return done(null, { name, role, password }, {scope: '*'})
	}))
}

module.exports = registerBearer;
```

```
//index.js
const passport = require('koa-passport');
const registerLocal = require('./strategies/local')
const registerBearer = require('./strategies/bearer')
const { UserModel } = require('../models');

//将user.name序列化为SESSIONID，存储于cookie, 见ctx.session.passport.user
passport.serializeUser((user,done) => done(null, user));
//session反序列化，参数为提交的SESSIONID，从数据库查询user，存储于ctx.request.user
passport.deserializeUser(async (user, done) => done(null, await UserModel.findOne({_id: user._id})));

//使用local策略，一般用于本地login,logout等操作
registerLocal(passport);
//使用bearer策略，一般用于client通过token获取服务器数据
registerBearer(passport);

const isLocalAuthenticated = async(ctx, next) => {
    if(ctx.isAuthenticated()){   //authenticate via session.passport
    	await next();
    else
      	await ctx.redirect('/server/login');
}

//带bearer tokens的请求不需要session support, 因此session option可被set to false.
const isBearerAuthenticated = passport.authenticate('bearer', {session: false})

module.exports = { passport, isLocalAuthenticated, isBearerAuthenticated };
```

```
// app.js
const { passport } = require('./rest/auth');
const app = new Koa();

app.use(passport.initialize()); //初始化
app.use(passport.session());    //使用session
```



## 3.2 其它中间件 

- format_response.js 用来格式化从服务器端请求的资源

```
const format_response = async (ctx, next) => {
	ctx.error = ({ data, message, error }) => {
	    ctx.body = { code: -200, message, data, error };
	}
	ctx.success = ({ data, message }) => {
		ctx.body = { code: 200, message, data };
	}
	await next();
}

module.exports = format_response;
```



- upload.js，使用busboy来处理form data类型的post请求，包括图片等类型的文件

```
const path = require('path');
const fs = require('fs');
const Busboy = require('busboy');
/* {FormData} 上传文件
 * @param  {object} ctx: koa上下文
 * @param  {object} options: 文件上传参数， path文件存放路径
 * @return {promise}         
 */
const upload = (ctx, options) => {
	let req = ctx.req
  	let busboy = new Busboy({ headers: req.headers })
	let _filePath = path.join(__dirname, '../../public/upload', options&&options.path||'/');
	if(!fs.existsSync(_filePath)) fs.mkdirSync(_filePath);
	
	return new Promise((resolve, reject) => {
  		let result = { fields: {}, files: {} }
    	
    	//form的普通字段和值
  		busboy.on('field',function(fieldname, val, fieldnameTruncated, valTruncated, encoding, mimetype){
  		
 			if(!result.fields[fieldname]){
 				result.fields[fieldname] = val
 			}else if(result.fields[fieldname] instanceof Array){
 				result.fields[fieldname].push(val)
 			}else{
 				result.fields[fieldname] = [].concat(result.fields[fieldname], val)
 			}
 		});

  		//form的文件
 		busboy.on('file', function(fieldname, file, filename, encoding, mimetype) {
			//过滤octet-stream类型的未知文件
			var name, url, size = 0;
			if(!mimetype.includes("application/octet-stream")){ 
				name = Math.random().toString(16).substr(2)+'.'+filename.split('.')[1];
				// 文件保存到指定路径
				file.pipe(fs.createWriteStream(path.join(_filePath, name)));
			}

  			file.on('data', (data) => { size = data.length;} );
			
			file.on('end', function(data) {
				if(name){
					url = path.join('/upload/', options&&options.path, name);
				   	if(!result.files[fieldname]){
	 					result.files[fieldname] = { name, url, size, mimetype }
		 			}else if(result.files[fieldname] instanceof Array){
		 				result.files[fieldname].push({ name, url, size, mimetype })
		 			}else{
		 				result.files[fieldname] = [].concat(result.files[fieldname], { name, url, size, mimetype })
		 			}
				}
			});
 		})

 		busboy.on('finish', () => resolve(result) );
 		
 		return req.pipe(busboy);
  	})
}

module.exports = upload;
```
