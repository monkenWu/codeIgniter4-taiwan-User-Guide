###########################
執行查詢
###########################

.. contents::
    :local:
    :depth: 2

************
查詢基礎
************

常規的查詢
===============

可以使用 **query** 來提交查詢：

::

	$db->query('YOUR QUERY HERE');

當對資料庫進行 "讀取" 查詢時，``query()`` 函數會以 **物件** 型態回傳資料庫產生的 :doc:`查詢結果 <results>`。當對資料庫進行 "寫入" 查詢時，會依據執行的成功或失敗來回傳TRUE 或 FALSE。
當對資料庫進行 "檢索" 查詢時，通常使用一個變數來接收查詢的結果，例如：

::

	$query = $db->query('YOUR QUERY HERE');

簡單化的查詢
==================

**simpleQuery** 方法是 ``$db->query()`` 的簡化版。它不會回傳資料庫結果集合，不會設定查詢計時器，不會編譯資料，不會儲存你的查詢以便除錯。它只是讓你提交查詢。大多數使用者很少會使用到這個函數。

它回傳資料庫驅動的 "execute" 函數的回傳的結果。通常在執行類型查詢，如： INSERT 、 DELETE ，或 UPDATE 語法（合適的應用時機），會根據成功或失敗，回傳TRUE/FALSE。對於可獲得結果的查詢，則會回傳一個資源／物件。

::

	if ($db->simpleQuery('YOUR QUERY'))
	{
		echo "Success!";
	}
	else
	{
		echo "Query failed!";
	}

.. note:: 在 PostgreSQL 中，例如： ``pg_exec()`` 函數在執行查詢成功後，都只會回傳資源，即使是執行類型查詢也是一樣。所以如果你是要找布林值的話，請注意這一點。

***************************************
手動設定資料庫字首
***************************************

如果你有設定資料庫字首，並且想要它加入在資料表名稱前，例如當你使用原生SQL，則你可以使用以下的語法：

::

	$db->prefixTable('tablename'); // outputs prefix_tablename

如果你因為某些原因需要修改字首，並不需要建立一個新的連線，可以使用以下方法：

::

	$db->setPrefix('newprefix');
	$db->prefixTable('tablename'); // outputs newprefix_tablename

你可以利用以下方式，隨時取得目前字首：

::

	$DBPrefix = $db->getPrefix();

**********************
保護標識符號
**********************

在許多資料庫中，保護資料表和欄位名稱是重要的。例如：在 MySQL 中使用反引號（backticks）。但是 **查詢生成器會自動保護** ，如果你需要手動保護這些標示符號，可以使用以下的方法：

::

	$db->protectIdentifiers('table_name');

.. important:: 儘管查詢生成器會盡力地正確使用你所提供的任何欄位和資料表名稱。請注意，它的設計並不是讓使用者任意的輸入。所以請不要輸入未經消毒的資料。

假如你的資料庫設定檔中有指定字首，這個函數會在你的資料表前加上字首。你可以設定第二個參數為 TRUE (boolean) 來啟動字首設定：

::

	$db->protectIdentifiers('table_name', TRUE);

****************
跳脫查詢
****************
在提交資料到資料庫前，先進行資料跳脫，這是一個非常良好的安全習慣。 CodeIgniter 有三種方法可以幫助你做到這一點：

#. **$db->escape()** 這個函數根據資料型態來運作，所以它只會對字串型態的資料進行跳脫。它也會自動增加單引號在字串的兩側，所以你不必加上單引號：

    ::

	$sql = "INSERT INTO table (title) VALUES(".$db->escape($title).")";

#. **$db->escapeString()** 這個函數可以跳脫任何資料型態。在大多數的情況下，你會看到以上的方式，而不是以下的。要使用這樣的功能可以參考以下：

    ::

	$sql = "INSERT INTO table (title) VALUES('".$db->escapeString($title)."')";

#. **$db->escapeLikeString()** 這個函數被用來使用在 SQL LIKE 的語法當中，像是萬用字元（ % ，\_ ）在此函數中也會被跳脫。

    ::

        $search = '20% raise';
        $sql = "SELECT id FROM table WHERE column LIKE '%" .
        $db->escapeLikeString($search)."%' ESCAPE '!'";

.. important:: ``escapeLikeString()`` 方法使用 "!" （驚嘆號） 來跳脫 *LIKE* 條件的特殊字元。因為這個方法可以跳脫部分的字串，你可以自己用引號進行封裝，所以這個方法無法替你自動加入 ``ESCAPE '!'`` 的條件，你需要手動加入。

**************
封裝查詢
**************

封裝可以簡化你的查詢語法，讓系統為你的查詢放入資料。請參考以下範例：

::

	$sql = "SELECT * FROM some_table WHERE id = ? AND status = ? AND author = ?";
	$db->query($sql, [3, 'live', 'Rick']);

這些問號會自動取代成查詢函數中的第二個參數的陣列參數值。

封裝也支援陣列參數，它將轉換成 IN 使用的集合：

::

	$sql = "SELECT * FROM some_table WHERE id IN ? AND status = ? AND author = ?";
	$db->query($sql, [[3, 6], 'live', 'Rick']);

封裝轉換後的查詢結果如下

::

	SELECT * FROM some_table WHERE id IN (3,6) AND status = 'live' AND author = 'Rick'

使用封裝的第二個好處是，這些值會自動跳脫，以產生更安全的查詢。你就不需要手動跳脫資料，系統會自動幫你處理。

封裝命名
==============

不使用問號來標記封裝值的位置，你可以使用封裝命名，讓 key 和 value 可以相互對應。並且讓傳遞進來的鍵值陣列的鍵名與查詢中的置換符號相符：

::

        $sql = "SELECT * FROM some_table WHERE id = :id: AND status = :status: AND author = :name:";
        $db->query($sql, [
                'id'     => 3,
                'status' => 'live',
                'name'   => 'Rick'
        ]);

.. note:: 每個要封裝的值必須用冒號（:）包起來。

***************
錯誤處理
***************

**$db->error();**

如果你需要取得上次查詢後的錯誤訊息，error() 這個方法會回傳一個包含錯誤編號和訊息的陣列，以下是一個簡單的範例：

::

	if ( ! $db->simpleQuery('SELECT `example_field` FROM `example_table`'))
	{
		$error = $db->error(); // Has keys 'code' and 'message'
	}

****************
預備查詢
****************

大多數的資料庫引擎，支援某些形式的預備語法，這些語法讓你可以準備一次查詢，然後使用新的資料集進行多次查詢。由於資料是用與查詢本身不同的格式傳送到資料庫，因此消除了SQL注入的可能性。當你需要執行多次相同的查詢，速度也會快很多。然而，對每次的查詢都使用它，會對效能產生重大的影響，因為你要加倍頻繁的呼叫資料庫。由於查詢生成器和資料庫連接已經為你處理了跳脫資料，因此安全方面就不需要特別的擔心。但是，有時候你需要藉由執行預備語法或預備查詢來最佳化查詢。

預備查詢
===================

透過 ``prepare()`` 方法可以簡單地完成預備。這需要一個會回傳查詢物件的匿名函數作為參數。查詢物件是由任何 "final" 類型查詢自動產生的，包含新增、更新、刪除、取代、取得。使用查詢生成器執行查詢，是最簡單的方式。實際上，該查詢並未執行，而且變數無關緊要，因為它們從未被應用，而是置換符號。這會回傳一個 PreparedQuery 的物件。

::

    $pQuery = $db->prepare(function($db)
    {
        return $db->table('user')
                   ->insert([
                        'name'    => 'x',
                        'email'   => 'y',
                        'country' => 'US'
                   ]);
    });

如果你不想使用查詢生成器，則可以使用問號為置換符號的值，手動建立一個查詢物件。

::

    use CodeIgniter\Database\Query;

    $pQuery = $db->prepare(function($db)
    {
        $sql = "INSERT INTO user (name, email, country) VALUES (?, ?, ?)";

        return (new Query($db))->setQuery($sql);
    });

在預備語法中，如果你需要傳送一個變數陣列來操作資料庫，你可以在第二個參數傳送這個陣列：

::

    use CodeIgniter\Database\Query;

    $pQuery = $db->prepare(function($db)
    {
        $sql = "INSERT INTO user (name, email, country) VALUES (?, ?, ?)";

        return (new Query($db))->setQuery($sql);
    }, $options);

執行查詢
===================
當你的預備查詢已經準備好了，你可以使用 ``execute()`` 方法去執行你的查詢。在語法中，你可以根據你的需求傳送任意多的變數，但是變數的數量比須符合置換符號的數量，且順序也必須與原查詢中一樣。

::

    // Prepare the Query
    $pQuery = $db->prepare(function($db)
    {
        return $db->table('user')
                   ->insert([
                        'name'    => 'x',
                        'email'   => 'y',
                        'country' => 'US'
                   ]);
    });

    // Collect the Data
    $name    = 'John Doe';
    $email   = 'j.doe@example.com';
    $country = 'US';

    // Run the Query
    $results = $pQuery->execute($name, $email, $country);

這裡將會回傳一個標準的 :doc:`結果集合 </database/results>` 。

其他方法
=============

除了以上的兩個主要的方法之外，預備查詢物件也有以下幾個方法：

**close()** 儘管PHP在關閉資料庫語法已經做得很好，但是在操作完資料庫後關閉預備語法也是一項重要的工作。

::

    $pQuery->close();

**getQueryString()**

這將會回傳一個字串型態的預備語法。

**hasError()**

在最後一次 execute() 後如果出現任何錯誤，將會回傳布林型態的 true/false 。

**getErrorCode()**
**getErrorMessage()**

如果出現任何錯誤，可以使用這兩個方法來檢視錯誤編碼和錯誤訊息。

**************************
操作查詢物件
**************************

在CodeIgniter的內部架構中，所有查詢都會當作為 \CodeIgniter\Database\Query 的實體進行處理和儲存。這些類別負責繫結變數，否則準備好查詢，並且儲存查詢相關的效能資訊。

**getLastQuery()**

當你需要檢索上次查詢的語法，使用 getLastQuery() 這個方法：

::

	$query = $db->getLastQuery();
	echo (string)$query;

查詢類別
===============

每個查詢物件會儲存查詢相關的一些資訊。在一定程度上這些方法被時間軸功能使用，但你也可以使用。

**getQuery()**

在完成所有處理後，回傳最後的查詢。這裡的查詢是確切發送資料庫的查詢。

::

	$sql = $query->getQuery();

將查詢物件轉換成陣列做相同的查詢。

::

	$sql = (string)$query;

**getOriginalQuery()**

回傳原本的SQL。這不會有任何繫結值或更換字首等等。

::

	$sql = $query->getOriginalQuery();

**hasError()**

如果在執行查詢時有發生錯誤，這個方法將將會回傳 true 。

::

	if ($query->hasError())
	{
		echo 'Code: '. $query->getErrorCode();
		echo 'Error: '. $query->getErrorMessage();
	}

**isWriteType()**

如果查詢被確認為寫入的類別查詢（例如：新增、更新、刪除等等），回傳true。

::

	if ($query->isWriteType())
	{
		... do something
	}

**swapPrefix()**

在最後的SQL中，用一個值替換掉一個資料表的字首。第一個參數為原本的字首，第二個參數為你想要替換的值。

::

	$sql = $query->swapPrefix('ci3_', 'ci4_');

**getStartTime()**

回傳查詢執行的時間，以微秒（ms）為單位。

::

	$microtime = $query->getStartTime();

**getDuration()**

回傳查詢持續時間的浮點數，以微秒（ms）為單位。

::

	$microtime = $query->getDuration();
