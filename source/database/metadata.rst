#################
資料庫詮釋資料
#################

.. contents::
    :local:
    :depth: 2

**************
資料表詮釋資料
**************

這些函數讓你可以提取資料表的資訊。

列出資料庫的資料表
================================

**$db->listTables();**

回傳一個包含你現在連線的資料庫裡所有資料表名稱。範例：

::

	$tables = $db->listTables();

    foreach ($tables as $table) {
        echo $table;
    }

.. note:: 有些資料庫內部具有系統的資料表，上述函數並不會回傳這些系統資料表。

確認資料表是否存在
===========================

**$db->tableExists();**

有時候在操作前確定特定資料表是否存在是重要的。回傳布林值 TRUE/FALSES。範例：

::

    if ($db->tableExists('table_name')) {
        // some code...
    }

.. note:: 將 *table_name* 替換成你想要尋找的資料表名稱。

**************
欄位詮釋資料
**************

列出資料表中的欄位
==========================

**$db->getFieldNames()**

回傳具有欄位名稱的陣列。這個查詢可以使用以下兩中方是呼叫：

1. 你可以傳入資料表名稱，並從 ``$db->object`` 中取得欄位名稱。

::

    $fields = $db->getFieldNames('table_name');

    foreach ($fields as $field) {
        echo $field;
    }

2. 你可以從查詢結果物件中，呼叫該函數得到相關聯查詢的欄位名稱：

::

    $query = $db->query('SELECT * FROM some_table');

    foreach ($query->getFieldNames() as $field) {
        echo $field;
    }

確認欄位是否出現在資料表
==========================================

**$db->fieldExists()**

有時候在操作前確定特定欄位是否存在是重要的。回傳布林值 TRUE/FALSES。範例：

::

	if ($db->fieldExists('field_name', 'table_name')) {
		// some code...
	}

.. note:: 將 *field_name* 替換成你想要尋找的欄位名稱，將 *table_name* 替換成你想要尋找的資料表名稱。

檢索欄位的詮釋資料
=======================

**$db->getFieldData()**

回傳具有欄位資訊的陣列物件。

有時候收集欄位名稱或其他詮釋資料是重要的，像是：欄位類型、最大長度等等。

.. note:: 並不是所有資料庫都有提供詮釋資料

使用範例：

::

    $fields = $db->getFieldData('table_name');

    foreach ($fields as $field) {
        echo $field->name;
        echo $field->type;
        echo $field->max_length;
        echo $field->primary_key;
    }

如果你已經執行查詢，則可以使用結果物件而不是資料表名稱

::

	$query  = $db->query("YOUR QUERY");
	$fields = $query->fieldData();

如果你的資料庫有支援，以下的內容是可以從上述函數得到的資訊：

-  name - 欄位名稱
-  max_length - 欄位的最大長度
-  primary_key - 如果該欄位是主鍵則為 1
-  type - 欄位的類型

列出資料表的索引
===========================

**$db->getIndexData()**

回傳具有索引資訊的陣列物件。

使用範例：

::

    $keys = $db->getIndexData('table_name');

    foreach ($keys as $key) {
        echo $key->name;
        echo $key->type;
        echo $key->fields; // array of field names
    }

鍵值類型在你使用的資料庫中應該是唯一。例如：MySQL會為每個跟資料表有關聯的鍵值，回傳主鍵、全文索引、空間索引、或唯一索引的其中一種。

**$db->getForeignKeyData()**

回傳一個包含外來鍵資訊的物件陣列

使用範例：

::

    $keys = $db->getForeignKeyData('table_name');

    foreach ($keys as $key) {
        echo $key->constraint_name;
        echo $key->table_name;
        echo $key->column_name;
        echo $key->foreign_table_name;
        echo $key->foreign_column_name;
    }

在你使用的資料庫中，物件欄位可能是唯一的。例如：SQLite3不會回傳欄位名稱，但對複合外來鍵會 *排序* 欄位。