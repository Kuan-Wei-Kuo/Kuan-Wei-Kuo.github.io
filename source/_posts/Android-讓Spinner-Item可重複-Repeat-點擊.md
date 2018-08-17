---
title: Android. 讓Spinner Item可重複(Repeat)點擊
date: 2016-03-01 10:49:28
tags: Android
---

Spinner是一種簡單的選單，但通常我們都只是簡單應用，並沒有做太多的改變，但我看到某些APP可以重複點擊耶！我也TMD很有事的研究了一下。

先說好我只Po KeyCode，其餘的自己想辦法應用啦~我可是跟Google研究很久耶!((被踹飛
如果你想讓Spinner Item可以重複點擊，那麼就用下面這個囉！

# 修改Spinner SourceCode
```java
try {
    Field field = AdapterView.class.getDeclaredField("mOldSelectedPosition");
    field.setAccessible(true);
    field.setInt(spinner_nav, AdapterView.INVALID_POSITION);
} catch (Exception e) {
    e.printStackTrace();
}
```

如果你有稍微研究一下SourceCode你會看到mOldSelectedPosition是怎樣的角色，當ItemPosition被記錄到mOldSelectedPosition的時候，代表我們已經點擊過，這時候他們就不會給你點擊，所以我們透過修改mOldSelectedPositio，讓他保持初始，這樣我們就可以瘋狂重複點擊啦！