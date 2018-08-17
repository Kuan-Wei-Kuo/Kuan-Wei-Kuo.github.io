---
title: Android. 猴子也學不會的SQLite(2)
date: 2016-01-10 09:56:42
tags:
  - Android
  - SQLite
---

在上一篇文章想必大家都建立好了一個資料庫(檔案型)，那麼接下來就是重頭戲了，如何新增、修改、刪除以及取得資料等動作呢?

# 新增
```java
public void insterInfo(SQLiteDatabase db, String name, String phone) {
    ContentValues contentValues = new ContentValues();
    contentValues.put(NAME, name);
    contentValues.put(PHONE, phone);

    db.insert(INFO_TABLE, null, contentValues);
}
```
在第一章節我們建立了INFO TABLE，內部欄位為RowId, Name, Phone，但因RowId每次自動累加，因此不需在新增一次。

# 修改
```java
public void updateInfo(SQLiteDatabase db, String rowId, String name, String phone) {
    ContentValues contentValues = new ContentValues();
    contentValues.put(NAME, name);
    contentValues.put(PHONE, phone);

    //問號對應到後面陣列的資料
    db.update(INFO_TABLE, contentValues, ROW_ID + "=?", new String[] {rowId});
}
```
修改部分與新增大同小異，唯一不同的就是多了個RowId，這個欄位是我們主要索引值，當RowId = 1時，那麼我們將會把新的資料塞進去第一筆資料當中，以此類推。

# 刪除
```java
public void removeInfo(SQLiteDatabase db, String rowId) {
    db.delete(INFO_TABLE, ROW_ID + "=?", new String[]{rowId});
}
```
利用PK值進行刪除，如果想要多欄位刪除的話，需要自行設計，方法雷同。

# 取得資料
```java
public Cursor getInfO(SQLiteDatabase db) {
    return  db.query(INFO_TABLE, new String[] {ROW_ID, NAME, PHONE}, null, null, null, null, null);
}
```
回傳值是Cursor物件，我們看看Cursor物件的使用方式

## Cursor使用方法
```java
MySQLiteManager mSQLiteManager = new MySQLiteManager(this);
Cursor mCursor = mSQLiteManager.getInfo(mSQLiteManager.getReadableDatabase());
if(mCursor != null) {
    mCursor.moveToFirst();
    for(int i = 0 ; i < mCursor.getCount() ; i++) {
        //Do Something
        mCursor.moverToNext();
    }
}
```

好了今天就到這邊，其實也就是一般的SQL操作而已。