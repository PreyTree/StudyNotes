# 动态代理DynamicProxy

## 定义和基本概念

**动态代理**是一种在运行时动态创建代理对象的技术。**它通过代理对象来间接访问目标对象，并在访问前后执行额外的操作。**动态代理可以在不修改目标对象代码的情况下，为对象添加额外的功能。

## 实现方式

### **动态代理主要有两种实现方式：**

> - **基于接口的动态代理‌：**通过‌Java的反射机制，使用Proxy类和InvocationHandler接口实现。
> - **基于类的动态代理**‌：使用CGLIB库，通过继承目标类并重写方法实现。

## 动态代理Demo

### User.java

```java
package com.pjy;

/**
 * User接口类
 */
public interface User {
    void delete();

    String save(String name);
}

```

### MyUser.java

```java
package com.pjy;

/**
 * User实现类
 */

public class MyUser implements User{

    private String name;

    public MyUser() {
    }

    public MyUser(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void delete() {
        System.out.println("删除成功");

    }

    @Override
    public String save(String name) {
        System.out.println("保存成功 name:" + name);
        return "200";
    }
}

```

### ProxyUtil.java

```java
package com.pjy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 动态代理工具类
 * 类的作用：
 *      创造一个代理
 */

public class ProxyUtil {

    /**
     *
     * 方法的作用：
     *      给一个User对象，创造一个代理
     *  形参：
     *      杯代理的User对象
     *   返回值：
     *      给User创建的代理
     *
     */

    public static User createProxy(MyUser myUser){
        /**
         * java.lang.reflect.Proxy类：提供了为对象产生代理对象的方法：
         * public static Object newProxyInstance(Class<?>[] interfaces, InvocationHandler h)
         * 参数一：用于指定用哪个类加载器，去加载生成的代理类
         * 参数二：指定接口，这些接口用于指定生成的代理有哪些方法
         * 参数三：用来指定生成的代理对象要干什么事情
         */

        User user = (User)Proxy.newProxyInstance(ProxyUtil.class.getClassLoader(),
                new Class[]{User.class},
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        /**
                         * 参数一：要代理的对象
                         * 参数二：就要运行的方法
                         * 参数三：调用运行方法时，传递的实参
                         */
                        if ("delete".equals(method.getName())){
                            System.out.println("准备删除");
                        } else if ("save".equals(method.getName())){
                            System.out.println("准备保存");
                        }


                        return method.invoke(myUser,args);//根据传入的myUser对象和实参args,调用method方法
                    }
                });

        return user;
    }
}

```

###


### UserTest.java

```java
package com.pjy;

/**
 *动态代理测试类
 */

public class UserTest {
    public static void main(String[] args) {

        //1.获取代理对象
        MyUser myUser = new MyUser("admin");
        User proxy = ProxyUtil.createProxy(myUser);

        //2.调用save方法
        //代理对象调用方法后自动调用底层的InvocationHandler接口的invoke方法
        //最后通过return Method类的invoke(Object  obj, Object ... args)实现代理的save方法
        String result = proxy.save("message");

        System.out.println(result);

        //2.调用delete方法
        proxy.delete();
    }
}

```







