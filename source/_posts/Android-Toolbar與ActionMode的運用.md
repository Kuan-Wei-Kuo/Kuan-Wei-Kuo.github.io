---
title: Android. Toolbar與ActionMode的運用
date: 2015-11-19 10:25:00
tags: Android
---

我想很多人不知道有這個ActionMode的存在，好啦...我承認之前真的不知道。

值得高興的是Toolbar也有支援ActionMode，可以讓我們更方便的在特殊時刻切換至想要的Menu。

# 接下來就來說明如何應用

```java
ActionMode actionMode; //為了可以對ActionMode做外部的更改
Toolbar toolbar = (Toolbar) findViewById(R.i.toolbar);
toolbar.toolbar.startActionMode( new ActionMode.Callback() {
    @Override
    public boolean onCreateActionMode(ActionMode mode, Menu menu) {
        actionMode = mode; //取得目前的ActionMode
        mode.getMenuInflater().inflate(R.menu.menu, menu);
        return true;
    }

    @Override
    public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
        //準備
        mode.setTitle("Title");
        return true;
    }

    @Override
    public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
        //Menu Item 點擊
        return false;
    }

    @Override
    public void onDestroyActionMode(ActionMode mode) {
        //當ActionMode移除
    }
});
```

其實用法就這麼簡單，但當完成的時候，肯定會發現為什麼沒有覆蓋Toolbar，這時候我們只要加入下面這行：
```xml
<item name="windowActionModeOverlay">true</item>
```
那麼ActionMode就可以覆蓋在Toolbar上面。

# Before
<img src="/2015/11/19/Android-Toolbar與ActionMode的運用/actionMode_0.png" width="300">

# After
<img src="/2015/11/19/Android-Toolbar與ActionMode的運用/actionMode_1.png" width="300">