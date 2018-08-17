---
title: Android. View的水平與慣性移動
date: 2016-04-01 16:10:37
tags: Android
---

猶如前幾天所發布的開源圖表套件，有嘗試的人應該會發現移動怪怪的，這部分確實當初並沒有對於這部分下功夫，所以我今天給他更改了一下，達到我們所想要的滑動效果。

今天比較平穩的Po文，因為我好累，懶得靠北太多程式的東西，但還是得分享分享。

首先我們得了解兩大Class，分別為Scroller與VelocityTracker，前者為滑動效果，後者為手勢速度，看到上述的物件應該都明白我們將之搭配起來，來讓我們有高級滑動的效果！

# Scroller
它不是物件，簡單的來說，它可以計算精確的滑動位置，讓我們使用這些數據做運用。
它包含了各種滑動的運算，當然我們通常都只用到其中幾種。

# VelocityTracker
這Class通常放在OnTouch觸發的狀態下，記錄當前手勢的移動位置與計算速度。

接下來以一個簡單的範例來說明。

```java
public class CustomView extands View {
    
    public static final int SCROLL_MODE = 0;
    public static final int FLING_MODE = 1;

    public int SCROLLER_MODE = -1;

    private Scroller mScroller;
    private VelocityTracker mVelocityTracker;

    private int mScaledMinimumFlingVelocity;
    private int mScaledMaximumFlingVelocity;

    private float offestX, offestY;

    public CustomView(Context context) {
    super(context);
        mScroller = new Scroller(context);
        mViewConfiguration = ViewConfiguration.get(context);
        mScaledMinimumFlingVelocity = mViewConfiguration.getScaledMinimumFlingVelocity();
        mScaledMaximumFlingVelocity = mViewConfiguration.getScaledMaximumFlingVelocity();
    }

    private void obtainVelocityTracker(MotionEvent event) {
        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(event);
    }

    private void releaseVelocityTracker() {
        if (mVelocityTracker != null) {
            mVelocityTracker.recycle();
            mVelocityTracker = null;
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        obtainVelocityTracker(event); //每次動作都得運算
        switch(event.getAction) {
            case MotionEvent.ACTION_DOWN:
                offestX = curLeft - event.getX();
                offestY = curTop - event.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                SCROLLER_MODE = SCROLL_MODE;
                mScroller.startScroll((int) event.getX(), (int) event.getY(),
                    (int) offestX, (int) offestY, 1);
                break;
            case MotionEvent.ACTION_UP:
                SCROLLER_MODE = FLING_MODE;
                int initialVelocity = (int) mVelocityTracker.getXVelocity();
                mVelocityTracker.computeCurrentVelocity(1000, mScaledMaximumFlingVelocity);
                if (Math.abs(initialVelocity) > mScaledMinimumFlingVelocity) {
                    // 這個方法不難看出，Scroller與VelocityTracker的緊密結合。
                    mScroller.fling((int) curLeft, (int) curTop,
                                (int) mVelocityTracker.getXVelocity(), (int) mVelocityTracker.getYVelocity(),
                                minX, maxX, minY, maxY);
                }
                // 釋放VelocityTracker資源
                releaseVelocityTracker();
                break;
        }
        // 如果沒有Invalidate狀態，Scroller是不會運作的，更不會運作computeScroll()
        ViewCompat.postInvalidateOnAnimation(this);
        return true;
    }
  
    /**
    * 這方法很明顯的就是要我們自行去計算Sroller所給予的數值。
    */
    @Override
    public void computeScroll() {
        /**
        *將兩種不同數據分開的原因為，我們Scroll更新速度快，因此直接用最終結果
        *然而Fling算是一種動畫效果，因此在ACTION_UP時會有這個動畫，這時候就需要使用當前數據。    
        */
        if(SCROLLER_MODE == SCROLL_MODE) {
            curLeft = mScroller.getFinalX();
            curTop = mScroller.getFinalY():
        } else {
            curLeft = mScroller.getCurrX();
            curTop = mScroller.getCurrY();
        }
        ViewCompat.postInvalidateOnAnimation(this);
    }
}
```

好吧！　今天下課了，看起來沒有很複雜，如果想有更棒的效果，
對於Scroller與VelocityTracker的了解是不可或缺的，除非你想自己寫Buffer >\<

程式碼只是把部分打出來，所以可能不能直接套用，自己先嘗試看看吧！