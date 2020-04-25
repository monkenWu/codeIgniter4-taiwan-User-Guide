##################################
快速入門： 範例程式
##################################

以下的頁面包含了示範如何使用資料庫類別的範例程式。請閱讀各個函數的說明頁面來了解完整的細節。

初始化資料庫類別
===============================

以下的程式根據你 :doc:`組態設定檔 <configuration>` 中的設定載入並初始化資料庫類別：

::

	$db = \Config\Database::connect();

類別一旦被載入，就可以像以下的說明做使用：

注意：如果你的所有頁面都需要存取資料庫，你可以設定自動連線。請參考 :doc:`資料庫連線 <connecting>` 來了解細節。

標準的多結果查詢（物件版）
=====================================================

::

	$query = $db->query('SELECT name, title, email FROM my_table');
	$results = $query->getResult();

	foreach ($results as $row)
	{
		echo $row->title;
		echo $row->name;
		echo $row->email;
	}

	echo 'Total Results: ' . count($results);

以上的 getResult() 函數會回傳一個 **物件** 陣列。範例：$row->title。

標準的多結果查詢（陣列版）
====================================================

::

	$query   = $db->query('SELECT name, title, email FROM my_table');
	$results = $query->getResultArray();

	foreach ($results as $row)
	{
		echo $row['title'];
		echo $row['name'];
		echo $row['email'];
	}

以上的 getResultArray() 函數會回傳一個標準的索引陣列。範例：$row['title']。

標準的單一結果查詢（物件版）
=================================

::

	$query = $db->query('SELECT name FROM my_table LIMIT 1');
	$row   = $query->getRow();
	echo $row->name;

以上的 getRow() 函數會回傳一個 **物件**。範例：$row->name。

標準的單一結果查詢（陣列版）
=================================================

::

	$query = $db->query('SELECT name FROM my_table LIMIT 1');
	$row   = $query->getRowArray();
	echo $row['name'];

以上的 getRowArray() 函數會回傳一個 **陣列**。範例：$row['name']。

標準新增
===============

::

	$sql = "INSERT INTO mytable (title, name) VALUES (".$db->escape($title).", ".$db->escape($name).")";
	$db->query($sql);
	echo $db->affectedRows();

查詢生成器 查詢
===================

:doc:`查詢生成器 <query_builder>` 提供一個取得資料的簡單方法：

::

	$query = $db->table('table_name')->get();

	foreach ($query->getResult() as $row)
	{
		echo $row->title;
	}

以上的 get() 函數會回傳指定的資料表中所有的結果。:doc:`查詢生成器 <query_builder>` 類別包含完整的資料操作方法

查詢生成器 新增
====================

::

	$data = [
		'title' => $title,
		'name'  => $name,
		'date'  => $date
	];

	$db->table('mytable')->insert($data);  // Produces: INSERT INTO mytable (title, name, date) VALUES ('{$title}', '{$name}', '{$date}')