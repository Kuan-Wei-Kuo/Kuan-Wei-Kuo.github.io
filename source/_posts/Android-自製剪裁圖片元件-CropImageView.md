---
title: Android. 自製剪裁圖片元件 CropImageView
date: 2016-01-30 14:24:16
tags: Android
---

這次分享簡單的剪裁圖片方式，我還沒做成Library，還滿懶惰的...((被踹飛

大家可以參考參考，程式碼方面不難，反正就一堆 ? : if else 的判定。

不過這次做起來有點瑕疵，在剪裁時候，有些許的不準確，目前判斷是因為Bitmap大小與剪裁不符合，大家可能都會想說，不是用Canvas去畫Bitmap嗎? No no ~ 我盡量不去使用Canvas去畫Bitmap，每次更新時都得在畫一次Bitmap，這是多麼恐怖的事情，你會非常Lag，效能之差，所以我直接使用ImageView.setImageBitmap。

當然這並不算是我第一次製作此類，在先前的項目就有使用過，但並未認真去抓取一些Bug，所以有時候會有些錯誤。

然而這次我將外圍、伸縮剪裁框等，小瑕疵都修改完畢，因此應該是不會發生瑕疵的問題了。

大家有興趣可以去GitHub看看，記得不要看在學時期的Project就別看了，很丟臉...((淚灑

P.S.也許有人說，我的藍點怎麼只有一個，應該四面八方都要有呀！ 我就是懶，我懶我驕傲!! ((又飛走

<img src="/2016/01/30/Android-自製剪裁圖片元件-CropImageView/Screenshot_2016-01-30-14-19-52.png" width="300">
<img src="/2016/01/30/Android-自製剪裁圖片元件-CropImageView/Screenshot_2016-01-30-14-20-02.png" width="300">

# [GitHub SourceCode](https://github.com/Kuan-Wei-Kuo/PhotoDesign)