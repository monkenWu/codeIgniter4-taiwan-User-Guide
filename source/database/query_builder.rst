###################
查詢生成器類別
###################

你可以在 CodeIgniter 中使用查詢生成器類別。
這個模式可以讓你用少量的程式碼就完成資料檢索、新增和更新。
在某些範例下，只需要一兩行程式碼就能操作資料庫。
CodeIgniter 並沒有要求每個資料表都要有自己的類別檔。
這代表著它能提供更簡單的介面。

除了簡單之外，查詢生成器的另一個特色是建立獨立於資料庫的應用程式，因為查詢語法是由每個獨立的資料庫適配器所產生。
此外，因為系統會自動跳脫字元，這使你的查詢更安全。

.. contents::
    :local:
    :depth: 2

*************************
載入查詢生成器
*************************

查詢生成器是由資料庫連接的  ``table()`` 這個方法所載入。
這是讓你設定查詢的 ``FROM`` ，並且回傳一個新的查詢生成器類別實體。

::

    $db      = \Config\Database::connect();
    $builder = $db->table('users');。

查詢生成器只有在你明確指定下，才會將請求載入到記憶體。因此預設情形下，不會有資源被使用。

**************
選取資料
**************

接下來的的函數將介紹如何使用 **SELECT** SQL語法。

**$builder->get()**

執行選取查詢並回傳結果。用來檢索所有資料表的資料。

::

    $builder = $db->table('mytable');
    $query   = $builder->get();  // Produces: SELECT * FROM mytable

第一個是限制筆數(limit)，第二個參數是忽略筆數(offset)。

::

	$query = $builder->get(10, 20);

	// 執行: SELECT * FROM mytable LIMIT 20, 10
	// (以上是MySQL的語法，這會與其他資料庫的語法有些許不同。)

以上的函數都被指定到 $query 這個變數中，如果要顯示這些結果，你可以使用以下方式：

::

    $query = $builder->get();

    foreach ($query->getResult() as $row) {
        echo $row->title;
    }

請參考 :doc:`產生查詢結果 <results>` 來了解關於產生結果的完整內容。

**$builder->getCompiledSelect()**

編譯選取查詢就像是 ``$builder->get()`` 的方法一樣，但實際上並不會 *執行* 查詢。
這個方法只是簡單的回傳字串格式的SQL查詢語法。

範例

::

	$sql = $builder->getCompiledSelect();
	echo $sql;

	// 輸出字串：SELECT * FROM mytable

第一個參數是設定是否重置查詢生成器的查詢。(預設是會重置，就像使用 ``$builder->get()`` )

::

	echo $builder->limit(10,20)->getCompiledSelect(false);

	// 輸出字串: SELECT * FROM mytable LIMIT 20, 10
	// (以上是MySQL的語法，這會與其他資料庫的語法有些許不同。)

	echo $builder->select('title, content, date')->getCompiledSelect();

	// 輸出字串: SELECT title, content, date FROM mytable LIMIT 20, 10

上述的範例要注意的關鍵是，第二次的查詢並沒有使用 ``$builder->from()`` ，也沒有傳送資料表名稱。
這是因為查詢並沒有使用 ``$builder->get()`` 來執行，這是重置值或直接使用 ``$builder->resetQuery()`` 來進行重置。

**$builder->getWhere()**

與 ``get()`` 函數相同，只是它能在第一個參數中加入 "where"  語法。而不是使用 db->where() 這個函數。

::

	$query = $builder->getWhere(['id' => $id], $limit, $offset);

請參考以下內容來得到更多關於 `where` 函數的使用方式。

**$builder->select()**

在你的查詢中加入 SELECT 的語法。

::

	$builder->select('title, content, date');
	$query = $builder->get();

	// 執行: SELECT title, content, date FROM mytable

.. note:: 如果你是要從資料表選取全部的欄位(\*)，你就不需要使用這個函數。當你忽略這個函數，CodeIgniter會假設你要選取所有欄位，自動幫你加入 ``SELECT \*`` 。

``$builder->select()`` 的第二個參數是可有可無的。如果你設定為 ``false``，CodeIgniter將不會保護你的語法或資料表名稱。
當你需要使用複合選取語法時這很有幫助，因為自動跳脫語法可能會破壞你的語法。


::

	$builder->select('(SELECT SUM(payments.amount) FROM payments WHERE payments.invoice_id=4) AS amount_paid', FALSE);
	$query = $builder->get();

**$builder->selectMax()**

當你要在查詢使用 ``SELECT MAX(field)`` 時，你可以利用第二個參數為你查詢的結果重新命名。

::

	$builder->selectMax('age');
	$query = $builder->get();  // 產生的語法: SELECT MAX(age) as age FROM mytable

	$builder->selectMax('age', 'member_age');
	$query = $builder->get(); // 產生的語法: SELECT MAX(age) as member_age FROM mytable

**$builder->selectMin()**

當你要在查詢使用 "SELECT MIN(field)"。就像 selectMax() 一樣，你可以利用第二個參數為你查詢的結果重新命名。

::

	$builder->selectMin('age');
	$query = $builder->get(); // 產生的語法: SELECT MIN(age) as age FROM mytable

**$builder->selectAvg()**

當你要在查詢使用 "SELECT AVG(field)"。就像 selectMax() 一樣，你可以利用第二個參數為你查詢的結果重新命名。
::

	$builder->selectAvg('age');
	$query = $builder->get(); // 產生的語法: SELECT AVG(age) as age FROM mytable

**$builder->selectSum()**

當你要在查詢使用 ``SELECT SUM(field)``。就像 ``selectMax()`` 一樣，你可以利用第二個參數為你查詢的結果重新命名。

::

	$builder->selectSum('age');
	$query = $builder->get(); // 產生的語法: SELECT SUM(age) as age FROM mytable

**$builder->selectCount()**

當你要在查詢使用 ``SELECT COUNT(field)``。就像 ``selectMax()`` 一樣，你可以利用第二個參數為你查詢的結果重新命名。

.. note:: 這個方法與 ``groupBy()`` 一起使用時非常方便。如果是要計算查詢的筆數，可以參考 ``countAll()`` 和 ``countAllResults()`` 。

::

	$builder->selectCount('age');
	$query = $builder->get(); // 產生的語法: SELECT COUNT(age) as age FROM mytable

**$builder->from()**

在你的查詢中加入 FROM 的語法。

::

	$builder->select('title, content, date');
	$builder->from('mytable');
	$query = $builder->get();  // 產生的語法: SELECT title, content, date FROM mytable

.. note:: 就像前面所介紹的，FROM 的語法可以在 ``$db->table()`` 中加入。額外呼叫 ``from()`` 只會在你的查詢中增加更多的資料表。

**$builder->join()**

在你的查詢中加入 JOIN 的語法。

::

    $builder->db->table('blog');
    $builder->select('*');
    $builder->join('comments', 'comments.id = blogs.id');
    $query = $builder->get();

    // 產生的語法:
    // SELECT * FROM blogs JOIN comments ON comments.id = blogs.id

如果你需要JOIN多個資料表，就需要呼叫多次函數。

如果你需要指定JOIN的類型，你可以在第三個參數中加入。可以選擇：left、right、outer、inner、left outer或right outer。

::

	$builder->join('comments', 'comments.id = blogs.id', 'left');
	// 產生的語法: LEFT JOIN comments ON comments.id = blogs.id

*************************
尋找特定的資料
*************************

**$builder->where()**


你可以使用以下四個方法的其中一個來設定查詢的 **WHERE** 條件。

.. note:: 除了使用自訂字串時，傳送到這個函數的數值都會自動跳脫，來產生安全的查詢。

.. note:: ``$builder->where()`` accepts an optional third parameter. If you set it to
    ``false``, CodeIgniter will not try to protect your field or table names.

#. **簡單的 key/value 方法：**

	::

		$builder->where('name', $name); // 產生的語法: WHERE name = 'Joe'

	請注意，= 這個符號將會自動幫你增加。

	如果你呼叫多個函數，它將會使用 AND 幫你串接在一起：

	::

		$builder->where('name', $name);
		$builder->where('title', $title);
		$builder->where('status', $status);
		// WHERE name = 'Joe' AND title = 'boss' AND status = 'active'

#. **客製化 key/value 方法：**

	你可以在第一個參數中包含一個符號來控制你的條件判斷：

	::

		$builder->where('name !=', $name);
		$builder->where('id <', $id); // 產生的語法: WHERE name != 'Joe' AND id < 45

#. **關聯陣列方法：**

	::

		$array = ['name' => $name, 'title' => $title, 'status' => $status];
		$builder->where($array);
		// 產生的語法: WHERE name = 'Joe' AND title = 'boss' AND status = 'active'

	也可以使用以下的方式，增加個別的判斷：

	::

		$array = ['name !=' => $name, 'id <' => $id, 'date >' => $date];
		$builder->where($array);

#. **自定字串：**

	你可以手動編寫你的語法。
	
	::

		$where = "name='Joe' AND status='boss' OR status='active'";
		$builder->where($where);

    If you are using user-supplied data within the string, you MUST escape the
    data manually. Failure to do so could result in SQL injections.

    ::

        $name = $builder->db->escape('Joe');
        $where = "name={$name} AND status='boss' OR status='active'";
        $builder->where($where);

#. **子查詢：**
	你可以使用匿名函數來建立子查詢。

    ::

        $builder->where('advance_amount <', function(BaseBuilder $builder) {
            return $builder->select('MAX(advance_amount)', false)->from('orders')->where('id >', 2);
        });
        // 產生的語法: WHERE "advance_amount" < (SELECT MAX(advance_amount) FROM "orders" WHERE "id" > 2)

**$builder->orWhere()**

這個函數與上述的功能相同，差別在於多個實體透過 OR 串接在一起。

    ::

	$builder->where('name !=', $name);
	$builder->orWhere('id >', $id);  // 產生的語法: WHERE name != 'Joe' OR id > 50

**$builder->whereIn()**

產生 WHERE 欄位 ``IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用AND串接。

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->whereIn('username', $names);
        // 產生的語法: WHERE username IN ('Frank', 'Todd', 'James')

你也可以使用子查詢而不是陣列。

    ::

        $builder->whereIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });
        // 產生的語法: WHERE "id" IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)

**$builder->orWhereIn()**

產生 WHERE 欄位 ``WHERE field IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用OR串接。

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->orWhereIn('username', $names);
        // 產生的語法: OR username IN ('Frank', 'Todd', 'James')

你也可以使用子查詢而不是陣列。

    ::

        $builder->orWhereIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });

        // 產生的語法: OR "id" IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)

**$builder->whereNotIn()**

產生 WHERE 欄位 ``NOT IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用AND串接。

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->whereNotIn('username', $names);
        // 產生的語法: WHERE username NOT IN ('Frank', 'Todd', 'James')

你也可以使用子查詢而不是陣列。

    ::

        $builder->whereNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });

        // 產生的語法: WHERE "id" NOT IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)


**$builder->orWhereNotIn()**

產生 WHERE 欄位 ``NOT IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用OR串接。

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->orWhereNotIn('username', $names);
        // 產生的語法: OR username NOT IN ('Frank', 'Todd', 'James')

你也可以使用子查詢而不是陣列。

    ::

        $builder->orWhereNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });

        // 產生的語法: OR "id" NOT IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)

************************
尋找相似的資料
************************

**$builder->like()**

這個方法可以產生 **LIKE** 語法，方便你搜尋資料。

.. note:: 傳送到這個函數的數值都會自動跳脫。

.. note:: 在這個函數的第五個參數傳送 ``true`` ，可以對所有 ``like*`` 的方法變體強制執行不區分大小寫的搜尋。這將會使用平台特有的功能，否則將會強制數值轉換成小寫，例如： ``WHERE LOWER(column) LIKE '%search%'`` 。這可能需要為 ``LOWER(column)`` 創建索引，而不是 ``column`` 本身，才會使功能有效地被執行。

#. **簡單的 key/value 方法：**

	::

		$builder->like('title', 'match');
		// 產生的語法: WHERE `title` LIKE '%match%' ESCAPE '!'

	如果你呼叫多個方法，它們將會用 AND 串接在一起。

	::

		$builder->like('title', 'match');
		$builder->like('body', 'match');
		// WHERE `title` LIKE '%match%' ESCAPE '!' AND  `body` LIKE '%match% ESCAPE '!'

	如果你想要控制萬用字元 (%) 放置的位置，可以在第三個參數中選擇。 可以設定的數值有： 'before' 、 'after' 和 'both' (預設為 'both' )。

	::

		$builder->like('title', 'match', 'before');	// Produces: WHERE `title` LIKE '%match' ESCAPE '!'
		$builder->like('title', 'match', 'after');	// Produces: WHERE `title` LIKE 'match%' ESCAPE '!'
		$builder->like('title', 'match', 'both');	// Produces: WHERE `title` LIKE '%match%' ESCAPE '!'

#. **關聯陣列方法：**

	::

		$array = ['title' => $match, 'page1' => $match, 'page2' => $match];
		$builder->like($array);
		// WHERE `title` LIKE '%match%' ESCAPE '!' AND  `page1` LIKE '%match%' ESCAPE '!' AND  `page2` LIKE '%match%' ESCAPE '!'

**$builder->orLike()**

這個方法與上述的方法相同，差別在於使用 OR 將多個實體串接在一起。

::

	$builder->like('title', 'match'); $builder->orLike('body', $match);
	// WHERE `title` LIKE '%match%' ESCAPE '!' OR  `body` LIKE '%match%' ESCAPE '!'

**$builder->notLike()**

這個方法與 ``like()`` 相同，差別在於產生的是 NOT LIKE 的字句。

::

	$builder->notLike('title', 'match');	// WHERE `title` NOT LIKE '%match% ESCAPE '!'

**$builder->orNotLike()**

這個方法與 ``notLike()`` 相同，差別在於使用 OR 將多個實體串接在一起。

::

	$builder->like('title', 'match');
	$builder->orNotLike('body', 'match');
	// WHERE `title` LIKE '%match% OR  `body` NOT LIKE '%match%' ESCAPE '!'

**$builder->groupBy()**

在你的查詢中加入 GROUP BY 的語法。

::

	$builder->groupBy("title"); // Produces: GROUP BY title

你也可以傳送多個數值的陣列。

::

	$builder->groupBy(["title", "date"]);  // 產生的語法: GROUP BY title, date

**$builder->distinct()**

在你的查詢中加入 "DISTINCT" 語法。

::

	$builder->distinct();
	$builder->get(); // 產生的語法: SELECT DISTINCT * FROM mytable

**$builder->having()**

在你的查詢中加入 GROUP BY 的語法。這有傳送一個或兩個參數的使用的方式：

::

	$builder->having('user_id = 45');  // Produces: HAVING user_id = 45
	$builder->having('user_id',  45);  // Produces: HAVING user_id = 45

你也可以傳送多個數值的陣列：

::

	$builder->having(['title =' => 'My Title', 'id <' => $id]);
	// 產生的語法: HAVING title = 'My Title', id < 45

如果你使用的是CodeIgniter會跳脫查詢的資料庫，你可以藉由傳入 FALSE 在第三個參數中來防止跳脫內容。

::

	$builder->having('user_id',  45);  // 產生的語法: HAVING `user_id` = 45 in some databases such as MySQL
	$builder->having('user_id',  45, FALSE);  // 產生的語法: HAVING user_id = 45

**$builder->orHaving()**

與 ``having()`` 相同，只用 "OR" 分開多個字句。

**$builder->havingIn()**

產生 HAVING 欄位 ``IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用AND串接。

    ::

        $groups = [1, 2, 3];
        $builder->havingIn('group_id', $groups);
        // 產生的語法: HAVING group_id IN (1, 2, 3)

你也可以使用子查詢而不是陣列。

    ::

        $builder->havingIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });
        // 產生的語法: HAVING "id" IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)

**$builder->orHavingIn()**

產生 HAVING 欄位 ``IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用OR串接。

    ::

        $groups = [1, 2, 3];
        $builder->orHavingIn('group_id', $groups);
        // 產生的語法: OR group_id IN (1, 2, 3)

你也可以使用子查詢而不是陣列。

    ::

        $builder->orHavingIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });

        // 產生的語法: OR "id" IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)

**$builder->havingNotIn()**

產生 HAVING 欄位 ``NOT IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用AND串接。

    ::

        $groups = [1, 2, 3];
        $builder->havingNotIn('group_id', $groups);
        // 產生的語法: HAVING group_id NOT IN (1, 2, 3)

你也可以使用子查詢而不是陣列。

    ::

        $builder->havingNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });

        // 產生的語法: HAVING "id" NOT IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)


**$builder->orHavingNotIn()**

產生 HAVING 欄位 ``NOT IN ('item', 'item')`` 的SQL查詢語法，如果合適的話就用OR串接。

    ::

        $groups = [1, 2, 3];
        $builder->havingNotIn('group_id', $groups);
        // 產生的語法: OR group_id NOT IN (1, 2, 3)

你也可以使用子查詢而不是陣列。

    ::

        $builder->orHavingNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });

        // 產生的語法: OR "id" NOT IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)

**$builder->havingLike()**

這個方法可以對HAVING產生 **LIKE** 語法，這對你在搜尋上很有幫助。

.. note:: 傳送到這個函數的數值都會自動跳脫，來產生安全的查詢。

.. note:: 在這個函數的第五個參數傳送 ``true`` ，可以對所有 ``havingLike*`` 的方法變體強制執行不區分大小寫的搜尋。這將會使用平台特有的功能，否則將會強制數值轉換成小寫，例如： ``HAVING LOWER(column) LIKE '%search%'`` 。這可能需要為 ``LOWER(column)`` 創建索引，而不是 ``column`` 本身，才會使功能有效地被執行。

#. **簡單的 key/value 方法：**

	::

		$builder->havingLike('title', 'match');
		// 產生的語法: HAVING `title` LIKE '%match%' ESCAPE '!'

	如果你呼叫多個函數，它將會使用 AND 幫你串接在一起：

	::

		$builder->havingLike('title', 'match');
		$builder->havingLike('body', 'match');
		// HAVING `title` LIKE '%match%' ESCAPE '!' AND  `body` LIKE '%match% ESCAPE '!'

	如果你想要控制萬用字元 (%) 放置的位置，可以在第三個參數中選擇。 可以設定的數值有： 'before' 、 'after' 和 'both' (預設為 'both' )。

	::

		$builder->havingLike('title', 'match', 'before');	// 產生的語法: HAVING `title` LIKE '%match' ESCAPE '!'
		$builder->havingLike('title', 'match', 'after');	// 產生的語法: HAVING `title` LIKE 'match%' ESCAPE '!'
		$builder->havingLike('title', 'match', 'both');		// 產生的語法: HAVING `title` LIKE '%match%' ESCAPE '!'

#. **關聯陣列方法：**

	::

		$array = ['title' => $match, 'page1' => $match, 'page2' => $match];
		$builder->havingLike($array);
		// HAVING `title` LIKE '%match%' ESCAPE '!' AND  `page1` LIKE '%match%' ESCAPE '!' AND  `page2` LIKE '%match%' ESCAPE '!'

**$builder->orHavingLike()**

這個方法與上述的方法相同，差別在於使用 OR 將多個實體串接在一起：

::

	$builder->havingLike('title', 'match'); $builder->orHavingLike('body', $match);
	// HAVING `title` LIKE '%match%' ESCAPE '!' OR  `body` LIKE '%match%' ESCAPE '!'

**$builder->notHavingLike()**

這個方法與 ``havingLike()`` 相同，差別在於產生的是 NOT LIKE 的字句。

::

	$builder->notHavingLike('title', 'match');	// HAVING `title` NOT LIKE '%match% ESCAPE '!'

**$builder->orNotHavingLike()**

這個方法與 ``notHavingLike()`` 相同，差別在於產生的是 NOT LIKE 的字句。

::

	$builder->havingLike('title', 'match');
	$builder->orNotHavingLike('body', 'match');
	// HAVING `title` LIKE '%match% OR  `body` NOT LIKE '%match%' ESCAPE '!'

****************
排序查詢結果
****************

**$builder->orderBy()**

讓你使用 ORDER BY 的語法。

第一個參數是你想要排序的欄位名稱。

第二個參數是設定你想要排序的方式。可以使用的數值有： **ASC** 、 **DESC** 和 **RANDOM** 。

::

	$builder->orderBy('title', 'DESC');
	// 產生的語法: ORDER BY `title` DESC

你也可以在第一個參數中傳送你想要的字串。

::

	$builder->orderBy('title DESC, name ASC');
	// 產生的語法: ORDER BY `title` DESC, `name` ASC

或者，呼叫多個函數來排序多個欄位。

::

	$builder->orderBy('title', 'DESC');
	$builder->orderBy('name', 'ASC');
	// 產生的語法: ORDER BY `title` DESC, `name` ASC

如果你使用 **RANDOM** ，則第一個參數將會被忽略，除非你指定一個種子值。

::

	$builder->orderBy('title', 'RANDOM');
	// 產生的語法: ORDER BY RAND()

	$builder->orderBy(42, 'RANDOM');
	// 產生的語法: ORDER BY RAND(42)

.. note:: Oracle 目前沒有支援亂數排序，預設將會使用ASC做排序。

****************************
限制或計算查詢結果
****************************

**$builder->limit()**

讓你限制查詢所要回傳的結果數量：

::

	$builder->limit(10);  // 產生的語法: LIMIT 10

第二個參數是設定查詢結果的偏移量。

::

	$builder->limit(10, 20);  // Produces: LIMIT 20, 10 (in MySQL. Other databases have slightly different syntax)


**$builder->countAllResults()**

讓你在特定查詢生成器的查詢中確定數量。它也可以增加其他查詢生成器的判斷函數，像是： ``where()`` 、 ``orWhere()`` 、 ``like()`` 、 ``orLike()`` 等等。範例如下：

::

	echo $builder->countAllResults();  // 產生integer結果，如：25。
	$builder->like('title', 'match');
	$builder->from('my_table');
	echo $builder->countAllResults();  // 產生integer結果，如：17。

不過這個方法會重置你在 ``select()`` 中傳送任何欄位值。如果你需要保留它們，你可以在第一個參數中傳送 ``FALSE`` 。

::

	echo $builder->countAllResults(false); // 產生integer結果，如：17。

**$builder->countAll()**

回傳在特定的資料表中的資料數量。
範例：

::

	echo $builder->countAll();  // 產生integer，如：25。

與 countAllResult 方法一樣，這個方法會重置你在 ``select()`` 中傳送任何欄位值。如果你需要保留它們，你可以在第一個參數中傳送 ``FALSE`` 。

**************
查詢分組
**************

查詢分組可以使用括號在WHERE子句中建立不同的群組。這樣就能在WHERE子句中建立複雜的查詢。支援巢狀的群組。範例：

::

    $builder->select('*')->from('my_table')
        ->groupStart()
            ->where('a', 'a')
            ->orGroupStart()
                ->where('b', 'b')
                ->where('c', 'c')
            ->groupEnd()
        ->groupEnd()
        ->where('d', 'd')
    ->get();

	// 產生的語法:
	// SELECT * FROM (`my_table`) WHERE ( `a` = 'a' OR ( `b` = 'b' AND `c` = 'c' ) ) AND `d` = 'd'

.. note:: 分組需要保持平衡，確保每個 ``groupStart()`` 都有相對應的 ``groupEnd()``。

**$builder->groupStart()**

開始一個新的分組，在查詢的WHERE子句中加入一個左括號。

**$builder->orGroupStart()**

開始一個新的分組，在查詢的WHERE子句中加入一個左括號，並在前面加入 'OR' 。

**$builder->notGroupStart()**

開始一個新的分組，在查詢的WHERE子句中加入一個左括號，並在前面加入 'NOT' 。

**$builder->orNotGroupStart()**

開始一個新的分組，在查詢的WHERE子句中加入一個左括號，並在前面加入 'OR NOT' 。

**$builder->groupEnd()**

結束目前的分組，在查詢的WHERE子句中加入一個右括號。

**$builder->groupHavingStart()**

開始一個新的分組，在查詢的HAVING子句中加入一個左括號。
Starts a new group by adding an opening parenthesis to the HAVING clause of the query.

**$builder->orGroupHavingStart()**

開始一個新的分組，在查詢的HAVING子句中加入一個左括號，並在前面加入 'OR' 。

**$builder->notGroupHavingStart()**

開始一個新的分組，在查詢的HAVING子句中加入一個左括號，並在前面加入 'NOT' 。

**$builder->orNotGroupHavingStart()**

開始一個新的分組，在查詢的HAVING子句中加入一個左括號，並在前面加入 'OR NOT' 。
Starts a new group by adding an opening parenthesis to the HAVING clause of the query, prefixing it with 'OR NOT'.

**$builder->groupHavingEnd()**

結束目前的分組，在查詢的HAVING子句中加入一個右括號。

**************
插入資料
**************

**$builder->insert()**

根據你提供的資料產生插入的語法，並執行查詢。你也可以傳送 **陣列** 或 **物件** 到函數中。以下是使用陣列的範例：

::

    $data = [
        'title' => 'My title',
        'name'  => 'My Name',
        'date'  => 'My date',
    ];

	$builder->insert($data);
	// 產生的語法: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')

第一個參數是傳送的關聯陣列變數。

以下是使用物件的範例：

::

	class Myclass
    {
        public $title   = 'My Title';
        public $content = 'My Content';
        public $date    = 'My Date';
    }

    $object = new Myclass;
    $builder->insert($object);
	// 產生的語法: INSERT INTO mytable (title, content, date) VALUES ('My Title', 'My Content', 'My Date')

第一個參數是傳送的物件變數。

.. note:: 傳送到這個函數的數值都會自動跳脫，來產生安全的查詢。

**$builder->ignore()**

根據你提供的資料產生忽略插入的語法，並執行查詢。
因此，如果相同主鍵的數值已經存在，就不會執行插入查詢。
你可以以傳送 **布林** 到這個函數中。
以下是使用上述傳送陣列的範例：

::

    $data = [
        'title' => 'My title',
        'name'  => 'My Name',
        'date'  => 'My date',
    ];

	$builder->ignore(true)->insert($data);
	// 產生的語法: INSERT OR IGNORE INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')


**$builder->getCompiledInsert()**

像是 ``$builder->insert()`` 一樣編譯插入查詢，但並不會 **執行** 查詢。這個方法只會回傳SQL查詢的語法字串。

範例：

::

    $data = [
        'title' => 'My title',
        'name'  => 'My Name',
        'date'  => 'My date',
    ];

	$sql = $builder->set($data)->getCompiledInsert('mytable');
	echo $sql;

	// 產生的語法字串: INSERT INTO mytable (`title`, `name`, `date`) VALUES ('My title', 'My name', 'My date')

第二個參數是設定是否重置查詢生成器的查詢(預設是會重置，就像使用 ``$builder->insert()`` 一樣)。

::

	echo $builder->set('title', 'My Title')->getCompiledInsert('mytable', FALSE);

	// 產生的語法字串: INSERT INTO mytable (`title`) VALUES ('My Title')

	echo $builder->set('content', 'My Content')->getCompiledInsert();

	// 產生的語法字串: INSERT INTO mytable (`title`, `content`) VALUES ('My Title', 'My Content')

上述的範例要注意的關鍵是，第二次的查詢並沒有使用 ``$builder->from()`` ，也沒有在第一個參數中傳送資料表名稱。
這是因為查詢並沒有使用 ``$builder->insert`()`` 來執行，這個查詢會重置值或直接使用 ``$builder->resetQuery()`` 來進行重置。


.. note:: 這個方法不適用在批次插入。

**$builder->insertBatch()**

根據你提供的資料產生插入的語法，並執行查詢。你也可以傳送 **陣列** 或 **物件** 到函數中。以下是使用陣列的範例：

::

    $data = [
        [
            'title' => 'My title',
            'name'  => 'My Name',
            'date'  => 'My date',
        ],
        [
            'title' => 'Another title',
            'name'  => 'Another Name',
            'date'  => 'Another date',
        ],
    ];

	$builder->insertBatch($data);
	// 產生的語法: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date'),  ('Another title', 'Another name', 'Another date')

第一個參數是傳送的關聯陣列變數。

.. note:: 傳送到這個函數的數值都會自動跳脫，來產生安全的查詢。

*************
更新資料
*************

**$builder->replace()**

這個方法會執行 REPLACE 的語句，基於標準 SQL 的 DELETE + INSERT 一樣，會使用 *PRIMARY* 和 *UNIQUE* 作為判斷的因素。
在我們的範例中，它將會使你減少呼叫  ``select()`` 、 ``update()`` 、 ``delete()`` 和 ``insert()`` 不同的組合來實現複雜的邏輯。

範例::

    $data = [
        'title' => 'My title',
        'name'  => 'My Name',
        'date'  => 'My date',
    ];

	$builder->replace($data);

	// 執行的語法: REPLACE INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')

在上述的範例中，假設 *title* 是這個欄位的主鍵，如果 *title* 有指定 'My title' 這個值，則整個列將會被刪除，並用新的值去替換它。

``set()`` 這個的用法也會將所有字句自動跳脫，就像是 ``insert()`` 。

**$builder->set()**

這個函數能夠讓你設定插入和更新的值。


**它可以直接用於插入和更新函數，而不是傳送一個資料陣列：**

::

	$builder->set('name', $name);
	$builder->insert();  // 產生的語法: INSERT INTO mytable (`name`) VALUES ('{$name}')

如果使用多個函數呼叫，它們將會根據你使用的是插入或更新，重新組合成合適的語法：

::

	$builder->set('name', $name);
	$builder->set('title', $title);
	$builder->set('status', $status);
	$builder->insert();

**set()**  也會接受可有可無的第三個參數 ( ``$escape`` )，如果它設定為 ``false`` 它將會避免資料被跳脫。
為了要說差異，以下的範例是使用 ``set()`` 來做比較，分別是有無使用跳脫參數。

::

	$builder->set('field', 'field+1', FALSE);
	$builder->where('id', 2);
	$builder->update(); // gives UPDATE mytable SET field = field+1 WHERE `id` = 2

	$builder->set('field', 'field+1');
	$builder->where('id', 2);
	$builder->update(); // gives UPDATE `mytable` SET `field` = 'field+1' WHERE `id` = 2

你也可以在這個函數中傳送一個關聯陣列：

::

    $array = [
        'name'   => $name,
        'title'  => $title,
        'status' => $status,
    ];

	$builder->set($array);
	$builder->insert();

或是一個物件

::

    class Myclass
    {
        public $title   = 'My Title';
        public $content = 'My Content';
        public $date    = 'My Date';
    }

	$object = new Myclass;
	$builder->set($object);
	$builder->insert();

**$builder->update()**

產生一個更新字串並根據你提供的資料執行查詢。你可以在函數中傳送一個 **陣列** 或 **物件**。
以下是使用陣列的範例：

::

    $data = [
        'title' => $title,
        'name'  => $name,
        'date'  => $date,
    ];

    $builder->where('id', $id);
    $builder->update($data);
    // 產生的語法:
    //
    // UPDATE mytable
    // SET title = '{$title}', name = '{$name}', date = '{$date}'
    // WHERE id = $id

或者你可以傳送物件：

::

    class Myclass
    {
        public $title   = 'My Title';
        public $content = 'My Content';
        public $date    = 'My Date';
    }

    $object = new Myclass;
    $builder->where('id', $id);
    $builder->update($object);
    // 產生的語法:
    //
    // UPDATE `mytable`
    // SET `title` = '{$title}', `name` = '{$name}', `date` = '{$date}'
    // WHERE id = `$id`

.. note:: 傳送到這個函數的數值都會自動跳脫，來產生安全的查詢。

``$builder->where()`` 這個函數，能夠使你設定 WHERE的語法。
你也可以將資料作為字串直接傳送到更新函數裡：

::

	$builder->update($data, "id = 4");

或是一個陣列：

::

	$builder->update($data, ['id' => $id]);

你也可以在更新時使用 ``$builder->set()`` 這個函數來闡述上述的內容。

**$builder->updateBatch()**

產生一個更新字串並根據你提供的資料執行查詢。你可以在函數中傳送一個 **陣列** 或 **物件**。
以下是使用陣列的範例：

::

	$data = [
	   [
	      'title' => 'My title' ,
	      'name'  => 'My Name 2' ,
	      'date'  => 'My date 2'
	   ],
	   [
	      'title' => 'Another title' ,
	      'name'  => 'Another Name 2' ,
	      'date'  => 'Another date 2'
	   ]
	];

	$builder->updateBatch($data, 'title');

	// 產生的語法：
	// UPDATE `mytable` SET `name` = CASE
	// WHEN `title` = 'My title' THEN 'My Name 2'
	// WHEN `title` = 'Another title' THEN 'Another Name 2'
	// ELSE `name` END,
	// `date` = CASE
	// WHEN `title` = 'My title' THEN 'My date 2'
	// WHEN `title` = 'Another title' THEN 'Another date 2'
	// ELSE `date` END
	// WHERE `title` IN ('My title','Another title')

第一個參數是數值的關聯陣列，第二個參數是 Where 的鍵值。

.. note:: 傳送到這個函數的數值都會自動跳脫，來產生安全的查詢。

.. note:: 因為 ``affectedRows()`` 的工作原理，這個方法不會回傳受影響的列數。你可以使用 ``updateBatch()`` 來取得受影響的列數。

**$builder->getCompiledUpdate()**

上述的方法與 ``$builder->getCompiledInsert()`` 完全一樣，只差在它是產生 UPDATE 的 SQL 字串，而不是 INSERT SQL字串。


可以參考 ``$builder->getCompiledInsert()`` ，了解更多的資訊。

.. note:: 這個方法不適用在批次更新。

*************
刪除資料
*************

**$builder->delete()**

產生刪除的 SQL 字串，並且執行查詢：

::

	$builder->delete(['id' => $id]);  // Produces: // DELETE FROM mytable  // WHERE id = $id

第一個參數是 where 的條件。
也可以使用 ``where()`` 或 ``or_where()`` 函數來取代你想要的條件判斷。

::

	$builder->where('id', $id);
	$builder->delete();

	// 產生的語法:
	// DELETE FROM mytable
	// WHERE id = $id

如果你想要刪除資料表裡的所有資料，你可以使用 ``truncate()`` 或是 ``emptyTable()`` 這兩個函數。

**$builder->emptyTable()**

產生刪除的 SQL 字串，並且執行查詢：

::

	  $builder->emptyTable('mytable'); // 產生的語法: DELETE FROM mytable

**$builder->truncate()**

產生截斷的 SQL 字串，並且執行查詢：

::

	$builder->truncate();

	// 產生的語法:
	// TRUNCATE mytable

.. note:: 如果 TRUNCATE 不能使用，``truncate()`` 將會執行 ``DELETE FROM table``。

**$builder->getCompiledDelete()**


這個方法與 ``$builder->getCompiledInsert()``  相同，差別在於產生的是 DELETE 的 SQL 字串，而不是 INSERT 的 SQL 字串。

可以參考 ``$builder->getCompiledInsert()``，了解更多的資訊。

***************
鏈式方法
***************

鏈式方法藉由結合多個函數，簡化你的語法。

::

	$query = $builder->select('title')
			 ->where('id', $id)
			 ->limit(10, 20)
			 ->get();

.. _ar-caching:

***********************
重置查詢生成器
***********************

**$builder->resetQuery()**

重置查詢生成器可以讓你重新開始查詢，而不需要先執行 ``$builder->get()`` 或是 ``$builder->insert()`` 這樣的方法來執行查詢 。

這在使用查詢生成器產生 SQL (例如： ``$builder->getCompiledSelect()`` ) 但是選擇它很有用，可以參考以下範例：

::

    // 請注意 get_compiled_select() 的第二個參數是 FALSE
    $sql = $builder->select(['field1','field2'])
                   ->where('field3',5)
                   ->getCompiledSelect(false);

    // ...
    // 在 SQL 中做一些瘋狂的事情，像是將 SQL 添加到 cron 的腳本中為了之後的執行之類的事情。
    // ...

    $data = $builder->get()->getResultArray();

    // 將會執行以下的查詢，並回傳一個陣列的結果：
    // SELECT field1, field1 from mytable where field3 = 5;

***************
類別參考
***************

.. php:class:: CodeIgniter\\Database\\BaseBuilder

	.. php:method:: db()

        :returns: The database connection in use
        :rtype:	``ConnectionInterface``

        Returns the current database connection from ``$db``. Useful for
        accessing ``ConnectionInterface`` methods that are not directly
        available to the Query Builder, like ``insertID()`` or ``errors()``.

	.. php:method:: resetQuery()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		重置目前查詢生成器的狀態。當你想要建立一個可以在特定條件下取消的查詢是很有用的。

	.. php:method:: countAllResults([$reset = TRUE])

		:param	bool	$reset: 是否重置 SELECT 的值
		:returns:	查詢結果中列的數量
		:rtype:	int

		產生特定於平台的查詢字串，用來計算查詢生成器回傳所有紀錄的數量。

	.. php:method:: countAll([$reset = TRUE])

		:param	bool	$reset: 是否重置 SELECT 的值
		:returns:	查詢結果中列的數量
		:rtype:	int

		產生特定於平台的查詢字串，用來計算查詢生成器回傳所有紀錄的數量。

	.. php:method:: get([$limit = NULL[, $offset = NULL[, $reset = TRUE]]]])

		:param	int	$limit: LIMIT 限制量
		:param	int	$offset: OFFSET 位移量
		:param 	bool $reset: 是否要清除查詢生成器的值？
		:returns:	``\CodeIgniter\Database\ResultInterface`` 實體 (鏈式方法)
		:rtype:	``\CodeIgniter\Database\ResultInterface``

		根據已經呼叫的查詢生成器方法，編譯和執行 ``SELECT`` 的語句。

	.. php:method:: getWhere([$where = NULL[, $limit = NULL[, $offset = NULL[, $reset = TRUE]]]]])

		:param	string	$where: WHERE 條件
		:param	int	$limit: LIMIT 限制量
		:param	int	$offset: OFFSET 位移量
		:param 	bool $reset: 是否要清除查詢生成器的值？
		:returns:	``\CodeIgniter\Database\ResultInterface`` 實體 (method chaining)
		:rtype:	``\CodeIgniter\Database\ResultInterface``

		跟 ``get()`` 一樣, 但是也允許直接加入 WHERE 條件判斷。

	.. php:method:: select([$select = '*'[, $escape = NULL]])

		:param	string	$select: 查詢的 SELECT
		:param	bool	$escape: 是否跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``SELECT`` 語法。

	.. php:method:: selectAvg([$select = ''[, $alias = '']])

		:param	string	$select: 計算平均值的欄位
		:param	string	$alias: 為計算平均值的欄位重新取名的名稱
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``SELECT AVG(欄位)`` 語法。

	.. php:method:: selectMax([$select = ''[, $alias = '']])

		:param	string	$select: 計算最大值的欄位
		:param	string	$alias: 為計算最大值的欄位重新取名的名稱
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``SELECT MAX(欄位)`` 語法。

	.. php:method:: selectMin([$select = ''[, $alias = '']])

		:param	string	$select: 計算最小值的欄位
		:param	string	$alias: 為計算最小值的欄位重新取名的名稱
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``SELECT MIN(欄位)`` 語法。

	.. php:method:: selectSum([$select = ''[, $alias = '']])

		:param	string	$select: 計算總和的欄位
		:param	string	$alias: 為計算總和的欄位重新取名的名稱
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``SELECT SUM(欄位)`` 語法。

	.. php:method:: selectCount([$select = ''[, $alias = '']])

		:param	string	$select: 計算數量的欄位
		:param	string	$alias: 為計算數量的欄位重新取名的名稱
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``SELECT COUNT(欄位)`` 語法。

	.. php:method:: distinct([$val = TRUE])

		:param	bool	$val: 是否使用 distinct
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		設立一個標記告訴查詢生成器在 ``SELECT`` 中加入 ``DISTINCT`` 的語法。

	.. php:method:: from($from[, $overwrite = FALSE])

                :param	mixed	$from: 資料表名稱；字串或陣列
                :param	bool	$overwrite: 是否要移除現有的第一個資料表？
                :returns:	``BaseBuilder`` 實體(鏈式方法)
                :rtype:	``BaseBuilder``

		指定查詢的 ``FROM`` 語法。

	.. php:method:: join($table, $cond[, $type = ''[, $escape = NULL]])

		:param	string	$table: 要 JOIN 的資料表名稱
		:param	string	$cond: JOIN ON 的條件
		:param	string	$type: JOIN 的類型
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		Adds a JOIN clause to a query.

	.. php:method:: where($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 要做判斷的欄位名稱或關聯陣列
		:param	mixed	$value: 如果只有一個欄位，判斷它的數值
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生查詢的 ``WHERE`` 語法。呼叫多次函式會使用 ``AND`` 將SQL語法串接在一起。

	.. php:method:: orWhere($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 要做判斷的欄位名稱或關聯陣列
		:param	mixed	$value: 如果只有一個欄位，判斷它的數值
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生查詢的 ``WHERE`` 語法。呼叫多次函式會使用 ``OR`` 將SQL語法串接在一起。

	.. php:method:: orWhereIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜尋的欄位
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool	        $escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 ``WHERE`` 欄位 ``IN('項目', '項目')`` 的 SQL 查詢語法。如果合適則使用 ``OR`` 將SQL語法串接在一起。

	.. php:method:: orWhereNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜尋的欄位
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool	        $escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 ``WHERE`` 欄位 ``NOT IN('項目', '項目')`` 的 SQL 查詢語法。如果合適則使用 ``OR`` 將SQL語法串接在一起。

	.. php:method:: whereIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要檢查的欄位名稱
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool            $escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 ``WHERE`` 欄位 ``IN('項目' , '項目')`` 的 SQL 查詢語法。如果合適則使用 ``AND`` 將SQL語法串接在一起。

	.. php:method:: whereNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要檢查的欄位名稱
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool	        $escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 ``WHERE`` 欄位 ``NOT IN('項目', '項目')`` 的 SQL 查詢語法。如果合適則使用 ``AND`` 將SQL語法串接在一起。

	.. php:method:: groupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個分組的語法，在判斷式中使用 ``AND`` 。

	.. php:method:: orGroupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個分組的語法，在判斷式中使用 ``OR`` 。

	.. php:method:: notGroupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個分組的語法，在判斷式中使用 ``AND NOT`` 。

	.. php:method:: orNotGroupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個分組的語法，在判斷式中使用 ``OR NOT`` 。

	.. php:method:: groupEnd()

		:returns:	BaseBuilder 實體
		:rtype:	object

		結束一個分組

	.. php:method:: like($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	bool    $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``LIKE`` 的語法，呼叫多次函式會使用 ``AND`` 將SQL語法串接在一起。

	.. php:method:: orLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	bool    $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``LIKE`` 的語法，呼叫多次函式會使用 ``OR`` 將SQL語法串接在一起。

	.. php:method:: notLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	bool    $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ``NOT LIKE`` 的語法，呼叫多次函式會使用 ``AND`` 將SQL語法串接在一起。

	.. php:method:: orNotLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	bool    $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 NOT LIKE 的語法，呼叫多次函式會使用  ``OR`` 將SQL語法串接在一起。

	.. php:method:: having($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 欄位/數值的識別符號(字串)或關聯陣列的組合
		:param	string	$value: 如果 $key 為識別符號，則代表要尋找的數值
		:param	string	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 HAVING 的語法，呼叫多次函式會使用 AND 將SQL語法串接在一起。

	.. php:method:: orHaving($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 欄位/數值的識別符號(字串)或關聯陣列的組合
		:param	string	$value: 如果 $key 為識別符號，則代表要尋找的數值
		:param	string	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 HAVING 的語法，呼叫多次函式會使用 OR 將SQL語法串接在一起。

	.. php:method:: orHavingIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜尋的欄位
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool	        $escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 HAVING 欄位 IN('項目', '項目') SQL 查詢語法，如果合適則使用 'OR' 將SQL語法串接在一起。

	.. php:method:: orHavingNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜尋的欄位
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool	        $escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 HAVING 欄位 NOT IN('項目', '項目') SQL 查詢語法，如果合適則使用 'OR' 將SQL語法串接在一起。

	.. php:method:: havingIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜尋的欄位
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool            $escape: 是否要跳脫數值或識別符號
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 HAVING 欄位 IN('項目', '項目') SQL 查詢語法，如果合適則使用 'AND' 將SQL語法串接在一起。

	.. php:method:: havingNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要檢查的欄位名稱
		:param	array|Closure   $values: 要被查詢的數值陣列，或是子查詢的匿名函式
		:param	bool	        $escape: 是否要跳脫數值或識別符號
		:param	bool            $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	BaseBuilder 實體
		:rtype:	object

		產生 HAVING 欄位 NOT IN('項目', '項目') SQL 查詢語法，如果合適則使用 'AND' 將SQL語法串接在一起。

	.. php:method:: havingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	bool    $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中將 ``LIKE`` 的語法加入至 ``HAVING`` 的部分，呼叫多次函式會使用 'AND' 將SQL語法串接在一起。

	.. php:method:: orHavingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	bool    $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中將 ``LIKE`` 的語法加入至 HAVING 的部分，呼叫多次函式會使用  ``OR`` 將SQL語法串接在一起。

	.. php:method:: notHavingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	bool    $insensitiveSearch: 是否使用不分大小寫的搜尋
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中將 NOT LIKE 的語法加入至 HAVING 的部分，呼叫多次函式會使用 'AND' 將SQL語法串接在一起。

	.. php:method:: orNotHavingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 欄位名稱
		:param	string	$match: 要做匹配的文字
		:param	string	$side: 在判斷式的哪一邊加入萬用字元 '%'
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中將 NOT LIKE 的語法加入至 HAVING 的部分，呼叫多次函式會使用  ``OR`` 將SQL語法串接在一起。

	.. php:method:: havingGroupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個給 HAVING 的分組的語法，在判斷式中使用 'AND' 。

	.. php:method:: orHavingGroupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個給 HAVING 的分組的語法，在判斷式中使用 'OR' 。

	.. php:method:: notHavingGroupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個給 HAVING 的分組的語法，在判斷式中使用 'AND NOT' 。

	.. php:method:: orNotHavingGroupStart()

		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		開始一個給 HAVING 的分組的語法，在判斷式中使用 'OR NOT' 。

	.. php:method:: havingGroupEnd()

		:returns:	BaseBuilder 實體
		:rtype:	object

		結束一個給 HAVING 的分組的語法，

	.. php:method:: groupBy($by[, $escape = NULL])

		:param	mixed	$by: 要做 group by 的欄位；字串或陣列
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 GROUP BY 的語法。

	.. php:method:: orderBy($orderby[, $direction = ''[, $escape = NULL]])

		:param	string	$orderby: 要做 order by 的欄位
		:param	string	$direction: 要排序的類型 - ASC 、 DESC 或 隨機
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 ORDER BY 的語法。

	.. php:method:: limit($value[, $offset = 0])

		:param	int	$value: 要限制結果列數的數量
		:param	int	$offset: 要忽略列數的數量
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 LIMIT 和 OFFSET 的語法。

	.. php:method:: offset($offset)

		:param	int	$offset: 要忽略列數的數量
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		在查詢中加入 OFFSET 的語法。

	.. php:method:: set($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: 欄位名稱或欄位/值的陣列
		:param	string	$value: 如果 $key 為單一欄位，則代表欄位的數值
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		加入欄位/數值的組合，之後再傳遞給 ``insert()`` 、 ``update()`` 或 ``replace()`` 。

	.. php:method:: insert([$set = NULL[, $escape = NULL]])

		:param	array	$set: 欄位/數值組合的關聯陣列
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	TRUE 代表插入成功，FALSE 則代表插入失敗
		:rtype:	bool

		編譯並執行 INSERT 的語法。

	.. php:method:: insertBatch([$set = NULL[, $escape = NULL[, $batch_size = 100]]])

		:param	array	$set: 要插入的資料
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:param	int	$batch_size: 一次插入的列數
		:returns:	成功插入的數量或回傳 FALSE插入失敗
		:rtype:	mixed

		編譯並執行批次的 ``INSERT`` 語法。

		.. note:: 當你提供超過 ``$batch_size`` 的資料，將會執行多個 ``INSERT`` 的查詢，每個查詢最多插入 ``$batch_size`` 的資料。

	.. php:method:: setInsertBatch($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: 欄位名稱或欄位/值的陣列
		:param	string	$value: 如果 $key 為單一欄位，則代表欄位的數值
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		加入欄位/數值的組合，以利稍後使用 ``insertBatch()`` 的插入。

	.. php:method:: update([$set = NULL[, $where = NULL[, $limit = NULL]]])

		:param	array	$set: 欄位/數值組合的關聯陣列
		:param	string	$where: WHERE 的字句
		:param	int	$limit: LIMIT 限制量
		:returns:	TRUE 代表更新成功，FALSE 則代表更新失敗
		:rtype:	bool

		編譯並執行 UPDATE 的語法。

	.. php:method:: updateBatch([$set = NULL[, $value = NULL[, $batch_size = 100]]])

		:param	array	$set: 欄位名稱或欄位/數值組合的關聯陣列
		:param	string	$value: 如果 $key 為單一欄位，則代表欄位的數值
		:param	int	$batch_size: 要在單一查詢中要分組的判斷式數量
		:returns:	成功更新的數量或回傳 FALSE 代表更新失敗
		:rtype:	mixed

		編譯並執行批次的 ``UPDATE`` 語法。

		.. note:: 當你提供欄位/數值的組合超過 ``$batch_size`` 的資料，將會執行多個的查詢，每次最多處理 ``$batch_size`` 的欄位/數值組合。

	.. php:method:: setUpdateBatch($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: 欄位名稱或欄位/值的陣列
		:param	string	$value: 如果 $key 為單一欄位，則代表欄位的數值
		:param	bool	$escape: 是否要跳脫數值或識別符號
		:returns:	``BaseBuilder`` 實體(鏈式方法)
		:rtype:	``BaseBuilder``

		加入欄位/數值的組合，以利稍後使用 ``updateBatch()`` 的更新。

	.. php:method:: replace([$set = NULL])

		:param	array	$set: 欄位/數值組合的關聯陣列
		:returns:	TRUE 代表執行成功, FALSE 代表執行失敗
		:rtype:	bool

		編譯並執行 REPLACE 的語法。

	.. php:method:: delete([$where = ''[, $limit = NULL[, $reset_data = TRUE]]])

		:param	string	$where: WHERE 條件
		:param	int	$limit: LIMIT 限制量
		:param	bool	$reset_data: TRUE 代表重置查詢中 "寫入" 的語法
		:returns:	``BaseBuilder`` 實體(鏈式方法) 或 FALSE 代表執行失敗
		:rtype:	mixed

		編譯並執行 DELETE 的查詢。

    .. php:method:: increment($column[, $value = 1])

        :param string $column: 要遞增的欄位名稱
        :param int    $value:  要遞增的數量

        將欄位的值增加指定的數量。如果欄位的類型不是數字，如 VARCHAR ，則很有可能被替換成 $value 。

    .. php:method:: decrement($column[, $value = 1])

        :param string $column: 要遞減的欄位名稱
        :param int    $value:  要遞減的數量

        將欄位的值減少指定的數量。如果欄位的類型不是數字，如 VARCHAR ，則很有可能被替換成 $value 。

	.. php:method:: truncate()

		:returns:	TRUE 代表執行成功， FALSE 代表執行失敗
		:rtype:	bool

		對資料表執行 TRUNCATE 的語法。

		.. note:: 如果資料庫平台沒有支援 TRUNCATE ，將會使用 DELETE 來取代。

	.. php:method:: emptyTable()

		:returns:	TTRUE 代表執行成功， FALSE 代表執行失敗
		:rtype:	bool

		使用 DELETE 語法刪除所有資料表內的資料

	.. php:method:: getCompiledSelect([$reset = TRUE])

		:param	bool	$reset: 是否重置目前查詢生成器的數值
		:returns:	將編譯後的 SQL 語法轉成字串類型
		:rtype:	string

		編譯一個 SELECT 語法，並以字串類型做回傳。

	.. php:method:: getCompiledInsert([$reset = TRUE])

		:param	bool	$reset: 是否重置目前查詢生成器的數值
		:returns:	將編譯後的 SQL 語法轉成字串類型
		:rtype:	string

		編譯一個 INSERT 語法，並以字串類型做回傳。

	.. php:method:: getCompiledUpdate([$reset = TRUE])

		:param	bool	$reset: 是否重置目前查詢生成器的數值
		:returns:	將編譯後的 SQL 語法轉成字串類型
		:rtype:	string

		編譯一個 UPDATE 語法，並以字串類型做回傳。

	.. php:method:: getCompiledDelete([$reset = TRUE])

		:param	bool	$reset: 是否重置目前查詢生成器的數值
		:returns:	將編譯後的 SQL 語法轉成字串類型
		:rtype:	string

		編譯一個 DELETE 語法，並以字串類型做回傳。
