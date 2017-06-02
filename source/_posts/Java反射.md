---
title: Java反射
date: 2017-05-04 08:45:50
tags:
    - Java
categories: 
    - Java
---

## Class类的使用
在面向对象的世界里，万事万物皆对象。*(java中，静态的成员，普通数据类型不是对象。)*
类也是对象，是java.lang.Class类的对象。这个对象的三种表示方法：

<!-- more -->

```
// 第一种，任何一个类都有一个隐含的静态成员变量class
Class c1 = Foo.class;
// 第二种，已知该类的对象
Class c2 = foo1.getClass();
// 第三种   
Class c3 = null;
c3 = Class.forName("com.example.reflect.Foo");
```

*说明：*
1) c1, c2, c3 表示了Foo类的类类型(class type)。
2) 一个类只可能是Class类的一个实例对象，即c1 == c2 == c3。
3) 可以通过类的类类型创建该类的对象。eg:Foo foo = (Foo)c1.newInstance();
  前提是：需要有无参数的构造方法。
4) 基本的数据类型，void关键字，都存在类类型
 Class c1 = int.class;       // int的类类型
 Class c2 = String.class;    // String的类类型
 Class c3 = double.class;    // double的类类型
 Class c4 = Double.class;    // Double的类类型，*与double.class不同*
 Class c5 = vold.class;      // void的类类型

## Class类动态加载类
编译时刻加载类是静态加载类，运行时刻加载类是动态加载类。
new创建对象是静态加载类，在编译时刻就需要加载所有可能使用到的类。

动态加载类的例子：
```
Office.java:
class Office{
    public static void main(String[] args){
        try{
            // 动态加载类，在运行时刻加载
            Class c = Class.forName(args[0]);
            // 通过类类型，创建该类对象
            OfficeAble oa = (OfficeAble) c.newInstance();
            oa.start();
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}

OfficeAble.java:
interface OfficeAble{
    public void start();
}

Word.java:
class Word implements OfficeAble{
    public void start(){
        System.out.println("Word start.");
    }
}
```

## 获取类的信息

*步骤：*
1) 获取对象的类类型
2) 调用API

程序示例：
```
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ClassUtil {

    public static void main(String[] args){
        ClassUtil cu = new ClassUtil();
        String s = "Hello world";
        cu.printClassMethodMessage(s);
        cu.printClassFieldMessage(s);
        cu.printClassConstructorMessage(s);
    }

    /**
     * 打印类的方法信息
     * @param obj
     */
    public static void printClassMethodMessage(Object obj){
        // 获取对象的类类型
        Class c = obj.getClass();

        /**
         * Method类，方法对象
         * 一个成员方法就是一个Method对象
         * getMethods() 获取所有public方法，包括从父类继承来的方法
         * getDeclaredMethods() 获取自己声明的全部方法，不论访问权限
         */
        Method[] ms = c.getMethods();

        System.out.println("类的名称是：" + c.getName());
        System.out.println("类的方法：");
        for(Method method : ms){
            // 返回值的类类型
            Class returnType = method.getReturnType();
            System.out.print(returnType.getName() + " ");

            // 方法名
            System.out.print(method.getName() + "(");

            // 参数列表
            Class[] parameterTypes = method.getParameterTypes();
            int parameterLength = parameterTypes.length;
            if(parameterLength > 0) {
                if(parameterLength > 1) {
                    for (int i = 0; i < parameterLength - 1; i++) {
                        System.out.print(parameterTypes[i].getSimpleName() + ", ");
                    }
                }
                System.out.print(parameterTypes[parameterLength - 1].getSimpleName());
            }

            System.out.println(")");
        }
    }

    /**
     * 打印类的成员变量信息
     * @param obj
     */
    public static void printClassFieldMessage(Object obj){
        // 获取对象的类类型
        Class c = obj.getClass();

        /**
         * Field类，成员变量对象
         * 一个成员方法就是一个Method对象
         * getFields() 获取所有public成员变量
         * getDeclaredFields() 获取自己声明的全部成员变量
         */
        Field[] fs = c.getDeclaredFields();

        System.out.println("类的成员变量：");
        for(Field field : fs){
            // 得到成员变量的类的类类型
            Class fieldType = field.getType();
            String typeName = fieldType.getName();

            // 得到成员变量的名称
            String fieldName = field.getName();

            System.out.println(fieldType + " " + fieldName);
        }
    }

    /**
     * 打印类的构造函数信息
     * @param obj
     */
    public static void printClassConstructorMessage(Object obj){
        // 获取对象的类类型
        Class c = obj.getClass();

        /**
         * Constructor类，构造函数对象
         * getContructors() 获取所有public构造函数
         * getDeclaredConstructors() 得到所有的构造函数
         */
        Constructor[] cs = c.getDeclaredConstructors();

        System.out.println("类的构造函数：");
        for(Constructor constructor : cs){
            System.out.print(constructor.getName() + "(");

            // 获取参数列表
            Class[] parameterTypes = constructor.getParameterTypes();
            int parameterLength = parameterTypes.length;
            if(parameterLength > 0) {
                if(parameterLength > 1) {
                    for (int i = 0; i < parameterLength - 1; i++) {
                        System.out.print(parameterTypes[i] + ",");
                    }
                }
                System.out.print(parameterTypes[parameterLength - 1]);
            }

            System.out.println(")");
        }
    }
}
```

## 方法反射的基本操作
1) 如何获取某个方法： 方法的名称和方法的参数列表才能唯一确定某个方法
2) 方法反射的操作： method.invoke(obj, paras...)

```
class Foo{
    public void print(int a, int b){
        System.out.println(a + b);
    }
}

class MethodReflect{
    public static void main(String[] args){
        // 1.获取类类型
        Foo foo = new Foo();
        Class c = foo.getClass();

        /**
         * 2.获取方法，名称和参数列表来决定
         *   getMethod() 获取public方法
         *   getDeclaredMethod() 获取自己声明的方法
         */
        // Method m = c.getMethod("print", new Class[]{int.class, int.class});
        Method m = c.getMethod("print", int.class, int.class);

        /**
         * 3.方法的反射操作
         * 等同于 foo.print(10, 20);
         * 方法没有返回值则返回null,否则返回具体的返回值
         */
        // Object obj = m.invoke(foo, new Class[]{10, 20});    
        Object obj = m.invoke(foo, 10, 20);
    }
}
```

## 集合中泛型的本质
Java中集合的泛型是防止错误输入的，只在编译阶段有效，绕过编译就无效了。*编译之后集合去泛型化*

验证：
```
ArrayList<String> list = new ArrayList<String>();
list.add("hello");
Class c = list.getClass();
Method m = c.getMethod("add", Object.class);
m.invoke(list, 20);    //没有错误
```
