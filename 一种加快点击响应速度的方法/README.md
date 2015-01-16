## 一种加快点击响应速度的方法

> 原创：余俊卿 转载请:<yujunqing@meizu.com>

----

##### 一般情况下我们是这样响应点击事件的：

    v.setOnClickListener(this);

##### 但今天在优化lancher启动其它应用速度 的时候发现，onClick的执行需要延时20-30ms

##### 虽然数值不是很高，但是在不需要响应双击事件的时候这是完全没有必要的时间浪费。我们可以使用

    v.setOnTouchListener(speedTouchListener);
    
##### 来优化这其中的时间浪费

####以下是实例代码，基于Lancher.java

简单来说就是去掉之前的 setOnClickListener ，保留 OnClickListener ，但是把 onClick()方法放到 自定义的 GestureDetector 里去做。

由于考虑了内存泄露的潜在风险所以加大了代码复杂度。

如果有更好的方法，请 @余俊卿 谢谢！


````
/**
     * 用来加快应用启动速度，把UP响应时间缩减到最短
     * 可以稳定减少20-30ms
     */
    public final SpeedTouchListener speedTouchListener = new SpeedTouchListener(this);

    static class SpeedTouchListener implements View.OnTouchListener {

        private final MyGestureDetector detector;
        private final WeakReference<Launcher> mLauncher;

        public SpeedTouchListener(Launcher launcher) {
            this.mLauncher = new WeakReference<Launcher>(launcher);
            this.detector = new MyGestureDetector(new GestureDetector.SimpleOnGestureListener() {
                @Override
                public boolean onSingleTapUp(MotionEvent e) {
                    if (mLauncher.get() != null) {
                        //执行之前的onClick()方法
                        mLauncher.get().onClick(detector.viewCache.get());
                    }
                    return true;
                }
            });

            detector.setIsLongpressEnabled(false);
            detector.setOnDoubleTapListener(null);
        }

        @Override
        public boolean onTouch(View v, MotionEvent event) {
            return detector.onTouchEvent(v, event);
        }

        class MyGestureDetector extends GestureDetector {

            public WeakReference<View> viewCache;

            public MyGestureDetector(OnGestureListener listener) {
                super(listener);
            }

            public boolean onTouchEvent(View v, MotionEvent ev) {
                if (viewCache == null || !v.equals(viewCache.get())) {
                    viewCache = new WeakReference<View>(v);
                }
                return super.onTouchEvent(ev);
            }
        }
    }
````