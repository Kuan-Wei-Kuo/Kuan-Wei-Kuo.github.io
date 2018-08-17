---
title: Android. MVP的簡單介紹與使用
date: 2015-11-19 20:30:00
tags: Android
---

關於這篇文章的發表，個人其實也非常忐忑，但卻始終忍不住與大家一起探討這些問題。

自從MVP問世以後，Android開發者中已經掀起了許多看法，也因每個人的看法不同，所以我想在此發表自己的看法，並且與各位討論！

MVP 與 MVC 的選擇是如何? 我想很多人其實也很疑惑，但我相信新手們肯定都不知道這兩個是什麼?

# 淺談MVC
## 經典模式中的MVC
{% asset_img MVC_ICON.jpg %}
## 事實上MVC會變成
{% asset_img Usual_MVC.jpg %}

大家應該可以看到第二張圖中的虛線，沒錯，我們MVC通常會變成這樣，因為當我們Model完成時，該如何溝通View?

除非我們讓View也可讓Model溝通，但這卻又不符合MVC的初始原則，也造就了現在許多人對於MVC的說法各種不一。

我也只能夠確定一點，MVC這一詞只會越來越亂，因為許多工程師都有自己對於MVC的看法。

# 淺談MVP
{% asset_img MVP_ICON.jpg %}

MVP其實就是由MVC所演變而來，從圖上可知道，View與Model靠著Presenter做溝通，兩者間並無法直接溝通，這樣的架構好處就是，當Model修改時，只要我們最後的數據型態是相同的，View就不需要修改。

在Android方面，Activity(Fragment)可以稱為是Controller、View，兩者都是，如果使用MVC這種模式，我認為對於程式的撰寫是非常不易的，因此也認為MVP的架構對於Android，是相對有利的。

# 為什麼要使用
不論MVP與MVC兩者間都會讓程式上較於複雜，但如果理解這些模式的人，應該會更加容易明白這些位置的作用。也因為如此，這兩種架構都不適用於小型程式，畢竟只有大量的程式碼，才能夠讓他們發揮真正的作用。
1. 模組、介面權責劃分明顯
2. 模組可重複使用
3. 增加靈活性
4. 降低依賴性

# MVP的使用方法
## 建立Presenter與Model的監聽器
```java
public interface OnFindItemListener {
    // 假設我們要取得Integer數值的List
    void onFindItem(ArrayList<Integer> integers);
}
```

## Model
## 建立Model要傳數值回OnFindItemListener的Interactor
```java
public interface FindItemInteractor {
    void onFind(OnFindItemListener onFindItemListener);
}
```

## 建立我們運算取得數據的方法
```java
public class FindItemInteractorImpl implements FindItemInteractor {

    @Override
    public void onFind(OnFindItemListener onFindItemListener) {
        // 回傳ArrayList<Integer>
        onFindItemListener.onFindItem(findIntegers()); 
    }

    private ArrayList<Integer> findIntegers() {
        ArrayList<Integer> integers = new ArrayList<>();
        for(int i = 0 ; i < 10 ; i++) {
            integers.add(i);
        }
        return integers;
    }
}
```

# View
## 與Presenter建立溝通，也是獲取數據的功能
```java
public interface FindItemView {
    // 基本上跟OnFindItemListener是一樣的
    void setItem(ArrayList<Integer> integers); 
}
```

# Presenter
## 建立我們主要的Presenter
```java
public interface FindItemPresenter {
    // 這邊可以算是命令，當我onStartFind可以做些事
    void onStartFind();
}
```

## Presenter應用
```java
public class FindItemPresenterImpl implements FindItemPresenter, OnFindItemListener {

    private FindItemView findItemView;
    private FindItemInteractor findItemInteractor;

    public FindItemPresenterImpl(FindItemView findItemView) {
        this.findItemView = findItemView;
        findItemInteractor = new FindItemInteractorImpl();
    }

    @Override
    public void onStartFind() {
        // 當onFind啟動時，也代表FindItemInteractorImpl開始運算
        findItemInteractor.onFind(this);
    }

    @Override
    public void onFindItem(ArrayList<Integer> integers) {
        // 使用View將數據回傳
        findItemView.setItem(integers);
    }
}
```

## Activity使用Presenter獲取我們想要的數據
```java
public class MainActivity extends AppCompatActivity implements FindItemView {
    // 為了讓我們可以監聽到FindItemView所回傳的數據，所以需要implements FindItemView
    
    // 宣告一個Presenter
    private FindItemPresenter findItemPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initButton();
        // new Presenter應用
        findItemPresenter = new FindItemPresenterImpl(this); 
    }

    private void initButton() {
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 如上述所說，這可以算是一個命令，Activity請求Presenter使用onStartFind方法。
                findItemPresenter.onStartFind(); 
            }
        });
    }

    // 當Model完成運算後，會將數據經由Presenter傳回至View
    @Override
    public void setItem(ArrayList<Integer> integers) {
        TextView textView = (TextView) findViewById(R.id.textView);
        textView.setText("Find " + integers.size() + " item.");
    }
}
```
# GitHub Download
[GitHub](https://github.com/Kuan-Wei-Kuo/ExampleMVP)
 
# 結語
這篇文章就到此結束了，對於這些設計模式，我們有時候其實很難抉擇與確實的實現，因此只能說，先把View分出來，在來考慮Model。
