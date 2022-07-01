[toc]



# 一	數據庫設計

用戶表

商家表

管理員表

商品表

訂單表

購物車

商品類型表

消費表

消費類型表

# 二	數據庫的創建

## 說明

我使用的是SqlSugar中的倉儲模式，中間出現一些問題，如：内部异常 1: FileNotFoundException: Could not load file or assembly 'Newtonsoft.Json, Version=10.0.0.0, Culture=neutral。這是版本不匹配的問題，如果沒有引用這個包，在Model層中引用包就行，如果引用了這個包，需要去改一下配置文件，改成符合條件的版本。(本次創建數據庫報錯時，我將沒有版本的包在Model層中引用，解決了問題。但引用了“報錯的包，版本號不匹配，需要修改版本號”這個是我在網上找的，沒有試過)

## 建庫建表

### 配置實體

| 特性                                               | 說明                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| IsIdentity=true                                    | 設置自增(如果是Oracle请设置OracleSequenceName 设置后和自增一样使用) |
| IsPrimaryKey=true                                  | 创建主键                                                     |
| ColumnDataType="指定類型" (如：nvarchar()，text等) | 指定數據庫中數據類型                                         |
| ColumnName                                         | 实体类数据库列名不一样设置数据库列名                         |
| IsIgnore                                           | ORM不处理该列 【忽略】                                       |
| IsOnlyIgnoreInsertIsOnlyIgnoreUpdate               | 插入操作时不处理该列 【插入中忽略】 更新操作不处理该列 【更新中忽略】 |
| OracleSequenceName                                 | 设置Oracle序列，设置后该列等同于自增列                       |



### 創建Model層

我使用的是code first，所以我先創建的是Model層，通過Model層生成數據庫和數據表。

首先，在Model層中使用NuGet引用SqlSugarCore包

創建類，創建的類到時候都會被映射成數據庫的表

BaseID類

~~~C#
using System;
using SqlSugar;

namespace Models.Entity
{
    public class BaseID
    {
        [SugarColumn(IsPrimaryKey =true,IsIdentity =true)]
        public int ID { get; set; }
    }
}

~~~

User類

~~~C#
using System;
using SqlSugar;

namespace Models.Entity
{
    /// <summary>
    /// 用戶類
    /// </summary>
    public class User:BaseID
    {
        [SugarColumn(ColumnDataType ="nvarchar(20)")]
        public string  UserName { get; set; }
        [SugarColumn(ColumnDataType ="nvarchar(20)")]
        public string  UserPwd { get; set; }
        [SugarColumn(ColumnDataType ="nvarchar(20)")]
        public string Name { get; set; }
        [SugarColumn(ColumnDataType ="varchar(11)")]
        public string UserPhone { get; set; }
        [SugarColumn(ColumnDataType ="nvarchar(100)")]
        public string DeliverAddress { get; set; }//收貨地址
        [SugarColumn(ColumnDataType ="nvarchar(200)")]
        public string DetailDeliverAddress { get; set; }//詳細收貨地址
        public double RemainingSum { get; set; }//余額
    }
}

~~~



### 創建Repository層

Repository層可以簡單理解為數據庫訪問層(DAL)，主要是對數據庫操作的封裝，需要什麼數據庫操作，就寫什麼對應的接口和方法。

Repository層主要包括兩個項目，IRepository(接口)和Repository(實現類)；在這兩項目中引用Model層

在IRepository中創建一個公共的接口IBaseRepository，IBaseRepository中主要放最底層的操作方法，引用SqlSugar命名空間。再寫一些單個類型的倉儲接口，繼承IBaseRepository接口。(例：Model層中有個user類，那麼IRepository中就要建立一個userIRepository接口)

在Repository中創建于IRepository對應的類，分別實現他們對應的接口。創建一個BaseRepository類，該類繼承并實現IBaseRepository接口；之後創建和Model層中的實體類對應的Repository類(例：Model層中有個user類，那麼Repository中就要建立一個userRepository類，該類繼承BaseRepository類和userIRepository接口)

### 創建Web API

創建一個web api，在web api中引用Model層，

