---
title: Activity与Fragment交互重构
date: 2017-02-09 12:48:08
tags: activity fragment
categories: Android
---
场景：一个Activity中含若干fragment，fragment调用Activity中方法来与其他fragment通信。
通常如果fragment调用activity的方法是将activity中方法公有化，在fragment中直接获取activity实例来调用其中的方法，但是这样做起来程序耦合性过高，也有违设计模式的开闭原则，如果fragment过多，activity也会比较混乱，这几天重构代码，试了下接口回调的方法，感觉还不错。

 - 把fragment调用activity中的方法封装到一个统一的接口中去
```java
 public interface Interaction {
//  fragment invoke
    void methodOne();
    void methodTwo();
}
```
 - activity实现这个接口
```java
public class MainActivity implements Interaction{
    @Override
    public void methodOne() {
    }
     @Override
    public void methodTwo() {
    }
}
```
 - 在fragment中可以在onAttach获取这个接口实例
```java
private Interaction interaction;
    
    private Context mContext;

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        try {
            interaction = (Interaction) context;
        }catch (ClassCastException e) {
            throw new ClassCastException(mContext.getClass().getName()
                    +" must implements interface Interaction");
        }
    }

    }
 ```
 
这样整个交互过程就比较清晰了，代码案例很简单，但封装重构思想可以多加利用，提升代码质量
