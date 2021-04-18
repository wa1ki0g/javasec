java.lang.Runtime因为有一个exec方法可以执行命令，所以在很多的payload中我们都可以看到反射调用Runtime类来执行本地系统命令，通过学习如何反射Runtime类也能让我们理解反射的一些基础用法以及一些攻击手法

不使用反射执行本地命令代码片段：

```plain
System.out.println(IOUtils.toString(Runtime.getRuntime().exec("whoami").getInputStream(),"UTF-8"));
```
如上可以看到，我们可以使用一行代码完成本地的rce，但是入砂锅使用反射就会比较麻烦了，我们不得不需要间接性的调用Runtime的exec方法
反射Runtime执行本地命令执行代码：

```java
//获取Runtime类对象
Class runtimeClass1 = Class.forname("java.lang.Runtime");
//获取构造方法
Constructor constructor = runtimeClass1.getDeclaredConstructor();
//如果方法是 private修饰的，当你用反射去访问的时候
//setAccessible(true); 之后 才能访问
setAccessible(true);
//创建Runtime类实例，等价于Runtime rt=new Runtime();
object runtimeInstance=constructor.newInstance();
//获取Runtime的exec(string cmd)方法
//getMethod 系列方法获取的是当前类中所有公共方法，包括从父类继承的方法
//getDeclaredMethod 系列方法获取的是当前类中“声明”的方法，是实在写在这个类里的,包括私有的方法，但从父类里继承来的就不包含了
Method runtimeMethod = runtimeClass1.getMethod("exec",String.class);
//调用exec方法，等价于exec(String cmd)方法
//method.invoke(方法实例对象, 方法参数值，多个参数值用","隔开);
Process runtimeMethod = (Process) runtimeMethod.invoke(runtimeInstance,cmd);
//获取命令执行结果
InputStream in = process.getInputStream();
//输出命令执行结果
System.out.println(IOUtiles.tostring(in,"UTF-8"));
```
反射调用Runtime实现本地命令执行的流程如下：
1.反射获取Runtime类对象(Class.forName("java.lang,Runtime"))

2.使用Runtime类的Class对象获取Runtime类的无参构造方法(getDeclareConstructor()),因为Runtime的构造方法是private的我们无法直接调用，所以我们需要通过反射去修改方法的访问权限（constructor.setAccessible(true)）。

3.获取Runtime类的exec(string)方法（runtimeClass1.getMethod("exec",String.class);）。

4.调用exec(String)方法（runtimeMethod.invoke(runtimeInstance,cmd)）。

#### 上面的代码每一步都写了非常清晰的注释，接下来我们将进一步深入的了解下每一步具体含义。

#### 反射创建类实例：

在java的任何一个类都必须有一个或者多个构造方法，如果代码中没有创建构造方法那么在类编译的时候会自动创建一个无参数的构造方法

Runtime类构造方法示例代码片段:

```java
public class Runtime {
   /** Don't let anyone else instantiate this class */
  private Runtime() {}
}
```
从上面的Runtime类代码注释我们可以看到他的构造方法是私有方法，看出来其本身是不希往望除了其自身以外的任何人去创建该类实例的，所以我们没办法new一个Runtime类实例即不能实用Runtime aa = new Runtime();的方式去创建Runtime对象，在示例中我们借助了反射机制，修改了方法访问权限从而间接的创建出了Runtime对象。
runtimeClass1.getDeclaredConstructor和runtimeClass1.getConstructor都可以获取到类的构造方法，区别在于后者无法获取到私有方法，所以一般在获取某个类的构造方法时侯，我们会去使用前者去获取构造方法。如果构造方法中有一个或多个参数的情况下我们应该在获取构造方法时候传入对应得参数类型数组，如：

clazz.getDeclaredConstructor(String.class,String.class)。

如果我们想获取类的所有构造方法我们可以使用:

clazz.getDeclaredConstructors来获取一个Constructor数组

获取到Constructor以后我们可以通过constructor.newInstance()来创建类实例，同理如果有参数的情况下我们应该传入对应的参数值。

如：constructor.new.Instance("admin","123456").当我们没有权限访问构造方法时，我们应该调用constructor.setAccessible(true)来修改访问权限就可以成功得创建出类的实例了。

### 反射调用类方法

Class对象提供了一个获取某个类的所有的成员方法的方法，也可以通过方法名和方法参数类型来获取指定成员方法：

#### 获取当前类所有的成员方法：

```java
Method[] methods = clazz.getDeclaredMethods()
```
#### 获取当前指定类方法：

```java
Method method = clazz.getDeclaredMethod("方法名")；
Method method = clazz.getDeclaredMethod("方法名"，参数类型如string.class,多个参数用","隔开;)
```
`getMethod`和`getDeclaredMethod`都能够获取到类成员方法，区别在于`getMethod`只能获取到`当前类和父类`的所有有权限的方法(如：`public`)，而`getDeclaredMethod`能获取到当前类的所有成员方法(不包含父类)。
#### 反射调用方法

获取到`java.lang.reflect.Method`对象以后我们可以通过`Method`的`invoke`方法来调用类方法。

#### 调用类方法代码片段：

```plain
method.invoke(方法实例对象, 方法参数值，多个参数值用","隔开);
```
`method.invoke`的第一个参数必须是类实例对象，如果调用的是`static`方法那么第一个参数值可以传`null`，因为在java中调用静态方法是不需要有类实例的，因为可以直接`类名.方法名(参数)`的方式调用。
`method.invoke`的第二个参数不是必须的，如果当前调用的方法没有参数，那么第二个参数可以不传，如果有参数那么就必须严格的`依次传入对应的参数类型`。

## 反射调用成员变量

Java反射不但可以获取类所有的成员变量名称，还可以无视权限修饰符实现修改对应的值。

获取当前类的所有成员变量：

```plain
Field fields = clazz.getDeclaredFields();
```
获取当前类指定的成员变量：
```plain
Field field  = clazz.getDeclaredField("变量名");
```
`getField`和`getDeclaredField`的区别同`getMethod`和`getDeclaredMethod`。
获取成员变量值：

```plain
Object obj = field.get(类实例对象);
```
修改成员变量值：
```plain
field.set(类实例对象, 修改后的值);
```
同理，当我们没有修改的成员变量权限时可以使用:`field.setAccessible(true)`的方式修改为访问成员变量访问权限。
如果我们需要修改被`final`关键字修饰的成员变量，那么我们需要先修改方法

```plain
copy
// 反射获取Field类的modifiers
Field modifiers = field.getClass().getDeclaredField("modifiers");
// 设置modifiers修改权限
modifiers.setAccessible(true);
// 修改成员变量的Field对象的modifiers值
modifiers.setInt(field, field.getModifiers() & ~Modifier.FINAL);
// 修改成员变量值
field.set(类实例对象, 修改后的值);
```
Java反射机制是Java动态性中最为重要的体现，利用反射机制我们可以轻松的实现Java类的动态调用。Java的大部分框架都是采用了反射机制来实现的(如:`Spring MVC`、`ORM框架`等)，Java反射在编写漏洞利用代码、代码审计、绕过RASP方法限制等中起到了至关重要的作用。
