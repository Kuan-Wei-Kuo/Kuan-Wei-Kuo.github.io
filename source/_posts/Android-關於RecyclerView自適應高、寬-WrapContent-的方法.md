---
title: Android. 關於RecyclerView自適應高、寬(WrapContent)的方法
date: 2016-02-20 11:01:46
tags: Android
---

我們要知道一件事情啦！ RecyclerView沒有對於Item做任何的改變，簡單的說，我們的Layout編排，都是靠LayoutManager去做修改，因此我們需要對於LayoutManager做運算。

所以我們得手動更改onMeasure，以下為LinearLayoutManager做範例。

```java
public class WrapContentLinearLayout extends LinearLayoutManager {

    public WrapContentLinearLayout(Context context) {
        super(context);
    }

    public WrapContentLinearLayout(Context context, int orientation, boolean reverseLayout) {
        super(context, orientation, reverseLayout);
    }

    public WrapContentLinearLayout(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }

    @Override
    public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state, int widthSpec, int heightSpec) {
        View view = recycler.getViewForPosition(0);
        if(view != null){
            measureChild(view, widthSpec, heightSpec); //計算View的高寬
            int measuredWidth = getItemCount() * view.getMeasuredWidth() > View.MeasureSpec.getSize(widthSpec) ? View.MeasureSpec.getSize(widthSpec) : getItemCount() * view.getMeasuredWidth();
            int measuredHeight = view.getMeasuredHeight();
            setMeasuredDimension(measuredWidth, measuredHeight);
        }
    }
}
```

如果全部的ItemView寬度大於RecyclerView的寬度就使用原先的寬度，反之就使用我們ItemView的寬度，如此一來就解決了。

然後就...江...上面這方法聽說不是很準確的抓到高寬度，但我覺得夠用了！反正使用者又看不太出來(感覺要被打了...)。

以上為今日不負責任教學講座，請看好看滿。