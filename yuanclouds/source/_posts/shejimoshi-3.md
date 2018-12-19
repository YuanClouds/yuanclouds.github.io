---
title: 设计模式---builder
date: 2018-09-25 23:20:11
tags: 设计模式
---

####1.什么是builder模式
   &nbsp;  &nbsp;简单解释，在程序设计的时候，如果面对的对象属性较多，对象复杂性比较大（例如对象包含对象）的情况，开发者选择这种设计模式去设计一个类的时候，开发者可以在不知道类内部结构构建细节的情况下，清晰地控制对象构造流程。例如，一个person类拥有很多属性的时候，最常见的比如name，age，weight，height等等。在这里两个地方体现出builder模式的优势：

（1）如果在创建一个person对象，只需要name、age属性，则需要创建多一个包含这两个属性的构造方法。需求不一样，构造方法就需要更多。累赘。
（2）对于普通的构造方法，当我们初始化weight、height两个属性的时候，我们一般会如下。
````
Person person = new Person("123","53");
````
如果属性一多，我们根本很模糊知道weight是哪个。

####2.模式适用范围
（1）当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式时。
（2）当构造过程必须允许被构造的对象有不同的表示时。
（3）Builder模式要解决的也正是这样的问题：
　　当我们要创建的对象很复杂的时候（通常是由很多其他的对象组合而成），
　　我们要复杂对象的创建过程和这个对象的表示（展示）分离开来，
　　这样做的好处就是通过一步步的进行复杂对象的构建，
　　由于在每一步的构造过程中可以引入参数，使得经过相同的步骤创建最后得到的对象的展示不一样。

####3.具体应用
笔者这里通过汽车Car类对象包含引擎Engine类、导航Naviaror类、车轮Wheel类作为一个复杂性类说明Builder模式的应用。

（1）公有抽象类Car
````

/**
 * 车的抽象类 较复杂的对象，对象包含对象
 * Created by wsy on 2016/2/17.
 */
public abstract class Car {

    protected Engine mEngine;
    protected Navigator mNavigator;
    protected Wheel mWheel;

    public Car() {
    }

    public Car(Engine mEngine, Navigator mNavigator, Wheel mWheel) {
        this.mEngine = mEngine;
        this.mNavigator = mNavigator;
        this.mWheel = mWheel;
    }

    /**
    public Car(Engine mEngine) {
        this.mEngine = mEngine;
    }

    public Car(Engine mEngine, Navigator mNavigator) {
        this.mEngine = mEngine;
        this.mNavigator = mNavigator;
    }
     **/

    public Engine getmEngine() {
        return mEngine;
    }

    public void setmEngine(Engine mEngine) {
        this.mEngine = mEngine;
    }

    public Navigator getmNavigator() {
        return mNavigator;
    }

    public void setmNavigator(Navigator mNavigator) {
        this.mNavigator = mNavigator;
    }

    public Wheel getmWheel() {
        return mWheel;
    }

    public void setmWheel(Wheel mWheel) {
        this.mWheel = mWheel;
    }

    @Override
    public String toString() {
        return "Car{" +
                "mEngine=" + mEngine +
                ", mNavigator=" + mNavigator +
                ", mWheel=" + mWheel +
                '}';
    }
}
````
（2） 具体实现奔驰类
````
/**
 * 奔驰实体类
 * Created by wsy on 2016/2/17.
 */
public class BenzCar extends Car {

    public BenzCar() {
    }

    public BenzCar(Engine mEngine, Navigator mNavigator, Wheel mWheel) {
        super(mEngine, mNavigator, mWheel);
    }

    /**
    public BenzCar(Engine mEngine) {
        super(mEngine);
    }

    public BenzCar(Engine mEngine, Navigator mNavigator) {
        super(mEngine, mNavigator);
    }
     **/

    // 这里如果重写了，直接通过this.mEngine，因为是父类projected类型，访问不到，因此日志打印会为空值，多态
//    @Override
//    public void setmEngine(String mEngine) {
//        mEngine = "Benz Engine 2016";
//    }
}
````
（3）Car车构建类Builder
````
/**
 * 奔驰构建类
 * Created by wsy on 2016/2/17.
 */
public class BenzBuilder{

    private Car mCar = new BenzCar();

    public BenzBuilder builderEngine(Engine mEngine) {
        mCar.setmEngine(mEngine);
        return this;
    }

    public BenzBuilder builderWheel(Wheel mWheel) {
        mCar.setmWheel(mWheel);
        return this;
    }

    @Override
    public Builder builderNavigator(Navigator mNavigator) {
        mCar.setmNavigator(mNavigator);
        return this;
    }

    public Car onCreate() {
        return mCar;
    }
}
````
（4）引擎Engine类、导航Naviaror类、车轮Wheel类（这里构建类builder都是静态内部类）
````
/**
 * 引擎实体类
 * Created by wsy on 2016/2/17.
 */
public class Engine {

    private String brand;
    private String power;

    public Engine(String brand, String power) {
        this.brand = brand;
        this.power = power;
    }

    public Engine() {
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public String getPower() {
        return power;
    }

    public void setPower(String power) {
        this.power = power;
    }

    @Override
    public String toString() {
        return "Engine{" +
                "brand='" + brand + '\'' +
                ", power='" + power + '\'' +
                '}';
    }

    public static class EngineBuilder{

        private Engine mEngine = new Engine();
        private String brand;
        private String power;

        public EngineBuilder setBrand(String brand) {
            mEngine.setBrand(brand);
          return this;
        }

        public EngineBuilder setPower(String power) {
            mEngine.setPower(power);
            return this;
        }

        public Engine create(){
            return mEngine;
        }

    }

}
````
````
package cn.wsy.builder.simple.common;

/**
 * 导航仪实体类
 * Created by wsy on 2016/2/17.
 */
public class Navigator {

    private String brand;
    private String price;

    public Navigator(String brand, String price) {
        this.brand = brand;
        this.price = price;
    }

    public Navigator() {
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public String getPrice() {
        return price;
    }

    public void setPrice(String price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Navigator{" +
                "brand='" + brand + '\'' +
                ", price='" + price + '\'' +
                '}';
    }

    public static class NavigatorBuilder {
        private Navigator mNavigator = new Navigator();
        private String brand;
        private String price;

        public NavigatorBuilder setPrice(String price) {
            mNavigator.setPrice(price);
            return this;
        }

        public NavigatorBuilder setBrand(String brand) {
            mNavigator.setBrand(brand);
            return this;
        }

        public Navigator create(){
            return mNavigator;
        }

    }

}
````
````
package cn.wsy.builder.simple.common;

/**
 * 车轮实体类
 * Created by wsy on 2016/2/17.
 */
public class Wheel {

    private String brand;
    private String weight;

    public Wheel() {
    }

    public Wheel(String brand, String weight) {
        this.brand = brand;
        this.weight = weight;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public String getWeight() {
        return weight;
    }

    public void setWeight(String weight) {
        this.weight = weight;
    }

    @Override
    public String toString() {
        return "Wheel{" +
                "brand='" + brand + '\'' +
                ", weight='" + weight + '\'' +
                '}';
    }

    public static class WheelBuilder{
        private Wheel mWhell = new Wheel();
        private String brand;
        private String weight;

        public WheelBuilder setBrand(String brand) {
            mWhell.setBrand(brand);
           return this;
        }

        public WheelBuilder setWeight(String weight) {
            mWhell.setWeight(weight);
            return this;
        }

        public Wheel create(){
            return mWhell;
        }
    }
}
````
（4）测试方法
    这里我们通过对比传统构造方法来对比Builder构造的优势:
1.如果类奔驰车类构造对象的时候不需要导航仪类，传统的构造方法需要定义多一个构造方法，具体如下：
````
 public BenzCar(Engine mEngine, Wheel mWheel) {
           super(mEngine, mWheel);
 }
````
构造如下：
````
BenzCar car1 = new BenzCar(new Engine("321","321"),new Wheel("123","123"));
````
2.利用builder模式构造
````
//这种方式创建 可观性清晰 构造器也可以减少
BenzBuilder builder = new BenzBuilder();//oncreate 可以拿到实体对象
BenzCar car = (BenzCar) builder
                       .builderEngine(new Engine.EngineBuilder().setBrand("brand").create())
                       .builderNavigator(new Navigator.NavigatorBuilder().setPrice("price").create())
                       .onCreate()
                      ;Log.i(TAG, car.toString());
````

 这样是不是更加清晰！需求改动、最初类属性没有修改的时候，可以直接不用修改类的构造方法，维护性更高！但是这种方式会严重增加的代码量，所以各有优缺点！！