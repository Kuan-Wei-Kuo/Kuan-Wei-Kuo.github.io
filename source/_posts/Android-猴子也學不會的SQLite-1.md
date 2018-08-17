---
title: Android. 猴子也學不會的SQLite(1)
date: 2016-01-04 11:13:42
tags: 
  - Android
  - SQLite
---

各位道友，貧道今天來分享一下魯魯der文章，在大家踏入這個Android開發以後，會發現其實SQLite在Android其實很常使用。

接下來就簡約介紹一下SQLite。

1. SQLite的優點
(1) 體積小，適用於一些簡單的應用也方便攜帶
(2) 在多讀少寫的情況下，讀取速度甚至優於MySQL
(3) 可以橫跨多種平台
(4) SQLite為檔案型資料庫，要備份直接Copy一個檔案就好。
(5) 支援多種語言

2. SQLite的缺點
(1) 容量受限
(2) 許多功能相較於一般資料庫來的少

其實SQLite的優點是大於缺點的，但對於大數據的情況下，SQLite是沒有辦法與知名資料庫比較的。更應該說，他們是不同領域下的佼佼者，在手機、嵌入式等等的小型系統，是非常受用的，反之。

那麼我們接下來教學將會以Android為主軸，來設計SQLite。

```java
public class MySQLiteManager extends SQLiteOpenHelper {
  
    // 資料庫版本
    private final static int DB_VERSION = 1;

    //資料庫名稱，附檔名為db
    private final static String DB_NAME = "MySQLite.db";
    
    //資料表名稱
    private final static String INFO_TABLE = "info_table";

    private final static String ROW_ID = "rowId";  

    private final static String NAME = "name";

    private final static String PHONE = "phone";

    public MySQLiteManager (Context context) {
          super(context, DB_NAME, null, DB_VERSION);
    }
    
    // 每次使用將會檢查是否有無資料表，若無，則會建立資料表
    @Override
    public void onCreate(SQLiteDatabase db) {
        //SQLite 所支援屬性帳面上的不多，但事實上也是能夠讀取一些其它屬性
        String createTable = "CREATE TABLE " + INFO_TABLE+ " ("
                          + ROW_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " + NAME + " TEXT, " + PHONE + " TEXT);";
        db.execSQL(createTable);
    }
    
    @Override
    public void onUpgrade(SQLiteDatabase db, int i, int i2) {
      
    }

}
```

這樣就可以建立出一個簡單的資料表了，大家不仿可以嘗試看看，
第一章節比較無聊與簡單，後面將會慢慢精彩起來，這也是必經過程，加油！
