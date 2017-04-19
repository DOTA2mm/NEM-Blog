## NodeJS&Express&MongoDB搭建简易博客

> [参考链接](https://github.com/nswbmw/N-blog)

### 使用模块清单
- `config-lite` (读取配置文件)
- `ejs` (模板引擎)
- `mongolass` (mongodb 的驱动库)
- `express` (web服务框架)
- `express-session` (会话（session）支持中间件)
- `connect-flash` (基于 session 的用于通知功能的中间件)
- `connect-mongo` (将session存入mongodb,类似的有connect-redis，需结合 express-session 使用)
- `express-formidable` (处理表单数据的中间件)
- `express-winston` (提供日志记录的中间件)
- `marked` (处理md格式内容)
- `moment` (时间格式化插件)
- `objectid-to-timestamp` (objectid装换成时间毫秒数)
- `sha1` (数据加密)
- `winston` (日志)
- `mocha` (测试库) 
- `supertest` (http接口断言库)

### 一些细节

#### 模板数据传递
有4种往模板中传递数据的方法：
1. 在路由中`res.render('posts', {posts: posts})`第二个参数传递的对象可在模板中直接引用
2. 通过`app.locals` (程序层面，存在于整个生命周期) 这一对象向模板中传递数据,`app.locals` 上通常挂载常量信息（如博客名、描述、作者信息）
3. 通过`res.locals` (请求层面，存在于当前请求中) 这一对象向模板中传递数据，一般`res.locals`上通常挂载变量信息，即每次请求可能的值都不一样（如请求者信息，
`res.locals.user = req.session.user`）
3. 通过 `app.set(key, value)` 得到的 `app.setting` 也可以在模板中访问： `setting.key`

#### 权限控制
只有登录才能发表文章评论，并且只有自己才能删自己ed文章评论（不考虑管理员权限）这些基本的权限控制。  
`middlewares/check.js`这个模块中有两个路由中间件：`checkLogin`和`checkNotLogin`:  
```js
module.exports = {
  checkLogin: function checkLogin(req, res, next) {
    if (!req.session.user) {
      req.flash('error', '未登录')
      return res.redirect('/signin')
    }
    next()
  },

  checkNotLogin: function checkNotLogin(req, res, next) {
    if (req.session.user) {
      req.flash('error', '已登录')
      return res.redirect('back') // 返回之前的页面
    }
    next();
  }
};
```
通过检查session中是否有用户信息来看有没有登录，在需要权限判断的路由中使用：  
```js
// GET /signin 登录页
router.get('/', checkNotLogin, function (req, res, next) {
  res.render('signin')
})

// POST /signin 用户登录
router.post('/', checkNotLogin, function (req, res, next) {
  var name = req.fields.name
  var password = req.fields.password
  ...
}
```

#### 其他
目前只想到总结这些注意点，还有些代码中有详细注释。还存在一些不太明了的地方需要查阅相关文档……

#### 单元测试
做单元测试需要导出`app`而不是启动程序  
所以我们这样做：
```js
if (module.parent) {
  module.exports = app
} else {
  // 监听端口，启动程序
  app.listen(config.port, function () {
    console.log(`${pkg.name} listening on port ${config.port}`)
  })
}
```
如果直接启动 index.js 则会监听端口启动程序， 如果 index.js 被 require 了，则导出 app 用于测试  
`module.parent`用于判断 require 的模块是否是程序的入口文件，如果是有`require.main === module`或`module.parent === null`

### 参考资料
- [Express官方文档](http://expressjs.com/zh-cn/)
- [Nodejs中connect-flash的使用](http://mclspace.com/2015/12/03/nodejs-flash-note/)
- [Express 模板传值对象app.locals、res.locals](https://itbilu.com/nodejs/npm/Ny0k0TKP-.html)
- [express-session文档](https://github.com/expressjs/session)