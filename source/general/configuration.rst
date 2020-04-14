#############
組態設定檔案
#############

常見的框架都會使用組態設定檔案來定義許多參數與初始設定。 CodeIgniter 則提供了簡單的類別，其中只需要設定相關的公開屬性。

與其他框架不同， CodeIgniter 的可設定選項並不包含在一個檔案中。相反，每個需要設定的類別都會有一個與它相同名稱的組態設定檔案。你可以在 **/app/Config** 資料夾中找到應用程式的設定文件。

.. contents::
    :local:
    :depth: 2

使用組態設定檔案
================================

你可以通過幾種不同的方式來訪問類別的組態設定檔案。

- 通過使用 ``new`` 保留字來創建一個實體：

::
 
	// Creating new configuration object by hand
	$config = new \Config\Pager();

- 通過 ``config()`` 函數：

::

	// Creating a new object with config function
	$config = config('Pager', false);

	// Get shared instance with config function
	$config = config('Pager');

	// Access config class with namespace
	$config = config( 'Config\\Pager' );

所有「組態設定物件」的屬性都是公開的，所以你可以像其他屬性一樣訪問設定檔：

::

        $config = config('Pager');
	// Access settings as object properties
	$pageSize = $config->perPage;

如果沒有提供命名空間，它將在所有已定義的命名空間與 **/app/Config/** 中尋找檔案。

所有 CodeIgniter 的組態設定檔案都以 ``Config`` 作為命名空間。在你的應用程式中使用這個命名空間將可以獲得最好的效能，因為它知道在哪裡可以找到這些檔案。

你可以通過使用不同的命名空間，將組態設定檔案放在任何你想要的資料夾中。這樣，你就可以把組態設定檔案放在正式伺服器上，且不可通過 web 造訪的資料夾中，同時將其保存在 **/app** 下，以便在開發的過程可以輕鬆訪問。

創建組態設定檔案
============================

當你需要一個新的組態設定時，首先，你得在你需要的地方創建一個新的檔案。預設的檔案位置（推薦用於大多數的情況）在 **/app/Config** 。這個類別應該使用合適的命名空間，並且它應該繼承 ``CodeIgniter\Config\BaseConfig`` ，確保它可以接收特定的環境設定。

相關的設定完成後，你可以開始定義這個類別，並用你設定的公開屬性來填充它。

::

    <?php namespace Config;

    use CodeIgniter\Config\BaseConfig;

    class CustomClass extends BaseConfig
    {
    	public $siteName  = 'My Great Site';
    	public $siteEmail = 'webmaster@example.com';

    }

環境變數
=====================

現今，「環境變數」是「應用程式設定」的最佳實踐之一。因為環境變數在部署的過程中，將替代程式碼被頻繁改變。例如：多個環環境，如開發者的開發機器與正式伺服器等多個環境，通常需要不同的設定值來滿足每個環境的特定情形。

密碼、API密鑰，或乃至任何隱私的敏感資訊，都應該使用環境變數作為儲存方案。

環境變數與 CodeIgniter
=====================================

CodeIgniter 通過使用「點 env」檔實現環境變數，這使得設定環境變數變得簡單又不用費功夫。「點 env」檔這個說法來自於 CodeIgniter 環境變數檔案的名稱，它指的是「 .env 」。

CodeIgniter 希望你將 **.env** 檔案置於根目錄下，緊鄰 ``system`` 與 ``app`` 目錄。在 CodeIgniter 中，有一個用於環境變數的樣板文件，它位於根目錄下，名為 **env** （注意！這個檔案的開頭並沒有 **點** ）。它內涵所有你的專案可能會使用到的變數集合，這些變數已經被分配了空值、虛值或默認值。你可以藉由將這個檔案重新命名成 **.env** 或將這個檔案複製一份命名成 **.env** 使它開始作用。這個檔案將會作為你的應用程式的起點。

.. important:: 為了不讓敏感資訊洩漏出去，請確定你的版本控制系統不追蹤 **.env** 檔案。以 *git* 來說，就是在  **.gitignore** 添加這個需要被忽略的檔案。

設定將會以「名稱 = 值」的方式，儲存在 **.env** 檔案中。

::

	S3_BUCKET = dotenv
	SECRET_KEY = super_secret_key
        CI_ENVIRONMENT = development

**.env** 檔會在你的應用程式運作時自動載入，並把設定放在運作的環境中。若一個變數已經存在於環境中，則不會被覆蓋過去。若需要取得環境變數，可以使用以下任何一個方法： ``getenv()`` 、 ``$_SERVER`` 或 ``$_ENV`` 。

::

	$s3_bucket = getenv('S3_BUCKET');
	$s3_bucket = $_ENV['S3_BUCKET'];
	$s3_bucket = $_SERVER['S3_BUCKET'];

巢狀變數
=================

為了減少程式量，你可以藉由在 ``${...}`` 內寫上變數名稱來重用已經在檔案裡宣告過的變數。

::

        BASE_DIR="/var/webroot/project-root"
        CACHE_DIR="${BASE_DIR}/cache"
        TMP_DIR="${BASE_DIR}/tmp"

命名空間變數
====================

有的時候，你會有幾個相同名稱的變數。這時，系統就需要一種方法來知道正確的設定應該是哪一個。這個問題可以透過變數的「命名空間」來解決。

命名空間變數使用「點」符號來限定變數名稱，這樣當他們被整合到同意個運作環境中，就會是唯一變數。先宣告一個區別性的前綴，再加上一個點（.），然後才是變數名稱。

::

    // not namespaced variables
    name = "George"
    db=my_db

    // namespaced variables
    address.city = "Berlin"
    address.country = "Germany"
    frontend.db = sales
    backend.db = admin
    BackEnd.db = admin

組態設定類別與環境變數
===============================================

當你實體化一個組態設定類別時，可能會考慮將命名空間中的環境變數合併到組態設定物件的屬性中。

如果命名空間變數的前綴與組態設定類別的命名空間完全相同，那麼設定的尾部（點後），將會作為屬性被設定。如果組態設定類別內已有這個變數的存在，再麼環境變數的值將會取代組態設定文件中的值。如果沒有任何同的變數存在，那麼組態設定類別的屬性將保持不變。在這種用法中，前綴必須與類別的完整命名空間（區分大小寫）相同。

::

    Config\App.CSRFProtection  = true    
    Config\App.CSRFCookieName = csrf_cookie
    Config\App.CSPEnabled = true

.. note::　命名空間前綴和屬性名稱區分大小寫。它們必須與組態設定類別檔案中定義的完整命名空間與屬性名稱完全相同。

你也可以使用 *短前綴* 來達成目的，它是一個只使用組態設定類別名稱的省略版命名空間。如果短前綴與類別名稱相符合，則 **.env** 的值將會取代組態設定文件的值。

::

    app.CSRFProtection  = true    
    app.CSRFCookieName = csrf_cookie
    app.CSPEnabled = true

note:: 當使用短前綴時，屬性名稱必須與類別所定義的屬性名稱完全一致。

將環境變數視為陣列
========================================

命名空間的環境變數還可以被設定成一個陣列。如果前綴與組態設定類別相同，在這之後可以加上「點」符號，並且命名陣列名稱後，再以「點」符號宣告屬性名稱，它就會被視為一個陣列。

::

    // regular namespaced variable
    Config\SimpleConfig.name = George

    // array namespaced variables
    Config\SimpleConfig.address.city = "Berlin"
    Config\SimpleConfig.address.country = "Germany"

根據上述所宣告例子， SimpleConfig 組態設定物件是我們所要設定的對象，這將會被視為：

::

    $address['city']    = "Berlin";
    $address['country'] = "Germany";

``$address`` 屬性的其他元素並不會被改變。

你也可以使用陣列屬性名稱作為前綴。如果環境檔案撰寫方與以下內容相同，那麼結果也會和上述一樣：

::

    // array namespaced variables
    Config\SimpleConfig.address.city = "Berlin"
    address.country = "Germany"


處理不同環境
===============================

藉由使用一個單獨的 **.env** 檔案，並根據環境的需要修改設定值，就可以很容易地完成對多個環境的組態設定。

請不要把你所有會使用到的設定都放在這個檔案中。事實上，它應該只記錄那些特定環境會使用到的設定項目，或者是密碼、 API 金鑰等不應該被揭露的敏感資訊。使用 **.env** 可以讓任何環境間的部屬所改變的東西都是相同的。

與大多數的設定相同，不論在哪個環境，將 **.env** 檔案放在專案根目錄下（與 ``system`` 以及 ``app`` 同層）。

為了避免在版本控制系統上公開程式碼後洩漏敏感資訊，你應該要確保你的版本控制系統沒有追蹤 **.env** 檔案。

.. _registrars:

註冊器
==========

組態設定檔案可以指定任意數量的「註冊器」，這可以是任何可能提供額外組態設定屬性的類別。可以藉由在你的組態設定檔案中新增一個註冊器屬性來實作，這個屬性包含了一個註冊器的名稱陣列。

::

    protected $registrars = [
        SupportingPackageRegistrar::class
    ];

為了讓類別被識別成「註冊器」，這個類別必須實作一個與組態設定類別相同的靜態函數，並且它應該要回傳一個關聯的屬性設定陣列。

當你的組態設定物件被實體化時，它將會 Foreach ``$registrars`` 所指定的類別。對於每一個包含與組態設定相同的方法名稱的類別，它將會呼叫該類別的方法，並且以與命名空間變數相同的方式，將所有回傳的屬性納入其中。

這就是一個組態設定類別的範例：

::

    <?php namespace App\Config;

    use CodeIgniter\Config\BaseConfig;

    class MySalesConfig extends BaseConfig
    {
        public $target        = 100;
        public $campaign      = "Winter Wonderland";
        protected $registrars = [
            '\App\Models\RegionalSales';
        ];
    }

與上例相關的 RegionalSales 模型可能是這個樣子的：

::

    <?php namespace App\Models;

    class RegionalSales
    {
        public static function MySalesConfig()
        {
            return ['target' => 45, 'actual' => 72];
        }
    }

通過上面的範例，當 `MySalesConfig` 被實體化時，它最終會宣告這兩個屬性，但 `$target` 屬性的值會被覆蓋，因為它把 `RegionalSalesModel` 視為「註冊器」，由此產生這樣子的組態設定屬性：

::

    $target   = 45;
    $campaign = "Winter Wonderland";
