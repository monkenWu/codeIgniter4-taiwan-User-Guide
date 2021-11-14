###########################
連接你的資料庫
###########################

.. contents::
    :local:
    :depth: 2

你可以在任何你需要的函數中，加入以下的程式碼來連接資料庫，或是在你的類別建構中宣告一個可用的全域參數。

::

	$db = \Config\Database::connect();

如果在上述的函數中，第一個參數 *不包含* 任何資訊，它將會以你系統指定的資料庫設定檔設定做連線。對於大多數人而言，這是第一首選方法。

以下的程式提供一個方便，且純粹是圍繞在上述內容中的方法：

::

    $db = db_connect();

可用的參數
--------------------

#. 資料庫群組名稱，必須是字串且完全符合類別設定的屬性名稱。預設參數為 ``$config->defaultGroup``　。
#. TRUE/FALSE （boolean）. 是否回傳分享的連線（查閱以下的多個資料庫連線）

手動連線到資料庫
---------------------------------

第一個參數是可以 **選擇的** ，從你的設定檔中指定一個特定的資料庫群組。例如：

從你的設定檔中選擇指定的群組，你可以這樣做：

::

	$db = \Config\Database::connect('group_name');

group_name 是設定檔中，要連線的群組名稱。

多個連線到同一個資料庫
-------------------------------------

預設情況下， ``connect()`` 方法每次都會回傳相同的資料庫連線。如果你需要單獨連線到同一個資料庫，則在第二個參數設定為 ``false`` ：

::

	$db = \Config\Database::connect('group_name', false);

連線到多個資料庫
================================

如果你需要同時連線到不只一個資料庫，你可以參考以下的程式碼：

::

	$db1 = \Config\Database::connect('group_one');
	$db  = \Config\Database::connect('group_two');

注意: 將 "group_one" 和 "group_two" 修改成你想要連接的群組名稱。

.. note:: 如果你只需要在同一個的連線中使用不同的資料庫，就不需要建立其他的資料庫設定。你可以參考以下程式碼，來切換到不同的資料庫： ``$db->setDatabase($database2_name);``

設定客製化的連線
===============================

要使用客製化的連線設定，你需要傳入一個陣列的資料庫設定值，而不是群組名稱。傳入陣列的格式，必須與設定檔中的格式相同。

::

    $custom = [
        'DSN'      => '',
        'hostname' => 'localhost',
        'username' => '',
        'password' => '',
        'database' => '',
        'DBDriver' => 'MySQLi',
        'DBPrefix' => '',
        'pConnect' => false,
        'DBDebug'  => (ENVIRONMENT !== 'production'),
        'charset'  => 'utf8',
        'DBCollat' => 'utf8_general_ci',
        'swapPre'  => '',
        'encrypt'  => false,
        'compress' => false,
        'strictOn' => false,
        'failover' => [],
        'port'     => 3306,
    ];
    $db = \Config\Database::connect($custom);


重新連線 / 保持有效的連線
===========================================

如果你正在執行複雜的PHP運算（例如：圖形處理）時，造成資料庫伺服器閒置逾時，你應該考慮在執行下一步的查詢前，用 reconnect() 方法 ping 伺服器，這可以正常地保持連線或重新建立連線。

.. important:: 如果你使用的是 MySQLi ，reconnect() 這個方法不會ping伺服器，而是會關閉連線並重新連線。

::

	$db->reconnect();

手動關閉資料庫連線
===============================

雖然CodeIgniter會智慧地幫助你關閉資料庫連線，但是你可以明確地關閉連線。

::

	$db->close();
