############
程式碼模組
############

CodeIgniter 支援使用程式碼模組化的方式，幫助你創建具有重用性的程式碼。模組通常是以特定的主題為中心，在你的大型應用程式中它可能是其中的小型應用程式。模組化的撰寫方式在框架中的任何標準檔案類型都有支援，例如：控制器、模型、視圖、設定檔案、輔助函數以及語言檔案等。你可以根據你的需求包含任意數量的模組。

.. contents::
    :local:
    :depth: 2

==========
命名空間
==========

模組功能的核心要素來自於 CodeIgniter 使用與 :doc:`PSR4 相容的自動載入 </concepts/autoloader>` 。雖然所有程式碼都可以使用 PSR4 的自動載入器和命名空間，但充分利用模組的主要方法，是在  **app/Config/Autoload.php** 的 ``psr4`` 的部分添加你的程式碼命名空間。

例如，我們想保留一個簡單的部落格模組，讓我們可以在不同的應用程式間重複使用。你可以創建一個名為公司名稱 Acme 的資料夾，用於儲存我們的模組。我們將這個資料夾放在專案根目錄的 **應用程式** 目錄旁邊：

::

    /acme        // New modules directory
    /application
    /public
    /system
    /tests
    /writable

打開 **app/Config/Autoload.php** 後，將 **Acme** 命名空間加入到 ``psr4`` 陣列屬性中：

::

    public $psr4 = [
        APP_NAMESPACE => APPPATH, // For custom namespace
        'Config'      => APPPATH . 'Config',
        'Acme'        => ROOTPATH . 'acme',
    ];

現在，我們可以透過 ``Acme`` 命名空間造訪 **acme** 資料夾。僅僅是這樣我們就能完成模組所需的 80% 工作，你應該要使用自己熟悉的命名空間並且熟練地使用它們。多個檔案類型將透過所有已經定義的命名空間進行自動掃描，這對於模組的工作要素來說是至關重要的。

一個模組內的通用資料夾結構將會與主應用程式的資料夾結構相同：

::

    /acme
        /Blog
            /Config
            /Controllers
            /Database
                /Migrations
                /Seeds
            /Helpers
            /Language
                /en
            /Libraries
            /Models
            /Views

當然，沒有誰可以強迫你使用這個結構進行開發，你應該以最適合自己的方式來組織你的模組，省去不需要的目錄。或是，替你的實體、介面或是儲存庫創建新的目錄。

===========================
自動載入無類別檔案
===========================

通常來說，你的模組可能不只有 PHP 類別，還會包含：程序式函數、引導檔案，模組常數檔案等。這些檔案通常不會以類別的方式被載入，這時可能會使用 ``require`` 的方式載入會使用到的檔案。

而 CodeIgniter 提供你一種自動載入這些非類別檔案的方式，就像是類別自動載入一樣。我們需要提供這些檔案的路徑清單，並將它們宣告在 **app/Config/Autoload.php** 中的 ``$files`` 屬性。

::

    public $files = [
        'path/to/my/functions.php',
        'path/to/my/constants.php',
        'path/to/my/bootstrap.php',
    ];


==============
自動探索
==============

很多時候，你需要替被引入的檔案指定完整的命名空間。但是，你可以透過組態設定將 CodeIgniter 設定成自動探索，讓它找到不同類型的檔案，並將模組集成到你的應用程式中，藉此使集成模組變得更加簡單：

- :doc:`事件 </extending/events>`
- :doc:`註冊器 </general/configuration>`
- :doc:`路由檔案 </incoming/routing>`
- :doc:`服務（ services ） </concepts/services>`

這是在 **app/Config/Modules.php** 檔案中進行設定的。

自動探索系統的工作原理是掃描 PSR4 命名空間內的特定目錄與檔案，而這些目錄和檔案已經在 **Config/Autoload.php** 中定義了。

為了讓自動探索系統在我們的 **Blog** 命名空間中工作，我們需要做一個小調整。 **Acme** 需要改成 **Acme\\Blog** 因為命名公籤中的每個「模組」都需要完全定義，探索的過程將在這個路徑上尋找可以被探索的項目，例如在： **/acme/Blog/Config/Routes.php** 中找到路由文件。

啟用或關閉探索
=======================

你可以使用 **$enabled** 這個類別變數開啟或關閉系統中所有自動探索的功能。 False 將禁用所有探索，最佳化效能，但模組的特殊功能將被否決。

指定被探索的項目
=======================

使用 **$activeExplorers**  選項，你可以指定哪些項目在自動探索的範圍。如果這個項目不存在，那麼這個項目將不會被自動探索，但是陣列中的其他項目仍在探索範圍。

Composer 與探索
======================

透過 Composer 安裝的軟體包也在預設探索的範圍之中，這是因為 Composer 的命名空間是採用 PSR4 規範的。 PSR0 的命名空間就不會被自動探索功能檢測到。

如果你不希望在定位檔案時掃描所有 Composer 的以知目錄，你可以透過編輯 ``Config\Modules.php`` 檔案，修改裡頭的 ``$discoverInComposer`` 變數來關閉這個功能：

::

    public $discoverInComposer = false;

==================
與檔案協同工作
==================

這個條目將針對每種檔案類型（控制器、視圖、語言檔案等）說明它們應該如何在模組中使用。其中的一些訊息在這份使用文件的相關位置有更詳細的描述，但我們將會在此簡述，以便你更容易地掌握所有部件事如何整合在一起的。

路由
======

在預設的情形下 :doc:`路由 </incoming/routing>` 會在模組內被自動掃描。他可以在 **Modules** （模組）的設定檔案中關閉。

.. note:: 因為這個檔案被包含在當前的作用域中，所以 ``$routes`` 實體已經替你自動宣告好了。如果你試圖重新定義這個類別，它將會導致錯誤發生。

過濾器
=======

預設的情況下，會自動掃描模組內的 :doc:`過濾器 </incoming/filters>` 。它可以在 **Modules** 的設定檔案中被關閉。

.. note:: 由於檔案包含在目前的作用域中，因此已經為你宣告好了 ``$filters`` 實體。如果你試圖重新宣告這個類別，則會導致錯誤。

在模組的 **Config/Filters.php** 檔案中，你需要宣告你使用的過濾器別名。

::

    $filters->aliases['menus'] = MenusFilter::class;

控制器
===========

我們不能透過 URI 檢測自動路由到 **app/Controllers** 目錄以外的控制器，所以你必須在路由檔案中進行指定：

::

    // Routes.php
    $routes->get('blog', 'Acme\Blog\Controllers\Blog::index');

為了減少這裡需要輸入的數量，**group** 這個路由的特色功能可以輔助你：

::

    $routes->group('blog', ['namespace' => 'Acme\Blog\Controllers'], function($routes)
    {
        $routes->get('/', 'Blog::index');
    });

設定檔案
============

在處理組態設定檔案時，不需要特別改變它們，這些檔案仍然是命名空間類別，並使用 ``new`` 保留字進行載入：

::

    $config = new \Acme\Blog\Config\Blog();

每當使用了 **config()** 函數時，將會自動探索相關的設定檔案。

遷移
==========

在已定義的命名空間內將會自動探索遷移的檔案。它會在每次執行時在所有的命名空間中找到所有的遷移。

填充
==========

只要提供了完整的命名空間，填充的檔案可以透過 CLI 使用，也可以從其他的填充檔案呼叫，若是你希望在 CLI 中呼叫填充檔案，則需要鍵入雙反斜線：

::

    > php public/index.php migrations seed Acme\\Blog\\Database\\Seeds\\TestPostSeeder

輔助函數
==========

當使用 ``helper()`` 方法時，只要是在命名空間 **Helpers** 目錄下的輔助函數，就會從已經定義的命名空間中被自動定位：

::

    helper('blog');

語言檔案
==============

當使用了 ``lang()`` 方法時，只要檔案遵照與主應用程式相同的目錄結構，語言檔案就會從已定義的命名空間中自動定位。

程式庫
=========

你得透過完全符合的類別名稱來實體化你的程式庫，所以我們不提供特殊的方式讓你造訪程式庫：

::

    $lib = new \Acme\Blog\Libraries\BlogLib();

模型
======

你得透過完全符合的類別名稱來實體化你的模型，所以我們不提供特殊的方式讓你造訪模型：

::

    $model = new \Acme\Blog\Models\PostModel();

視圖
=====

視圖可以按照 :doc:`視圖 </outgoing/views>` 使用說明中的描述，使用類別命名空間來進行載入：

::

    echo view('Acme\Blog\Views\index');
