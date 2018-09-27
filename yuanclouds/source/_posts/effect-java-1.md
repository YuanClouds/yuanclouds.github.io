---
title: 小知识 - 灵活对象的构建与销毁篇
date: 2018-09-12 19:50:50
tags: 一些比较少见的java优化
top_img: http://d.5857.com/mxr_170206/001.jpg
---

# # 小知识 - 灵活对象的构建与销毁篇
## 一、静态方法替代构造方法
### 优点
#### （一）表达更加清晰
总所周知，java中的构造方法是允许定义多个的。当一个对象具有不同特征属性的时候，需要构造不同的属性对象的方法有两种。
+ 提供多个不同对象属性的构造方法
+ 提供set方法，开发者进行实例化时针对性进行需要的set
其中，提供不同的对象属性构造方法无疑会造成使用的开发者理解不清晰，举下下面的例子：
``` java
     /** 假如一个猫cat类，有一是否有斑纹的属性（默认没有）**/
    public class Cat{
        private boolean hasStripe;
        private String name;
        ...
        public Cat(String name){
            // ...
         }

        public Cat(String name,boolean hasStripe){
            // ...
        }
    }
```
如果有更多的特殊属性，那无疑就是会增加更多表达模糊的构造方法。如果借鉴工厂方法的思路，也许会不会更清晰点？例如：
``` java
    public class Cat{    
        //,,,
        public static Cat getHasStripeCat(){
                // 实例有虎纹的猫...
         }
          public static Cat getNormalCat(){
                //. 实例默认的猫..
          }
    }
```
当然有更加推荐的办法去解决这种多构造方法造成表达模糊的方法，即设计模式Builder模式。其中在android源码、其他依赖库原来越多利用这种构造模式，灵活又具有针对性

#### （二）灵活控制创建的对象(从实例化的角度)
先看下这句源码：Boolean.valueOf(***) ，如下：

``` java
public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
 }
```
valueOf是一个静态方法（替换构造方法提供实例化对象实例），TRUE与FALSE是Boolean类中的属性Boolean对象，因此利用静态方法替换构造方法一个优势就是可以动态得返回实例化的对象，例如Boolean的valueOf方法，这样即可以循环利用Boolean实例对象，又可以在大对象时new造成的内存的浪费

#### （三）灵活控制返回对象(从继承的角度)
面向对象开发的较大的优势-“继承”。父类可以代表其下所有的子类引用，基于这点静态方法有一个较大的优势，例如A为一个定义的基类，B是继承于A的子类，C也是继承于A的子类。

### 缺点
如果类中实例化对象只有通过静态工厂方法而没有自身的构造方法，则这个类将会失去子类化。即静态工厂方法无法被子类进行继承

## 二、及时销毁不引用的对象
回想起好几年前写c++，再遇到java的时候，jvn的垃圾回收器简直就是女神。虽然有了垃圾回收机制，大大的减少手动释放内存空间的工作量。但是并不是刻意"完全"不用管理内存，垃圾回收器也会遇到内存泄漏问题。
引用案例的代码：

``` java 
public class Stack {
    /** 某一堆栈管理类，Push表示入栈，pop表示出栈**/
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
}
```
这类问题比较特殊，由于栈内部会维护对象的过期引用（过期引用：用于不会再解除引用）。在方法pop（）中只是返回了elements[i]的对象，但是实际elements依然是持有这个对象的引用的。因此这个对象将不能被正常的回收。优化如下：

``` java
public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
```

## 三、适当维护程序的对象池
new Object() 的时候会创建实例对象，如果在程序运行期间频繁地进行实例操作，也许会带来内存抖动。特别是重量级的对象，会引发一定的性能问题。
Boolean 内部就典型地维护了个对象轻量级的对象池，如下：
``` java
 /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code true}.
     */
    public static final Boolean TRUE = new Boolean(true);

    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code false}.
     */
    public static final Boolean FALSE = new Boolean(false);
```
其中在 valueOf 的时候会直接饮用这个对象池实例。valueOf 本身是作为转化对象而不是创建对象，因此通过对象池进行对象的转换。即减少了内存空间，又实现了目的。

## 四、finalize 终结方法
finalize 可以理解成C++的析构函数。在讲finalize的时候，先举例有内存泄漏隐患的例子：
``` java
public class TranslateTool {
    
    private ITranslateTool mITranslateTool;
    
    public interface ITranslateTool{
        void onResult(String result);
    }

    public TranslateTool() {
    }

    public void doing(String text, ITranslateTool iTranslateTool){
        this.mITranslateTool = iTranslateTool;
        //...异步操作 begin
        iTranslateTool.onResult("");
        //...异步操作 end
    }

    public void onDestory(){
        mITranslateTool = null;
    }
}
```
TranslateTool是一个翻译工具类，调用者通过调用doing进行异步翻译工作。首先mITranslateTool是外部持有的一个监听回调器，假如在TranslateTool翻译异步工作中（mITranslateTool由Activity实现持有），这时候在销毁Activity的时候如果忘记调用TranslateTool的 onDestory 方法。此时mITranslateTool的生命周期将会比Activity长，所以将会引起Activity不能被及时释放导致内存泄漏。

#### **为什么要说什么的例子呢？**
很多时候代码自己写的，但是维护不一定是你自己。所以这里就很容易出现忘记调用onDestory解除Activity的引用。这里onDestory释放属于显示调用。那么有没有办法可以隐式调用释放解除Activity的引用呢？答案就是 终结方法 finalize。

#### **finalize**
finalize 有个好处，为了避免开发者忘记显示调用释放方法而提供一个"安全网"，但是并不能保证finalize是及时调用的。原因在于如果是一些需要及时关闭的场景，例如stream、cusor、file等，不能完全依赖finalize，而是需要尝试try...catch...finally..

如果类中实现了自己的finalize方法，通过继承该类中，如果忘记调用了父类的finalize，依然会出现上面的问题，如下：

``` java
public class Parent {
        @Override
    protected void finalize() throws Throwable {
            Log.d("siven","父类终结...");
            super.finalize();
        }
}

public class Child extends Parent{
        @Override
        protected void finalize() throws Throwable {
                Log.d("siven","子类终结");
               // super.finalize(); // 忘记调用父类了
        }
}
```
**如何解决？**
Parent可以定义一个对象属性，用于决定Parent的一些引用释放，从而解除了必须由子类要进行supper.finalize()的必要性：

``` java
public class Parent {

    private final Object finalizerGuardian = new Object(){
        @Override
        protected void finalize() throws Throwable {
            onDestory();
            super.finalize();
        }
    };

   public void onDestory(){
       // 移除
   }
}
```
finalizerGuardian属于私有对象属性，因此引用只有被该类的实例对象引用，生命周期与该类的实例对象一致。因此子类的finalize方法是否要调用super.finalize不是必要的问题了。