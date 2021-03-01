#####################
呼叫自定函數
#####################

$db->callFunction();
============================

這個函數使你能以無關平台的方式呼叫 CodeIgniter 中，沒有實作到的的 PHP 資料庫函數。例如：假設你要呼叫 ``mysql_get_client_info()`` ，但是 CodeIgniter 並沒有支援該函數，你就可以這樣做：

::

	$db->callFunction('get_client_info');

在第一個參數中你必須放入函數的名稱 **不帶有** mysql\_ 的字首。字首會根據你現在使用的資料庫驅動自動加上。這可以讓你在不同的資料庫系統中，執行相同的函數。但是，在資料庫系統之間，並不是所有的函數使用方式都一樣，所以這個函數在移植性方面還是有限制的。

你使用的函數如果有任何參數，要加在第二個參數上。

::

	$db->callFunction('some_function', $param1, $param2, etc..);

通常你需要提供資料庫連線 ID 或資料庫結果 ID 。連線 ID 可以使用以下方式取得：

::

	$db->connID;

可以從查詢物件中取得資料庫結果 ID ，例如：

::

	$query = $db->query("SOME QUERY");

	$query->resultID;