######################
資料庫組態設定
######################

.. contents::
    :local:
    :depth: 2

CodeIgniter 提供一個組態設定檔讓你設定資料庫連線資料（使用者名稱、密碼、資料庫名稱等...）。這一個設定檔位於 **app/Config/Database.php** 。你也可以藉由 .env 檔設定資料庫連線資料。參閱以下說明了解更多。

資料庫設定值被存放在一個遵照以下規範的類別屬性裡面：

::

    public $default = [
        'DSN'      => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => '',
        'database' => 'database_name',
        'DBDriver' => 'MySQLi',
        'DBPrefix' => '',
        'pConnect' => true,
        'DBDebug'  => true,
        'charset'  => 'utf8',
        'DBCollat' => 'utf8_general_ci',
        'swapPre'  => '',
        'encrypt'  => false,
        'compress' => false,
        'strictOn' => false,
        'failover' => [],
    ];

類別屬性的名稱就是連線名稱，並且可以用特殊群組名稱連線。

.. note:: SQLite3 資料庫的預設位置位於 ``writable`` 資料夾內，如果你需要更改為至，則必須設定新資料夾的完整路徑。

某些資料庫驅動例如：PDO、PostgreSQL、Oracle、ODBC）可能需要提供完整的DSN字串。在這些狀況下，你就需要使用 DSN 設定參數，就像是使用原生PHP的驅動擴充模組一樣，例如：

::

	// PDO
	$default['DSN'] = 'pgsql:host=localhost;port=5432;dbname=database_name';

	// Oracle
	$default['DSN'] = '//localhost/XE';

.. note:: 如果你沒有為驅動指定 DSN 字串，CodeIgniter 將會使用你提供的其他設定去建構你的資料庫。

您還可以使用通用方式（類似 URL）設定資料來源名稱。在這種情況下，DSN 必須具有下列原型：

::

    $default['DSN'] = 'DBDriver://username:password@hostname:port/database';

要在使用 DSN 字串進行資料庫連線時覆蓋預設的設定值時，請將設定用的變數作為查詢字串加入到 DSN 中。

::

    // MySQLi
    $default['DSN'] = 'MySQLi://username:password@hostname:3306/database?charset=utf8&DBCollat=utf8_general_ci';
    // Postgre
    $default['DSN'] = 'Postgre://username:password@hostname:5432/database?charset=utf8&connect_timeout=5&sslmode=1';

.. note:: 如果你提供的 DSN 字串缺少了一些有效的參數（例如：資料庫的字元集），若這些參數出現在其他設定中，CodeIgniter 將自動在DNS的字串中附加上這些參數。

當主要連線因為某些原因無法連線時，你可以設定故障排除。可以透過以下的連接設定做故障排除：

::

    $default['failover'] = [
        [
            'hostname' => 'localhost1',
            'username' => '',
            'password' => '',
            'database' => '',
            'DBDriver' => 'MySQLi',
            'DBPrefix' => '',
            'pConnect' => true,
            'DBDebug'  => true,
            'charset'  => 'utf8',
            'DBCollat' => 'utf8_general_ci',
            'swapPre'  => '',
            'encrypt'  => false,
            'compress' => false,
            'strictOn' => false,
        ],
        [
            'hostname' => 'localhost2',
            'username' => '',
            'password' => '',
            'database' => '',
            'DBDriver' => 'MySQLi',
            'DBPrefix' => '',
            'pConnect' => true,
            'DBDebug'  => true,
            'charset'  => 'utf8',
            'DBCollat' => 'utf8_general_ci',
            'swapPre'  => '',
            'encrypt'  => false,
            'compress' => false,
            'strictOn' => false,
        ]
    ];

你可以指定任意數量的故障排除。

你可以選擇保存多個資料庫連線設定。例如，在一個系統下執行多個環境（開發、正式、測試等），你可以為了每一個開發環境建立獨立的資
料庫設定，並且可以按照你的需求任意切換。如果要設定 test 的資料庫環境，可以參閱以下範例：

::

    public $test = [
        'DSN'      => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => '',
        'database' => 'database_name',
        'DBDriver' => 'MySQLi',
        'DBPrefix' => '',
        'pConnect' => true,
        'DBDebug'  => true,
        'charset'  => 'utf8',
        'DBCollat' => 'utf8_general_ci',
        'swapPre'  => '',
        'compress' => false,
        'encrypt'  => false,
        'strictOn' => false,
        'failover' => []
    );

然後，要以全域的方式告訴系統，在設定檔中使用 test 這組連線：

::

	$defaultGroup = 'test';

.. note:: test 的名稱可以由你去任意更改。預設的情況下，主要的連線會使用 default 。但你也可以更改成跟你的專案有關係的名稱。你可以更改設定檔來檢測環境，並且在類別的建構函數中新增需要的邏輯，將 defaultGroup 自動更新成正確的數值：

You could modify the config file to detect the environment and automatically update the defaultGroup value to the correct one by adding the required logic within the class’ constructor:

::

    class Database
    {
        public $development = [...];
        public $test        = [...];
        public $production  = [...];

        public function __construct()
        {
            $this->defaultGroup = ENVIRONMENT;
        }
    }

使用 .env 檔設定
--------------------------

你也可以使用目前伺服器資料庫的設定，儲存你的設定參數在 ``.env`` 檔中。你只需要在預設值設定中輸入你想要改變的參數即可。
參數的命名必須遵守以下的格式，其中 default 是這個群組的名稱：

::

	database.default.username = 'root';
	database.default.password = '';
	database.default.database = 'ci4';

如同其他所有的。

設定值說明：
----------------------

======================  ===========================================================================================================
 設定名稱               說明
======================  ===========================================================================================================
**dsn**				DSN 連線字串 （所有設定一次完成的設定方式）。
**hostname** 		你的資料庫伺服器的 hostname 。通常本地端是'localhost'。
**username**		用以連線資料庫的使用者名稱。
**password**		用以連線資料庫的使用者密碼。
**database**		你所要連線的資料庫名稱。
**DBDriver**		資料庫驅動。例如： MySQLi 、Postgre 等。名稱必須完全符合驅動的名稱。
**DBPrefix**		資料表字首。當使用 :doc:`查詢生成器 <query_builder>` 查詢資料時，會自動新增該值到資料表的字首。這允許了多個 CodeIgniter 共用同個資料庫。
**pConnect**		TRUE/FALSE （boolean） - 是否使用保持連線的功能。
**DBDebug**			TRUE/FALSE （boolean） - 是否顯示資料庫的錯誤訊息。
**charset**	    	與資料庫溝通時，所使用的字元集。
**DBCollat**		與資料庫溝通時，所使用的字元排序。

			.. note:: 只有在 MySQLi 中才能使用。

**swapPre**			可以與dbprefix交換的資料表字首。這對於分散式的應用程式很有用，當你可能需要手動編輯查詢，並且需要由終端使用者去定義字首。
**schema**			資料庫綱目，預設為'public'。被PostgreSQL和ODBC的驅動做使用。
**encrypt**			是否使用加密連線。

			- sqlsrv 和 pdo/sqlsrv 驅動使用 TRUE/FALSE
			- MySQLi 和 pdo/mysql 驅動使用以下的陣列參數:

			    - ssl_key    - 私密金鑰檔案的路徑。
			    - ssl_cert   - 公開金鑰認證檔案的路徑。
			    - ssl_ca     - 認證機構檔案的路徑。
			    - ssl_capath - 包含PEM格式的可信任數位認證的目錄路徑。
			    - ssl_cipher - 加密密碼中， `允許` 使用的密碼列表，使用冒號（:）做區隔。
			    - ssl_verify - TRUE/FALSE。是否驗證伺服器認證（僅限 MySQLi 使用）。

**compress**		是否使用使用者端壓縮（MySQL專用）
**strictOn**		TRUE/FALSE （boolean） - 是否強制 "Strict Mode" 連線，使用嚴格的SQL對應用程式的開發是有幫助的。
**port**			設定資料庫 port 。要使用這項設定你需要在資料庫設定參數陣列當中加入。

			::

				$default['port'] = 5432;

======================  ===========================================================================================================

.. note:: 並不是所有設定值都需要被設定，這會根據你所使用的資料庫平台（ MySQL 、PostgreSQL 等）。例如：當你在使用 SQLite 時，你不需要設定使用者名稱或密碼，而且你的資料庫名稱就是資料庫的路徑。以上的資訊都是假設你在使用的是 MySQL 。