---
title: Android. 猴子也學不會的SQLite(3)
date: 2016-01-18 11:05:42
tags:
  - Android
  - SQLite
---

有時候會因應一些需求，需要修改資料庫欄位設計，這時候SQLite稍嫌不太方便，需要利用onUpgrade方法，進行ALTER動作。

```java
private final static String DATE = "date"; //欄位名稱

@Override
public void onUpgrade(SQLiteDatabase sqLiteDatabase, int oldVersion, int newVersion) {
    if(newVersion > oldVersion) {
        sqLiteDatabase.beginTransaction();
    if(oldVersion == 1) {
        try {
            // 新增一個日期欄位，預設為NULL字串
            sqLiteDatabase.execSQL("ALTER TABLE " + INFO_TABLE + " ADD COLUMN " + DATE + " TEXT DEFAULT " + "NULL";
            Log.d("SQLite", "版本更新成功，結束交易...");
            sqLiteDatabase.setTransactionSuccessful();
            sqLiteDatabase.endTransaction();
        } catch (Exception e) {
            Log.d("SQLite", "版本更新失敗，結束交易...");
            sqLiteDatabase.endTransaction();
        }        
    } else {
        onCreate(sqLiteDatabase);
    }
}
```

由上述程式碼可以看出我們新增需要自己去編寫程式碼，就是這麼簡單，但SQLite是放在使用者的地方，因此也不建議大量更新資料庫。
