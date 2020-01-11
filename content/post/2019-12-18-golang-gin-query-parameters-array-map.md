+++
title="Java lambda表达式入门"
tags=["java"]
categories=["Java"]
date="2019-12-18T21:55:00+08:00"
url="/2019/12/18/golang-gin-query-parameters-array-map.html"
toc=false
+++
### 什么是lambda表达式
看看lambda表达式在其他语言中怎么使用的：

> 在python中有lambda表达式，在python中可以使用它定义匿名方法，并赋值给一个变量，然后那个变量就成为了一个方法，比如
```python
f = lambda x:  (x + 1 ) if x > 100 else x - 1
# 相当于
def f(x):
	return (x + 1 ) if x > 100 else x - 1;

```
或者把lambda表达式传给一个方法，例如：
```python
def m(fun,x):
	fun(x)
m(lambda x: print(x),1000)
```

由上面的示例可以看出来，lambda表达式是一段可执行的代码块，可传入方法内执行，且可以赋给任意变量由变量执行。
这其实也是支持函数式编程的特征，一段代码块可以在任何场所执行，不像java，需要借助对象这个承载来执行代码块。从python的执行方式就可以看出来，
我们可以在一个py文件中输入`print 'hello world'`,然后就可以直接执行输出；但是在java中，你需要先创建一个类，然后创建main方法，在main方法里面执行`System.out.println("hello world")`;
可以看出来，支持函数式编程能够减少很多的冗余代码，但是也因为灵活性使得代码的风格十分多变导致代码不好维护。
Java引入lambda表达式的目的，自然是想引入函数式编程的优点，**让代码简洁**，这也是lambda表达式的作用。

### 在java中怎么使用lambda表达式
既然知道了什么是lambda表达式，那么再来了解一下再Java中使用这种语言特性，有哪些方式又有那些限制和注意事项。

#### 函数式接口
在Java中，lambda是基于函数式接口的，也可以理解成它是替换函数式接口的实例的匿名对象，且它**只能**转换成函数式接口。
那么什么是函数式接口呢？**只有一个**抽象方法的接口，只能有一个抽象接口，但是接口是可以包含default方法的（Java 8特性），如下的接口就是合法的：
```java
@FunctionalInterface
interface FuncInter {
	void test();

	default void testdefault() {
		System.out.println("hello default");
	}
	// void test2(); error it is not a Functional interface
}

@FunctionalInterface
interface FuncInter2 {
	void test(String x);
}
```
> @FunctionalInterface是一个标识注解，表明本接口只有一个抽象方法，如果再次添加抽象方法，会在编译期报错，加上此注解可以防止误加抽象方法。

函数式接口有了，怎么用呢，先看下在java8 以前时怎么用的。
```java
	public static void main(String[] args) {
		// 匿名类的方法
		FuncInter2 f = new FuncInter2() {

			@Override
			public void test(String x) {
				// TODO Auto-generated method stub

			}

		};
		f.test("hello");
	}

```
如果不用匿名类，就需要创建一个类实现接口，再创建接口实例，最后调用test方法；
#### 试用lambda表达式
既然说lambda表达式是用来替换函数式接口的，使用lambda的方法应该就可以减少相关代码的编写:
```java
@Test
	public void testLambdasyntax() {

		// lambda表达式是用来替换函数式接口的（匿名）实例
		/*
		 * 无参抽象方法
		 */
		// 1. 单行语句
		FuncInter fi = () -> System.out.println("hello");
		fi.test();
		// 2. 多行语句使用{}包围
		fi = () -> {
			// TODO
			System.out.println("line 1");
			System.out.println("line 2");
		};
		fi.test();
		/*
		 * 有参抽象方法
		 */
		FuncInter2 fi2 = (String a) -> System.out.println(a);
		fi2.test("www");
		// 可以推断参数类型可以不写类型
		fi2 = (a) -> System.out.println("no type " + a);
		fi2.test("hello ");
		// 单个参数，且参数类型可以推断，可以不加括号
		fi2 = a -> System.out.println("no parentheses " + a);
		fi2.test("hello ");
	}

	/**
	 * 把lambda传给方法 </br>
	 * 实际上就是把函数式接口的匿名对象传给方法 </br>
	 * 可以理解成把函数传递给方法，但是在java里面实际上还是基于对象
	 */
	@Test
	public void deliverToMethod() {
		// 定义一个局部类
		class MyClassNeedInterface {
			FuncInter fi = null;

			MyClassNeedInterface(FuncInter fi) {
				this.fi = fi;
			}

			void doSomeThing() {
				fi.test();
			}

			void doSomeThing(FuncInter2 fi, String x) {
				fi.test(x);
			}
		}
		MyClassNeedInterface c = new MyClassNeedInterface(() -> System.out.println("This is from a lambda expression"));
		c.doSomeThing();
		String x = "a string ";
		// 等价于 c.doSomeThing(a->System.out.println(a), x);
		c.doSomeThing(System.out::println, x);
		c.doSomeThing(a -> System.out.println(a), x);
	}

}

```
确实简洁了很多。可以发现java中一个lambda表达式的形式是 **参数、箭头、表达式**，且可以根据情况省略括号或者参数类型，定义出来的表达式相当于一个接口的实例对象。
箭头后面的表达式如果是多行代码，使用花括号括起来。
实际上lambda表达式的用法，也就限于此类情况（替代函数式接口的匿名实例）。
#### 方法引用
在示例`deliverToMethod`中有`System.out::println`这种形式的使用，是引用了已经存在的方法来填充接口的抽象方法，它支持三种形式：
* instance::instanceMethod
* Class::staticMethod
* Class::instanceMethod  **此种方法的调用 第一个参数是实例方法的调用者（方法的目标）**。

#### 构造器引用
和上节的方法引用类似，只是引用的方法名是new。

#### 闭包
> 闭包（closure）是计算机编程领域的专业名词，指可以包含自由（未绑定到特定对象）变量的代码块，子函数可以使用父函数中的局部变量。


lambda表达式使得java拥有了闭包这个特性。 简单来说，闭包是有数据的行为（函数、方法）,是可以读取其他函数内部变量的函数。
lambda表达式具有代码块，那么自由变量是什么？所谓自由变量，是未绑定到特定对象的变量，那么外部的局部变量肯定是自由变量；
子函数可以使用父函数中的局部变量，也就是代码块可以访问外围的局部变量，还要能够携带出去。
```java
@FunctionalInterface
interface Closure {
	int test(int a);
}
	@Test
	public void testClosure() {
		final int a = 100; // 这里的final可以不加但是得建立在后面不会修改的前提上，所以还是建议加上；
		Closure ci = (b) -> {
			System.out.println("I catch a int value of a :" + a);
			return (a + b);
		};
		// a month later
		int v = ci.test(1);
		System.out.println(v);
	}

```
**在java中，在lambda中访问的外部局部变量，必须是final变量，即引用不可修改，如果是基本类型，必须是常量。**
另外有一个很重要的规则，**lambda表达式的体和使用这个表达式的块具有同一个作用域。所以同名变量会命名冲突**
如：
```java
@Test
	public void testSocpe() {
		int a = 1000;
		Closure closure = a -> a + 100; // ERROR
	}
	@Test
	public void testSocpe() {
		int b = 1000;
		Closure c = a -> this.add100(a);//this的作用域是在testScope方法里面
		System.out.println(c.test(b));// 1100

	}

	private int add100(int a) {
		return a + 100;
	}
```
#### 什么时候使用lambda表达式
lambda表达式定义的可执行代码块，如果是需要立即执行的，可以直接写在执行方法里面，何必绕个圈子去调用接口的方法呢。所以lambda表达式的使用场景是，暂时不要执行的代码块，在未来因为某个事件触发而执行，所以重点在于延迟执行。

jdk提供了一系列的函数式接口，涵盖很多使用场景。常用的有：

| 接口 | 抽象方法| 参数类型 | 返回值类型 |
| :----| :----|:-----|:-----|
| Runnable  | run| - |void| 
| Supplier<T> | get| - | T|
| Consumer<T> | accept |T|void|
| Predicate<T> | test |T|boolean|


* 参考文献
[Java核心技术 卷I 第十版]
