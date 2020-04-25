########################
產生查詢結果
########################

There are several ways to generate query results:

.. contents::
    :local:
    :depth: 2

*************
結果陣列
*************

**getResult()**

這個方法會將查詢結果以 *物件* 陣列或 *空* 陣列(查詢失敗時)回傳。
一般情況下可以使用foreach迴圈，參考以下範例：

::

    $query = $db->query("YOUR QUERY");

    foreach ($query->getResult() as $row)
    {
        echo $row->title;
        echo $row->name;
        echo $row->body;
    }

上述方法的別名是 ``getResultObject()`` 。

如果你想要得到多重陣列的查詢結果，你可以在參數中傳送 'array'：

::

    $query = $db->query("YOUR QUERY");

    foreach ($query->getResult('array') as $row)
    {
        echo $row['title'];
        echo $row['name'];
        echo $row['body'];
    }

上述方法的別名是 ``getResultArray()`` 。

你也可以傳送一個字串給 ``getResult()`` ，為每一個物件建立一個類別名稱。
which represents a class to instantiate for each result object

::

    $query = $db->query("SELECT * FROM users;");

    foreach ($query->getResult('User') as $user)
    {
        echo $user->name; // access attributes
        echo $user->reverseName(); // or methods defined on the 'User' class
    }

上述方法的別名是 ``getCustomResultObject()`` 。

**getResultArray()**

這個方法將會回傳純陣列或空陣列(查詢結果為空時)的查詢結果。
一般情況下可以使用foreach迴圈，參考以下範例：

::

    $query = $db->query("YOUR QUERY");

    foreach ($query->getResultArray() as $row)
    {
        echo $row['title'];
        echo $row['name'];
        echo $row['body'];
    }

***********
結果列
***********

**getRow()**

這個方法會回傳單筆查詢結果。如果你查詢的資料不只一筆，它只會回傳第一筆資料。
查詢結果會以 **物件** 的方式回傳。參考以下範例：

::

    $query = $db->query("YOUR QUERY");

    $row = $query->getRow();

    if (isset($row))
    {
        echo $row->title;
        echo $row->name;
        echo $row->body;
    }

如果你想要回傳特定的第幾筆資料，可以設定第一個參數為你想要的該筆資料：

::

	$row = $query->getRow(5);

你也可以傳入第二個參數，這個結果物件將被實體化為的類別名稱。

::

	$query = $db->query("SELECT * FROM users LIMIT 1;");
	$row = $query->getRow(0, 'User');

	echo $row->name; // access attributes
	echo $row->reverse_name(); // or methods defined on the 'User' class

**getRowArray()**

同等於上述 ``row()`` 方法，不同的是它回傳是陣列。參考以下範例：

::

    $query = $db->query("YOUR QUERY");

    $row = $query->getRowArray();

    if (isset($row))
    {
        echo $row['title'];
        echo $row['name'];
        echo $row['body'];
    }

如果你想要回傳特定的第幾筆資料，可以設定第一個參數為你想要的該筆資料：

::

	$row = $query->getRowArray(5);

除此之外，你可以利用底下方式查詢到 前一筆/下一筆/第一筆/最後一筆 資料：

	| **$row = $query->getFirstRow()**
	| **$row = $query->getLastRow()**
	| **$row = $query->getNextRow()**
	| **$row = $query->getPreviousRow()**

預設回傳值為物件，除非設定第一個參數為 "array"：

	| **$row = $query->getFirstRow('array')**
	| **$row = $query->getLastRow('array')**
	| **$row = $query->getNextRow('array')**
	| **$row = $query->getPreviousRow('array')**

.. note:: 上述的所有方法都會將結果載入到記憶體(prefetching)中。使用 ``getUnbufferedRow()`` 處理大量的資料集。

**getUnbufferedRow()**

這個方法會回傳單筆查詢結果，但並不會像 ``row()`` 將所有查詢結果預載到記憶體中。
如果你的查詢結果超過一筆，它將回傳該筆資料並將內部資料指標移動到下一筆資料。

::

    $query = $db->query("YOUR QUERY");

    while ($row = $query->getUnbufferedRow())
    {
        echo $row->title;
        echo $row->name;
        echo $row->body;
    }

你可以選擇傳入 'object' （預設）或 'array' 來指定回傳的資料型態：

::

	$query->getUnbufferedRow();		    // object
	$query->getUnbufferedRow('object');	// object
	$query->getUnbufferedRow('array');	// associative array

*********************
客製化查詢結果
*********************

你可以將查詢結果以自定義類別的實例做回傳，而不是像 ``getResult()`` 和 ``getResultArray()`` ，以stdClass或陣列的方式回傳。
如果類別沒有載入到記憶體中，Autoloader將會自動載入。
物件將會把資料庫中回傳的所有變數設定為屬性。
如果這些變數已經被宣告為非公開，那你應該要提供一個 ``__set()`` 的方法，讓它可以被設定。

範例：

::

	class User
	{
		public $id;
		public $email;
		public $username;

		protected $last_login;

		public function lastLogin($format)
		{
			return $this->lastLogin->format($format);
		}

		public function __set($name, $value)
		{
			if ($name === 'lastLogin')
			{
				$this->lastLogin = DateTime::createFromFormat('U', $value);
			}
		}

		public function __get($name)
		{
			if (isset($this->$name))
			{
				return $this->$name;
			}
		}
	}

除了以下列出的兩個方法之外，這些方法(例如： ``getFirstRow()``, ``getLastRow()``,
``getNextRow()``, and ``getPreviousRow()`` )也可以使用類別名稱的方式做回傳查詢結果。

**getCustomResultObject()**

以類別的實例之陣列an array of instances of the class requested.，回傳所有查詢結果集。
唯一要傳入的參數是要做為類別實例的名稱class to instantiate.

範例：

::

	$query = $db->query("YOUR QUERY");

	$rows = $query->getCustomResultObject('User');

	foreach ($rows as $row)
	{
		echo $row->id;
		echo $row->email;
		echo $row->last_login('Y-m-d');
	}

**getCustomRowObject()**

從查詢結果中回傳單筆資料。第一個參數為要回傳的該筆資料，第二個參數為實例化的類別名稱。
The second parameter is the class name to instantiate.

範例：

::

	$query = $db->query("YOUR QUERY");

	$row = $query->getCustomRowObject(0, 'User');

	if (isset($row))
	{
		echo $row->email;   // access attributes
		echo $row->last_login('Y-m-d');   // access class methods
	}

你也能以完全相同的方式，使用 ``getRow()`` 這個方法。

範例：

::

	$row = $query->getCustomRowObject(0, 'User');

*********************
結果輔助方法
*********************

**getFieldCount()**

回傳查詢後的欄位數量。確保使用查詢結果物件來呼叫這個方法。

::

	$query = $db->query('SELECT * FROM my_table');

	echo $query->getFieldCount();

**getFieldNames()**

回傳查詢後的欄位名稱陣列，確保使用查詢結果物件來呼叫這個方法。

::

    $query = $db->query('SELECT * FROM my_table');

	echo $query->getFieldNames();

**freeResult()**

它將會釋放查詢後的記憶體並且刪除資源ID。
通常PHP會在腳本執行結束後自動釋放記憶體。
但是，如果你是在特定腳本中運行大量資料的查詢，你可能需要在每次查詢都需要釋放記憶體，以減少記憶體的消耗。

範例：

::

	$query = $thisdb->query('SELECT title FROM my_table');

	foreach ($query->getResult() as $row)
	{
		echo $row->title;
	}

	$query->freeResult();  // The $query result object will no longer be available

	$query2 = $db->query('SELECT name FROM some_table');

	$row = $query2->getRow();
	echo $row->name;
	$query2->freeResult(); // The $query2 result object will no longer be available

**dataSeek()**

這個方法是設定下一次查詢的索引。它只會與 ``getUnbufferedRow()`` 結合使用。

它容納一個正整數，預設為0，成功回傳TRUE；失敗回傳FALSE。

::

	$query = $db->query('SELECT `field_name` FROM `table_name`');
	$query->dataSeek(5); // 忽略前5筆
	$row = $query->getUnbufferedRow();

.. note:: 並不是所有的資料庫驅動都會支援此方法或回傳FALSE。例如：你不可能在PDO中使用此方法。

***************
物件參考
***************

.. php:class:: CodeIgniter\\Database\\BaseResult

	.. php:method:: getResult([$type = 'object'])

		:param	string	$type: Type of requested results - array, object, or class name
		:returns:	Array containing the fetched rows
		:rtype:	array

		A wrapper for the ``getResultArray()``, ``getResultObject()``
		and ``getCustomResultObject()`` methods.

		Usage: see `結果陣列`_.

	.. php:method:: getResultArray()

		:returns:	Array containing the fetched rows
		:rtype:	array

		Returns the query results as an array of rows, where each
		row is itself an associative array.

		Usage: see `結果陣列`_.

	.. php:method:: getResultObject()

		:returns:	Array containing the fetched rows
		:rtype:	array

		Returns the query results as an array of rows, where each
		row is an object of type ``stdClass``.

		Usage: see `結果陣列`_.

	.. php:method:: getCustomResultObject($class_name)

		:param	string	$class_name: Class name for the resulting rows
		:returns:	Array containing the fetched rows
		:rtype:	array

		Returns the query results as an array of rows, where each
		row is an instance of the specified class.

	.. php:method:: getRow([$n = 0[, $type = 'object']])

		:param	int	$n: Index of the query results row to be returned
		:param	string	$type: Type of the requested result - array, object, or class name
		:returns:	The requested row or NULL if it doesn't exist
		:rtype:	mixed

		A wrapper for the ``getRowArray()``, ``getRowObject()`` and
		``getCustomRowObject()`` methods.

		Usage: see `結果列`_.

	.. php:method:: getUnbufferedRow([$type = 'object'])

		:param	string	$type: Type of the requested result - array, object, or class name
		:returns:	Next row from the result set or NULL if it doesn't exist
		:rtype:	mixed

		Fetches the next result row and returns it in the
		requested form.

		Usage: see `結果列`_.

	.. php:method:: getRowArray([$n = 0])

		:param	int	$n: Index of the query results row to be returned
		:returns:	The requested row or NULL if it doesn't exist
		:rtype:	array

		Returns the requested result row as an associative array.

		Usage: see `結果列`_.

	.. php:method:: getRowObject([$n = 0])

		:param	int	$n: Index of the query results row to be returned
                :returns:	The requested row or NULL if it doesn't exist
		:rtype:	stdClass

		Returns the requested result row as an object of type
		``stdClass``.

		Usage: see `結果列`_.

	.. php:method:: getCustomRowObject($n, $type)

		:param	int	$n: Index of the results row to return
		:param	string	$class_name: Class name for the resulting row
		:returns:	The requested row or NULL if it doesn't exist
		:rtype:	$type

		Returns the requested result row as an instance of the
		requested class.

	.. php:method:: dataSeek([$n = 0])

		:param	int	$n: Index of the results row to be returned next
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		Moves the internal results row pointer to the desired offset.

		Usage: see `結果輔助方法`_.

	.. php:method:: setRow($key[, $value = NULL])

		:param	mixed	$key: Column name or array of key/value pairs
		:param	mixed	$value: Value to assign to the column, $key is a single field name
		:rtype:	void

		Assigns a value to a particular column.

	.. php:method:: getNextRow([$type = 'object'])

		:param	string	$type: Type of the requested result - array, object, or class name
		:returns:	Next row of result set, or NULL if it doesn't exist
		:rtype:	mixed

		Returns the next row from the result set.

	.. php:method:: getPreviousRow([$type = 'object'])

		:param	string	$type: Type of the requested result - array, object, or class name
		:returns:	Previous row of result set, or NULL if it doesn't exist
		:rtype:	mixed

		Returns the previous row from the result set.

	.. php:method:: getFirstRow([$type = 'object'])

		:param	string	$type: Type of the requested result - array, object, or class name
		:returns:	First row of result set, or NULL if it doesn't exist
		:rtype:	mixed

		Returns the first row from the result set.

	.. php:method:: getLastRow([$type = 'object'])

		:param	string	$type: Type of the requested result - array, object, or class name
		:returns:	Last row of result set, or NULL if it doesn't exist
		:rtype:	mixed

		Returns the last row from the result set.

	.. php:method:: getFieldCount()

		:returns:	Number of fields in the result set
		:rtype:	int

		Returns the number of fields in the result set.

		Usage: see `結果輔助方法`_.

    .. php:method:: getFieldNames()

		:returns:	Array of column names
		:rtype:	array

		Returns an array containing the field names in the
		result set.

	.. php:method:: getFieldData()

		:returns:	Array containing field meta-data
		:rtype:	array

		Generates an array of ``stdClass`` objects containing
		field meta-data.

	.. php:method:: freeResult()

		:rtype:	void

		Frees a result set.

		Usage: see `結果輔助方法`_.
