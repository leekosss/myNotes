[TOC]



## [java安全]反射

### 定义 

在讲反射之前，我们需要知道什么是反射

> 运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；
> 对于任意一个对象，都能够调用它的任意属性和方法；
> 这种动态获取信息以及动态调用对象方法的功能称为Java语言的反射机制。

简单的说：反射是指在运行时动态地获取一个类的信息，包括其成员变量、方法和构造函数等，以及在运行时对它们进行操作的能力。

使用反射，你可以在编译时期并不知道某个类的具体类型，而在运行时期动态地创建该类的对象、调用该类的方法，或访问和修改该类的属性。这种能力使得Java具有了很高的灵活性和扩展性。

反射就像是一个工具箱，它提供了一系列方法来检查和操作类的信息，包括：

- 获取一个类的相关信息，如名称、修饰符、继承关系等；
- 获取一个类的所有成员变量，并可以通过反射的方式修改它们的值；
- 获取一个类的所有方法，并可以通过反射的方式调用它们；
- 获取一个类的所有构造函数，并可以通过反射的方式创建它们的实例。
- （包括私有属性）



### 反射的运用

#### 1、反射获取类对象

获取类对象主要有以下三种形式：

- Class.forName()
- Object.class
- obj.getClass()

在我们获取了类对象后，我们就可以去使用类对象去获取类中的方法、成员变量、构造方法...了



##### 1.1、Class.forName()

Class.forName(parameter)方法是Class类中的一个静态方法，直接通过Class调用，其中的参数传入一个类的全限定名（包名+类名）

例如：

```java
public class Reflect1 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> runtime = Class.forName("java.lang.Runtime");
        System.out.println(runtime);
    }
}

输出：
class java.lang.Runtime
```

成功创建了一个Runtime的class类



##### 1.2、Object.class

通过类的class属性直接获取类对象：

```java
public class Reflect1 {
    public static void main(String[] args) {
        Class<Runtime> runtime = Runtime.class;
        System.out.println(runtime);
    }
}

输出：
class java.lang.Runtime
```

此处Runtime类处于 `java.lang`包下，所以不需要导包



##### 1.3、obj.getClass()

```java
public class Reflect1 {
    public static void main(String[] args)  {
        Runtime runtime = Runtime.getRuntime();
        Class<? extends Runtime> run = (Class<? extends Runtime>) runtime.getClass();
    }
}
```

首先我们先创建`runtime`对象，然后调用对象的 `getClass()` 方法获取类对象





#### 2、反射获取成员方法

- getMethods()
- getDeclaredMethods()
- getMethod(String functionName, Class<?>... parameterTypes)
- getDeclaredMethod(String functionName, Class<?>... parameterTypes)



##### 2.1、getMethods()

通过class类对象的`getMethods()`方法可以返回所有**public**公共成员方法(包括继承的)数组

例如：

```java
import java.lang.reflect.Method;
public class Reflect1 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> run = Class.forName("java.lang.Runtime");
        Method[] methods = run.getMethods();
        for (Method method : methods) {
            System.out.println(method);
        }
    }
}
```

![image-20230528163233262](https://s2.loli.net/2023/05/28/MtPJQvfwlLZkIGR.png)

其中也包含了非Runtime类的成员方法



##### 2.2、getDeclaredMethods()

通过class类对象的`getDeclaredMethods()`方法可以返回**所有权限**的成员方法(包括私有的、不包括继承的)数组

```java
import java.lang.reflect.Method;
public class Reflect1 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> run = Class.forName("java.lang.Runtime");
        Method[] methods = run.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method);
        }
    }
}
```

![image-20230528163626019](https://s2.loli.net/2023/05/28/zcYbpHNw2qWXglR.png)

如图，所有方法都是Runtime类的，无继承方法，并且可以获得除public权限外的方法



##### 2.3、getMethod()

返回单个public公共成员方法对象（包括继承）

`getMethod(String functionName, Class<?>... parameterTypes)`

参数1：代表想要获取的函数名称

参数2~x：代表对应函数声明类型的字节码对象

```java
public class Reflect1 {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException {
        Class<?> run = Class.forName("java.lang.Runtime");
        Method method = run.getMethod("equals", Object.class); 
		// 此处equals(Object name)的参数为string类型，所以需要传递Object.class字节码对象
    }
}
```



##### 2.4、getDeclaredMethod()

返回单个任意权限成员方法对象（包括私有，不包括继承）

`getDeclaredMethod(String functionName, Class<?>... parameterTypes)`

参数1：代表想要获取的函数名称

参数2~x：代表对应函数声明类型的字节码对象

```java
public class Reflect1 {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException {
        Class<?> run = Class.forName("java.lang.Runtime");
        Method method = run.getDeclaredMethod("exec", String.class);

    }
}
```



#### 3、反射获取构造方法

- getConstructors()
- getDeclaredConstructors()
- getConstructor()
- getDeclaredConstructor()

| 方法名                                                       | 说明                           |
| ------------------------------------------------------------ | ------------------------------ |
| Constructor<?>[] getConstructors()                           | 返回所有公共构造方法对象的数组 |
| Constructor<?>[] getDeclaredConstructors()                   | 返回所有构造方法对象的数组     |
| Constructor<T> getConstructor(Class<?>... parameterTypes)    | 返回单个公共构造方法对象       |
| Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) | 返回单个构造方法对象           |

后两个函数的参数都是构造方法的函数声明中形参的字节码文件，与上面类似



#### 4、反射创建对象

##### 4.1、通过Constructor创建对象

###### 方法介绍

- newInstance()  根据指定的构造方法创建对象
- setAccessible()  设置为true,表示取消访问检查

我们这里使用`getDeclaredConstructor()`先获取`Runtime`类的构造方法，然后使用Constructor对象的`newInstance(Object...initargs)`方法(括号中传入构造方法所需的值)创建对象

```java
public class Reflect1 {
    public static void main(String[] args) throws Exception {
        Class<?> run = Class.forName("java.lang.Runtime");
        Constructor<?> constructor = run.getDeclaredConstructor();
        Object runtime = constructor.newInstance();
        System.out.println(runtime);
    }
}
```

但是这里报错了：

![image-20230528170640904](https://s2.loli.net/2023/05/28/cNvo9DfUBeEHOxC.png)

我们进Runtime类看一下构造方法的权限：

![image-20230528170721782](https://s2.loli.net/2023/05/28/ruxUaWnKwL8d7FS.png)

我们发现Runtime类的构造方法是private修饰的，私有的

> 在反射中被private修饰的不能直接使用，需要使用`setAccessible(true)`取消访问检查，暴力反射

因此，我们在创建对象之前，先使用`setAccessible(true)`取消访问检查，然后创建对象

```java
public class Reflect1 {
    public static void main(String[] args) throws Exception {
        Class<?> run = Class.forName("java.lang.Runtime");
        Constructor<?> constructor = run.getDeclaredConstructor();
        constructor.setAccessible(true);
        Object runtime = constructor.newInstance();
        System.out.println(runtime);
    }
}
```

![image-20230528172256663](https://s2.loli.net/2023/05/28/ZKluSn9H26xMGws.png)

成功创建对象

##### 4.2、通过Class类对象创建对象

在Class类对象中可以直接调用`newInstance()` 方法调用无参构造方法创建一个空参对象

但是这里不能调用`setAccessible()`方法访问private修饰的构造方法

因此，如果我们使用这种方法创建Runtime对象就会失败：

![image-20230528173022759](https://s2.loli.net/2023/05/28/b4Mos6G7mRyFD9H.png)

但是我们可以使用Class类对象的`newInstance()`方法创建构造方法不为private的对象

```java
public class Reflect1 {
    public static void main(String[] args) throws Exception {
        Class<?> run = Class.forName("java.lang.String");
        Object runtime = run.newInstance();
    }
}
```



#### 5、反射调用成员方法

+ 方法介绍

  | 方法名                                    | 说明     |
  | ----------------------------------------- | -------- |
  | Object invoke(Object obj, Object... args) | 运行方法 |

  参数一: 用obj对象调用该方法

  参数二: 调用方法的传递的参数(如果没有就不写)

  返回值: 方法的返回值(如果没有就不写)

在我们获取到Method对象后，我们就可以使用Method对象的 `invoke()`方法去调用函数了



##### 通过Runtime对象的exec方法去调用计算器

首先我们创建Runtime的Class类，然后获取构造方法创建Runtime对象、获取exec方法，最后调用exec方法

```java
public class Reflect1 {
    public static void main(String[] args) throws Exception {
        Class<?> run = Class.forName("java.lang.Runtime");
        Constructor<?> constructor = run.getDeclaredConstructor();
        //注意需要取消访问检查，因为Runtime的构造方法是私有的
        constructor.setAccessible(true); 
        Object runtime = constructor.newInstance();
        Method exec = run.getMethod("exec", String.class);
        exec.invoke(runtime,"C:\\Windows\\System32\\calc.exe");
    }
}
```

成功弹出计算器，getshell

![image-20230528174206005](https://s2.loli.net/2023/05/28/EQmkzWDRCluniaK.png)



#### 6、反射获取成员变量并使用

方法分类

| 方法名                              | 说明                                 |
| ----------------------------------- | ------------------------------------ |
| Field[] getFields()                 | 返回所有public公共成员变量对象的数组 |
| Field[] getDeclaredFields()         | 返回所有成员变量对象的数组           |
| Field getField(String name)         | 返回单个公共成员变量对象             |
| Field getDeclaredField(String name) | 返回单个成员变量对象                 |

用法和上面类似，参数名就是变量的名字。`getField()`和`getFields()` 都可以获取继承的属性



##### 6.1、获取成员变量的值

- get()

```java
class Student extends Teacher {
    public String name = "stu";
}

public class Reflect1 {
    public static void main(String[] args) throws Exception {
        Class<?> stu = Class.forName("com.leekos.Student");
        Object newInstance = stu.newInstance();
        Field name = stu.getDeclaredField("name");
        System.out.println(name.get(newInstance));
    }
}

输出：
stu
```



##### 6.2、设置成员变量的值

- set()

```java
public class Reflect1 {
    public static void main(String[] args) throws Exception {
        Class<?> stu = Class.forName("com.leekos.Student");
        Object newInstance = stu.newInstance();
        Field name = stu.getDeclaredField("name");
        name.set(newInstance,"666");
        System.out.println(name.get(newInstance));
    }
}

输出：
666
```





### 总结

反射常用的方法

- 获取类的⽅法： Class.forName()
- 实例化类对象的⽅法： newInstance()
- 获取函数的⽅法： getMethod()
- 执⾏函数的⽅法： invoke()
- 限制突破方法：setAccessible()













