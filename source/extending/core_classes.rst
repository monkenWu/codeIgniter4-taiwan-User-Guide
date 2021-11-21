****************************
建立核心系統類別
****************************

每當 CodeIgniter 運作時，都會有幾個基本類別作為核心框架的一部份，自動地初始化。但是，你可以使用自己的版本來替換掉任何一個核心的系統類別，甚至可以繼承後擴充核心版本。

**大多數的使用者永遠不需要這麼做，但對於想顯著改變 CodeIgniter 核心的使用者，可以選擇替換或擴充它們。**

.. note:: 搞亂核心系統類別將會有很多影響，所以在嘗試這件事情之前，要確定你知道自己在做什麼。

系統類別列表
=================

以下是每次 CodeIgniter 在運作時呼叫的系統檔案列表

* Config\\Services
* CodeIgniter\\Autoloader\\Autoloader
* CodeIgniter\\Config\\DotEnv
* CodeIgniter\\Controller
* CodeIgniter\\Debug\\Exceptions
* CodeIgniter\\Debug\\Timer
* CodeIgniter\\Events\\Events
* CodeIgniter\\HTTP\\CLIRequest (if launched from command line only)
* CodeIgniter\\HTTP\\IncomingRequest (if launched over HTTP)
* CodeIgniter\\HTTP\\Request
* CodeIgniter\\HTTP\\Response
* CodeIgniter\\HTTP\\Message
* CodeIgniter\\HTTP\\URI
* CodeIgniter\\Log\\Logger
* CodeIgniter\\Log\\Handlers\\BaseHandler
* CodeIgniter\\Log\\Handlers\\FileHandler
* CodeIgniter\\Router\\RouteCollection
* CodeIgniter\\Router\\Router
* CodeIgniter\\Security\\Security
* CodeIgniter\\View\\View
* CodeIgniter\\View\\Escaper


替換核心類別
======================

要使用自己的系統類別來替代掉預設的系統類別，請確定 :doc:`Autoloader <../concepts/autoloader>` 可以找得到你的類別，以及你的新類別實作了相應的介面，並修改 :doc:`Service <../concepts/services>` 來載入你的類別替換掉核心類別。

例如，你有一個新的 ``App\Libraries\RouteCollection`` 類別，你想用它來替換掉系統核心類別，你就可這樣創建你的類別：

::

    <?php

    namespace App\Libraries;

    use CodeIgniter\Router\RouteCollectionInterface;

    class RouteCollection implements RouteCollectionInterface
    {
        // ...
    }


然後，你得修改在 **app/Config/Services.php** 的 ``routes`` 服務，改成載入你的類別：

::

    public static function routes(bool $getShared = true)
    {
        if ($getShared)
        {
            return static::getSharedInstance('routes');
        }

        return new RouteCollection(static::locator(), config('Modules'));
    }

擴充核心系統類別
======================

如果你只需要在現有的程式庫中添加一些功能（也許是一兩個方法），那麼重新撰寫整個程式庫有點矯枉過正了。在這種情形下，最簡單的方法就是擴充類別，擴充類別與替換類別幾乎一模一樣，只有一個例外：

* 類別必須繼承父類別。

舉個例子，要擴充原生的 RouteCollection 類別，你得這麼做：

::

    <?php

    namespace App\Libraries;

    use CodeIgniter\Router\RouteCollection;

    class RouteCollection extends RouteCollection
    {
        // ...
    }

如果你需要在你的類別中使用建構函數，請確定你同時運作了父類別的擴充函數：

::

    <?php

    namespace App\Libraries;

    use CodeIgniter\Router\RouteCollection as BaseRouteCollection;

    class RouteCollection extends BaseRouteCollection
    {
        public function __construct()
        {
            parent::__construct();

            // your code here
        }
    }

**Tip ：** 在你的類別中，任何與父類別相同的函數都將被使用，而不會使用原生的函數，這就是所謂的方法覆寫，你可以利用這種方式大幅度地修改 CodeIgniter 的核心。