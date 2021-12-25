#######################
服務（services）
#######################

.. contents::
    :local:
    :depth: 2

介紹
============

CodeIgniter 中的所有的核心類別都由服務（services）提供。這意味著，要呼叫的類別不是使用寫死的方式載入，而是非常簡單的在設定檔中定義。此檔案使用工廠模式，用於創建所需類別的新實體。

我們舉個例子可能會比較好理解，假設你需要拉取 Timer 類別的實體。最簡單的方法是創建該類別的新實體::

	$timer = new \CodeIgniter\Debug\Timer();

這滿有用的，但如果你想要在這裡使用別的 Timer 類別，而他有著預設所沒有的功能。為此，你必須尋找散落在應用程式中所有已使用 Timer 類別的位置，這是耗時又耗力的，於是 服務 可以派上用場了。

我們讓中心類別為我們創建實體，而不是自己創建。這個類別非常的簡單，它只包含了每個我們想要用作 services 的類別的方法。該方法通常會回傳該類別的共用實體，任何依賴項都藉由它來維護。然後，我們來試試用這種方式創建 Timer 實體::

	$timer = \Config\Services::timer();

當你需要更改應用設定時，可以修改服務設定檔，接著無須任何操作，整個應用程式都會自動響應。現在，你可以利用這個特性輕鬆使用任何新方法，非常簡單且可靠。

.. note:: 建議僅在控制器內創建服務。其他檔案（如模型和程式庫）應將依賴項傳遞到建構函數或透過 setter 方法創建。

便利功能
---------------------

我們提供了兩個隨時可用的方便功能來獲取 services。

第一個是 ``service()`` ，回傳請求服務的新實體。唯一必需輸入的引數是服務的名稱。這與服務檔中的方法名稱，都會回傳類別的 SHARED 實體相同，因此多次調用函數應該都會回傳相同的實體::

	$logger = service('logger');

如果創建方法需要其他的引數，可以在服務名稱之後傳遞它們::

	$renderer = service('renderer', APPPATH.'views/');

第二個函數 ``single_service()`` 的工作方式類似：``service()`` ，但是他是回傳類別的新實體::

	$logger = single_service('logger');

定義 Services
=================

為了使 services 能正常工作，你必須能夠依賴每個具有常數 API 或 `介面 <http://php.net/manual/en/language.oop5.interfaces.php>`_ 的類別來使用。幾乎所有 CodeIgniter 的類別都提供了它們遵循的介面。你只需要確保能滿足介面的要求，並且知道這些類別是否是相容的，你就能擴充或替換核心的類別。

舉例來說，``RouterCollection`` 類別實作了 ``RouterCollectionInterface`` ，所以當你要替換創建路由的方式時，只需創建可以實作 ``RouterCollectionInterface`` 的新類別即可::

    class MyRouter implements \CodeIgniter\Router\RouteCollectionInterface
    {
        // Implement required methods here.
    }

最後，修改 **/app/Config/Services.php** 的內容來創建 ``MyRouter`` 的新實體，而不是自己創建一個::

    public static function routes()
    {
        return new \App\Router\MyRouter();
    }

允許引數
-------------------

在某些情況下，你會需要選項來在創建實體的時候傳遞設定給類別。由於 services 檔案是一個非常簡單的類別，因此實作起來很容易。

``renderer`` service 就是一個很好的例子。預設情況下，我們希望此類別能夠在 ``APPPATH.views/`` 中找到視圖。但是，我們希望開發人員有需要的話，也能夠更改這個路徑。因此，類別接受 ``$viewPath`` 作為建構函數的引數。service 的方法如下所示::

    public static function renderer($viewPath=APPPATH.'views/')
    {
        return new \CodeIgniter\View\View($viewPath);
    }

這將設置建構函數方法中的預設路徑，但也能輕鬆的更改它使用的路徑::

    $renderer = \Config\Services::renderer('/shared/views');

共用類別
-----------------

在某些情況下，你需要只創建一個 service 的實體。這很容易，只要從工廠方法內調用 ``getSharedInstance()`` 方法，就可以辨認是否已在類別中創建和保存實體，如果沒有，則它會創建一個新實體。所有的工廠方法都提供 ``$getShared = true`` 值作為最後一個引數，你也應該依照這個模式來實作::

    class Services
    {
        public static function routes($getShared = false)
        {
            if (! $getShared)
            {
                return new \CodeIgniter\Router\RouteCollection();
            }

            return static::getSharedInstance('routes');
        }
    }

搜尋 Service
-----------------

CodeIgniter 可以自動搜尋你在任何定義的命名空間中創建的任何 Config_Services.php 檔案，這讓使用模組服務檔變得很簡單。為了自動搜尋自訂服務檔，它們必須滿足以下要求：

- 他的命名空間必須定義在 ``Config\Autoload.php`` 
- 在命名空間內，這個檔案必須在 ``Config\Services.php`` 中
- 它必須繼承自 ``CodeIgniter\Config\BaseService``

用個小例子來說明一下，假設你在根目錄中創建了一個新目錄 ``Blog`` 。這將容納一個帶有控制器、模型等的 **blog 模組** ，並且你希望將某些類別作為 service 提供。第一步是創建新檔案 ``Blog\Config\Services.php`` ，他的架構應為：

::

    <?php namespace Blog\Config;

    use CodeIgniter\Config\BaseService;

    class Services extends BaseService
    {
        public static function postManager()
        {
            ...
        }
    }

現在，你可以像上文所述的那樣使用這個檔案。當你想要從任何控制器獲取貼文 service 時，只需使用框架的 ``Config\Services`` 類別來獲取服務：

::

    $postManager = Config\Services::postManager();

.. note:: 如果多個服務檔具有相同的方法名稱，則會回傳第一個找到的檔案的實體。