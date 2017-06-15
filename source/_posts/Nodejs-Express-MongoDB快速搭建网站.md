---
title: Nodejs + Express + MongoDB快速搭建网站
date: 2017-05-26 22:17:56
tags:
    - Nodejs
    - Express
    - MongoDB
categories:
    - Nodejs
---

### 基本架构（前期准备）
#### 后台：
- Node.js
- Express框架(通过npm安装)
- Moment.js——处理时间格式(通过npm安装)
- mongoose——mongoDB数据库插件(通过npm安装)
- underscore——mongoDB相关插件，可实现数据更新(通过npm安装)
- cookie-parser——解析cookie(通过npm安装)
- express-session——处理session(通过npm安装)
- connect-mongo——用mongodb做会话的持久化(通过npm安装)
- bcrypt——密码加密处理模块(通过npm安装)

#### 数据库：
- mongoDB

#### 前端：
- jade模板引擎(通过npm安装)
- Bower(通过npm安装)
- jQuery(通过Bower安装)
- Bootstrap(通过Bower安装)

* * * 

### 项目流程（编码实现）：
#### 项目前后端流程打通

- Node入口文件分析
```
let port = 3000;
let express = require('express');
let app = express();
let bodyParser = require('body-parser');  // 将表单数据格式化
let path = require('path');               // 路径模块

app.use(bodyParser.urlencoded({ extended: true}));
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));  // 静态资源路径
app.set('view engine', 'jade');           // 视图模板引擎为jade
app.set('views', './views/pages')         // 视图模板引擎路径
app.locals.moment = require('moment');    // 时间格式化模块

// index page
app.get('/', function(req, res){
  Movie.fetch(function(err, movies){
    if(err){
      console.log(err);
    }

    res.render('index', {   // index.jade
      title: 'MovieGoGo 首页',
      movies: movies
    });
  })
});

app.listen(port);
console.log('MovieGoGo is running at http://localhost:' + port);
```
  
- 目录初始化：

```
MovieGoGo/
├── node_modules/         // nodejs依赖库
├── public/               // 静态资源
│   ├── images/           // 图片
│   ├── js/               // 自定义js文件
│   └── libs/
│       ├── bootstrap/
│       └── jquery/
├── schemas               // 模式
│   └── movie.js
├── models                // 模型
│   └── movie.js
├── views                 // 视图
│   ├── includes
│   │   ├── header.jade
│   │   └── head.jade
│   ├── layout.jade
│   └── pages
│       ├── admin.jade
│       ├── detail.jade
│       ├── index.jade
│       └── list.jade
├── app.js                // 入口文件
├── bower.json            // bower配置文件
├── package.json          // nodejs配置文件
└── README.md
```


- 创建jade视图及入口文件中处理

head.jade:
```
.container
  .row
    .page-header
      h1 #{title}
```

header.jade
```
link(href="/libs/bootstrap/dist/css/bootstrap.min.css", rel="stylesheet")
script(src="/libs/jquery/dist/jquery.min.js")
script(src="/libs/bootstrap/dist/js/bootstrap.min.js")
```

layout.jade:
```
doctype
html
  head
    meta(charset='utf-8')
    title #{title}
    include ./includes/head
  body
    include ./includes/header
    block content
```

index.jade：
```
extends ../layout
block content
  .container
    .row
      for item in movies
        .col-md-2
          .thumbnail
            a(href="/movie/#{item._id}")
              img(src="#{item.poster}", alt="#{item.title}")
            .caption
              h3 #{item.title}
              p: a.btn.btn-primary(href="/movie/#{item._id}", role="button")
                观看预告片
```

admin.jade：
```
extends ../layout
block content
  .container
    .row
      form.form-horizontal(method="post", action="/admin/movie/new")
        input(type="hidden", name="movie[_id]", value="#{movie._id}")
        .form-group
          label.col-sm-2.control-label(for="inputTitle") 电影名字
          .col-sm-8
            input#inputTitle.form-control(type="text", name="movie[title]", value="#{movie.title}")
        .form-group
          label.col-sm-2.control-label(for="inputDirector") 导演
          .col-sm-8
            input#inputDirector.form-control(type="text", name="movie[director]", value="#{movie.director}")
        .form-group
          label.col-sm-2.control-label(for="inputCountry") 国家
          .col-sm-8
            input#inputCountry.form-control(type="text", name="movie[country]", value="#{movie.country}")
        .form-group
          label.col-sm-2.control-label(for="inputLanguage") 语言
          .col-sm-8
            input#inputLanguage.form-control(type="text", name="movie[language]", value="#{movie.language}")
        .form-group
          label.col-sm-2.control-label(for="inputPoster") 海报
          .col-sm-8
            input#inputPoster.form-control(type="text", name="movie[poster]", value="#{movie.poster}")
        .form-group
          label.col-sm-2.control-label(for="inputFlash") 预告片
          .col-sm-8
            input#inputFlash.form-control(type="text", name="movie[flash]", value="#{movie.flash}")
        .form-group
          label.col-sm-2.control-label(for="inputShowAt") 上映时间
          .col-sm-8
            input#inputShowAt.form-control(type="text", name="movie[showAt]", value="#{movie.showAt}")
        .form-group
          label.col-sm-2.control-label(for="inputSummary") 简介
          .col-sm-8
            input#inputSummary.form-control(type="text", name="movie[summary]", value="#{movie.summary}")
        .form-group
          .col-sm-6
          button.btn.btn-default(type="submit") 录入
```

detail.jade：
```
extends ../layout
block content
  .container
    .row
      .col-md-7
        img(src="#{movie.poster}", alt="#{movie.title}")
        //- embed(src="#{movie.flash}", allowFullScreen="true", quality="high", width="720", height="600", align="middle", type="application/x-shockwave-flash")
      .col-md-5
        dl.dl-horizontal
          dt 电影名字
          dd #{movie.title}
          dt 导演
          dd #{movie.director}
          dt 国家
          dd #{movie.country}
          dt 语言
          dd #{movie.language}
          dt 上映时间
          dd #{movie.showAt}
          dt 简介
          dd #{movie.summary}
```

list.jade：
```
extends ../layout
block content
  .container
    .row
      table.table.table-hover.table-bordered
        thead
          tr
            th 电影名字
            th 导演
            th 国家
            th 语言
            th 上映时间
            th 录入时间
            th 简介
            th 查看
            th 更新
            th 删除
        tbody
          each item in movies
            tr(class="item-id-#{item._id}")
              td #{item.title}
              td #{item.director}
              td #{item.country}
              td #{item.language}
              td #{item.showAt}
              td #{moment(item.meta.createAt).format('MM/DD/YYYY HH:mm:SS')}
              td.col-sm-4 #{item.summary}
              td: a.btn.btn-default(target="_blank", href="/movie/#{item._id}") 查看
              td: a.btn.btn-success(target="_blank", href="/admin/update/#{item._id}") 更新
              td
                button.btn.btn-danger.del(type="button", data-id="#{item._id}") 删除
  script(src="/js/admin.js")
```

- 伪造模板数据跑通前后端交互流程(略)

#### 项目数据库实现

- mongodb模式模型设计及编码

schemas/movie.js:
```
let mongoose = require('mongoose');

let MovieSchema = new mongoose.Schema({
  title: String,
  director: String,
  country: String,
  language: String,
  poster: String,
  flash: String,
  summary: String,
  showAt: Number,
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

MovieSchema.pre('save', function(next){
  if(this.isNew){
    this.meta.createAt = this.meta.updateAt = Date.now();
  }else{
    this.meta.updateAt = Date.now();
  }
  next();
});

MovieSchema.statics = {
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

module.exports = MovieSchema
```

models/movie.js:
```
let mongoose = require('mongoose');
let MovieSchema = require('../schemas/movie');
let Movie = mongoose.model('Movie', MovieSchema);

module.exports = Movie
```

- 编写数据库交互代码

app.js文件中添加代码：
```
let dbUrl = 'mongodb://127.0.0.1:27017/moviegogo';

let mongoose = require('mongoose');
let _ = require('underscore');
let Movie = require('./models/movie');

mongoose.Promise = global.Promise;
mongoose.connect(dbUrl);

// detail page
app.get('/movie/:id', function(req, res){
  let id = req.params.id;

  Movie.findById(id, function(err, movie){
    if(err){
      console.log(err);
    }

    res.render('detail', {
      title: 'MovieGoGo ' + movie.title,
      movie: movie
    });
  })
});

// admin new movie form page
app.get('/admin/movie', function(req, res){
  res.render('admin', {
    title: 'MovieGoGo 后台录入页',
    movie: {
      title: "",
      director: "",
      country: "",
      language: "",
      poster: "",
      flash: "",
      showAt: "",
      summary: ""
    }
  });
});

// admin new movie
app.post('/admin/movie/new', function(req, res){
  let id = req.body.movie._id;
  let movieObj = req.body.movie;
  let _movie;

  if(id != 'undefined'){
    Movie.findById(id, function(err, movie){
      if(err){
        console.log(err);
      }

      _movie = _.extend(movie, movieObj);

      _movie.save(function(err){
        if(err){
          console.log(err);
        }

        res.redirect('/admin/list');
      });
    });
  }else{
    _movie = new Movie({
      title: movieObj.title,
      director: movieObj.director,
      country: movieObj.country,
      language: movieObj.language,
      poster: '/images/sjbbb.jpg',  //movieObj.poster,
      flash: movieObj.flash,
      summary: movieObj.summary,
      showAt: movieObj.showAt
    });

    _movie.save(function(err){
      if(err){
        console.log(err);
      }

      res.redirect('/admin/list');
    });
  }
});

// admin update movie
app.get('/admin/update/:id', function(req, res){
  let id = req.params.id;

  if(id){
    Movie.findById(id, function(err, movie){
      res.render('admin', {
        title: 'imooc 后台更新页',
        movie: movie
      });
    });
  }
});

// list page
app.get('/admin/list', function(req, res){
  Movie.fetch(function(err, movies){
    if(err){
      console.log(err);
    }

    res.render('list', {
      title: 'MovieGoGo 列表页',
      movies: movies
    });
  })
});
```

#### 删除功能及项目生成配置文件
- 删除功能

app.js文件中添加代码:
```
// admin delete page
app.delete('/admin/delete', function(req, res){
  let id = req.query.id;

  if(id){
    Movie.remove({_id: id}, function(err){
      if(err){
        console.log(err);
      }else{
        res.json({success: 1});
      }
    });
  }
});
```

新建public/js/admin.js:
```
$(function(){
  $('.del').click(function(e){
    let target = $(e.target);
    let id = target.data('id');
    let tr = $('.item-id-' + id);

    $.ajax({
      type: 'DELETE',
      url: '/admin/delete?id=' + id
    }).done(function(results){
      if(results.success === 1){
        if(tr.length > 0){
          tr.remove();
        }
      }
    });
  });
});
```

- 生成项目配置文件
使用<code>bower init</code>和<code>cnpm init</code>生成配置文件。

#### 开发用户的注册登录功能

- 用户模型及密码处理

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

models/user.js:
```
let mongoose = require('mongoose');
let UserSchema = require('../schemas/user');
let User = mongoose.model('User', UserSchema);

module.exports = User
```

- 登录注册前端视图

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

- 注册用户后台存储

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

- 实现登录逻辑

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

- 利用mongodb做会话的持久化

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

测试：在 app.js 中 app.get('/',function(req,res){})加入测试代码：

```
console.log("user in session: ");
console.log(req.session.user);
```

- 注销用户、用户退出功能实现

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

app.js文件：
```
// ---------- 修改 ----------
// index page
app.get('/', function(req, res){
  // console.log("user in session: ");
  // console.log(req.session.user);

  let _user = req.session.user;
  if(_user){
    app.locals.user = _user;
  }

  Movie.fetch(function(err, movies){
    if(err){
      console.log(err);
    }

    res.render('index', {
      title: 'MovieGoGo 首页',
      movies: movies
    });
  })
});

// ---------- 添加 ----------
// logout
app.get('/logout', function(req, res){
  delete req.session.user;
  delete app.locals.user;
  res.redirect('/');
});
```

* * *

### 注意事项及技巧
1）使用bower安装依赖的库和文件时，应事先在项目根目录建立.bowerrc文件，并在文件中添加配置信息，表示库和文件的安装目录：
  ```
  {
    "directory": "public/libs"
  }
  ```
2）安装相关模块时，使用<code>--save</code>参数，自动将配置信息写入配置文件（如：cnpm install cookie-parser --save)

### 项目具体源代码：
Github: [https://github.com/Jochen-M/MovieGoGo.git](https://github.com/Jochen-M/MovieGoGo.git)