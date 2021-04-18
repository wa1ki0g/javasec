J        ava反射(`Reflection`)是Java非常重要的动态特性，通过使用反射我们不仅可以获取到任何类的成员方法(`Methods`)、成员变量(`Fields`)、构造方法(`Constructors`)等信息，还可以动态创建Java类实例、调用任意的类方法、修改任意的类成员变量值等。Java反射机制是Java语言的动态性的重要体现，也是Java的各种框架底层实现的灵魂。

一般情况下我们使用某个类的时候，比如创建个对象或者直接引用等，都会必定知道他是什么类，是用来做什么的。于是我们可以直接对其进行实例化，之后使用这个类实例化出来的对象进行操作

```java
Apple apple = new Apple(); //直接初始化，「正射」
apple.setPrice(4);
```
上面这个样子的初始化我们可以理解为:"正"，而发射则是我们一开始并不知道我们要初始化的类对象是什么，自然也无法通过上述例子进行实例化操作。以及调用。
这时候，我们使用 JDK 提供的反射 API 进行反射调用：

```java
Class clz = Class.forName("reflect.Apple");
Method method = clz.getMethod("setPrice", int.class);
Constructor constructor = clz.getConstructor();
Object object = constructor.newInstance();
method.invoke(object, 4);
```
上面两段代码的执行结果，其实是完全一样的。但是其思路完全不一样，第一段代码在未运行时就已经确定了要运行的类（Apple），而第二段代码则是在运行时通过字符串值才得知要运行的类
## 一个简单的例子

上面提到的示例程序，其完整的程序代码如下：

```java
public class Apple {
    private int price;
    public int getPrice() {
        return price;
    }
    public void setPrice(int price) {
        this.price = price;
    }
    public static void main(String[] args) throws Exception{
        //正常的调用
        Apple apple = new Apple();
        apple.setPrice(5);
        System.out.println("Apple Price:" + apple.getPrice());
        //使用反射调用
        Class clz = Class.forName("com.chenshuyi.api.Apple");
        Method setPriceMethod = clz.getMethod("setPrice", int.class);
        Constructor appleConstructor = clz.getConstructor();
        Object appleObj = appleConstructor.newInstance();
        setPriceMethod.invoke(appleObj, 14);
        Method getPriceMethod = clz.getMethod("getPrice");
        System.out.println("Apple Price:" + getPriceMethod.invoke(appleObj));
    }
}
```
从代码中可以看到我们使用反射调用了 setPrice 方法，并传递了 14 的值。之后使用反射调用了 getPrice 方法，输出其价格。上面的代码整个的输出结果是：
ApplePrice:5

ApplePrice:14

从这个简单的例子可以看出，一般情况下我们使用反射获取一个对象的步骤：

* 获取类的 Class 对象实例
```plain
Class clz = Class.forName("com.zhenai.api.Apple");
```
* 根据 Class 对象实例获取 Constructor 对象
```plain
Constructor appleConstructor = clz.getConstructor();
```
* 使用 Constructor 对象的 newInstance 方法获取反射类对象
```plain
Object appleObj = appleConstructor.newInstance();
```
而如果要调用某一个方法，则需要经过下面的步骤：
* 获取方法的 Method 对象
```plain
Method setPriceMethod = clz.getMethod("setPrice", int.class);
```
* 利用 invoke 方法调用方法
```plain
setPriceMethod.invoke(appleObj, 14);
```
到这里，我们已经能够掌握反射的基本使用。但如果要进一步掌握反射，还需要对反射的常用 API 有更深入的理解。
