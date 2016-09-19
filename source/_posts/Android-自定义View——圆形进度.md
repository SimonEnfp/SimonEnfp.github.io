---
title: Android 自定义View——圆形进度
date: 2016-09-18 18:13:14
tags:
categories: 技术分享
---
发现有很多项目在刚进入app时的欢迎页后的广告页右上角有一个点击跳过的按钮，并且有个环形进度，对难以忍受广告的用户来说，这比显示广告页的时候只能傻傻的在那等广告,体验就好多了，于是自己写了个view，无图无真相，先来一发效果图：
![此处输入图片的描述][1]

这个view是个圆形，且有环形进度，中心有文字进度，所以我选择继承自原始View,原理就是根据时间的流逝，计算进度，重绘View。
主要分为5个部分：
> * 自定义View属性
> * 重写OnMeasure
> * 重写OnDraw
> * 更新进度
#### 1. 自定义View所需属性
 在res/values/attr.xml中
```python
<?xml version="1.0" encoding="utf-8"?>
<resources>

    
    <declare-styleable name="CustomCircleProgress">
        <attr name="centerText" format="string"/>
        <attr name="centerTextColor" format="color"/>
        <attr name="centerTextSize" format="dimension"/>
        <attr name="circleBackground" format="color" />
        <attr name="ringProgressColor" format="color"/>
        <attr name="ringProgressWidth" format="dimension"/>
        <attr name="ringProgressBackground" format="color"/>
    </declare-styleable>
    
    <declare-styleable name="CustomView2">
        <attr name="contentText" format="string"/>
        <attr name="contentTextColor" format="color"/>
        <attr name="contentTextSize" format="dimension"/>
    </declare-styleable>
</resources>
```
定义属性变量，构造方法中获取属性
```python

/*
    * 中心文字
    * */
    private String mCenterText;
    /*
    * 中心文字颜色
    * */
    private int mCenterTextColor;
    /*
    * 中心文字大小
    * */
    private float mCenterTextSize;
    /*
    * 圆形背景
    * */
    private int mCircleBackground;
    /*
    * 圆环进度颜色
    * */
    private int mRingProgressColor;
    /*
    * 圆环进度背景颜色
    * */
    private int mRingProgressBackground;
    /*
    * 圆环进度宽度
    * */
    private float mRingProgressWidth;
     public CustomCircleProgress(Context context, AttributeSet attributeSet, int defStyleAttr){
        super(context,attributeSet,defStyleAttr);
        //        获得定义的自定义样式属性
        TypedArray a = context.obtainStyledAttributes(attributeSet,                 R.styleable.CustomCircleProgress,defStyleAttr,0);

        mCenterText = a.getString(R.styleable.CustomCircleProgress_centerText);
        mCenterTextColor =     a.getColor(R.styleable.CustomCircleProgress_centerTextColor,Color.WHITE);
        mCenterTextSize = a.getDimensionPixelSize(R.styleable.CustomCircleProgress_centerTextSize,(int) TypedValue.applyDimension(
                TypedValue.COMPLEX_UNIT_SP, 14, getResources().getDisplayMetrics()));
        mCircleBackground = a.getColor(R.styleable.CustomCircleProgress_circleBackground,Color.BLACK);

        mRingProgressColor = a.getColor(R.styleable.CustomCircleProgress_ringProgressColor,Color.GRAY);
        mRingProgressBackground = a.getColor(R.styleable.CustomCircleProgress_ringProgressBackground,Color.TRANSPARENT);
        mRingProgressWidth = a.getDimensionPixelSize(R.styleable.CustomCircleProgress_ringProgressWidth,(int) TypedValue.applyDimension(
                TypedValue.COMPLEX_UNIT_DIP, 4, getResources().getDisplayMetrics()));

        a.recycle();
        initialize();

    }
    
```

#### 2. 重写OnMeasure
由于继承自View，所以为了使wrap_content属性有效，必须重写OnMeasure。
```python
 @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

    //如果宽或高的SpecMode为AT_MOST，就设置为默认宽高
        if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST){
            setMeasuredDimension(mDefaultRadius,mDefaultRadius);
        }else if (widthSpecMode == MeasureSpec.AT_MOST){
            setMeasuredDimension(mDefaultRadius,heightSpecSize);
        }else if (heightSpecMode == MeasureSpec.AT_MOST){
            setMeasuredDimension(widthSpecSize,mDefaultRadius);
        }
        int width = getMeasuredWidth();
        int height = getMeasuredHeight();
        //宽和高取较小的那一个，并设置自定义圆形View尺寸
        int mViewSize = ((height < width ? height : width));
        setMeasuredDimension(mViewSize,mViewSize);
    }
```

#### 3. 重写OnDraw
```python
@Override
    protected void onDraw(Canvas canvas) {

        //设置绘制区的矩形，为圆环外界留出圆环宽度一半的大小
        mRectFBound.set(mRingProgressWidth/2,mRingProgressWidth/2,getWidth() - mRingProgressWidth/2,getHeight() - mRingProgressWidth/2);
        mPaint.setAntiAlias(true);

//        画实心背景 半径为矩形区域减去圆环宽度的一半
        mPaint.setColor(mCircleBackground);
        mPaint.setStyle(Paint.Style.FILL);
        canvas.drawCircle(mRectFBound.centerX(),mRectFBound.centerY(),mRectFBound.width()/2 - mRingProgressWidth/2,mPaint);


        //画中心文字 中心文字起点为左下，文字绘制在圆形中间位置
        mPaint.setColor(mCenterTextColor);
        mPaint.setStyle(Paint.Style.FILL);
        if (mShowPercent){
            mCenterText = String.valueOf(mPercent) + "%";
        }
        mPaint.setTextSize(mCenterTextSize);
        mPaint.getTextBounds(mCenterText,0,mCenterText.length(),mTextRect);
        canvas.drawText(mCenterText,mRectFBound.centerX() - mTextRect.width()/2,mRectFBound.centerY() + mTextRect.height()/2,mPaint);

        //        画圆环   因为圆环有宽度，所以是环的中间位置与矩形四边相切
        mPaint.setStrokeWidth(mRingProgressWidth);
        mPaint.setStyle(Paint.Style.STROKE);

        mPaint.setColor(mRingProgressBackground);
        canvas.drawArc(mRectFBound,0,360,false,mPaint);

        mPaint.setColor(mRingProgressColor);
        canvas.drawArc(mRectFBound,0,mArc,false,mPaint);

        updateProgress();
    }
```

#### 4. 更新进度
先定义一些更新进度需要的变量
```python
  /*
   * 圆环起始弧度,默认360
   * */
    private int mArc = 360;

    /*
    * 当前进度
    * */
    private float mCurrProgress;


    /*
    * 进度开始时刻
    * */
    private long mStartTime ;

    /*
    * 进度是否已开始
    * */
    private boolean mStart = false;

    /**
     * 默认进度完成总时间。
     */
    private int mDuration = 3000;

    /*
    * 进度完成时间倒数
    * */
    private float mDurationReciprocal;

    /*
    * 进度是否渐增，默认是递减
    * */
    private boolean mIncreasing = false;

    /*
    * 是否显示百分比进度，默认不显示
    * */
    private boolean mShowPercent = false;
    /*
    * 百分比进度，默认100%
    * */
    private int mPercent = 100;
```
这里有三个变量可以在使用view的自己设定mDuration（设置进度总时间）、mIncreasing(设置进度递增或递减)、mShowPercent（是否显示进度百分比）、
```python
 /*
    * 进度开始
    * */
    public void startRun(){
        mStartTime =  AnimationUtils.currentAnimationTimeMillis();
        mStart = true;
        mDurationReciprocal = 1.0f / (float) mDuration;
        updateProgress();
    }
    
     /*
    * 更新进度
    * */
    private void updateProgress(){
        if (computeProgressOffset()){

            if (mIncreasing){
                mArc = Math.round(360 * mCurrProgress);
                mPercent = Math.round(100 * mCurrProgress);
            }else {
                mArc = Math.round(360 * (1-mCurrProgress));
                mPercent = Math.round(100 * (1-mCurrProgress));
            }

            postInvalidate();
        }
    }

    /*
    * 根据时间流逝计算进度比例
    * */
    private boolean computeProgressOffset(){
        if (!mStart){
            return false;
        }
        int timePass = (int) (AnimationUtils.currentAnimationTimeMillis() - mStartTime);
        if (timePass < mDuration){
            mCurrProgress = timePass * mDurationReciprocal;
        }else {
            mStart = false;
            mCurrProgress = 1;
        }
        return true;
    }

```
由于是在一定的时间内更新完成进度(进度0~1)，所以可以在更新进度的时候先通过 当前系统时间 - 开始更新进度时间 计算已经过去的时间timePass，这样就根据时间的流逝计算占总进度的比例，每次更新进度时调用computeProgressOffset()，判断是否完成进度，如果未完成(即时间未到)，设置已完成进度，返回true，调用postInvalidate()更新View；更新完后又会调用updateProgress()，如此循环；一旦timePass超过或等于设定时间，就完成了，设置进度为1，此时仍需要更新一次，所以computeProgressOffset()仍然返回true，但更新标识mStart应为false了，以此终止更新。
这样简单的圆形进度基本功能就实现了，项目中够用了。
    


 

  [1]: http://o6w24t5lg.bkt.clouddn.com/aa.gif