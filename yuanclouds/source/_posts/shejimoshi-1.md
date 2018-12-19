---
title: 设计模式---StateMode
date: 2018-09-20 20:40:37
tags: 设计模式
---

####一、什么是StateMode（状态模式）
&nbsp;&nbsp;&nbsp;状态模式与上一篇写的[《[设计模式---Strategy模式](http://www.jianshu.com/p/a94376693afc)》](http://www.jianshu.com/p/a94376693afc)具体实现结构有点相似，两者之间都是通过注入不同的子对象得到不同的操作行为，但是两者的实现目的完全不同。例如，状态模式的行为是平行，我们可以直接通过注入不同的状态子类对象从而获得不同状态的操作，并且状态是可以切换的。而策略模式，则是通过注入不同的操作对象从而获得不同的操作行为，不同的注入方式直接，他们的行为是独立的。下面做一个区分：

(一)区别：
&nbsp;&nbsp;&nbsp;状态模式将各个状态所对应的操作分离开来，即对于不同的状态，通过注入状态，由不同的子类实现具体操作，不同状态的切换由子类实现，当发现传入参数不是自己这个状态所对应的参数，则自己给Context类切换状态；而策略模式是直接依赖注入到Context类的参数进行选择策略，不存在切换状态的操作。

(二)联系：
&nbsp;&nbsp;&nbsp;状态模式和策略模式都是为具有多种可能情形设计的模式，把不同的处理情形抽象为一个相同的接口，符合对扩展开放，对修改封闭的原则。

总而言之！两者就像孪生兄弟！！！

####二、该模式适应范围
（一）针对同一类型问题有多种处理方式，具体行为也有差别的时候
（二）需要安全地封装多种同一类型的操作时
（三）出现同一抽象类有多个子类，有需要通过if..else..或switch..case选择具体子类时

####三、具体应用
&nbsp;&nbsp;&nbsp;笔者对于这个模式比较有感触，这里先用电视机根据关机与开机两者不同状态，遥控器有不同操作行为作为案例分析。其次在通过我们实战中，对于现在有一些APP应用，不一定强制性要求登录才可以进行某些操作，因此则出现登录与非登录状态的时候会出现不同的状态，所以自然我们可以使用----状态模式！！！

一、电视机应用---状态模式
1.首先定义电视机操作接口
````
package cn.wsy.mymode.stateMode;

/**
 * 电视机操作
 * Created by wsy on 2016/2/23.
 */
public interface TvStateControl {

    public void upVolume();

    public void downVolume();

    public void nextChannel();

    public void beforeChannel();

}
````
2.不同状态之间的电视机操作（开机、关机）
````
package cn.wsy.mymode.stateMode;

import android.util.Log;

/**
 * Created by wsy on 2016/2/23.
 */
public class PowerOnState implements TvStateControl {

    private final String TAG = "PowerOnState";

    @Override
    public void upVolume() {
        Log.i(TAG,"音量提高中..,");
    }

    @Override
    public void downVolume() {
        Log.i(TAG,"音量降低中..,");
    }

    @Override
    public void nextChannel() {
        Log.i(TAG,"下一个频道..,");
    }

    @Override
    public void beforeChannel() {
        Log.i(TAG,"上一个频道..,");
    }
}
````
````
package cn.wsy.mymode.stateMode;

import android.util.Log;

/**
 * 关机的操作
 * Created by wsy on 2016/2/23.
 */
public class PowerOffState implements TvStateControl{

    private final String TAG = "PowerOffState";

    @Override
    public void upVolume() {
        Log.i(TAG,"关机咯");
    }

    @Override
    public void downVolume() {
        Log.i(TAG,"关机咯");
    }

    @Override
    public void nextChannel() {
        Log.i(TAG,"关机咯");
    }

    @Override
    public void beforeChannel() {
        Log.i(TAG,"关机咯");
    }
}
````
3.电视中遥控控制类
````
package cn.wsy.mymode.stateMode;

/**
 * 电视机控制类
 * Created by wsy on 2016/2/23.
 */
public class TvControl {

    TvStateControl stateControl;

    public TvControl(TvStateControl stateControl) {
        this.stateControl = stateControl;
    }

    /**
     * 这里作为切换状态接口
     * @param stateControl
     */
    public void setStateControl(TvStateControl stateControl) {
        this.stateControl = stateControl;
    }

    public void upVolume() {
        stateControl.upVolume();
    }

    public void downVolume() {
        stateControl.downVolume();
    }

    public void nextChannel() {
        stateControl.nextChannel();
    }

    public void beforeChannel() {
        stateControl.beforeChannel();
    }
}
````
4.测试
&nbsp;&nbsp;&nbsp;如果我们不采用模式设计程序，一般我们设计思路会是if...else...硬编码的判断模式，例如以下代码：
````
    public void traditionalTest(int tvState) {
        //int tvState = 0;//0 开机 1关机
        //调声音
        upVolume(tvState);
        //调频道
        nextChannel(tvState);
    }

 //传统
    public void upVolume(int state) {
        if (state == 0) {
            Log.i(TAG, "音量提高中..,");
        }else{
            Log.i(TAG, "关机啦..,");
        }
    }

    public void downVolume(int state) {
        if (state == 0) {
            Log.i(TAG,"音量降低中..,");
        }else{
            Log.i(TAG, "关机啦..,");
        }
    }

    public void nextChannel(int state) {
        if (state == 0) {
            Log.i(TAG,"下一个频道..,");
        }else{
            Log.i(TAG, "关机啦..,");
        }
    }

    public void beforeChannel(int state) {
        if (state == 0) {
            Log.i(TAG,"上一个频道..,");
        }else{
            Log.i(TAG, "关机啦..,");
        }
    }
````
&nbsp;&nbsp;&nbsp;这种方式，会增加更多if..else重复代码，可维护性也不高，如果状态增加，我们将要去每个涉及到状态判断的类文件去修改对应逻辑，这样同样违反了程序设计的封闭原则，不推荐！
&nbsp;&nbsp;&nbsp;相反，利用状态模式的实现代码，如下：
````
   public void modeTest() {
        //注入状态操作
        TvControl tvControl = new TvControl(new PowerOnState());
        //调声音 开机
        tvControl.upVolume();
        //关机 再操作
        tvControl.setStateControl(new PowerOffState());
        tvControl.nextChannel();
    }
````
结果：
![](http://upload-images.jianshu.io/upload_images/2516602-e93c5fc9c62c96ad?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&nbsp;&nbsp;&nbsp;相反，这种模式不用硬生生利用if..else..等判断语句去处理，再切换状态的时候，我们只需要通过注入新的状态子对象，即可以获取不同的操作方式！如果需求增加状态属性，我们也不需要直接去修改判断语句，只需要重新增加状态子对象，再次注入即可！！！大大增加程序维护性！

####二、实战应用（登录与非登录行为）---状态模式
&nbsp;&nbsp;&nbsp;APP部分功能。这样，便会出现登录与非登录两种状态会出现不同的行为，因此这里便可以联想到这篇博客学习的----状态模式！！！下面举出简单例子（这里约定要登录才能进入个人中心、进行评论，进行转发，否则会自动进入登录界面。）
1.涉及到登录的所有行为 接口
````
package cn.wsy.mymode.stateMode.modeForLogin;

/**
 * 涉及到登录操作的行为动作
 * Created by wsy on 2016/2/23.
 */
public interface LoginState {

    /**
     * 进入个人中心
     */
    public void toPersonalCenter();

    /**
     * 进行评论
     */
    public void commenting();

    /**
     * 进行转发
     */
    public void transpondMsg();

}
````
2.不同状态的操作（登录与非登录）
````
package cn.wsy.mymode.stateMode.modeForLogin;

import android.util.Log;

/**
 * 已经登录
 * Created by wsy on 2016/2/23.
 */
public class LoginedState implements LoginState{

    private final String TAG =  "LoginedState";

    @Override
    public void toPersonalCenter() {
        Log.i(TAG,"进入个人中心界面");
    }

    @Override
    public void commenting() {
        Log.i(TAG,"进行评论");
    }

    @Override
    public void transpondMsg() {
        Log.i(TAG,"进行转发");
    }

}
````

````
package cn.wsy.mymode.stateMode.modeForLogin;

import android.util.Log;

/**
 * 登出操作
 * Created by wsy on 2016/2/23.
 */
public class LoginOutState implements LoginState{

    private final String TAG =  "LoginedState";

    @Override
    public void toPersonalCenter() {
        Log.i(TAG,"进入个人中心失败，进入登录界面");
    }

    @Override
    public void commenting() {
        Log.i(TAG,"进行评论失败，进入登录界面");
    }

    @Override
    public void transpondMsg() {
        Log.i(TAG,"进行转发失败,进入登录界面");

````
3.登录状态行为操作静态控制类
````
/**
 * 登录控制类
 * Created by wsy on 2016/2/23.
 */
public class LoginControl {

    LoginState loginedState;

    public LoginControl(LoginState loginedState) {
        this.loginedState = loginedState;
    }

    public void setLoginedState(LoginState loginedState) {
        this.loginedState = loginedState;
    }

    public void toPersonalCenter() {
        loginedState.toPersonalCenter();
    }

    public void commenting() {
        loginedState.commenting();
    }

    public void transpondMsg() {
        loginedState.transpondMsg();
    }
}
````

4.测试结果
````
 @Override
    public void modeTest() {
        LoginControl loginControl = new LoginControl(new LoginOutState());
        //默认没有登录
        loginControl.toPersonalCenter();
        //登录后 再操作
        loginControl.setLoginedState(new LoginedState());
        loginControl.toPersonalCenter();
    }
````

