---
title: Android. 使用getIdentifier獲取資源ID
date: 2015-12-29 13:15:28
tags: Android
---

最近在開發App的時候發現一個問題，如果我們早先就將Image的位置放入SQLite，然後加入新的Image位置，發現會產生id錯亂的問題，這問題其實也就只是因為我們加入新的Image而產生的，因為每加入新的圖片，可能就會造成id的從新編排。

那避免方法其實很簡單，方法如下
```java
/**
* @param name 這邊請填入我們所想要的id名稱 (ex:ic_luncher)
* @param defType 填入我們想要在哪個資料夾尋找上述的id名稱 (ex:mipmap)
* @param defPackage 想要從哪個Package取得上述的內容
**/
getResources().getIdentifier(String name, String defType, String defPackage);

int resId = getResources().getIdentifier("ic_luncher", "mipmap", getPackageName());
```
其實就這麼簡單而已，這樣也不怕因為更新圖片而導致id位置錯亂。