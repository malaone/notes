---
title: Java基础
categories: Java
tags: [Java]
---

#### java创建类

##### .java文件

- java的类名可以小写开头，不过一般第一个字母为大写


- 一个.java文件只能有一个public class

```java
public class A {}
```

- .java文件名

A.java

```java
//文件只能有一个public class，且文件名必须是A.java
public class A {}//表示全局类，其他包可以import

//compile error: The public type A must be defined in its own file
public class B { }

class C {}//局部类，只能被同一包中的其他类引用

class D {}//可以有多个局部类
```

任意文件名.java

```java
//可以没有public类，此时文件名随便
class E {}

class F {}
```

##### class的修饰

- class前面的修饰只能是public, abstract & final

```java
//Illegal modifier for the class A; only public, abstract & final are permitted
private class A {}//error

protected class B {}//error
```

- final class表示类不可以被继承



#### interface与abstract

##### 参考

[Java 8 默认方法和多继承](http://colobu.com/2014/11/04/Java-8-default-method-and-multiple-inheritance/)

##### 两者比较

interface是一种特殊的abstract，都不能实例化。

```java
public interface InterfaceTest {
	int a = 0;//默认是public static final，这里省略了
	
	//不能有构造方法
	//public InterfaceTest() {} 
	
	public static void staticMethod() {
		System.out.println("just test interface");
	}
	
	//java8中，可以在接口中实现方法，供子类使用
	public default void defaultMethod() {
		System.out.println("just test interface");
	}
  
	//默认是abstract方法，这里省略了
  	//这里不能是private
	public InterfaceTest abstractMethod();
}

public abstract class AbstractTest {
	public int a;
	
	public AbstractTest() {}
	
	public void test() {//供子类使用
		System.out.println("just test");
	}
	
	public static void staticMethod() {
		System.out.println("just test");
	}
	
	public abstract AbstractTest abstractFunction();
}

public class App {
    public static void main( String[] args ) {
        //两者都可以直接调用静态方法和静态属性
    	int a = InterfaceTest.a;
    	InterfaceTest.staticMethod();
    	AbstractTest.staticMethod();
      
    	//抽象类不能实例化
    	//AbstractTest ab = new AbstractTest();
      
      	//可以这样实例化
    	AbstractTest ab = new AbstractTest(){
    		public AbstractTest abstractMethod() {
    			return null;
    		}
    	};
    	
    	InterfaceTest ift = new InterfaceTest(){
    		public InterfaceTest abstractMethod() {
    			return null;
    		}
    	};
    }
}
```

##### java8的default method与多继承

java8以前，interface中是不能有实现过的方法default method的，java是单继承类多实现接口，因为接口中的方法都没有实现，不会存在方法冲突的问题。

```java
interface A {
  default String say(String name) {
  	return "hello " + name;
  }
}
 
interface B {
  default String say(String name) {
  	return "hi " + name;
  }
}

//接口可以多继承
interface C extends A, B{
  //这里必须重写，否则报错，因为无法确定使用A、B中的哪一个say
  default String say(String name) {
 	 return "greet " + name;
  }
}
```

从下面例子可以看出，子类优先继承父类的方法， 如果父类没有相同签名的方法，才继承接口的默认方法；

```java
interface A {
  default void say() {
  	System.out.println("A");
  }
}
 
static class B {
  public void say() {
  	System.out.println("B");
  }
}
 
static class C extends B implements A{}
 
public static void main(String[] args) {
	C c = new C();
	c.say(); //B
}
```

接口多层继承：

```java
interface A {
	default void say(int a) {
		System.out.println("A");
	}
	
	default void run() {
		System.out.println("A.run");
	}
}
interface B extends A{
	default void say(int a) {
		System.out.println("B");
	}
	
	default void play() {
		System.out.println("B.play");
	}
}
interface C extends A,B{//B、A虽然有相同的方法say，但B覆盖了A，C继承B的Say
	
}
```

总结

- 类优先于接口。 如果一个子类继承的父类和接口有相同的方法实现。 那么子类继承父类的方法
- 子类型中的方法优先于父类型中的方法。 
- 如果以上条件都不满足， 则必须显示覆盖/实现其方法，或者声明成abstract。



#### static

参考：[java static关键字的四种用法](http://www.cnblogs.com/dotgua/p/6354151.html)



#### final

参考：[java final关键字的几种用法](http://www.cnblogs.com/dotgua/p/6357951.html)

1. final修饰变量表示该变量值不能改变，只能赋值一次
2. final修饰的引用表示引用中存放的地址不能变
3. final修饰的方法表示不能被覆盖
4. 类中所有的private方法都隐式地指定为是final的，由于无法在类外使用private方法，所以也就无法覆盖它
5. 用来修饰方法参数，表示在变量的生存期中它的值不能被改变
6. 修饰类，表示该类无法被继承

#### 类型相关

##### 枚举enum

```java
public enum Color {
  RED, GREEN, BLANK, YELLOW
}

public enum Color {
    RED("红色", 1), 
  	GREEN("绿色", 2), 
  	BLANK("白色", 3), 
  	YELLO("黄色", 4);
    // 成员变量
    private String name;
    private int index;

    // 私有构造方法
    private Color(String name, int index) {
      this.name = name;
      this.index = index;
    }
}

public enum WebEnvironment {
    MOCK(false),
    RANDOM_PORT(true),
    DEFINED_PORT(true),
    NONE(false);

    private final boolean embedded;

    private WebEnvironment(boolean embedded) {
        this.embedded = embedded;
    }

    public boolean isEmbedded() {
        return this.embedded;
    }
}
```

##### 类型转换

- 参考：

[Java类型强制转换](http://blog.csdn.net/czengze/article/details/52976820)

- 引用类型

在Java中由于继承和向上转型，子类可以非常自然地转换成父类，但是父类转换成子类则需要强制转换。因为子类拥有比父类更多的属性、更强的功能，所以父类转换为子类需要强制。

```java
public class Father {}

public class Son extends Father {}

public class App {
    public static void main( String[] args ) {
    	Father father = new Son(); //父类引用指向子类对象 
    	System.out.println(father instanceof Son);//true
    	
    	//父类强制转换为子类，运行正常
        //在进行强转前，要进行类型检查
    	Son son = (Son)father; 
    	 
    	Father father2 = new Father(); //父类引用没有指向子类对象 
    	System.out.println(father2 instanceof Son);//false
    	
    	//运行出错，ClassCastException：Father cannot be cast to Son
    	Son son2 = (Son) father2; 
      
        father.eat();
//    	father.study(); //编译报错,父类引用只能调用父类中的方法
    	son.eat();
    	son.study();
    }
}
```

- 基本数据类型

  *java中整数类型默认的int类型；小数类型默认的double；

  *char 可以当做一中特殊的整数类型；

  *int无法转换为boolean；

  *小数类型转为整数类型，小数可能被舍弃，所有出现精度损失，所以需要强制转换；

  *boolean 类型不能转换成任何其它数据类型；

| 数据类型    | 所占字节 |
| ------- | ---- |
| boolean | 未定   |
| byte    | 1字节  |
| char    | 2字节  |
| short   | 2字节  |
| int     | 4字节  |
| long    | 8字节  |
| float   | 4字节  |
| double  | 8字节  |

```java
char ch = '8';
int num = ch - '0';//char可以直接赋给int，不过是ASCII，所以减去'0'的ASCII即可

//整数默认是int
int a = 3;//这里的3是int
long b = 3L;

//浮点数默认是double
double c = 3.14;
float d = 3.14F;
float f1 = (float) 100.9;

//没报错的原因：
//编译时候，进行检查，看赋值大小是否超过变量的类型所容纳的范围
//如果超过，报错：从int转换到byte可能会有损失，如果没超过，编译通过
short a = 2;
byte b2 = 120;

//面试陷阱1
byte b1 = 10;
byte b2 = 11;
//java为了防止两个小数相加超出自己所表示的范围，把b1 ,b2变成 int 再相加
byte b3 = b1 + b2 //错误
byte b4 = (byte)(b1 + b2); //正确

//面试陷阱2
short s1 = 1; 
s1 = s1 + 1; //错误: s1+1中1为int，s1自动转为int

short s2 = 1; 
s2 += 1; // 等同于short s2 = (short)(s2 + (short)1); //正确
```

隐式类型转换：

```
byte ->short(char)->int->long->float->double
```

显示类型转换，也叫强制转换，可能会失去精度：

```
double→float→long→int→short(char)→byte
```



#### 路劲相关

- classpath 和 classpath* 区别

```
classpath：只会到你的class路径中查找找文件; 
classpath*：不仅包含class路径，还包括jar文件中(class路径)进行查找. 
```



#### 泛型

##### 参考

[Java泛型深入理解](http://blog.csdn.net/sunxianghuang/article/details/51982979)（包含常见面试题）

##### 什么是泛型

泛型就是参数化类型，使得代码可以适应多种类型，提高了代码的可读性和安全性，减少了强制类型转换以及确保类型安全。

不使用泛型：

```java
public class BeforeGeneric {  
  
  public static void main(String[] args) {  
    ArrayList stringValues = new ArrayList();  
    stringValues.add(1);//可以向数组中添加任何类型的对象  
    
    //问题1——获取值时必须强制转换，因为get返回的是Object
    //问题2——编译时不会出错，而运行时报java.lang.ClassCastException，因为前面存入的是Integer，不能转String
    String str = (String) stringValues.get(0);    
  }  
}  
```

使用泛型后可以指定集合中放入的类型：

```java
public class GenericType {  
  	public static void main(String[] args) {    
      ArrayList<String> stringValues = new ArrayList<String>();//可读性
      stringValues.add("str");  
      stringValues.add(1); //编译错误，安全性，只能添加String
      String str = stringValues.get(0);//这里不需要强转
  }
}  
```

##### 泛型只在编译阶段有效

泛型基本上完全是在编译器中实现的，由编译器执行类型检测和类型推断，然后生成普通的非泛型的字节码，就是虚拟机完全无感知泛型的存在。这种实现技术称为擦除（erasure）。编译器使用泛型类型信息保证类型安全，然后在生成字节码之前将其清除。

```java
public class App {
    public static void main( String[] args ) {
    	List<String> stringArrayList = new ArrayList<String>();
    	List<Integer> integerArrayList = new ArrayList<Integer>();

    	Class classStringArrayList = stringArrayList.getClass();
    	Class classIntegerArrayList = integerArrayList.getClass();

//在编译器将泛型类编译完成之后，泛型类的类型参数都被全部擦除，
//类参数初始化的泛型类其实是共用的一个泛型类字节码，而并不会一种类参数就生成一种对应的泛型类字节码
    	if(classStringArrayList.equals(classIntegerArrayList)){
    	    System.out.println("类型相同");//List<Integer>和List<String>类型,编译后都是List
    	}
    }
}

//泛型类型  
class Pair<T> {    
    private T value;    
    public T getValue() {    
        return value;    
    }    
    public void setValue(T  value) {    
        this.value = value;    
    }    
}

//擦除后原始类型
//因为在Pair<T>中，T是一个无限定的类型变量，所以用Object替换
class Pair {    
    private Object value;    
    public Object getValue() {    
        return value;    
    }    
    public void setValue(Object  value) {    
        this.value = value;    
    }    
}   

//泛型类型  
class Pair<T extends Number> {//不能使用<T super Number>，注意与通配符上下界区别
    private T value;
}

//擦除后原始类型
class Pair<Number> {    
    private Number value;
}
```

##### 类型擦除和多态的冲突和解决 

如下代码所示：按照上面所说的类型会被擦除的话，那么B类继承A类，B的get/set方法按照常理来说是和父类A的get/set方法不同的，因此不能叫做重写，但是实际上就是重写，为啥呢，还是因为编译器在编译期间自动给B类补充了一个叫桥的方法，也就是java中的Bridge设计模式。

```java
class A<T>{
    private T t;
    public T get(){
        return t;
    }
    public void set(T t){
        this.t=t;
    }
}

class B extends A<Number>{
    private Number n;

    @Override
    public Number get() {//返回类型是Number，与A中返回Object(原型)不一样，不能算重写
        return n;
    }

    @Override
    public void set(Number n) {//入参类型与A不一样
        this.n=n;
    }
}
```

从这段代码编译后生成的字节码中可以发现，编译器帮B类生成了两个方法：

```java
@Override
public Object get() {//与A的原型一致了
  return this.get();//调用B中返回Number的那个get方法
}

@Override
public void set(Object t ) {//与A的原型一致了
  this.set((Number)t ); //调用B中参数是Number的那个set方法
}
```

所以我们自己的重写并非真正意义上的重写，而是java编译器通过桥手段帮我么完成了重写。

在编写java代码时是不允许仅仅通过返回类型来判断方法的，上面A、B类编译后的代码为：

```java
class A {
    private Object t;
    public Object get() {
        return t;
    }
    public void set(Object t) {
        this.t=t;
    }
}

class B extends A {
    private Number n;

    public Number get() {
        return n;
    }

    public void set(Number n) {
        this.n=n;
    }
  
    //与上面的get只有返回类型不一样，会报错：方法已经存在，重复定义
    @Override
    public Object get() {
      	return this.get();
    }

    @Override
    public void set(Object t ) {
        this.set((Number)t );
    }
}
```

但是虚拟机可以通过参数类型和返回类型共同确定一个方法，所以编译器为了实现泛型的多态允许自己来做这样“看似不合法”的事情（典型的走后门），然后交给虚拟机自己去区别处理了。

##### 泛型类

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{ 
    //key这个成员变量的类型为T,T的类型由外部指定  
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }
}

//泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
//传入的实参类型需与泛型的类型参数类型相同，即为Integer.
Generic<Integer> genericInteger = new Generic<Integer>(123456);

//传入的实参类型需与泛型的类型参数类型相同，即为String.
Generic<String> genericString = new Generic<String>("key_vlaue");

//可以不传入泛型实参
Generic generic = new Generic("111111");
Generic generic1 = new Generic(4444);
Generic generic2 = new Generic(55.55);
Generic generic3 = new Generic(false);

//不能对确切的泛型类型使用instanceof操作，以下操作是非法的
if(ex_num instanceof Generic<Number>){   
} 
```

##### 泛型接口

```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}

//实现泛型接口
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}
//正确
class FruitGenerator<T> implements Generator {}
//编译器会报错："Unknown class"
class FruitGenerator implements Generator<T> {}

/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口,虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next();中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```

##### 泛型方法

泛型类，是在实例化类的时候指明泛型的具体类型；泛型方法，是在调用方法的时候指明泛型的具体类型。

```java
/**
 * 泛型方法的基本介绍
 * @param tClass 传入的泛型实参
 * @return T 返回值为T类型
 * 说明：
 *     1）public 与 返回值中间<T>非常重要，可以理解为声明此方法为泛型方法。
 *     2）只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
 *     3）<T>表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
 *     4）与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。
 */
public <T> T genericMethod(Class<T> tClass)throws InstantiationException ,
  IllegalAccessException {
        T instance = tClass.newInstance();
        return instance;
}

Object obj = genericMethod(Class.forName("com.test.test"));


public class GenericTest {
   //泛型类
   public class Generic<T>{
        private T key;

        public Generic(T key) {
            this.key = key;
        }
        //不是一个泛型方法
        public T getKey(){
            return key;
        }
    }

    //泛型方法
    //泛型可以有多个，如：public <T,K> K showKeyName(Generic<T> container){...}          
    public <T> T showKeyName(Generic<T> container){
        T test = container.getKey();
        return test;
    }
  
    //泛型方法
  	public <T> void f(T t){
        System.out.println(t.getClass().getSimpleName());
    }

    //不是泛型方法，只是使用了Generic<Number>这个泛型类做形参而已
    public void showKeyValue1(Generic<Number> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

    //不是泛型方法
    public void showKeyValue2(Generic<?> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }
}
```

泛型与静态方法：

因为泛型类中的泛型参数的实例化是在定义**泛型类型对象（例如ArrayList<Integer>）**的时候指定的，而静态变量和静态方法不需要使用对象来调用。对象都没有创建，如何确定这个泛型参数是何种类型，所以当然是错误的。

```java
//泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数
public class Test2<T> {      
    public static T one;   //编译错误      
    public static  T show(T one){ //编译错误      
      	return null;      
    }      
}    

public class Test2<T> {      
    public static <T> T show(T one){//这是正确的
        return null;      
    }      
} 
```

泛型方法类型的判断：

在调用泛型方法的时候，可以指定泛型类型，也可以不指定。在不指定泛型类型的情况下，泛型类型为该方法中的几种参数类型的**共同父类的最小级**，直到Object。在指定泛型类型的时候，该方法中的所有参数类型必须是该泛型类型或者其子类。

```java
public class Test {
            
    //这是一个简单的泛型方法    
    public static <T> T add(T x,T y){    
        return y;    
    } 
  
    public static void main(String[] args) {    
        /**不指定泛型的时候*/    
        int i=Test.add(1, 2); //这两个参数都是Integer，所以T为Integer    
        Number f=Test.add(1, 1.2);//这两个参数是Integer和Float，所以取共同父类的最小级，T为Number    
        Object o=Test.add(1, "asd");//这两个参数一个是Integer，另一个是String，所以取同一父类的最小级，为Object  
    
        /**指定泛型的时候*/    
        int a=Test.<Integer>add(1, 2);//指定了Integer，所以只能为Integer类型或者其子类    
        int b=Test.<Integer>add(1, 2.2);//编译错误，指定了Integer，不能为Float    
        Number c=Test.<Number>add(1, 2.2); //指定为Number，所以可以为Integer和Float    
    }      
} 
```

##### 泛型通配符？

- 泛型是不能继承的

```java
public void showKeyValue1(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

Generic<Integer> gInteger = new Generic<Integer>(123);
Generic<Number> gNumber = new Generic<Number>(456);

// 编译报错：Generic<java.lang.Integer> cannot be applied to Generic<java.lang.Number>
showKeyValue(gInteger);
```

使用通配符修改上面的方法：

```java
public void showKeyValue1(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```

类型通配符一般是使用？代替具体的**类型实参，不是类型形参**，可以把？看成所有类型的父类，是一种真实的类型。

- 通配符上界

```java
class Fruit{}
class Apple extends Fruit{}
class Orange extends Fruit{}

public class TestBuChang<T>{
    public static void main(String[] args) {
        List<? extends Fruit> list = new ArrayList<>();
        
        //编译错误，?代表Fruit的所有子类，但具体类型不确定
        list.add(new Apple());
        list.add(new Orange());
        list.add(new Fruit());
      	
      	//null没有类型
        list.add(null);
		
      	//入参为Object
        list.contains(new Apple());
        list.indexOf(new Apple());

        Apple apple = (Apple)list.get(0); //get返回Fruit类型
    }
}
```

- 通配符下界

```java
class Fruit{}
class Apple extends Fruit{}
class Apple1 extends Apple{}
class Orange extends Fruit{}

public class TestBuChang<T>{

    public static void main(String[] args) {
        List<? super Apple> list = new ArrayList<>();
      	//这里？代表Apple的父类，根据多态，可以接受？的所有子类
        list.add(new Apple());
        list.add(new Apple1());
        
        //编译错误，？代表Apple的所有父类，不能确定是哪一个
      	list.add(new Fruit());
		
      	//对于super，get返回的是Object，因为编译器不能确定?是Apple的哪个父类，所以只能返回Object
        Object apple = list.get(0);
    }
}
```

##### 泛型不支持数组

```java
//compile error: Cannot create a generic array of ArrayList<String>
List<String>[] ls = new ArrayList<String>[10];

//正确
List<?>[] ls = new ArrayList<?>[10];
List<String>[] ls2 = new ArrayList[10];

//sun 官方例子
List<String>[] lsa = new List<String>[10]; // Not really allowed.   
Object o = lsa;    
Object[] oa = (Object[]) o; //数组类型也是Object的子类   

List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
//lsa[1]中实际存放的是li，lsa[1].get(0)返回为Integer
String s = lsa[1].get(0); // Run-time error: ClassCastException.

//使用通配符就可以
List<?>[] lsa = new List<?>[10]; // OK, array of unbounded wildcard type.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Correct.    
Integer i = (Integer) lsa[1].get(0); // OK 
```

##### 总结

1. 使用<T>来声明一个泛型T供泛型类或泛型方法使用，T可以是其他字母或字符串。
2. 通配符？是泛型实参，用来传入泛型类或泛型方法中。List<String>、List<Integer>可以赋值给List<?>，但是不能赋值给List<Object>，泛型没有继承。
3. 泛型不会存在于字节码中，只作用于编译阶段，这就是擦除，<T extends BaseType>在字节码中的类型替换为BaseType，<T>相当于<T extends Object>。
4. 泛型方法如 public  <T> T add(T x,T y) {...}，T的类型根据x和y共同决定，是x和y的最小父类。
5. Array不支持泛型，所以建议使用List来代替Array，List可以保证编译期的安全保证。




#### 反射

##### 静态加载与动态加载类

```java
public class Office {
    public static void main(String[] args) {
        //new是静态加载类，在编译时刻就需要加载所有可能用到的类
        //通过动态加载类可以解决该问题
        //如果没有Word或Excel类，就会编译报错
        if("Word".equals(args[0])) {
            Word word = new Word();
            word.start();
        }
        if("Excel".equals(args[0])) {
            Excel excel = new Excel();
            excel.start();
        }
    }
}


public class OfficeBetter {
    public static void main(String[] args) {
        //动态加载类，在运行时刻加载
        //编译不报错
        try{
            Class c = Class.forName(args[0])
            //创建对象
            OfficeAble oa = (OfficeAble)c.newInstance();
            oa.start();
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}

public interface OfficeAble {
    public void start();
}
//添加OfficeAble接口的实现后，只需编译该类，OfficeAble无需重新编译
public class Word implements OfficeAble {
    public void start() {
        System.out.println("Word start...");
    }
}
//Office
//Ppt
```

##### 获取类类型的三种方式

```java
Class c1 = Class.forName("java.lang.String");
String s = "hello";
Class c2 = s.getClass();
Class c3 = String.class;
```

每个类都是Class的实例，Class对象可以看作类的字节码。

##### 通过反射来认识泛型

```java
import java.lang.reflect.Method;
import java.util.ArrayList;

public class MethodDemo {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		
		ArrayList<String> list1 = new ArrayList<String>();
		list1.add("hello");
		//list1.add(20);错误的
		Class c1 = list.getClass();
		Class c2 = list1.getClass();
		System.out.println(c1 == c2);
		//反射的操作都是编译之后的操作
		
		/*
		 * c1==c2结果返回true说明编译之后集合的泛型是去泛型化的
		 * Java中集合的泛型，是防止错误输入的，只在编译阶段有效，
		 * 绕过编译就无效了
		 * 验证：我们可以通过方法的反射来操作，绕过编译
		 */
		try {
			Method m = c2.getMethod("add", Object.class);
			m.invoke(list1, 20);//绕过编译操作就绕过了泛型
			System.out.println(list1.size());
			System.out.println(list1);
			/*for (String string : list1) {
				System.out.println(string);
			}*///现在不能这样遍历
		} catch (Exception e) {
		  e.printStackTrace();
		}
	}
}
```

#### String





#### 内部类

```java
//编译后会生成三个class：C.class、C$InerClass.class、C$InerClass2.class
class C{
	
	public class InerClass{
		
	}
	
	private class InerClass2{
		
	}
}

//编译后会生成两个class：Test.class、Test$1.class
public class Test {
	
	public static void main(String[] args) {

		new Thread(new Runnable() {
			@Override
			public void run() {
				// TODO Auto-generated method stub				
			}			
		}).start();
	}
}
```

