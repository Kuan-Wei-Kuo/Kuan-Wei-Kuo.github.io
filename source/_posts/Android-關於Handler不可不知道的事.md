---
title: Android. 關於Handler不可不知道的事
date: 2015-11-17 15:02:00
tags: Android
---

大家都應該都有使用過Handler更新UI的時刻，但我們的使用方法是對的嗎?

# 我們通常都會像下面這樣寫
```java
private Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        // TODO
    }
}
```

各種關於Handler的教學文章應該都是如此使用吧?

那麼大家有沒有看過下面這行錯誤警告↓
Warn:This Handler class should be static or leaks might occur
這句話給的意義其實非常重大，Handler應該是static，否則內存可能洩漏！
就讓小魯解釋一下，他這個可恨的東西到底是如何洩漏的呢?

問題在於Runnable、Thread，我們獲取大量數據時當然會使用這些方法，並且加上msg讓他完成時更新UI，但如果我們將這個Activity(Fragment)註銷掉呢?

很不幸的，當你應用程式開啟framework也跟著開啟，內部有一個Looper，整個App靠他，如果msg沒有執行前，這些正在執行中的Runnable、Thread將會繼續執行，直至msg被執行，這根本就是驚天動地，令人無法接受，唉聲嘆氣的消耗程式資源!

但不要吃驚，Romain Guy(Adroid framework engineer)也幫助我們，給我們建議[使用方法](https://groups.google.com/forum/?fromgroups=#%21msg/android-developers/1aPZXZG6kWk/lIYDavGYn5UJ)

# 就讓咱們繼續看下去...
```java
class OuterClass {
    class InnerClass {

        private final WeakReference<OuterClass> mTarget;
                                        
        InnerClass(OuterClass target) {
            mTarget = new WeakReference<OuterClass>(target);
        }
                                    
        void doSomething() {
            OuterClass target = mTarget.get();
            if (target != null) {
                target.do();    
            }
        }
    }
}
```

上述程式碼可以看到一個不常看見的東西叫WeakReference，也是所謂的"弱引用"，小魯在這邊也大概說一下，弱引用就是不能阻擋任何關於回收的動作。

# 那麼我們的程式碼將會
```java
private static class OuterHandler extends Handler {
    private final WeakReference<MainActivity> mActivity;
        
    public OuterHandler(MainActivity activity) {
        mActivity = new WeakReference<MainActivity>(activity);
    }
        
    @Override
    public void handleMessage(Message msg) {
        MainActivity activity = mActivity.get();
        if (activity != null) {
        // do something...
        }
    }
}
```

所以其實當我們使用到了handler以及msg，如果我們無法取得想要WeakReference的物件，那麼回收機制也將會直接收走，連一點殘渣也不想留給你，這樣也解決的上述內存洩漏的可能。

# 參考資料
[泡在網上的日子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1106/1922.html)
[Google開發論壇](https://groups.google.com/forum/?fromgroups=#%21msg/android-developers/1aPZXZG6kWk/lIYDavGYn5UJ)
[Java之WeakReference與SoftReference使用講解](http://codex.wiki/post/132087-711)