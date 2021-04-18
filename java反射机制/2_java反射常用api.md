# 获取反射中的class对象

在反射中要获取一个类或调用一个类的方法，我们首先要获取到该类的class对象

在java API中，获取Class类对象有三种方法：

#### 第一种，使用Class.forName 静态方法。当你知道该类的全路径名时，你可以使用该方法获取Class类对象

```java
Class clz = Class.forName("java.lang.String");
```
#### 第二种，使用类的.class方法，这种方法只适合在编译前就知道操作的Class

```plain
Class clz = String.class;
```
#### 第三种，使用类对象的getClass()方法。

```java
String str = new String("hello");
Class clz = str.getClass();
```
### 通过以上任意一种方式就可以获取Class对象了，反射调用内部类的时候需要使用`$`来代替`.`,如`Test`类有一个叫做`Hello`的内部类，那么调用的时候就应该将类名写成：`Test$Hello`。

### 通过反射创建类对象

通过反射创建类对象主要有两种方式：通过Class对象的newInstance()方法，通过Constructor对象的newInstance()方法

#### 第一种，通过Class对象的newInstance()方法

```java
Class clz =Apple.class;
Apple apple = (Apple)clz.newInstance();
```
#### 第二种，通过Constructor 对象的newInstance()方法

```plain
Class clz = Apple.class;
Constructor sonstructor = clz.getConstructor();
Apple apple =(Apple)constructor.newInstance();
```
使用Constructor对象创建类对象可以选择特定构造方法，而通过Class对象则只能使用默认的无参数构造方法。下面的代码就调用了一个有参数的构造方法进行了类对象的初始化。
```java
Class clz = Apple.class;
Constructor constructor = clz.getConstructor(String,class,int.class);
Apple apple = (Apple)constructor.newInstance("红富士"，15)；
```
### 通过反射获取类属性，方法，构造器

我们通过Class对象的getFields()方法可以获取Class类的属性。但无法获取私有属性。

```java
Class clz = Apple.class;
Field [] fields = clz.getFields();
for (Field field : fields){
    System.out.println(field.getName());
}
```
输出结果是price
而如果使用Class对象的getDeclareFields()方法则可以获取包括私有属性在内的所有属性:

```java
Class clz = Apple.class;
Field [] fields = clz.getDeclareFields()
for (Field field : fields){
   System.out.println(field.getName());
}
```
输出结果是
name

price

与获取类属性一样，当我们去获取类方法、类构造器时，如果要获取私有方法或私有构造器，则必须使用有 declared 关键字的方法。

