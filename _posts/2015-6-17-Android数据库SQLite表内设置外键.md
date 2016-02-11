---
layout: post
author: mxn
title: Android数据库SQLite表内设置外键
category: 技术博文
tag: [android]
---

####介绍

Android默认的数据是SQLite，但SQLite3.6.19之前(在2.2版本中使用的是3.6.22，因此如果你的应用只兼容到2.2版本就可以放心使用外键功能)是不支持外键的，如果有两张表需要关联，用外键是最省事的，但不支持的话怎么办呢？这里就有一个解决办法，就是用事务将两张表关联起来，并且最后生成一张视图。

####现有两张表
Employees
Dept
####视图
![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img2.JPG)
ViewEmps:显示雇员信息和他所在的部门.

 <!-- more -->
 
####创建数据库
自定义一个辅助类继承SQLiteOpenHelper类.

onCreate(SQLiteDatabase db): 当数据库被创建的时候，能够生成表，并创建视图跟触发器。
onUpgrade(SQLiteDatabse db, int oldVersion, int newVersion): 更新的时候可以删除表和创建新的表。

代码如下：
{% highlight java %}
public class DatabaseHelper extends SQLiteOpenHelper {  
    static final String dbName="demoDB";  
    static final String employeeTable="Employees";  
    static final String colID="EmployeeID";  
    static final String colName="EmployeeName";  
    static final String colAge="Age";  
    static final String colDept="Dept";  
    static final String deptTable="Dept";  
    static final String colDeptID="DeptID";  
    static final String colDeptName="DeptName";  
    static final String viewEmps="ViewEmps";  
    public DatabaseHelper(Context context) {  
      super(context, dbName, null,33);   
    }  
    // 创建库中的表，视图和触发器
    public void onCreate(SQLiteDatabase db) {  
      db.execSQL("CREATE TABLE "+deptTable+" ("+colDeptID+ " INTEGER PRIMARY KEY , "+  
        colDeptName+ " TEXT)");  
      db.execSQL("CREATE TABLE "+employeeTable+"   
        ("+colID+" INTEGER PRIMARY KEY AUTOINCREMENT, "+  
            colName+" TEXT, "+colAge+" Integer, "+colDept+"   
        INTEGER NOT NULL ,FOREIGN KEY ("+colDept+") REFERENCES   
        "+deptTable+" ("+colDeptID+"));");  
      //创建触发器  
      db.execSQL("CREATE TRIGGER fk_empdept_deptid " +  
        " BEFORE INSERT "+  
        " ON "+employeeTable+  
        " FOR EACH ROW BEGIN"+  
        " SELECT CASE WHEN ((SELECT "+colDeptID+" FROM "+deptTable+"   
        WHERE "+colDeptID+"=new."+colDept+" ) IS NULL)"+  
        " THEN RAISE (ABORT,'Foreign Key Violation') END;"+  
        "  END;");  
     //创建视图  
      db.execSQL("CREATE VIEW "+viewEmps+  
        " AS SELECT "+employeeTable+"."+colID+" AS _id,"+  
        " "+employeeTable+"."+colName+","+  
        " "+employeeTable+"."+colAge+","+  
        " "+deptTable+"."+colDeptName+""+  
        " FROM "+employeeTable+" JOIN "+deptTable+  
        " ON "+employeeTable+"."+colDept+" ="+deptTable+"."+colDeptID  
        );  
     }  
    // 更新库中的表
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {  
          db.execSQL("DROP TABLE IF EXISTS "+employeeTable);  
          db.execSQL("DROP TABLE IF EXISTS "+deptTable);  
          db.execSQL("DROP TRIGGER IF EXISTS fk_empdept_deptid");  
          db.execSQL("DROP VIEW IF EXISTS "+viewEmps);  
          onCreate(db);  
     }  
}
{% endhighlight  %}

加入数据

{% highlight java %}
SQLiteDatabase db=this.getWritableDatabase();  
 ContentValues cv=new ContentValues();  
   cv.put(colDeptID, 1);  
   cv.put(colDeptName, "Sales");  
   db.insert(deptTable, colDeptID, cv);  
   cv.put(colDeptID, 2);  
   cv.put(colDeptName, "IT");  
   db.insert(deptTable, colDeptID, cv);  
                     db.close();  
{% endhighlight  %}

更新数据

{% highlight java %}
public int UpdateEmp(Employee emp)  
  {  
   SQLiteDatabase db=this.getWritableDatabase();  
   ContentValues cv=new ContentValues();  
   cv.put(colName, emp.getName());  
   cv.put(colAge, emp.getAge());  
   cv.put(colDept, emp.getDept());  
   return db.update(employeeTable, cv, colID+"=?",   
    new String []{String.valueOf(emp.getID())});     
  }  
{% endhighlight  %}


删除数据
{% highlight java %}
public void DeleteEmp(Employee emp)  
  {  
   SQLiteDatabase db=this.getWritableDatabase();  
   db.delete(employeeTable,colID+"=?", new String [] {String.valueOf(emp.getID())});  
   db.close();  
  }  
{% endhighlight  %}

取得所有部门信息

{% highlight java %}
Cursor getAllDepts()  
  {  
   SQLiteDatabase db=this.getReadableDatabase();  
   Cursor cur=db.rawQuery("SELECT "+colDeptID+" as _id,   
    "+colDeptName+" from "+deptTable,new String [] {});  
 
   return cur;  
  }  
{% endhighlight  %}

取得部门内雇员信息
{% highlight java %}
public Cursor getEmpByDept(String Dept)  
  {  
   SQLiteDatabase db=this.getReadableDatabase();  
   String [] columns=new String[]{"_id",colName,colAge,colDeptName};  
   Cursor c=db.query(viewEmps, columns, colDeptName+"=?",   
    new String[]{Dept}, null, null, null);  
   return c;  
  }  
{% endhighlight  %}

取得部门ID
{% highlight java %}
public int GetDeptID(String Dept)  
  {  
   SQLiteDatabase db=this.getReadableDatabase();  
   Cursor c=db.query(deptTable, new String[]{colDeptID+" as _id",colDeptName},  
    colDeptName+"=?", new String[]{Dept}, null, null, null);  
   //Cursor c=db.rawQuery("SELECT "+colDeptID+" as _id FROM "+deptTable+"   
   //WHERE "+colDeptName+"=?", new String []{Dept});  
   c.moveToFirst();  
   return c.getInt(c.getColumnIndex("_id"));    
  }  
  {% endhighlight  %}
 上面部门和雇员信息的表因为实现的关联，所以更新和删除都会对对应的信息更新。
