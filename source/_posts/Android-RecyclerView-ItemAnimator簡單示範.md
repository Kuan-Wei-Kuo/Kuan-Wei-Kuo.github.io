---
title: Android. RecyclerView ItemAnimator 簡單示範
date: 2015-08-08 06:08:00
tags: Android
---

目前在GitHub找到的資源，內部大多是使用DefaultItemAnimator.java(默認動畫程式碼)，並且做修改，因為它非常的好用，可以為您省去許多時間。

如果想要使用動畫就像下面那行程式碼就好了。
```java
recyclerView.setItemAnimator(new DefaultItemAnimator());
```
當然也有可能會是SlideInTop、OvershootInLeft、ScaleIn等等的動畫，但記住，這些都是需要自行製作的。

# Video
https://youtu.be/3Y9oEKV9h4w
[![Video](https://lh3.googleusercontent.com/kaAu9WJ9stE3X9ta0HaUozhLK3QJ57ipXo8_ipKuIjg=w852-h512-no)](https://youtu.be/3Y9oEKV9h4w)

# Reference Sources
[wasabeef/recyclerview-animators](https://github.com/wasabeef/recyclerview-animators)