####################
查詢輔助函數
####################

取得執行查詢的資訊
==================================

**$db->insertID()**

資料庫執行新增時的新增ID。

.. note:: 如果將 PDO 驅動連接 PostgreSQL 或是使用 Interbase 驅動，這個函數需要 $name 這個參數。用來指定適當的序列來檢查新增ID。

**$db->affectedRows()**

當執行 "寫入" 類型的查詢（新增、更新等等）時，顯示受影響的列數。

.. note:: 在 MySQL 中， "DELETE FROM TABLE" 會回傳 0 個受影響的列數。資料庫類別做了一個小 hack ，讓它可以回傳正確受影響的列數。在預設下，這個hack是被開啟的，不過也可以從資料庫驅動程式檔案中將它關閉。

**$db->getLastQuery()**

回傳最近一次的查詢物件（查詢字串，不是查詢結果）。

取得資料庫資訊
===============================

**$db->countAll()**

讓你可以知道一個資料表中資料的列數。第一個參數為資料表名稱。這是查詢生成器的一部分。範例：

::

	echo $db->table('my_table')->countAll();

	// Produces an integer, like 25

**$db->getPlatform()**

輸出你目前在運作的資料庫系統（例如：MySQL、MS SQL、Postgres等等）：

::

	echo $db->getPlatform();

**$db->getVersion()**

輸出你目前在運作的資料庫版本：

::

	echo $db->getVersion();
