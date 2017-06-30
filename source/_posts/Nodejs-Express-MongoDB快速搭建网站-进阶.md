---
title: Nodejs-Express-MongoDB快速搭建网站(进阶)
date: 2017-06-15 15:08:07
tags:
    - Nodejs
    - Express
    - MongoDB
categories:
    - Nodejs
---

> 注：承接[《Nodejs + Express + MongoDB快速搭建网站》](https://jochen-m.github.io/2017/05/26/Nodejs-Express-MongoDB快速搭建网站/)

### 开发用户的注册登录功能

#### 用户模型及密码处理

schemas/user.js:
```
let mongoose = require('mongoose');
let bcrypt = require("bcrypt");
const SALT_WORK_FACTOR = 10;

let UserSchema = new mongoose.Schema({
  name: {
    unique: true,
    type: String
  },
  password: String,
  meta: {
    createAt: {
      type: Date,
      default: Date.now() 
    },
    updateAt: {
      type: Date,
      default: Date.now()
    }
  }
});

UserSchema.pre('save', function(next){
  let user = this;
  if(this.isNew){
    this.meta.createAt = this.meta.updateAt = Date.now();
  }else{
    this.meta.updateAt = Date.now();
  }

  // 将密码与盐混合后加密处理
  bcrypt.genSalt(SALT_WORK_FACTOR, function(err, salt){
    if(err) return next(err);
    
    bcrypt.hash(user.password, salt, function(err, hash){
      if(err) return next(err);

      user.password = hash;
      next();
    })
  });
});

UserSchema.methods = {
  /**
   * 验证密码是否正确
   */
  verifyPassword: function(_password, cb){
    bcrypt.compare(_password, this.password, function(err, isMatch){
      if(err){
        return cb(err);
      }
      cb(null, isMatch);
    });
  }
};

UserSchema.statics = {
  fetch: function(callback){
    return this
      .find({})
      .sort('meta.updateAt')
      .exec(callback);
  },
  findById: function(id, callback){
    return this
      .findOne({_id: id})
      .exec(callback);
  }
};

module.exports = UserSchema
```

#### 登录注册前端视图

修改head.jade文件：
```
.navbar.navbar-default
  .container
    .navbar-header
      a.navbar-brand(href="/") MovieGoGo
    p.navbar-text.navbar-right
      a.navbar-link(href="#", data-toggle="modal", data-target="#signupModal") 注册
      span &nbsp;|&nbsp;
      a.navbar-link(href="#", data-toggle="modal", data-target="#signinModal") 登录
#signupModal.modal.fade
  .modal-dialog
    .modal-content
      form(method="POST", action="/user/signup")
        .modal-header 注册
        .modal-body
          .form-group
            label(for="signupName") 用户名
            input#signupName.form-control(name="user[name]", type="text")
          .form-group
            label(for="signupPassword") 密码
            input#signupName.form-control(name="user[password]", type="password")
        .modal-footer
          button.btn.btn-danger(type="button", data-dismiss="modal") 关闭
          button.btn.btn-success(type="submit") 提交
#signinModal.modal.fade
  .modal-dialog
    .modal-content
      form(method="POST", action="/user/signin")
        .modal-header 登录
        .modal-body
          .form-group
            label(for="signinName") 用户名
            input#signinName.form-control(name="user[name]", type="text")
          .form-group
            label(for="signinPassword") 密码
            input#signinName.form-control(name="user[password]", type="password")
        .modal-footer
          button.btn.btn-danger(type="button", data-dismiss="modal") 关闭
          button.btn.btn-success(type="submit") 登录
```

#### 注册用户后台存储

app.js文件中添加代码：
```
let User = require('./models/user');

// signup
app.post('/user/signup', function(req, res){
  let _user = req.body.user;

  User.find({name: _user.name}, function(err, user){
    if(err){
      console.log(err);
    }

    if(user.length > 0){
      return res.redirect("/");
    }else{
      let user = new User(_user);

      user.save(function(err, user){
        if(err){
          console.log(err);
        }

        res.redirect('/admin/userlist');
      });
    }
  });
});
```

#### 实现登录逻辑

```
// signin
app.post('/user/signin', function(req, res){
  let user_form = req.body.user;
  let name = user_form.name;
  let password = user_form.password;

  User.findOne({name: name}, function(err, _user){
    if(err){
      console.log(err);
    }

    if(!_user){
      return res.redirect('/');
    }

    let user = new User(_user); 
    user.verifyPassword(password, function(err, isMatch){
      if(err){
        console.log(err);
      }

      if(isMatch){
        console.log("Password is matched");
        req.session.user = user;
        return res.redirect('/');
      }else{
        console.log("Password is not matched");
      }
    });
  });
});
```

#### 利用mongodb做会话的持久化

app.js文件中添加代码：
```
let cookieParser = require('cookie-parser');
let session = require('express-session');
let mongoStore = require('connect-mongo')(session);

app.use(cookieParser());
app.use(session({
  secret: 'moviegogo',
  store: new mongoStore({
    url: dbUrl,
    collection: 'sessions'
  })
}));
```

#### 注销用户、用户退出功能实现

修改head.jade文件：
```
.navbar.navbar-default
  .container
    .navbar-header
      a.navbar-brand(href="/") MovieGoGo
    if user
      p.navbar-text.navbar-right
        span 欢迎您,#{user.name}
        span &nbsp;|&nbsp;
        a.navbar-link(href="/logout") 退出
    else
      p.navbar-text.navbar-right
        a.navbar-link(href="#", data-toggle="modal", data-target="#signupModal") 注册
        span &nbsp;|&nbsp;
        a.navbar-link(href="#", data-toggle="modal", data-target="#signinModal") 登录
```

app.js文件中添加代码：
```
// pre handle user
app.use(function(req, res, next){
    let _user = req.session.user;

    if(_user){
        app.locals.user = _user;
    }
    
    return next();
});

// logout
app.get('/logout', function(req, res){
  delete req.session.user;
  delete app.locals.user;
  res.redirect('/');
});
```

#### 配置入口文件app.js:

```
let logger = require('morgan');

// 用于本地测试
if( 'development' === app.get('env')){
    app.set('showStackError', true);            // 显示栈错误信息
    app.use(logger(':method :url :status'));    // 显示客户端请求信息（请求方式 url 状态）
    app.locals.pretty = true;                   // 格式化代码
    mongoose.set('debug', true);                // mongoose调试模式
}
```

#### 调整目录结构，彻底分离MVC层

```
.
├── app/
│   ├── controllers/
│   │   ├── index.js
│   │   ├── movie.js
│   │   └── user.js
│   ├── models/
│   │   ├── movie.js
│   │   └── user.js
│   ├── schemas/
│   │   ├── movie.js
│   │   └── user.js
│   └── views/
│       ├── includes/
│       │   ├── head.jade
│       │   └── header.jade
│       ├── pages/
│       │   ├── admin.jade
│       │   ├── detail.jade
│       │   ├── index.jade
│       │   ├── list.jade
│       │   └── userlist.jade
│       └── layout.jade
├── config/
│       └── routes.js
│── public/
│   ├── images/
│   ├── js/
│   └── libs/
│       ├── bootstrap/
│       └── jquery/
├── .bowerrc
├── README.md
├── app.js
├── bower.json
└── package.json
```

将app.js文件中路由相关代码移到控制器中，并在app.js中添加：
```
require('./config/routes')(app);
```

简化后的routes.js:
```
let Index = require('../app/controllers/index');
let User = require('../app/controllers/user');
let Movie = require('../app/controllers/movie');

module.exports = function(app) {
    // pre handle user
    app.use(function(req, res, next){
        let _user = req.session.user;
        app.locals.user = _user;
        next();
    });

    // Index
    app.get('/', Index.index);

    // User
    app.post('/user/signup', User.signup);
    app.post('/user/signin', User.signin);
    app.get('/logout', User.logout);
    app.get('/admin/userlist', User.list);

    // Movie
    app.post('/admin/movie/new', Movie.save);
    app.get('/movie/:id', Movie.detail);
    app.get('/admin/movie', Movie.new);
    app.get('/admin/update/:id', Movie.update);
    app.get('/admin/list', Movie.list);
    app.delete('/admin/delete', Movie.delete);
};
```

/app/controllers/index.js:
```
let Movie = require('../models/movie');

// index page
exports.index = function(req, res){
    // console.log("user in session: ");
    // console.log(req.session.user);

    Movie.fetch(function(err, movies){
        if(err){
            console.log(err);
        }

        res.render('index', {
            title: 'MovieGoGo 首页',
            movies: movies
        });
    })
};
```

/app/controllers/movie.js、 /app/controllers/user.js 与 /app/controllers/index.js 类似。

#### 增加注册登录跳转页面

routes.js中增加两条路由：
```
app.get('/signup', User.showSignup);
app.get('/signin', User.showSignin);
```

controllers/user.js中增加两个方法:
```
exports.showSignup = function(req, res){
    res.render('signup', {
        title: "注册"
    });
};

exports.showSignin = function(req, res){
    res.render('signin', {
        title: "登录"
    });
};
```

增加signup.jade和signin.jade两个页面（与之前的模态窗口类似）。

#### 用户权限管理

schemas/user.js中增加字段：
```
role: {
        // 0 - normal user
        // 1 - verified user
        // 2 - professional user
        // ...
        // >10 - admin
        // >50 - super admin
        type: Number,
        default: 0
    }
```

controllers/user.js中增加两个中间件:
```
// middleware for user
exports.signinRequired = function(req, res, next){
    let user = req.session.user;
    if(!user){
        res.redirect('/signin');
    }
    next();
};

exports.adminRequired = function(req, res, next){
    let user = req.session.user;
    if(user.role <= 10){
        res.redirect('/signin');
    }
    next();
};
```

规范路由命名，并修改相关文件的路由信息，如： public/js/admin.js 等。
为所有以 admin 开头的路由增加中间件 User.signinRequired 和 User.adminRequired。
```
// Index
app.get('/', Index.index);

// User
app.post('/user/signup', User.signup);
app.post('/user/signin', User.signin);
app.get('/signup', User.showSignup);
app.get('/signin', User.showSignin);
app.get('/logout', User.logout);
app.get('/admin/user/list', User.signinRequired, User.adminRequired, User.list);

// Movie
app.post('/admin/movie/save', User.signinRequired, User.adminRequired, Movie.save);
app.get('/movie/:id', Movie.detail);
app.get('/admin/movie/new', User.signinRequired, User.adminRequired, Movie.new);
app.get('/admin/movie/update/:id', User.signinRequired, User.adminRequired, Movie.update);
app.get('/admin/movie/list', User.signinRequired, User.adminRequired, Movie.list);
app.delete('/admin/movie/delete', User.signinRequired, User.adminRequired, Movie.delete);
```

### 开发评论和回复功能
*注*：只有登陆的用户才能发表评论和回复
#### 设计评论和回复的数据模型
schemas/comment.js:
```
let mongoose = require('mongoose');
let Schema = mongoose.Schema;
let ObjectId = Schema.Types.ObjectId;

let CommentSchema = new mongoose.Schema({
  movie: {
    type: ObjectId, 
    ref: 'Movie'
  },
  from: {
    type: ObjectId,
    ref: 'User'
  },
  reply: [{
    from: {
      type: ObjectId,
      ref: 'User'
    },
    to: {
      type: ObjectId,
      ref: 'User'
    },
    content: String
  }]，
  content: String,
  meta: {
    createAt: {
      type: Date,
      default: Date.now() 
    },
    updateAt: {
      type: Date,
      default: Date.now()
    }
  }
});
```

#### 评论及回复的存储与展现
在detail.jade中添加评论区和回复区：
```
.panel.panel-default
  .panel-heading
    h3 评论区
  .panel-body
    ul.media-list
  if comments
    each item in comments
      li(style="list-style-type:none").media
        .pull-left
          a.comment(href="#comment", data-cid="#{item._id}", data-tid="#{item.from._id}")
            img.media-object(src="/images/avatar.jpg", style="width: 48px; height: 48px;")
        .media-body
          h4.media-heading #{item.from.name}
          p #{item.content}
          //- 回复部分
          if item.reply && item.reply.length > 0
            each reply in item.reply
              .media
                .pull-left
                  a.comment(href="#comment", data-cid="#{item._id}", data-tid="#{reply.from._id}")
                    img.media-object(src="/images/avatar.jpg", style="width: 48px; height: 48px;")
                .media-body
                  h4.media-heading 
                    | #{reply.from.name}
                    span.text-info &nbsp;回复&nbsp;
                    | #{reply.to.name}
                  p #{reply.content}
            //- 回复部分结束
  #comment
    form#commentForm(method="post", action="/user/comment")
      input(type="hidden", name="comment[movie]", value="#{movie._id}")
      if user
        input(type="hidden", name="comment[from]", value="#{user._id}")
      .form-group
        textarea.form-group(name="comment[content]", row="3")
      if user
        button.btn.btn-primary(type='submit') 提交
      else
        a.navbar-link(href="#", data-toggle="modal", data-target="#signinModal") 登录后评论
  script(src="/js/detail.js")
```

添加路由：
```
app.post('/user/comment', User.signinRequired, Comment.save);
```

评论存储，新建models/comment.js：
```
let Comment = require('../models/comment');

// Routes for comment
exports.save = function(req, res){
  let _comment = req.body.comment;
  let movieId = _comment.movie;

  if(_comment.cid){
    Comment.findById(_comment.cid, function(err, comment){
      let reply = {
        from: _comment.from,
        to: _comment.tid,
        content: _comment.content
      };
      comment.reply.push(reply);
      comment.save(function(err, comment){
        if(err){
          console.log(err);
        }
        res.redirect('/movie/' + movieId);
      });
    });
  }else{
    let comment = new Comment(_comment);
    comment.save(function(err, comment){
      if(err){
        console.log(err);
      }
      res.redirect('/movie/' + movieId);  // 评论后返回当前电影
    });
  }
};
```

为头像增加点击事件：
public/js/detail.js:
```
$(function(){
  $('.comment').click(function(e){
    let target = $(this);
    let toId = target.data('tid');
    let commentId = target.data('cid');

    if($("#toId").length > 0){
      $("#toId").val(toId);
    }else{
      $('<input>').attr({
        type: 'hidden',
        id: 'toId',
        name: 'comment[tid]',
        value: toId
      }).appendTo('#commentForm');
    }

    if($("#commentId").length > 0){
      $("#commentId").val(commentId);
    }else{
      $('<input>').attr({
        type: 'hidden',
        id: 'commentId',
        name: 'comment[cid]',
        value: commentId
      }).appendTo('#commentForm');
    }
  });
});
```

为了展示评论和回复，需要对controllers/movie.js稍作修改：
```
exports.detail = function(req, res){
  let id = req.params.id;
  Movie.findById(id, function(err, movie){
    Comment
    .find({movie: id})
    .populate('from', 'name')       // 连表查询from的name字段，即评论人的名字
    .populate('reply.from reply.to', 'name')  // 连表查询reply.from和reply.to的name字段，即回复人的名字和被回复人的名字
    .exec(function(err, comments){
      res.render('detail', {
        title: "" + movie.title,
        movie: movie,
        comments: comments
      });
    });
  });
};
```

### 电影分类功能的实现
#### 分类的数据模型 schemas/category.js:
```
let CategorySchema = new mongoose.Schema({
  name: String,
  movies: [{
    type: ObjectId,
    ref: 'Movie'
  }],
  meta: {
    createAt: {
      type: Date,
      default: Date.now() 
    },
    updateAt: {
      type: Date,
      default: Date.now()
    }
  }
});
```

#### 分类后台录入及存储
增加路由信息routes.js：
```
let Category = require('../app/controllers/category');

// Category
app.post('/admin/category/save', User.signinRequired, User.adminRequired, Category.save);
app.get('/admin/category/new', User.signinRequired, User.adminRequired, Category.new);
app.get('/admin/category/list', User.signinRequired, User.adminRequired, Category.list);
```

分类的控制器 controllers/category.js:
```
let Category = require('../models/category');

// Routes for category
exports.new = function(req, res){
  res.render('admin_category', {
    title: '后台分类录入页',
    category: {}
  });
};

exports.save = function(req, res){
  let _catetory = req.body.catetory;
  let category = new Caterory(__category);

  category.save(function(err){
    if(err){
      console.log(err);
    }

    res.redirect('/admin/category/list');
  });
};

exports.list = function(req, res){
  Category.fetch(function(err, categories){
    if(err){
      console.log(err);
    }

    res.render('catetorylist', {
      title: '分类列表页',
      categories: categories
    });
  })
};
```

增加相关页面 admin_category.jade、categorylist.jade。（类似于admin.jade、list.jade，略）

