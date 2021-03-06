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
#####　在detail.jade中添加评论区和回复区：
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

#####　添加路由：
```
app.post('/user/comment', User.signinRequired, Comment.save);
```

#####　评论存储，新建models/comment.js：
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

#####　为头像增加点击事件：
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

#####　展示评论和回复
对controllers/movie.js稍作修改：
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
#### 分类的数据模型
schemas/category.js:
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
##### 增加路由信息
routes.js：
```
let Category = require('../app/controllers/category');

// Category
app.post('/admin/category/save', User.signinRequired, User.adminRequired, Category.save);
app.get('/admin/category/new', User.signinRequired, User.adminRequired, Category.new);
app.get('/admin/category/list', User.signinRequired, User.adminRequired, Category.list);
```

##### 分类的控制器
controllers/category.js:
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
  let _category = req.body.category;
  let category = new Caterory(_category);

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

##### 增加相关页面
admin_category.jade、categorylist.jade。（类似于admin.jade、list.jade，略）

#### 电影录入增加分类选择
##### 为电影增加字段
修改schemas/movie.js
```
let Schema = mongoose.Schema;
let ObjectId = Schema.Types.ObjectId;

category: {
    type: ObjectId,
    ref: 'Category'
  }
```

##### 增加“电影分类”表单项
修改admin.jade：
```
.form-group
  label.col-sm-2.control-label(for="inputCategory") 电影分类
    .col-sm-8
      input#inputCategory.form-control(type="text", name="movie[category_new]")
.form-group
  label.col-sm-2.control-label(for="radioCategory")
  each cat in categories
    label.radio-inline
      if movie._id
        input#radioCategory(type="radio", name="movie[category]", value=cat._id, checked=cat._id.toString()==movie.category.toString())
        | #{cat.name}
      else
        input#radioCategory(type="radio", name="movie[category]", value=cat._id)
        | #{cat.name}
```

##### 修改movie控制器
controllers/movie.js：
```
exports.new = function(req, res){
  Category.find({}, function(err, categories){
    res.render('admin', {
      title: '后台录入页',
      categories: categories,
      movie: {}
    })
  });
};

exports.save = function(req, res){
  let id = req.body.movie._id;
  let movieObj = req.body.movie;
  let _movie;

  if(id){
    //...
  }else{
    let categoryId = movieObj.category;
    _movie = new Movie(movieObj);
    _movie.save(function(err, movie){
      if(err){
        console.log(err);
      }
      Category.findById(categoryId, function(err, category){
        category.movies.push(movie._id);
        category.save(function(err, category){
          res.redirect('/admin/movie/list');
        });
      })
    });
  }
};

exports.update = function(req, res){
  let id = req.params.id;
  if(id){
    Movie.findById(id, function(err, movie){
      Category.find({}, function(err, categories){
        res.render('admin', {
          title: '后台更新页',
          movie: movie,
          categories: categories
        });
      });
    });
  }
};
```

#### 分类效果展示
##### 首页控制器
index.js：
```
exports.index = function(req, res){
  Category
    .find({})
    .populate({path: 'movies', options: {limit: 5}})
    .exec(function(err, categories){
        if(err){
          console.log(err);
        }
        res.render('index', {
          title: '首页',
          categories: categories
        });
    })
};
```

##### 首页视图
index.jade
```
for cat in categories
  .panel.panel-default
    .panel-heading
      h3 #{cat.name}
    .panel-body
      if cat.movies && cat.movies.length > 0
        for item in cat.movies
        //...
```

#### jsonp同步豆瓣数据
admin.jade 增加"豆瓣同步"表单项, 并引入admin.js:
```
.form-group
  label.col-sm-2.control-label(for="inputTitle") 豆瓣同步
    .col-sm-8
      input#douban.form-control(type="text")

script(src="/js/admin.js")
```

编辑admin.js, 通过ajax加载豆瓣数据：
```
$(function(){
  //...
  $('#douban').blur(function(){
    let id = $(this).val();   // 电影id
    $.ajax({
      url: 'https://api.douban.com/v2/movie/subject/' + id,
      cache: true,
      type: 'get',
      dataType: 'jsonp',
      crossDomain: true,
      jspnp: 'callback',
      success: function(data){
        $('#inputTitle').val(data.title);
        $('#inputDirector').val(data.directors[0].name);
        $('#inputCountry').val(data.countries[0]);
        $('#inputPoster').val(data.images.large);
        $('#inputShowAt').val(data.year);
        $('#inputSummary').val(data.summary);
      }
    })
  });
}
```

在豆瓣电影中查找电影id，如：
```
https://movie.douban.com/subject/26363254/?from=showing   // id=26363254
```
输入表单后，效果如图：
![Alt Text](/uploads/douban-example.png)

> 豆瓣开发者文档： https://developers.douban.com/wiki/?title=guide

#### 电影录入增加自定义分类
修改controllers/movie.js:
```
exports.save = function(req, res){
  if(id){
    //...
  }else{
    let categoryId = movieObj.category;
    let categoryName = movieObj.category_new;
    _movie = new Movie(movieObj);
    _movie.save(function(err, movie){
      if(err){
        console.log(err);
      }
      if(categoryId){ // 已有分类
        Category.findById(categoryId, function(err, category){
          category.movies.push(movie._id);
          category.save(function(err, category){
            res.redirect('/admin/movie/list');
          });
        })
      }else if(categoryName){ // 新分类
        let category = new Category({
          name: categoryName,
          movies: [movie._id]
        });
        category.save(function(err, category){
          _movie.category = category._id;
          _movie.save(function(err, movie){ // 注意：movie要保存两次
            res.redirect('/admin/movie/list');
          });
        });
      }
    });
  }
};
```

#### 增加分类列表及分页
修改index.jade,为分类标题增加链接：
```
h3
  a(href="/category/search?catId=#{cat._id}&page=1") #{cat.name}
```

增加路由：
```
app.get('/category/search', Category.search);
```

增加后台逻辑处理：
```
exports.search = function(req, res){
	let catId = req.query.catId;
	let page = parseInt(req.query.page, 10) || 0;
	let perPage = 5;
	let index = (page - 1) * perPage;

	Category
		.find({_id: catId})
		.populate({
			path: 'movies',
			select: 'title poster'
		})
		.exec(function(err, categories){
			if(err){
				console.log(err);
			}
			let category = categories[0] || {};
			let movies = category.movies || [];
			let results = movies.slice(index, index + perPage);

			res.render('results', {
				title: '分类',
				query: 'catId=' + catId,
				keyword: category.name,
				currentPage: page,
				totalPage: Math.ceil(movies.length / perPage),
				movies: results
			});
		});
};
```

展示页面results.jade：
```
extends ../layout

block content
	.container
		.row
			.panel.panel-default
				.panel-heading
					h3
						if query
							a(href="/category/search?#{query}&page=1") #{keyword}
						else
							a(href="/movie/search?keyword=#{keyword}&page=1") #{keyword}
				.panel-body
					if movies && movies.length > 0
						for item in movies
							.col-md-2
								.thumbnail
									a(href="/movie/#{item._id}")
										img(src="#{item.poster}", alt="#{item.title}")
									.caption
										h3 #{item.title}
										p: a.btn.btn-primary(href="/movie/#{item._id}", role="button") 观看预告片
			ul.pagination.col-md-6.col-md-offset-3
				- for(let i = 0; i < totalPage; i++){
					- if(currentPage == (i + 1)){
						li.active
							span #{currentPage}
					- }else{
						li
							if query
								a(href='/category/search?#{query}&page=#{i + 1}') #{i + 1}
							else
								a(href='/movie/search?keyword=#{keyword}&page=#{i + 1}') #{i + 1}
					- }
				- }
```

#### 搜索功能的实现
增加搜索框head.jade:
```
form.navbar-form.navbar-left(method="get", action="/movie/search")
  .form-group
    input.form-control(type="text", placeholder="Search", name="keyword")
    button.btn.btn-default(type="submit") Submit
```

增加路由：
```
// 应放在 app.get('/movie/:id', Movie.detail) 之前，否则会有冲突
app.get('/movie/search', Movie.search);   
```

增加后台逻辑处理：
```
exports.search = function(req, res){
	let keyword = req.query.keyword;
	let page = parseInt(req.query.page) || 1;
	let perPage = 5;
	Movie
		.find({title: new RegExp('.*' + keyword + '.*')}, function(err, movies){
			if(err){
				console.log(err);
			}
			res.render('results', {
				title: '分类',
				keyword: keyword,
				currentPage: page,
				totalPage: Math.ceil(movies.length / perPage),
				movies: movies
			})
		})
};
```

#### 上传海报
增加文件选择表单项
admin.jade:
```
form.form-horizontal(method="post", action="/admin/movie/save", enctype="multipart/form-data")
  .form-group
    label.col-sm-2.control-label(for="uploadPoster") 上传海报
    .col-sm-8
      input#uploadPoster(type="file", name="uploadPoster")
```

为系统增加中间件：(cnpm install connect-multiparty --save)
app.js:
```
let multipart = require('connect-multiparty');
app.use(multipart());
```

为保存电影增加中间件posterUploaded，以保证图片上传成功（这里最好采用异步方式）:
```
app.post('/admin/movie/save', User.signinRequired, User.adminRequired, Movie.posterUploaded, Movie.save);
```

具体实现，controllers/movie.js:
```
exports.posterUploaded = function(req, res, next){
	let posterData = req.files.uploadPoster;
	let filePath = posterData.path;
	let originalFilename = posterData.originalFilename;

	if(originalFilename){
		fs.readFile(filePath, function(err, data){
			let timestamp = Date.now();
			let type = posterData.type.split('/')[1];
			let newPosterName = timestamp + '.' + type;
			let newPath = path.join(__dirname, '../../public/images/' + newPosterName);

			fs.writeFile(newPath, data, function(err){
				req.newPosterName = '/images/' + newPosterName;
				next();
			})
		})
	}else{
		next();
	}
}
```

在保存电影之前，判断是否上传了图片，并替换之：
```
exports.save = function(req, res){
	let movieObj = req.body.movie;

	if(req.newPosterName){
		movieObj.poster = req.newPosterName;
	}
  //...
}
```

#### 访客统计
为movie增加字段:
```
pv: {
  type: Number,
  default: 0
},
```

每当访问详情页时，访问量 +1 ：
```
exports.detail = function(req, res){
	let id = req.params.id;
	Movie.update({_id: id}, {$inc: {pv: 1}}, function(err){
		if(err){
			console.log(err);
		}
	})
	// ...
};
```

展示：在列表页增加“访客统计”字段，即可。


> [《Nodejs + Express + MongoDB快速搭建网站》](https://jochen-m.github.io/2017/05/26/Nodejs-Express-MongoDB快速搭建网站/)
