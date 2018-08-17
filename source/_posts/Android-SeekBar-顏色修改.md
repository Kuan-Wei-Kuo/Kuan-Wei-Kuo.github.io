---
title: Android. SeekBar 顏色修改
date: 2015-08-31 12:38:00
tags: Android
---

日前有機會做個顏色選擇器，因此去研究了SeekBar的相關文獻，主要還是使用xml達到修改的目的，如下方所示：

### red_seekbar_progress.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item android:id="@android:id/background">
        //SeekBar滾動條的背景
        <shape android:shape="rectangle" >
            <solid
                android:color="@color/GREY_800" />
        </shape>
    </item>
    <item android:id="@android:id/progress">
        //SeekBar Progress的顏色
        <clip>
            <shape android:shape="rectangle" >
                <solid
                    android:color="@color/RED_300" />
            </shape>
        </clip>
    </item>
</layer-list>
```

### seekbar_layout.xml
```xml
<SeekBar
    android:id="@+id/slideR"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="10dp"
    android:max="255"
    android:maxHeight="3dp" 
    android:minHeight="3dp"
    android:progressDrawable="@drawable/red_seekbar_progress"
    android:thumbTint="@color/RED_300"/>
```

red_seekbar_progress.xml部分其實也沒有什麼需要探討的，簡單改個顏色與背景罷了，然而在seekbar_layout.xml就需要去注意一些事項：
1. 記得要設置maxHeight and minHeight否則會看到Progress Background比SeekBar滑動按鈕還要大的情形出現。
2. progressDrawable為剛剛所設置的red_seekbar_progress.xml。
3. thumbTint其實就是我們SeekBar的滑動按鈕顏色。