---
title: Android. 使用ValueAnimator令顏色漸變轉換
date: 2016-02-05 09:22:46
tags: Android
---

大家好 大gay嚇 摳擬機挖，肥宅今天來分享一下如何取得連續漸變的顏色呢!?在目前手機的發達上，UX的顧慮也是特別重要，因此我們發現動作效果是不是更多了?

沒錯，以前只要切來切去，現在就是要有一點順暢的切換，毫無為何感的fu，所以我們常常要使用一些動畫來輔助一下，因此，所以，也就是我們不可以不學習這東西囉！

其實Value簡單的說就是數值，這個東東可以讓數值漸變，由0~100之類的慢慢轉換。

<img src="http://developer.android.com/images/animation/animation-nonlinear.png" width="600">
(圖片取自Google)

然而我們的Value預設下是使用非線性的樣子去做變換，因此會像上圖那樣。
使用方法如下：
```java
int colorFrom = Color.RED;
int colorTo = Color.BLUE;
ValueAnimator colorAnimation = ValueAnimator.ofObject(new ArgbEvaluator(), colorFrom, colorTo);
colorAnimation.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animator) {
        toolbar.setBackgroundColor((int) animator.getAnimatedValue());
    }
});
colorAnimation.setDuration(300); //設定速度
colorAnimation.start();
```

要記得加入AnimatorUpdateListener監聽我們的動畫運行以及取得目前的Value，來達到漸變切換的感覺。Animator的應用實在是非常多，我想簡單介紹即可，如果要在更深入使用，就在說吧！