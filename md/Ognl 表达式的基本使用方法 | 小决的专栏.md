> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [jueee.github.io](https://jueee.github.io/2020/08/2020-08-15-Ognl%E8%A1%A8%E8%BE%BE%E5%BC%8F%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95/)

> OGNL 是 Object-Graph Navigation Language 的缩写，它是一种功能强大的表达式语言，通过它简单一致的表达式语法，可以存取对象的任意属性，调用对象的方法，遍历整个对象的结构图，实现字段类型转化等功能。

OGNL 是 `Object-Graph Navigation Language`（**对象导航图语言**）的缩写，它是一种功能强大的表达式语言，通过它简单一致的表达式语法，可以存取对象的任意属性，调用对象的方法，遍历整个对象的结构图，实现字段类型转化等功能。它使用相同的表达式去存取对象的属性。这样可以更好的取得数据。

### [](#Ognl-语言介绍 "Ognl 语言介绍")Ognl 语言介绍

OGNL 表达式官方指南：[https://commons.apache.org/proper/commons-ognl/language-guide.html](https://commons.apache.org/proper/commons-ognl/language-guide.html)

#### [](#介绍 "介绍")介绍

Ognl 是一个功能强大的表达式语言，用来获取和设置 java 对象的属性 ，它旨在提供一个更高抽象度语法来对 java 对象图进行导航。

另外，java 中很多可以做的事情，也可以使用 OGNL 来完成，例如：列表映射和选择。

对于开发者来说，使用 OGNL，可以用简洁的语法来完成对 java 对象的导航。通常来说：通过一个 “路径” 来完成对象信息的导航，这个 “路径” 可以是到 java bean 的某个属性，或者集合中的某个索引的对象，等等，而不是直接使用 get 或者 set 方法来完成。

#### [](#三要素 "三要素")三要素

**首先来介绍下 OGNL 的三要素：**

*   **表达式（Expression）**：
    
    表达式是整个 OGNL 的核心内容，所有的 OGNL 操作都是针对表达式解析后进行的。通过表达式来告诉 OGNL 操作到底要干些什么。因此，表达式其实是一个带有语法含义的字符串，整个字符串将规定操作的类型和内容。OGNL 表达式支持大量的表达式，如 “链式访问对象”、表达式计算、甚至还支持 Lambda 表达式。
    
*   **Root 对象**：
    
    OGNL 的 Root 对象可以理解为 OGNL 的操作对象。当我们指定了一个表达式的时候，我们需要指定这个表达式针对的是哪个具体的对象。而这个具体的对象就是 Root 对象，这就意味着，如果有一个 OGNL 表达式，那么我们需要针对 Root 对象来进行 OGNL 表达式的计算并且返回结果。
    
*   **上下文环境**：
    
    有个 Root 对象和表达式，我们就可以使用 OGNL 进行简单的操作了，如对 Root 对象的赋值与取值操作。但是，实际上在 OGNL 的内部，所有的操作都会在一个特定的数据环境中运行。这个数据环境就是上下文环境（Context）。OGNL 的上下文环境是一个 Map 结构，称之为 OgnlContext。Root 对象也会被添加到上下文环境当中去。
    
    说白了上下文就是一个 MAP 结构，它实现了 java.utils.Map 的接口。
    

### [](#使用-Ognl "使用 Ognl")使用 Ognl

引入 Maven：

```
<dependency>
	<groupId>ognl</groupId>
	<artifactId>ognl</artifactId>
	<version>3.1.19</version>
</dependency>
```

示例代码：

示例类：`sample.ognl.Address`

```
@Data
public class Address {

	private String port;
	private String address;
	
	public Address(String port,String address) {
		this.port = port;
		this.address = address;
	}
}
```

示例类 `sample.ognl.User`：

```
@Data
public class User {

	private String name;
	private int age;
	private Address address;
	
	public User() {}
	
	public User(String name, int age) {
		this.name = name;
		this.age = age;
	}
}
```

### [](#Ognl-的基本语法 "Ognl 的基本语法")Ognl 的基本语法

#### [](#对Root对象的访问 "对Root对象的访问")对 Root 对象的访问

OGNL 使用的是一种链式的风格进行对象的访问。

```
User user = new User("test", 23);
Address address = new Address("330108", "杭州市滨江区");
user.setAddress(address);
System.out.println(Ognl.getValue("name", user));	// test
System.out.println(Ognl.getValue("name.length", user));		// 4
System.out.println(Ognl.getValue("address", user));		// Address(port=330108, address=杭州市滨江区)
System.out.println(Ognl.getValue("address.port", user));	// 110003
```

#### [](#对上下文对象的访问 "对上下文对象的访问")对上下文对象的访问

使用 OGNL 的时候如果不设置上下文对象，系统会自动创建一个上下文对象，如果传入的参数当中包含了上下文对象则会使用传入的上下文对象。

当访问上下文环境当中的参数时候，需要在表达式前面加上 '#' ，表示了与访问 Root 对象的区别。

```
public static String demo2() throws OgnlException {
	User user = new User("test", 23);
	Address address = new Address("330108", "杭州市滨江区");
	user.setAddress(address);
	Map<String, Object> context = new HashMap<String, Object>();
	context.put("init", "hello");
	context.put("user", user);
	System.out.println(Ognl.getValue("#init", context, user));	// hello
	System.out.println(Ognl.getValue("#user.name", context, user));	// test
	System.out.println(Ognl.getValue("name", context, user));	// test
	return "this is demo2 method";
}
```

#### [](#对静态变量的访问 "对静态变量的访问")对静态变量的访问

在 OGNL 表达式当中也可以访问静态变量或者调用静态方法，格式如 @[class]@[field/method ()]。

```
public static String ONE = "one";
// 对静态变量的访问（@[class]@[field/method()]）
public static void demo3() throws OgnlException {
	Object object1 = Ognl.getValue("@sample.ognl.OgnlDemo@ONE", null);
	Object object2 = Ognl.getValue("@sample.ognl.OgnlDemo@demo2()", null);	// hello、test、test
	System.out.println(object1);	// one	
	System.out.println(object2);	// this is demo2 method
}
```

#### [](#方法的调用 "方法的调用")方法的调用

如果需要调用 Root 对象或者上下文对象当中的方法也可以使用.+ 方法的方式来调用。甚至可以传入参数。

赋值的时候可以选择上下文当中的元素进行给 Root 对象的 name 属性赋值。

```
User user = new User();
Map<String, Object> context = new HashMap<String, Object>();
context.put("name", "rcx");
context.put("password", "password");
System.out.println(Ognl.getValue("getName()", context, user));	// null
Ognl.getValue("setName(#name)", context, user);
System.out.println(Ognl.getValue("getName()", context, user));	// rcx
```

#### [](#对数组和集合的访问 "对数组和集合的访问")对数组和集合的访问

OGNL 支持对数组按照数组下标的顺序进行访问。此方式也适用于对集合的访问，对于 Map 支持使用键进行访问。

```
User user = new User();
Map<String, Object> context = new HashMap<String, Object>();
String[] strings  = {"aa", "bb"};
ArrayList<String> list = new ArrayList<String>();
list.add("aa");
list.add("bb");
Map<String, String> map = new HashMap<String, String>();
map.put("key1", "value1");
map.put("key2", "value2");
context.put("list", list);
context.put("strings", strings);
context.put("map", map);
System.out.println(Ognl.getValue("#strings[0]", context, user));	// aa
System.out.println(Ognl.getValue("#list[0]", context, user));	// aa
System.out.println(Ognl.getValue("#list[0 + 1]", context, user));	// bb
System.out.println(Ognl.getValue("#map['key1']", context, user));	// value1
System.out.println(Ognl.getValue("#map['key' + '2']", context, user)); 	// value2
```

从上面代码不仅看到了访问数组与集合的方式同时也可以看出来 OGNL 表达式当中支持操作符的简单运算。有如下所示：

*   2 + 4 // 整数相加（同时也支持减法、乘法、除法、取余 [% /mod]、）
*   "hell" + "lo" // 字符串相加
*   i++ // 递增、递减
*   i == j // 判断
*   var in list // 是否在容器当中

#### [](#投影与选择 "投影与选择")投影与选择

OGNL 支持类似数据库当中的选择与投影功能。

*   **投影**：选出集合当中的相同属性组合成一个新的集合。语法为 collection.{XXX}，XXX 就是集合中每个元素的公共属性。
    
*   **选择**：选择就是选择出集合当中符合条件的元素组合成新的集合。语法为 collection.{Y XXX}，其中 Y 是一个选择操作符，XXX 是选择用的逻辑表达式。
    
    选择操作符有 3 种：
    
    *   ? ：选择满足条件的所有元素
    *   ^：选择满足条件的第一个元素
    *   $：选择满足条件的最后一个元素

```
User p1 = new User("name1", 11);
User p2 = new User("name2", 22);
User p3 = new User("name3", 33);
User p4 = new User("name4", 44);
Map<String, Object> context = new HashMap<String, Object>();
ArrayList<User> list = new ArrayList<User>();
list.add(p1);
list.add(p2);
list.add(p3);
list.add(p4);
context.put("list", list);
System.out.println(Ognl.getValue("#list.{age}", context, list));	
// [11, 22, 33, 44]
System.out.println(Ognl.getValue("#list.{age + '-' + name}", context, list));	
// [11-name1, 22-name2, 33-name3, 44-name4]
System.out.println(Ognl.getValue("#list.{? #this.age > 22}", context, list));	
// [User(name=name3, age=33, address=null), User(name=name4, age=44, address=null)]
System.out.println(Ognl.getValue("#list.{^ #this.age > 22}", context, list));	
// [User(name=name3, age=33, address=null)]
System.out.println(Ognl.getValue("#list.{$ #this.age > 22}", context, list));	
// [User(name=name4, age=44, address=null)]
```

#### [](#创建对象 "创建对象")创建对象

OGNL 支持直接使用表达式来创建对象。主要有三种情况：

*   构造 List 对象：使用 {}, 中间使用 ',' 进行分割如 {"aa", "bb", "cc"}
*   构造 Map 对象：使用 #{}，中间使用 ', 进行分割键值对，键值对使用':'区分，如 #{"key1":"value1","key2":"value2"}
*   构造任意对象：直接使用已知的对象的构造方法进行构造。

```
System.out.println(Ognl.getValue("#{'key1':'value1'}", null));	// {key1=value1}
System.out.println(Ognl.getValue("{'key1','value1'}", null));	// [key1, value1]
System.out.println(Ognl.getValue("new sample.ognl.User()", null));	
// User(name=null, age=0, address=null)
```