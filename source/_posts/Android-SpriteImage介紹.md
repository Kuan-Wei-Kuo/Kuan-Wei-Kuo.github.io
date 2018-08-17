---
title: Android. SpriteImage介紹
date: 2016-04-24 13:44:28
tags: Android
---

有些人常問我，怎一天到晚都在寫程式?

TMD肥宅我覺得還是進入主體，牢騷沒人要聽啦！看看上次發文4/1，似乎今天有點久了，最近都在修改MyChart以及製作履歷表，剛好這次履歷表有用到SpriteImage，這種東西通常都用在遊戲上面。

如果製作2D遊戲的人可能會常看到以下這種圖片
<img src="/2016/03/01/Android-SpriteImage介紹/test2.png" width="600">
(圖片資源取自於[Game 2D Art](http://www.gameart2d.com/freebies.html))

為的就是減少效能使用，我們把一個連續動作放在一張大圖內，使用程式碼，裁減我們所需要的部分即可，這樣我們就不用一次讀取多張圖片。

因為我這邊已經有自己寫好的類別，沒意外直接套用即可，所以就直接打上來

# 主類別
```java
public class SpriteAnimation {

    protected Context context;

    protected Rect bitmapRect;

    protected long lastFrameChangeTime = 0; //累計時間

    protected long frameLengthInMilliseconds = 100; //更新速度

    /**
    * 對圖片預算，垂直與水平的框架都要計算
    * countFrame 為所有框架的數量
    */
    protected int countHorizontalFrame = 0, countVerticalFrame = 0, countFrame = 0;
    
    /**
    * 目前運行的框架位置
    */
    protected int currentHorizontalFrame = 0, currentVerticalFrame = 0;
    
    /**
    * 裁減的大小
    */
    protected int frameWidth = 0;
    protected int frameHeight = 0;

    private OnUpdateListener onUpdateListener;

    public SpriteAnimation(long frameLengthInMilliseconds, Rect bitmapRect, int frameWidth, int frameHeight, int countFrame) {
        this.frameLengthInMilliseconds = frameLengthInMilliseconds;
        this.bitmapRect = bitmapRect;
        this.frameWidth = frameWidth;
        this.frameHeight = frameHeight;
        this.countFrame = countFrame;

        countHorizontalFrame = bitmapRect.width() / frameWidth;
        countVerticalFrame = bitmapRect.height() / frameHeight;
    }

    private int animateCount = 0;

    public void start() {
        long time  = System.currentTimeMillis();
        if ( time > lastFrameChangeTime + frameLengthInMilliseconds) {
            lastFrameChangeTime = time;
            currentHorizontalFrame++;
            if (currentHorizontalFrame >= countHorizontalFrame) {
                currentHorizontalFrame = 0;
                currentVerticalFrame++;
                if(currentVerticalFrame >= countVerticalFrame)
                    currentVerticalFrame = 0;
            }
            animateCount++;
        }
        if(onUpdateListener != null) {
            onUpdateListener.onUpdate(currentHorizontalFrame, currentVerticalFrame);
            if(animateCount >= countFrame) {
                currentVerticalFrame = 0;
                currentHorizontalFrame = 0;
                animateCount = 0;
                onUpdateListener.end();
            }
        }
    }

    public interface OnUpdateListener {
        void onUpdate(int currentHorizontalFrame, int currentVerticalFrame);
        void end();
    }

    public void setOnUpdateListener(OnUpdateListener onUpdateListener) {
        this.onUpdateListener = onUpdateListener;
    }

    public int getFrameHeight() {
        return frameHeight;
    }

    public int getFrameWidth() {
        return frameWidth;
    }

    public int getCountFrame() {
        return countFrame;
    }
}
```

# 使用方法
```java
mSpriteAnimation = new SpriteAnimation(100, 
                    new Rect(0, 0, bitmap.getWidth(), bitmap.getHeight()), 192, 192, 10);
mSpriteAnimation.setOnUpdateListener(onUpdateListener);
mSpriteAnimation.start();
private SpriteAnimation.OnUpdateListener onUpdateListener = new SpriteAnimation.OnUpdateListener() {
    @Override
    public void onUpdate(int currentHorizontalFrame, int currentVerticalFrame) {
        int left = currentHorizontalFrame * mSpriteAnimation.getFrameWidth();
        int right = left + mSpriteAnimation.getFrameWidth();
        int top = currentVerticalFrame * mSpriteAnimation.getFrameHeight();
        int bottom = top + mSpriteAnimation.getFrameHeight();
        srcRect.set(left, top, right, bottom); //更新剪裁位置
        canvas.drawBitmap(runBitmap, srcRect, dstRect, null);
    }

    @Override
    public void end() {

    }
}
```