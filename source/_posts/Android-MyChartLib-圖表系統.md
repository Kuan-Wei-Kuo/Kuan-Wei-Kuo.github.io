---
title: Android. MyChartLib 圖表系統
date: 2016-03-28 09:09:00
tags: Android
top: 10
---

# New Update. 目前更新可以讓ViewPager 與 ScrollView三者結合
<br/>

# 長條圖
<img src="/2016/03/28/Android-MyChartLib-圖表系統/columnChartView.png" width="300">

# 摺線圖
<img src="/2016/03/28/Android-MyChartLib-圖表系統/lineChartView.png" width="300">

# 圓餅圖
<img src="/2016/03/28/Android-MyChartLib-圖表系統/pieChartView.png" width="300">

# [GitHub SourceCode](https://github.com/Kuan-Wei-Kuo/MyChart)
<br/>

# How to use

## XML
```xml
<!-- 這邊長寬高可以自行設定-->
<com.kuo.mychartlib.view.ColumnChartView
    android:id="@+id/columnChartView"
    android:layout_width="match_parent" 
    android:layout_height="match_parent"/>
```

## Java
```java
private ArrayList<ColumnData> computeColumnData(int size, int maxValue) {
    int[] colors = {ChartRendererUntil.CHART_GREEN, ChartRendererUntil.CHART_PINK,
    ChartRendererUntil.CHART_RED, ChartRendererUntil.CHART_YELLOW, ChartRendererUntil.CHART_BROWN,
    ChartRendererUntil.CHART_ORANGE, ChartRendererUntil.CHART_GREY,
    ChartRendererUntil.CHART_PURPLE};

    ArrayList<ColumnData> test = new ArrayList<>();
    Random random = new Random();
    for(int i = 0 ; i < size ; i++) {
        test.add(new ColumnData("Axis X",
        random.nextInt(maxValue), colors[random.nextInt(colors.length)]));
    }
    return test;
}
columnChartView.setColumnData(computeColumnData(size, maxValue));
```

### UpdateChart
```java
columnChartView.setColumnData(computeColumnData(size, maxValue));
// updateChart() 方法為更新用
columnChartView.upadteChart(); 
```

目前我覺得方法這邊還可以在改一下，updateChart()這邊有點難以與一般的Lib雷同，起因是因為每次更改都得查看是否有更改View大小，防止錯誤。
