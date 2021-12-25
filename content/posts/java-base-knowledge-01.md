---
title: "【Java 回炉计划】01：基础知识流水账"
date: 2021-12-25T12:22:10+08:00
draft: false
tags: ["Java"]
---

详细内容可参考大佬整理的资料 [Java 拾遗卷1：语言特性 · 语雀](https://www.yuque.com/inuter/nghb49)，以下的内容是从个人角度觉得需要额外记录的内容。

<!--more-->

# 面向对象

**里氏代换原则**（Liskov Substitution Principle, LSP）：凡是父类能够出现的地方，都可以用子类替换。

多态是一种允许子类或接口可以有多种实现的特性，这种特性使得代码在运行时，同一个行为可能在不同的情况下，具有不同的表现形式。

子类 `Override` 父类方法时需要满足以下 4 个条件：

1. 方法的访问权限不能变小（private <  package < protected < public）。
2. 返回类型能够向上转型成为父类的返回类型。
3. 异常也要能向上转型成为父类的异常。
4. 方法名、参数类型及个数必须严格一致。

把这个案例看明白：

```java
class A {
    public void m(A a) {
        System.out.println("AA");
    }

    public void m(D d) {
        System.out.println("AD");
    }
}

class B extends A {
    @Override
    public void m(A a) {
        System.out.println("BA");
    }

    public void m(B b) {
        System.out.println("BD");
    }
}

class C extends B {
}

class D extends B {
}

public class Test {

    public static void main(String[] args) {
        A a = new B();
        B b = new B();
        C c = new C();
        D d = new D();
        a.m(a); // BA
        a.m(b); // BA
        a.m(c); // BA
        a.m(d); // AD
    }
}
```

# 类

普通的内部类可以与外部类互相访问私有属性。

构造方法调用的途径：

- 通过 `new` 关键字
- 在子类的构造方法通过 `super` 关键字调用父类的构造方法
- 通过反射的方式获取并调用

在创建类对象时，会先执行父类和子类的静态代码块，然后再执行父类和子类的构造方法。另外，静态代码块只会执行一次，而构造方法在每次实例化对象时都会被执行。

# 接口&注解

注解是一种接口，但使用 `@interface` 修饰符修饰。

- `@Target` 注解用于限定注解的适用范围
- `@Retention` 注解用于表示一个注解保存到哪个阶段
- `@Inherited` 注解仅作用于类，默认情况下，父类的普通注解不能被子类继承，但如果父类的注解被 `@Inherited` 注解修饰，则该注解可以被子类自动继承。

Java 8 新增了一个 `@Repeatable` 元注解，使得某些注解可在同一元素上修饰多次。

# 常用数据类型及问题

Java 语言规范中并没有规定 `boolean` 类型数据的大小，其大小与 JVM 有关。

子类对象允许转换为父类对象，但父类对象不允许强制转换为子类对象。

由于计算机无法准确表示浮点数，`BigDemical` 不要使用入参为 `double` 类型的构造方法，而优先使用入参为 `String` 类型的构造方法。

`RoundingMode` 类

[https://www.yuque.com/inuter/nghb49/gakkv5?inner=g5LDq](https://www.yuque.com/inuter/nghb49/gakkv5?inner=g5LDq)

`BigDecimal` 类重写了 `equals` 方法，当且仅当两个 `BigDecimal` 对象表示的数值和 `scale` 都相同时，返回为 `true` ，如果两个对象表示的数值相同，但 `scale` 不同，返回 `false`。

如果希望只比较两个对象表示的数值，可以使用 `compareTo()` 方法

所有包装类都是 **immutable** 的（不可修改，不可继承），自动装箱都是通过包装类的 `valueOf()` 方法实现的，自动拆箱都是通过包装类的 `xxxValue()` 方法实现的。只有在自动装箱时（即调用 `valueOf()` 方法），相应的缓存机制才会生效。

对于 `表达式 1 ？ 表达式 2 ： 表达式 3` ，如果 `表达式 2` 和 `表达式 3` 有一项是基本数据类型，而另一项是对应的包转类型，那么包装类型的结果应该自动拆箱。

# String

先看如下代码，判断变量 `str1` 和 `str2` 的值分别为什么？

```java
String str1 = 1 + 2 + "3";	// 33
String str2 = "1" + 2 + 3;	// 123
```

如果用 `+` 操作符进行拼接时，拼接对象包含其它类型的常量。则左边第一个字符串常量前的所有 `+` 操作符都会视为加运算。

在循环体内，字符串的链接方法应该使用 `StringBuilder` 的 `append` 方法，而不是使用 `+` 操作符。

字符串的长度等于字符串所包含的 Unicode 代码点总数，即 length() 方法返回的是 Unicode 代码点总数。

**代码点**（*code point*）是 Unicode 独有的定义，最简单的理解是：Unicode 中的每个字符都是一个代码点。说白了就是，“字符”在 Unicode 中被称为“代码点”。

**代码单元**（*code unit*）是代码点的最小组成单位，可以理解为“一个代码点是由多个代码单元组成的”。根据一个代码单元所占内存的大小，可以分为 UTF-8、UTF-16、UTF-32。比如 UTF-8 就表示一个代码单元占用 8 个 bit，即 1 个字节。

那对于 UTF-8 来说，一个字符（或代码点）包含几个代码单元呢，答案是 1 - 4 个代码单元。这类似于计算机组成原理中说的“变长编码”或“赫夫曼编码”，中心思想是“使用频率较高的字符的编码长度尽可能的小”。

- UTF-8：一个代码点可能由 1- 4 个代码单元组成，一个代码单元占用 1 个字节；
- UTF-16：一个代码点可能由 1- 2 个代码单元组成，一个代码单元占用 2 个字节；
- UTF-32：一个代码点由 1 个代码单元组成，一个代码单元占用 4 个字节。

# JVM

### **静态常量池**

静态常量池也称为 **class 文件常量池**，它是指 class 文件中的 **Contant Pool** 结构，可以通过 `javap -verbose` 工具反编译 class 文件查看。

静态常量池只是 class 文件的一个部分，仅存在于 class 文件中，与 JVM 无关。静态常量池用于存放编译阶段生成的各种**字面量**和**符号引用**。

**字面量**包括字符串（如 String str = "a"）、基本类型的常量（即 `final` 修饰的变量）。**符号引用**包括类和方法的全限定名（如 `String` 类的全限定名为 `java/lang/String`）、字段的名称和描述符、方法的名称和描述符。

### **运行时常量池**

运行时常量池是方法区的一部分，全局共享。

当类被加载到内存中时，JVM 会将 class 文件中的静态常量池内容加载到运行时常量池中。在解析阶段，JVM 会把符号引用替换为直接引用（对象的索引值）。

### **字符串常量池**

字符串常量池也是方法区的一部分，全局共享。可以认为字符串常量池是一个对 `String` 进行缓存的结构。

### **方法区&永久代**

**方法区（Method Area）**是 Java 虚拟机规范中的一个概念，主要用来存放已被虚拟机加载的类的相关信息，包括类的信息、运行时常量池、字符串常量池等。

但 Java 虚拟机规范并未规定如何实现方法区，而常用的 **HotSpot** 虚拟机使用永久代（**PermGen**）来实现方法区，所以习惯将方法区称为永久代，但其实这两者并不等价。

# Object 类

**`wait()` 方法和 `Thread.sleep()` 方法有什么区别** ？

- `sleep()` 是 `Thread` 类中的静态方法，而 `wait()` 是 `Object` 类中本地方法；
- `sleep()` 不会释放锁，而 `wait()` 会释放，并自动加入到等待队列中；
- `sleep()` 不依赖于 synchronized 关键字，但 `wait()` 依赖，即 `wait()` 只能在 synchronized 块内使用；
- `sleep()` 不需要被主动唤醒（休眠之后退出阻塞），但 `wait()` 需要使用 `notify()` / `notifyAll()` 唤醒。

Java 语言规范规定 `equals()` 方法必须具备以下几个特性：

- **自反性**：如果 `x` 不为 `null` ， `x.equals(x)` 返回 `true` （ `x.equals(null)` 返回 `false` ）
- **对称性**：如果 `x.equals(y)` 为 `true` ，则 `y.equals(x)` 为 `true`
- **传递性**：如果 `x.equals(y)` 为 `true` 且 `y.equals(z)` 为 `true` ，则 `x.equals(z)` 为 `true`
- **一致性**：如果 `x` 和 `y` 未修改，则 `x.equals(y)` 的结果应该不变

**equals 与 hashCode 默认原则**

- 只要重写 `equals` ，就必须重写 `hashCode`
- 如果 `equals()` 返回 `true`，则 `hashCode()` 必须也返回 `true`
- 如果 `equals()` 返回 `false`，那 `hashCode()` 可以返回 `true` 或 `false`
- 因为 `Set` 存储的是不重复的对象，依据 `hashCode` 和 `equals` 进行判断，所以 `Set` 存储的对象必须重写这两个方法
- 如果自定义对象作为 `Map` 的 key，那么必须重写这两个方法

**equals 方法重写流程**

1. 可以先进行指针判断，如果是同一个对象，则直接返回 `true` ；
2. 判断传入的参数是否为 `null` ，如果是，直接返回 `false` ；
3. 判断两个对象的类型是否相同，如果不同，直接返回 `false` ；
4. 在确保类型相同的前提下进行强制类型转换，再对逐一对字段进行比较。

# 泛型

泛型本质是类型参数化，其与 JVM 无关，仅在编译时做语法检查，编译后被替换，这就是所谓的“类型擦除”。对于类型擦除，总结为一句话就是：**无边界，替换为 Object ；有边界，替换为边界**。

在编译一个泛型子类或泛型接口实现类时，编译器可能会生成一个 synthetic method，这种方法就叫桥接方法（bridge method）。桥接方法的出现是为了解决类型擦除对多态的破坏。

**正确理解 <? extends T> 与 <? super T>**

- `<? extends T>` 表示对象要么是 T 类型，要么是 T 的子类；
- `<? super T>` 表示对象要么是 T 类型，要么是 T 的父类。

**如何选择 <? extends T> 和 <? super T>**

- **不论是 `<? extends T>` 还是 `<? super T>` ，都是以类型 `T` 为中心的；**
- **如果你想使用 `T` 类型对象，则用 `<? extends T>` ；如果你想添加 `T` 类型对象，则用 `<? super T>` 。**

`<?>` 称为无界通配符（unbounded wildcard），可以匹配任何类型，但赋值之后就不能再添加元素了（ null 可以），只能进行获取或删除操作。

# 异常

`Error` 类及其子类和 `RuntimeException` 类及其子类被称为 **Unchecked Exception**；其它所有继承自 `Throwable` 接口的子类（包括 `Exception` 类自身）则称为 **Checked Exception。**Checked Exception 需要在代码中显式处理，否则会编译出错。

`NoClassDefFoundError` 和 `ClassNotFoundException` 有什么不同 ？

- `ClassNotFoundException` 表示在运行时找不到指定的类，如调用 `Class.forName()` 方法时，如果找不到目标类，则抛出该异常。
- `NoClassDefFoundError` 表示未找到目标类的定义，如在编译成功后，故意删除一个类的 class 文件，在执行程序时，就会抛出该错误

当存在 `try` 时，可以只有 `catch` 代码块，也可以只有 `fianlly` 代码块，但就是不能只有 `try` 部分。

`finally` 是在 `return` 表达式运行后执行的，此时要 `return` 的结果已经被先暂存起来，等 `finally` 代码块执行结束后再返回之前暂存的返回值。

**不要在 `finally` 代码块中执行 `return` 语句。**

### **为什么 finally 代码块总能被执行？**

因为编译器在编译 Java 代码时，会复制 `finally` 代码块，并分别放在 `try-catch` 所有正常执行路径及异常执行路径的出口中，这样不管发生什么情况， `finally` 代码块都会执行。

`Throwable` 类的（直接或间接）子类不能是泛型类

# 反射

[https://www.yuque.com/inuter/nghb49/op33g0?inner=IimI7](https://www.yuque.com/inuter/nghb49/op33g0?inner=IimI7)

其中 `getFields()` 、 `getConstructors()` 、 `getMethods()` 三个方法返回所有 public 修饰的属性和方法（包括从父类继承的）；而 `getDeclaredFields()` 、 `getDeclaredConstructors()` 、 `getDeclaredMethods()` 则仅返回本类定义的属性和方法，但不限于 `public`。
