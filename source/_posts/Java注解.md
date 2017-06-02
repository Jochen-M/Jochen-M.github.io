---
title: Java注解
date: 2017-05-07 21:57:48
tags:
    - Java
categories:
    - Java
---

## 概念
Java提供了一种原程序中的元素关联任何信息和任何元数据的途径和方法。

## 注解的分类
### 按照运行机制分
*源码注解：*注解只在源码中存在，编译成.class文件就不存在了。
*编译时注解：*注解在源码和.class文件中都存在。（ JDK自带注解）
*运行时注解：*在运行阶段还起作用，甚至会影响运行逻辑。（@Autowired）

<!-- more -->

### 按照来源分
#### JDK自带注解
@Override    覆盖父类的方法
@Deprecated    过时的方法
@SuppressWarnings    抑制警告。如：@SuppressWarnings("deprecation")
#### Java第三方注解
*Spring：*
@Autowired
@Service
@Repository
*Mybatis:*
@InsertProvider
@UpdateProvider
@Options
#### 自定义注解
### 元注解：注解的注解

## 自定义注解
### 自定义注解的语法要求
1) 使用@interface关键字定义注解；
2) 成员以无参无异常方式声明；
3) 可以用default为成员指定一个默认值；
4) 成员类型是受限的，合法的类型包括原始类型及String,Class,Annotation,Enumeration；
5) 如果注解只有一个成员，则成员名必须为value()，在使用时可以忽略成员名和赋值号(=)；
6) 注解类可以没有成员，没有成员的注解称为标识注解。
例如：
```
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Description{
    String desc();
    String author();
    int age() default 18;
}

```
或者：
```
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Description{
    String value();
}
```
*说明：*
@Target —— 注解的作用域：
+ CONSTRUCTOR 构造方法声明
+ FIELD 字段声明
+ LOCAL_VARIABLE 局部变量声明
+ METHOD 方法声明
+ PACKAGE 包声明
+ PARAMETER 参数声明
+ TYPE 类/接口

@Retention —— 生命周期：
+ SOURCE 只在源码显示，编译时会丢弃
+ CLASS 编译时会记录到class中，运行时忽略
+ RUNTIME 运行时存在，可以通过反射读取

@Inherited —— 允许子类继承
@Documented —— 生成javadoc时会包含该注解

### 使用自定义注解
语法说明：
@<注解名>(<成员名1> = <成员值1>， <成员名2> = <成员值2>,...)
例如：
```
@Description("I am class annotation")
public class Person{
    private String name;

    @Description("I am method annotation")
    public String getName(){
        return name;
    }
}

```

### 解析注解
通过反射获取类、函数或成员上的运行时注解信息，从而实现动态控制程序运行的逻辑。
```
import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

public class ParseAnn {
    public static void main(String[] args){
        // 1.使用类加载器加载类
        try{
            Class c = Class.forName("com.example.annotation.Person");
            // 2.找到类上的注解
            boolean isExist = c.isAnnotationPresent(Description.class);
            if(isExist){
                // 3.拿到注解实例
                Description d = (Description)c.getAnnotation(Description.class);
                System.out.println(d.value());
            }

            // 4.1 找到方法上的注解
            Method[] ms = c.getMethods();
            for(Method m : ms){
                boolean isMExist = m.isAnnotationPresent(Description.class);
                if(isMExist){
                    Description d = (Description)m.getAnnotation(Description.class);
                    System.out.println(d.value());
                }
            }
            // 4.2 另外一种解析方式
            for(Method m : ms){
                Annotation[] as = m.getAnnotations();
                for(Annotation a : as){
                    if(a instanceof Description){
                        Description d = (Description)a;
                        System.out.println(d.value());
                    }
                }
            }
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}

// 运行结果：
I am class annotation
I am method annotation
I am method annotation
```
