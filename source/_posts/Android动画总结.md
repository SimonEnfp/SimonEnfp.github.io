---
title: Android动画总结
date: 2017-3-20 16:28:08
tags: android动画
categories: Android
---


**android动画可以分为三种：View动画，帧动画，属性动画。**


----------


 - ***View动画（Animation）***四种效果：动画的XML文件路径为：res/anim/filename.xml
     - TranlateAnimation（标签<`translate`>）
     - ScaleAnimation（标签<`scale`>）
     - RotateAnimation（标签<`rotate`>）
     - AlphaAnimation（标签<`alpha`>）
 - View动画 可以通过AnimationListener设置动画监听：
     - onAnimationStart、onAnimationEnd、onAnimationRepeat；
 - View动画自定义只需继承Animation，重写它的initialize和applyTransformation（进行矩阵变换）方法，很少用；
 - View动画特殊使用场景：
  - LayoutAnimation作用于ViewGroup，为Viewgroup指定一个动画，当它的子元素出场时都会有这种动画效果，常常被用于ListView上；
  - Activity有默认的切换效果，但是这个效果我们可以自定义的，主要用到overridePendingTransition（int enterAnim，int exitAnim）这个方法，这个方法必须再startActivity（Intent）或者finish之后被调用才能生效；
  


----------


 - ***帧动画（AnimationDrawable）***类似电影播放，顺序播放一组预先定义好的图片；


----------


 - ***属性动画***  是API 11新加入的特性（在此之前可以采用开源动画库nineoldandroids来兼容以前版本网址：*http://nineldandroids.com*），和View动画不同，它对作用对象进行了扩展，属性动画可以对任何对象做动画，甚至可以没有对象。（属性动画中ValueAnimator、ObjectAnimator，AnimatorSet等概念）一般通过代码动态创建属性动画；
 - **时间插值器（TimeInterpolator）**：
    - 它的作用是根据时间流逝的百分比来计算出当前属性值改变的百分比，系统预置的有LinearInterpolator（线性插值器：匀速动画）、AccelerateDecelerateInterpolator（加速减速插值器：动画起始慢中间快）和DecelerateInterpolator（减速插值器：动画越来越慢）等。
 - **估值器（TypeEvaluator）**：
  - 它的作用是根据当前属性改变的百分比来计算改变后的属性值，系统预置的有IntEvaluator（针对整型属性）、FloatEvaluator（针对浮点型属性）和ArbgEvaluator（针对Color属性）。
 - 属性动画的监听器，主要有两个接口：AnimatorUpdateListener和AnimatorListener；
    - **AnimatorListener**可以监听动画开始、结束、取消以及重复播放。

 ```java
    public static interface AnimatorListener {
        void onAnimationStart(Animator animation);
        void onAnimationEnd(Animator animation);
        void onAnimationCancel(Animator animation);
        void onAnimationRepeat(Animator animation);
    }
 ```
    - **AnimatorUpdateListener**会监听整个动画过程，动画是由许多帧构成的，每播放一帧（动画默认刷新率10ms/帧），onAnimationUpdate就会被调用一次。
 ```java
public static interface AnimatorUpdateListener {
    void onAnimationUpdate(ValueAnimator animation);
}
 ```
 - **属性动画原理**：属性动画要求动画作用的对象提供该属性的get和set方法，属性动画根据外界传递的该属性的初始值和最终值，以动画的效果多次去调用set方法，每次传递给set方法的值都不一样，确切的来说是随着时间的推移，所传递的值越来越接近最终值。因此，如果对某个对象做属性动画，需满足：
- 对象必须要提供set方法，如果动画的时候没有传递初始值，那么还要提供get方法，因为系统要去取该属性的初始值，否则，程序会Crash；
- 对象的set方法对属性所做的改变必须能通过某种方法反映出来，比如会带来UI改变之类的，否则动画会无效果。
 - 如果对象不满足上述两点，可以通过下面三种方法解决：
     1. 给你的对象加上get和set方法，当然你得有权限；
     2. 如果没有权限，你可以用一个包装类来包装原始对象，间接为其提供get和set方法；
     3. 采用ValueAnimator，监听动画过程，自己实现属性的改变。


----------


 - **使用动画需要注意的事情**：
    - OOM问题，主要出现在帧动画中，当图片数量较多且较大时极易出现OOM；
    - 内存泄漏问题，在属性动画中有一类无限循环的动画，这类动画需要在Activity退出时及时停止，否则将导致Activity无法释放从而造成内存泄漏，通过验证后View动画并不存在此问题。
    - 兼容性问题，动画在3.0以下的系统上有兼容性问题，在某些特殊场景下可能无法正常工作，因此做好适配工作很重要。
    - View动画问题，View动画是对View的影像做动画，并不是真正的改变View的状态，因此会出现动画完成后View无法隐藏的现象，即setVisibility（View.GONE）失效，这个时候只要调用view.clearAnimarion()清除View动画即可。
    - 动画元素的交互，将View移动后，在Android3.0以前的系统上，不管是View动画还是属性动画（因为3.0以前属性动画原理仍然是View动画），新位置均无法触发单击事件，同时，老位置仍然可以触发。尽管View已经在视觉上不存在了，将View移回原位置后，原位置的单机事件继续生效。从3.0以后，属性动画的单击事件触发位置为移动后的位置，但View动画仍在原来位置。
    - 使用动画过程中建议开启硬件加速，可以提高动画的流畅性。


----------


**参考书目**：***Android开发艺术探索***


