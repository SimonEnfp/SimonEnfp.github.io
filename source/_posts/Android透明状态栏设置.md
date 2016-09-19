---
title: Android透明状态栏设置
date: 2016-06-27 11:23:10
tags: 状态栏
categories: Android
---
应公司需求，为了让app界面在4.4版本之后的客户端更加美观和谐，欲使手机顶部状态栏(statusBar)与app主题色保持一致，所以想让statusBar保持透明，显示UI主题背景。

 - 

适用于Android KITKAT(API 19) 之后

 - 
使用主题`Theme.Design.NoActionBar`，这里是自定义的Actionbar。
 
*先上代码*
 
-----
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT 
            && Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP){
            Window window = activity.getWindow();

    //      set transparent status and let main layout extend to status
            window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
            
    //      set transparent navigation and let main layout extend to navigation
    //      window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
 
            ViewGroup mContentView = (ViewGroup) activity.findViewById(Window.ID_ANDROID_CONTENT);
    //      set contentView rootView padding in order to set aside the location of the power strip
            View rootView = mContentView.getChildAt(0);
            if (rootView != null){
    //          rootView.setFitsSystemWindows(true);
                rootView.setPadding(0,getStatusBarHeight(activity),0,0);
            }
        }
        
----------

 


#### 1. 在API 19 ~ API 21之间：
window增加这个Flag，[FLAG_TRANSLUCENT_STATUS][1]，作用是使系统状态栏背景透明，这时候会发现我们的ContentView布局延伸到了手机顶部被状态栏覆盖，如图:
![此处输入图片的描述][2]
这样显然是不行的，我们自己的UI被挡住，所以需要下面的代码:

		//rootView.setFitsSystemWindows(true);
        rootView.setPadding(0,getStatusBarHeight(activity),0,0);
这里有两种方法:第一种是根布局设置[FitsSystemWindows][3]为ture，Android官方文档是这样解释的：

***Boolean internal attribute to adjust view layout based on system windows such as the status bar. If true, adjusts the padding of this view to leave space for the system windows.***
我的理解是如果设置为true，那就调整这个布局的padding，为系统窗口留出空间，这里指的就是顶部的statusBar和底部的Navigationbar，但是我的需求是仅仅让顶部预留空间即可，底部保持系统样式，所以，根据这个属性的原理，可以有另一种设置方法：

    `rootView.setPadding(0,getStatusBarHeight(activity),0,0);`
这里直接为自己的根布局设置一个与statusBar相同高度的padding即可。
完成之后，效果如下：
![此处输入图片的描述][4]

#### 2. 在API 21 以上：

>           window.getDecorView().setSystemUiVisibility(
>                       View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
>                     | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
>             window.setStatusBarColor(Color.TRANSPARENT);

这里第一步是通过DecorView设置其属性SystemUiVisibility，SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN来使自己的根布局充满全屏，第二部是通过window在API 21之后新增属性StatusBarColor，设置statusbar透明，后面再使根布局为系统窗口留出Padding与API 19方法同理，无需多说


----------


**具体源码在这里查看：**
[**https://github.com/simonenfp/007-A**][5]


  [1]: https://developer.android.com/reference/android/view/WindowManager.LayoutParams.html#FLAG_TRANSLUCENT_STATUS
  [2]: http://o6w24t5lg.bkt.clouddn.com/statusBar1.png?%09%20watermark/2/text/U2ltb25FbmZwJ3MgYmxvZw==/font/5a6L5L2T/fontsize/500/fill/IzBFQUZFMg==/dissolve/99/gravity/NorthEast/dx/10/dy/10
  [3]: https://developer.android.com/reference/android/view/View.html#attr_android:fitsSystemWindows
  [4]: http://o6w24t5lg.bkt.clouddn.com/statusBar2.png
  [5]: https://github.com/simonenfp/007-A