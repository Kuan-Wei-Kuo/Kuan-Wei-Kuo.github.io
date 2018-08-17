---
title: Android. Export from sqlite to excel
date: 2015-12-21 10:25:00
tags: Android
---

最近剛好有學習到如何將SQLite的資料拉出來建立一個Excel在手機上。

# 下載[Jexcel.jar](http://www.java2s.com/Code/Jar/j/Downloadjxljar.htm)
# 建立AsyncTask
```java
public class ExportDatabaseExcelTask extends AsyncTask<String, Void, Boolean> {

    private Context context;

    // 這個可以快速顯示一個Progress
    private ProgressDialog progressDialog; 


    public ExportDatabaseExcelTask(Context context) {
        this.context = context;
        progressDialog = new ProgressDialog(context);
    }
    
    @Override
    protected void onPreExecute() {
        super.onPreExecute();
        progressDialog.setMessage("正在匯出Excel檔");
        progressDialog.show();
    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        super.onPostExecute(aBoolean);
        if(progressDialog.isShowing())
            progressDialog.dismiss();

        if(!aBoolean)
            Toast.makeText(context, "匯出失敗", Toast.LENGTH_SHORT).show();
    }
    
    @Override
    protected Boolean doInBackground(String... params) {
        // 創立Excel的檔名
        String fileName = "Test" + ".xls";

        // 選擇要匯出的Table
        SQLiteManager sqLiteManager = new SQLiteManager(context);
        SQLiteDatabase db = sqLiteManager.getReadableDatabase();
        Cursor cursor = db.rawQuery("SELECT * FROM " + CurrentAccountData.getMoneyTableName(), null);
       
        // 將會在內部空間內建立一個資料夾
        File exportDir = new File(Environment.getExternalStorageDirectory().getPath(), "TestExport");

        if (!exportDir.exists())
            exportDir.mkdirs();

        // 在TestExport下創建一個檔案
        File file = new File(exportDir, fileName);
        
        /設定輸出相關格式
        WorkbookSettings wbSettings = new WorkbookSettings();
        wbSettings.setLocale(new Locale("en", "EN"));
        WritableWorkbook workbook;
        try {
            // 創立一個Excel檔
            workbook = Workbook.createWorkbook(file, wbSettings);
            
            // 建立分頁
            WritableSheet sheet = workbook.createSheet(CurrentAccountData.getAccountName(), 0);
            /**
            * 如果要建立第二分頁就會像 workbook.createSheet(CurrentAccountData.getAccountName(), 1);
            */

            //先建立我們Excel的標題
            sheet.addCell(new Label(0, 0, "學號"));
            sheet.addCell(new Label(1, 0, "姓名"));

            //開始放入表的資料，這邊請都用getString()
            if (cursor.moveToFirst()) {
                do {
                    int i = cursor.getPosition() + 1;
                    sheet.addCell(new Label(0, i, cursor.getString(0)));
                    sheet.addCell(new Label(1, i, cursor.getString(1)));
                } 
                while (cursor.moveToNext());
            }
            cursor.close();
            workbook.write();
            workbook.close();
            return true;
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }  catch (RowsExceededException e) {
            e.printStackTrace();
            return false;
        } catch (WriteException e) {
            e.printStackTrace();
            return false;
        }
    }
}
```
Done！
其實不管是Csv還是Excel最終都還是得將Table的資料一一取出來，然後再使用迴圈建立資料，因此使用AsnyTask是有其道理的，有人會說怎麼不使用Thread、Handler，其實效果大同小異。

但AsnyTask是為了Android而產生的，它在Android比上述兩個速度更快，美中不足的是沒辦法像Thread可以運算非常大量的資料。