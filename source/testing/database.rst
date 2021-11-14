=====================
資料庫測試
=====================

.. contents::
    :local:
    :depth: 2

建立一個測試類別
===================

為了利用 CodeIgniter 提供的內建資料庫工具進行測試，你的測試類別必須繼承 ``CIDatabaseTestCase`` ：

::

    <?php

    namespace App\Database;

    use CodeIgniter\Test\CIUnitTestCase;
    use CodeIgniter\Test\DatabaseTestTrait;

    class MyTests extends CIUnitTestCase
    {
        use DatabaseTestTrait;

        // ...
    }

因為大多數的特殊功能是在 ``setUp()`` 與 ``tearDown()`` 階段執行的，所以你必須確定在使用某些方法前呼叫了父類別的方法，否則你將無法實作這一章所描述的大部分功能：

::

    <?php

    namespace App\Database;

    use CodeIgniter\Test\CIUnitTestCase;
    use CodeIgniter\Test\DatabaseTestTrait;

    class MyTests extends CIUnitTestCase
    {
        use DatabaseTestTrait;

        public function setUp()
        {
            parent::setUp();

            // Do something here....
        }

        public function tearDown()
        {
            parent::tearDown();

            // Do something here....
        }
    }

建立一個測試用資料庫
==========================

在運作資料庫測試時，需要提供一個可以在測試時使用的資料庫。而測試用資料庫 CodeIgniter 框架提供了特別的工具，你不需要使用 PHPUnit 內建的資料庫功能。第一步，請確認你已經在  **app/Config/Database.php** 設定檔案中設定了一個 ``tests``  測試用資料庫組。你可以透過這個設定指定一個只有在運作測試時才使用的資料庫，以確保你它資料的安全。

如果你的團隊中有多個工程師，你可以將屬於你的設定存在 **.env** 檔案中。若想達成這個效果，請編輯檔案並且確認在下面提到的設定確實存在並且擁有正確的資訊：

::

    database.tests.dbdriver = 'MySQLi';
    database.tests.username = 'root';
    database.tests.password = '';
    database.tests.database = '';

遷移與資料填充
--------------------

在運作測試時，你需要確認你的資料庫有正確的 schema （綱目）設定，並且每次測試時都需要確保它處於全知的狀態。你可以透過在測試中添加幾個類別屬性，用於資料庫遷移和資料填充，藉此設定你的資料庫。

::

    <?php

    namespace App\Database;

    use CodeIgniter\Test\CIUnitTestCase;
    use CodeIgniter\Test\DatabaseTestTrait;

    class MyTests extends CIUnitTestCase
    {
        use DatabaseTestTrait;

        protected $refresh  = true;
        protected $seed     = 'TestSeeder';
        protected $basePath = 'path/to/database/files';
    }

**$migrate**

This boolean value determines whether the database migration runs before test.
By default, the database is always migrated to the latest available state as defined by ``$namespace``.
If false, migration never runs. If you want to disable migration, set false.

**$migrateOnce**

This boolean value determines whether the database migration runs only once. If you want
to run migration once before the first test, set true. If not present or false, migration
runs before each test.

**$refresh**

這個布林值決定了在每次測時前資料庫是否需要完全初始化，如果為 true ，則所有的遷移都將回歸到 0 版本，資料庫會被遷到你所設定的最新且可用的遷移。

**$seed**

如果這個屬性存在且不為 empty （空值），則這個屬性所宣告指定的資料填充檔案，將會在每次運作測試前用於資料庫填充使用。

**$seedOnce**

This boolean value determines whether the database seeding runs only once. If you want
to run database seeding once before the first test, set true. If not present or false, database seeding
runs before each test.

**$basePath**

在預設的情形下， CodeIgniter 將會在 **tests/_support/Database/Seeds** 中查找測試過程中應該執行的資料填充檔案。你可以透過指定 ``$basePath`` 屬性來改變這個預設的目錄。而你所宣告的路徑不應該包括 **seeds** 資料夾，應該包括的是擁有這個子路的單一目錄的路徑。

**$namespace**

在預設的情形下， CodeIgniter 會在 **tests/_support/Database/Migrations** 中查找在測試過程中應該運作的遷移檔案。你可以透過在 ``$namespace`` 屬性中宣告一個新的命名空間來更改這個位置。你所需告的新命名空間不應該包括 **Database/Migrations** 這個路徑，而應該是去除了這些路徑的基本命名空間。要從所有可用的命名空間中運作遷移，請將這個屬性設定為 ``null`` 。

輔助方法
==============

**CIDatabaseTestCase** 類別提供了幾個輔助方法來幫助你測試你的資料庫。

**regressDatabase()**

Called during ``$refresh`` described above, this method is available if you need to reset the database manually.

**migrateDatabase()**

Called during ``setUp``, this method is available if you need to run migrations manually.

**seed($name)**

手動將資料庫填充載入資料庫中。唯一的參數使指你所要運作的資料填充檔案，這個檔案必須存在於 ``$basePath`` 所指定的路徑中。

**dontSeeInDatabase($table, $criteria)**

假設資料庫中不存在 ``$criteria`` 中所宣告的鍵值陣列所提到的資料。

::

    $criteria = [
        'email'  => 'joe@example.com',
        'active' => 1
    ];
    $this->dontSeeInDatabase('users', $criteria);

**seeInDatabase($table, $criteria)**

假設資料庫中存在著與 ``$criteria`` 所宣告的鍵值陣列相同的一筆資料。

::

    $criteria = [
        'email'  => 'joe@example.com',
        'active' => 1
    ];
    $this->seeInDatabase('users', $criteria);

**grabFromDatabase($table, $column, $criteria)**

回傳指定的資料表中 ``$column`` 的值。如果這個值與 ``$criteria`` 提到的相同，則會回傳 ``$column`` 的值。如果這個方法找到了一筆以上的資料，那麼它將只會對第一筆資料進行測試。

::


    $username = $this->grabFromDatabase('users', 'username', ['email' => 'joe@example.com']);

**hasInDatabase($table, $data)**

在資料庫中插入一條新的資料，這筆資料將會在測試運作完畢後被刪除。 ``$data`` 是一個鍵值陣列，這其中包含著你需要插入置資料表中資料。

::

    $data = [
        'email' => 'joe@example.com',
        'name'  => 'Joe Cool'
    ];
    $this->hasInDatabase('users', $data);

**seeNumRecords($expected, $table, $criteria)**

斷言在資料庫中擁有與 ``$criteria`` 相符的資料。

::

    $criteria = [
        'active' => 1
    ];
    $this->seeNumRecords(2, 'users', $criteria);