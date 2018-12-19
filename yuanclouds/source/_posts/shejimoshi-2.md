---
title: 设计模式---Strategy
date: 2018-09-22 19:32:11
tags: 设计模式
---


####1.什么是Strategy模式（策略模式）
&nbsp;&nbsp;&nbsp;策略模式，举个现实应用开发的例子：在我们的程序设计中，如果需要提供多种排序算法（冒泡排序、二分法排序、归并排序、插入排序等等），有一种想法我们可以将所有排序方法作为静态方法统一封装到一个类里面，当我们要使用具体哪个方法的时候通过传入标志位，通过if...else...这种形式确定判断调用哪个排序算法。

&nbsp;&nbsp;&nbsp;对于这种方法，优点在于只需要一个类文件即可以解决调用问题。缺点在于通过if..else..这种判断方法属于硬编码实现方式，如果在该类中存在多种排序算法，则会造成臃肿的现象，可维护性差，并且当我们要新增一个排序算法的时候，我们要重新修改算法类的源代码，明显也违法了我们程序设计的ocp原则和单一职责原则。

&nbsp;&nbsp;&nbsp;<b>解决思路</b>：如果我们将每个算法策略抽象出来，提供一个当独的接口，不同的算法有不同的实现，我们在调用的时候只需要注入不同的对象即可以实现算法，这样的可维护性便大大增强，因而出现了今天要讲的策略模式。

####2.该模式适应范围
（一）针对同一类型问题有多种处理方式，具体行为也有差别的时候
（二）需要安全地封装多种同一类型的操作时
（三）出现同一抽象类有多个子类，有需要通过if..else..或switch..case选择具体子类时

####3.具体应用
&nbsp;&nbsp;&nbsp;在交通管理局中，一般不同的交通工具有不同的乘车价格，同种交通工具根据路程不同又有不同的乘车价格，因此笔者在这里举这个现实中的例子作为应用案例来解说。

（一）一般将所有情况封装到一个类中，通过静态方法直接引用的时候
````
    public void strategyMode_tradition_Test(int tranFlag,int km ){
        //车类型标志
//        int tranFlag = 0;//1 巴士 2的士 3地铁
//        int km = 25;//行走路程
        if (tranFlag == 1){
            if (km < 10){
                ///..一种价格
            }else if (km>10 && km<25){
                //..一种价格
            }//else if...
        }else if(tranFlag == 2){
            if (km < 3){
                ///..一种价格
            }else if (km>3 && km<20){
                //..一种价格
            }//else if...
        }else if(tranFlag == 3){
            if (km < 15){
                ///..一种价格
            }else if (km>15 && km<40){
                //..一种价格
            }//else if...
        }

    }
````
&nbsp;&nbsp;&nbsp;这里我们明显能看到，如果条件越多，会造成这个方法，整个类更加臃肿，如果我们要添加一种新的交通工具的时候，也需要重新回到类源文件中修改。

（二） 利用策略模式实现
1.首先是定义一个抽象价格计算接口
````
/**
 * Created by wsy on 2016/2/19.
 */
public interface CalculateUtil {
    public int calculating(int km);
}
````

2.分别抽象出不同交通工具的算法
````
package cn.wsy.mymode.strategyMode;

import android.util.Log;

/**
 * 巴士
 * Created by wsy on 2016/2/19.
 */
public class BusCalculate implements CalculateUtil{

    private final String TAG = this.getClass().getName();
    @Override
    public int calculating(int km) {
        Log.i(TAG,"巴士行程 "+km+"公里,价格为**");
        //复杂计算..
        return 0;
    }
}
````
````
package cn.wsy.mymode.strategyMode;

import android.util.Log;

/**
 * 地铁
 * Created by wsy on 2016/2/19.
 */
public class SubwayCalculate implements CalculateUtil{

    private final String TAG = this.getClass().getName();
    @Override
    public int calculating(int km) {
        Log.i(TAG, "地铁行程 " + km + "公里,价格为**");
        //复杂计算..
        return 0;
    }
}
````

````
package cn.wsy.mymode.strategyMode;

import android.util.Log;

/**
 * 的士
 * Created by wsy on 2016/2/19.
 */
public class TaxiCalculatie implements CalculateUtil {

    private final String TAG = this.getClass().getName();

    @Override
    public int calculating(int km) {
        Log.i(TAG, "的士行程 " + km + "公里,价格为**");
        //复杂计算..
        return 0;
    }
}
````

3.具体抽象价格算法控制类
````
/**
 * 交通工具价格控制类
 * Created by wsy on 2016/2/19.
 */
public class TransportControl {

    private CalculateUtil calculateUtil;

    public TransportControl(CalculateUtil calculateUtil) {
        this.calculateUtil = calculateUtil;
    }

    public int calculating(int km){
        return calculateUtil.calculating(km);
    }

}

````
4.测试方法
````
 public void strategyModeTest(){
        TransportControl transportSub = new TransportControl(new SubwayCalculate());
        transportSub.calculating(25);

        TransportControl transportBus = new TransportControl(new BusCalculate());
        transportBus.calculating(55);
    }
````

总之，从我们上面的利用策略模式与直接封装类的测试方法中，我们能观察到策略模式带来的有点是更直观、更清晰将每个算法职责单一起来。在案例应用程序中，我们只需要在定义TransportControl对象的时候，通过注入不同的对象，就可以引用不同的算法。对比与直接使用if...else..硬编码形式，更加简洁。








