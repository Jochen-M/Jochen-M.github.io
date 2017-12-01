---
title: mongoose简单操作实例
date: 2017-06-15 10:24:46
tags:
    - Nodejs
    - MongoDB
    - mongoose
categories:
    - Nodejs
    - MongoDB
---

Mongoose API: [http://mongoosejs.com/docs/api.html](http://mongoosejs.com/docs/api.html)

### mongoose 链接
```
let mongoose = require('mongoose');
let db       = mongoose.createConnection('mongodb://127.0.0.1:27017/dbname');

// 链接错误
db.on('error', function(error) {
    console.log(error);
});
```

### Schema 结构
```
let MongooseSchema = new mongoose.Schema({
    username : {type : String, default : '匿名用户'},
    title    : {type : String},
    content  : {type : String},
    time     : {type : Date, default: Date.now},
    age      : {type : Number}
});
```

### model
```
let MongooseModel = db.model('mongoose', MongooseSchema);
```

### 添加方法
#### 添加 mongoose 实例方法
```
MongooseSchema.methods.findByUsername = function(username, callback) {
    return this.model('mongoose').find({username: username}, callback);
}
```
#### 添加 mongoose 静态方法，静态方法在Model层就能使用
```
MongooseSchema.statics.findByTitle = function(title, callback) {
    return this.model('mongoose').find({title: title}, callback);
}
```

### 增加记录
#### 基于 entity 操作
```
let doc = {username : 'emtity_demo_username', title : 'emtity_demo_title', content : 'emtity_demo_content'};
let mongooseEntity = new MongooseModel(doc);
mongooseEntity.save(function(error) {
    if(error) {
        console.log(error);
    } else {
        console.log('saved OK!');
    }
    // 关闭数据库链接
    db.close();
});
```
#### 增加记录 基于model操作
```
let doc = {username : 'model_demo_username', title : 'model_demo_title', content : 'model_demo_content'};
MongooseModel.create(doc, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('save ok');
    }
    // 关闭数据库链接
    db.close();
});
```

### 修改记录
```
MongooseModel.update(conditions, update, options, callback);
let conditions = {username : 'model_demo_username'};
let update     = {$set : {age : 27, title : 'model_demo_title_update'}};
let options    = {upsert : true};
MongooseModel.update(conditions, update, options, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('update ok!');
    }
    //关闭数据库链接
    db.close();
});
```

### 查询
#### 基于实例方法的查询
```
let mongooseEntity = new MongooseModel({});
mongooseEntity.findByUsername('model_demo_username', function(error, result){
    if(error) {
        console.log(error);
    } else {
        console.log(result);
    }
    //关闭数据库链接
    db.close();
});
```
#### 基于静态方法的查询
```
MongooseModel.findByTitle('emtity_demo_title', function(error, result){
    if(error) {
        console.log(error);
    } else {
        console.log(result);
    }
    //关闭数据库链接
    db.close();
});
```
#### mongoose model find
```
let conditions = {title : 'emtity_demo_title'};      // 查询条件
let fields     = {title : 1, content : 1, time : 1}; // 待返回的字段
let options    = {};
MongooseModel.find(conditions, fields, options, function(error, result){
    if(error) {
        console.log(error);
    } else {
        console.log(result);
    }
    //关闭数据库链接
    db.close();
});
```

### 删除记录
```
let conditions = {username: 'emtity_demo_username'};
MongooseModel.remove(conditions, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('delete ok!');
    }
    //关闭数据库链接
    db.close();
});
```
