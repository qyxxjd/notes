#### View绘制

- requestLayout：会调用三大流程，进行测量、布局、绘制。measure -> onMeasure -> layout -> onLayout -> draw -> onDraw
- invalidate: 只进行重绘视图, 只能在主线程调用。draw -> onDraw
- postInvalidate: 内部执行的invalidate，在子线程调用
- onMeasure 
  > onMeasure(int widthMeasureSpec, int heightMeasureSpec)  
  > measure(int widthMeasureSpec, int heightMeasureSpec)  
  > 用于测量视图的大小.  
  > measure和onMeasure方法都接收两个参数(widthMeasureSpec, heightMeasureSpec)，这两个值分别用于确定视图的宽度和高度.  
  > MeasureSpec的值由specSize和specMode共同组成的，其中specSize记录的是大小，specMode记录的是规格。  
  > specMode一共有三种类型：  
  > 	EXACTLY - 子视图的大小应该是由specSize的值来决定的。  
  > 	AT_MOST - 子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。  
  > 	UNSPECIFIED - 可以设置成任意的大小，没有任何限制。  
  > widthMeasureSpec和heightMeasureSpec这两个值又是从哪里得到的呢？是通过getRootMeasureSpec()方法去获取  

- onLayout
  > onLayout(boolean changed, int l, int t, int r, int b);    
  > layout(int l, int t, int r, int b)  
  > measure过程结束后，视图的大小就已经测量好了，接下来就是给视图进行布局的，也就是确定视图的位置。  
  > 调用View的layout()方法来执行此过程  
  > 在layout()方法中，首先会调用setFrame(int l, int t, int r, int b)方法来判断视图的大小是否发生过变化，以确定有没有必要对当前的视图进行重绘  

- onDraw
  > onDraw(Canvas canvas)  
  > draw(Canvas canvas)  
  > measure和layout的过程都结束后，接下来就进入到draw的过程了。同样，根据名字你就能够判断出，在这里才真正地开始对视图进行绘制。  


#### 层级关系:
> Activity -> PhoneWindow -> DecorView -> 界面布局文件  
> 每个Activity包含一个PhoneWindow对象，PhoneWindow设置DecorView为应用窗口的根视图，所有的UI部件都是放在  DecorView中。  
> Window是一个抽象类，PhoneWindow是Window的唯一子类。  
> DecorView是PhoneWindow类的内部类。继承自FrameLayout，所以说DecorView是一个ViewGroup。  


#### 事件传递:
> View的事件分发机制其实就是点击事件的传递（一个Down事件,若干个Move事件，一个Up事件构成的事件序列的传递）  

传递事件  
> boolean dispatchTouchEvent(MotionEvent ev)  
  - ture:当前View消耗所有事件
  - false:停止分发，交由上层控件的onTouchEvent方法进行消费，如果本层控件是Activity，则事件将被系统消费处理

拦截事件  
> boolean onInterceptTouchEvent(MotionEvent event)  
  - ture:对事件拦截，交给本层的onTouchEvent进行处理
  - false:不拦截，分发到子View，由子View的dispatchTouchEvent进行处理
  - super.onInterceptTouchEvent(ev):默认不拦截
  
处理事件  
> boolean onTouchEvent(MotionEvent event)  
  - true:表示onTouchEvent处理后消耗了当前事件
  - false:不响应事件，不断的传递给上层的onTouchEvent方法处理，直到某个View的onTouchEvent返回true,则认为该事件被消费，如果到最顶层View还是返回false,则该事件不消费，将交由Activity的onTouchEvent处理
  - super.onTouchEvent(ev):默认消耗当前事件，与返回true一致

事件从Activity.dispatchTouchEvent()开始传递，然后由父View(ViewGroup)传递给子View，ViewGroup可以通过onInterceptTouchEvent()对事件做拦截，停止传递。子View可以通过onTouchEvent()对事件进行处理。如果事件一直没有处理，最终会回到Activity的onTouchEvent  
如果View不消耗事件，则在同一事件序列中，当前View无法再接收到剩下的事件  
处理事件的优先级为：OnTouchListener > onTouchEvent > OnClickListener  


#### onMeasure 和 onLayout 区别
- onMeasure 
	- 属于View的方法，用来测量自己和内容的来确定宽度和高度 
	- view的measure方法体中会调用onMeasure 
- onLayout 
	- 属于ViewGroup的方法，用来获取当前ViewGroup的子元素的位置和大小 ，进行布局
	- View的layout方法体中会调用onLayout 
- onMeasure在onLayout之前调用 
- 设置background后，会重新调用onMeasure和onLayout


#### 嵌套滑动
- 事件拦截和分发方案
	- 灵活性高，也最繁琐
- 基于NestingScroll机制的实现方案
	- 对原始的事件拦截机制进行封装，通过子View实现NestedScrollingChild接口，父View实现NestedScrollingParent接口
- 基于CoordinatorLayout与Behavior的实现方案
	- 对NestingScroll机制再次封装。CoordinatorLayout默认实现了NestedScrollingParent接口。
	- 第二种方案只能由子View通知父View，但有时候需要通知父View或者兄弟View，这个时候需要Behavior。


#### 滑动冲突
> 同方向的滑动冲突（比如ScrollView嵌套ListView或者ScrollView嵌套自己）  
> 不同方向的滑动冲突（比如ScrollView嵌套ViewPager）  

- 从外部拦截，就是在父控件的onInterceptTouch事件中，明确是否拦截此事件
- 通过子View的调用getParent().requestDisallowIntercaptTouchEvent()设定父View是否拦截Touch事件


#### 绘制
> Paint  相当于笔  
> Canvas 相当于纸  

Canvas API
- drawArc：绘制弧线
- drawColor：填充颜色
- drawCircle：圆
- drawOval：椭圆
- drawPoint：点
- drawLine：线条
- drawRect：矩形
- drawRoundRect：圆角矩形
- drawText：文本
- drawPath：路径
- drawPicture：图片
- drawBitmap：bitmap
