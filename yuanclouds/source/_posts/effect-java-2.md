---
title: 小知识 - 类与接口篇
date: 2018-09-12 19:50:53
tags: EffectJava
---

# # 小知识 - 类与接口篇

## 类与对象的基础
### 类成员减少破坏性
软件基本原则之一：隐藏与封装。先查看下面的这段代码：

``` java
public static class Test{

  public static final String str[] = {"1","2"};

  public static String test() {
   	String string = "";
  	for(String s:str){
   	 string = string + s + " ";
  	}
  	return string;
  }
 }

 public static void main(final String[] args){
	 System.out.println(Test.test());
	 Test.str[1] = "3"; // 猜猜会不会报错？
	 System.out.println(Test.test());
 }
```
再看下执行结果：
```
1 2
1 3
```
这里final修饰的str数组，指的是str指向的地址是不能被改变的，但是内容是可变的。在java数组中，任何非零长度的数组都是可变的。公有访问方式的类成员即使被final进行修饰，但是完全避免不了可变类型的类成员被外部的破坏。所以这是所有开发者设计API需要考虑的一个工作之一，类似这种有两个解决方式：
** 1、对象拷贝 clone **
``` java

 // 1、将数组str[] 修改成私有进行保护
 private static final String str[] = {"1","2"};

 // 2、增加public提供方法，并且通过clone返回拷贝对象
 public static String[] getStrArray() {
		return str.clone();
 }

 // 3、验证
 System.out.println(Test.test());
 Test.getStrArray()[1] = "3";
 System.out.println(Test.test());
```

** 2、转换其他对象 **

``` java

  // 1、将数组str[] 修改成私有进行保护
  private static final String str[] = {"1","2"};

  // 2、定义转换类
  public class Model{

  	private String str[];
  	public Model(String[] data) {
	   		this.str = new String[data.length];
	   		for(int i =0;i<str.length;i++){
				this.str[i] = data[i];
	   		}
  	}

  	public static Model setStr(String[] data){
	  		Model model = new Model(data);
	   		return model;
  	}

  	public String[] getStr(){
   	   return str;
  	}

 }

	// 3、增加public提供方法
    public static String[] getStrArray() {
		return return Model.setStr(str).getStr();
    }

	// 4、验证
		System.out.println(Test.test());
		Test.getStrArray()[1] = "3";
		System.out.println(Test.test());

```
3、结果
通过以上两种修改，均可以达到保护效果，运行结果如下:
```
1 2
1 2
```


### 不可变的类
** 何为不可变得类？**
类成员不允许被修改、不能被继承、所用方法作用于都是私有的。同时不可变得类是不用考虑线程同步的问题，因为它们本身是不可变，相对于可变类更加设计、实现以及安全。设计不可变类，需要遵循下面五大原则：

1、不要提供任何会修改对象状态的方法
2、保证类不会被扩展。防止类被继承，构造方法需要是私有private的
3、使所有的域都是final的。所有类成员大多数作用于是final的，如果需要向外提供类成员属性，通过clone、对象转换进行传递
4、使所有的域都成为私有的。即使final修饰的成员变量(例如上面的str[]案例)，即可能通过公有get方法进行访问，避免set进行的状态改变
5、确保对于任何可变组件的互斥访问。对象引用必须保证唯一性

** 总结优缺点：**
优点：不可变类具有安全性，并保证是线程安全的
缺点：因为保证引用的唯一性，所以在访问类成员的时候，每个值都会保证是一个新的单独对象。

## 何为复合？
### 复合的定义
不用扩展现有的类，而是在新的类中增加一个私有域，它引用现有类的一个实例
### 复合由于继承
这里将会涉及几个定义: 接口、包装、继承
#### 举例为什么复合由于继承[参考]
目的：实现可以统计HasSet add操作的次数
思路：选择继承HasSet，通过重写add()、addAll()的方法计算count数量
``` java
public class MyHashSet<E> extends HashSet<E>{

 private int addCount;

 @Override
 public boolean add(E e) {
  addCount++;
  return super.add(e);
 }

 @Override
 public boolean addAll(Collection<? extends E> c) {
  addCount+=c.size();
  return super.addAll(c);
 }

 public int getAddCount() {
  return addCount;
 }

}
```
验证：
``` java
MyHashSet<String> hashSet = new MyHashSet<String>();
  hashSet.add("hello");
  List<String> data = new ArrayList<String>();
  data.add("hello1");
  data.add("hello2");
  hashSet.addAll(data);

  System.out.println("getCount " + hashSet.getAddCount()); // 预计打出3
```
结果：
``` java
getCount 5
```
分析：
这里结果显示是5的原因，我们可以先看下addAll()的源码：
``` java
 public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```
这里addAll调用了自己的成员方法add，所以在执行addall的时候，add统计会翻倍进行。如果在没了解源码的情况下，就会出现这种继承的弊端：
1、继承重写方法，除非不调用super,不可侵入父类方法
2、父类库版本变更，逻辑修改。子类较为被动弱化。

#### 复合的正确姿势
目的：实现可以统计HasSet add操作的次数
思路：HashSet一开始的设计，已经通过Set<E>提供基本的API接口，很大程度地提供了HashSet拓展性。因此我们可以通过包装的方式代理HashSet实现HashSet的基本接口，在包装类里面加入统计的逻辑，这便是复合的设计原理。

** "实现-代理-包装-业务拓展" **

``` java
     //1、实现Set<E>接口
    public class MyEffectHashSet<E> implements Set<E>{

	 //2、定义HashSet代理类
	private HashSet<E> deleagate = new HashSet<E>();
	private int addCount;

     // 省略其他实现接口...

    @Override
    //3、包装HashSet代理类，实现代理类
	public boolean add(E e) {
		addCount++; /4、实现业务拓展
		return deleagate.add(e);
	}


    @Override
    //3、包装HashSet代理类，实现代理类
	public boolean addAll(Collection<? extends E> c) {
		addCount+=c.size(); //4、实现业务拓展
		return deleagate.addAll(c);
	}

```
验证：
``` java
	MyEffectHashSet<String> hashSet = new MyEffectHashSet();
	hashSet.add("hello");
	List<String> data = new ArrayList<String>();
	data.add("hello1");
	data.add("hello2");
	hashSet.addAll(data);

	System.out.println("getCount " + hashSet.getAddCount());
```
结果:
``` java
getCount 3
```

总之，如果目标类的确与父类存在子父关系，则可以选择继承操作。但是如果并非子父关系，则继承将会因为父类的API导致继承后子类的脆弱性，因此建议选择复合实现。


## 重新认识接口
### 抽象类与接口怎么选择？
接触过java的同学都很清楚抽象类与接口，并且两者最大的区别是：抽象类有默认的实现方法，并且有构造方法；而接口只有对方法的声明，但是支持多继承。
由于正是java的设计，类的继承只能是单继承，所有很大程度地限制了类的拓展。因此接口的优势就体现在下面几个点：
#### 基类方法经常会进行更新
如果在基类A中，有以下继承关系
```
B extends A
C extends A
D extents B
```
如果基类A中作为抽象类，当新增一个抽象方法的时候，D类要想拥有这个抽象方法，那只能间接的也将B类修改成抽象类（或者是public的方法进行重写，但是失去了保护性）。因此这里违反了封装性。
借鉴于这种会经常更新的基类，所以更合理设计是基于接口。

#### 接口定义拥有mixin类型(不是很理解)
#### 接口允许定于非层次级类型架构
+ 层次级别类型架构
即是子父这种层级关系的类型。例如定义一个抽象的BaseActivity，他拥有抽象setTitle、setLayoutResId等方法，继承于BaseActivity的子类都有与父级BaseActivity的一些共性，他们两者直接是层级关系的。

+ 非层级类型架构
例如继承的Activity后，定义的可以实现缓存接口：IMemory内存缓存、IDisk磁盘缓存，A Activity需要内存缓存则实现IMemory , B Activity 需要磁盘缓存则实现IDisk , C Activity两种缓存则定义一个接口ISuperCache (继承于IMemory、IDisk)并实现它。这三者：IMemory  IDisk  ISuperCache都是平级关系的。

总之，在多继承的时候，接口就是救世主了。但是在一般的设计原则中，都是遵循骨架实现AbstractInterface的设计原则，即Abstract与Interface的混合使用。例如上面提供的案例HashSet，本质继承于AbstractSet，同AbstractSet实现了Set接口。在上面的复合案例正是体现了骨架实现的优美之处，即为子类提供了抽象类的实现上的帮助，但是又不极限于抽象类作为定义类型的限制。
``` java
public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
  //...
}
```
## 减少标签类
### 状态模式与策略模式
标签代表状态，如果了解过状态模式的同学，应该对标签类非常熟悉。
+ 举个案例的代码：
``` java
private static class NavAtion{

		private static final int IS_MEMBER = 1; // 表示会员
		private static final int IS_NOMAL = 0; // 表示普通用户

		// 判断用户是不是会员
		public static boolean isMember(int userid){
			///...
			if(getPermission(userid) == IS_MEMBER){
				return true;
			}
			return false;
		}

		// 获取用户权限
		private static int getPermission(int userId){
			//...
			return IS_NOMAL;
		}

		public static void goToVedioPage(){
			if (isMember(1233)) {
				// 大佬会员，免费看电影吧...
			}else{
				// 不是会员用啥用...
			}
		}

	}

```
代码中表示一些跳转电影页处理，里面通过判断是否是会员有相对应的跳转操作。NavAtion典型属于标签类，里面有IS_MEMBER、IS_NOMAL这些状态的判断，不同的状态又充斥着不同的业务逻辑。随着业务的增大，NavAtion类肯定会越来越臃肿，设置不好维护。如果状态的增加，无疑也多了更多的if...else...代码。
+ 状态模式优化：
1、抽象一个业务跳转接口 INavAtion
``` java
public interface INavAtion{
		void goToVedioPage();
	}
```
2、实现会员跳转业务
``` java
public class MemberNavAction implements INavAtion{

		@Override
		public void goToVedioPage() {
			// 大佬会员，免费看电影吧...
		}

	}
```
3、实现普通用户跳转业务
``` java
public class NomalNavAction implements INavAtion{

		@Override
		public void goToVedioPage() {
			// 不是会员用啥用...
		}

	}

```
与状态模式相似的还有策略模式，虽然代码文件增加，但是无疑加强了可维护性，将不同策略、不同状态的业务进行隔离。




