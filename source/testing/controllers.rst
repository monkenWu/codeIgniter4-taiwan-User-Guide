###################
控制器測試
###################

透過一些新的輔助類別與 trait（特性），將會讓測試控制器變得更加便利。測試控制器時，你可以在控制器內執行程式碼，而不需要先執行整個應用程式 bootstrap （啟動）的過程。大多數時候，使用 `特性測試工具 <feature.html>`_ 會更加簡單，但這個功能將始終存在，以備不時之需。

.. note:: 因為整個框架沒有經過啟動，所以在某些特殊的狀況，你會無法使用這種方式測試控制器。

.. contents::
    :local:
    :depth: 2

使用輔助特性
================

你只需要在測試中宣告使用 ``ControllerTester`` 特性，便可以在任何的基礎測試類別中使用它。

::

    <?php

    namespace CodeIgniter;

    use CodeIgniter\Test\ControllerTestTrait;
    use CodeIgniter\Test\CIUnitTestCase;
    use CodeIgniter\Test\DatabaseTestTrait;

    class TestControllerA extends CIUnitTestCase
    {
        use ControllerTestTrait, DatabaseTestTrait;
    }

一旦包含了這個特性在你的測試類別中，你就可以開始設定環境，包括請求、響應類別、請求 body ，與 URI 等內容。當你用 ``controller()`` 方法指定你所要使用的 controller 時，請傳入該 controller 全稱的類別名稱。最後，使用 ``execute()`` 呼叫你所想要執行的方法名稱。

::

    <?php

    namespace CodeIgniter;

    use CodeIgniter\Test\ControllerTestTrait;
    use CodeIgniter\Test\CIUnitTestCase;
    use CodeIgniter\Test\DatabaseTestTrait;

    class TestControllerA extends CIUnitTestCase
    {
        use ControllerTestTrait, DatabaseTestTrait;

        public function testShowCategories()
        {
            $result = $this->withURI('http://example.com/categories')
                           ->controller(\App\Controllers\ForumController::class)
                           ->execute('showCategories');

            $this->assertTrue($result->isOK());
        }
    }

輔助方法
==============

**controller($class)**

指定你所要測試的控制器類別名稱，第一個參數必須是完全正確的類別名稱（包括命名空間）：

::

    $this->controller(\App\Controllers\ForumController::class);

**execute($method)**

在控制器中執行你所指定的方法，唯一的參數就是你所要執行的方法名稱：

::

    $results = $this->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

這將回傳一個新的輔助類別，它本身提供了一些用於檢查響應本身的方法，詳情請繼續閱讀下文。

**withConfig($config)**

允許你透過修改後的 **Config\App.php** 版本來測試不同的設定。

::

    $config = new Config\App();
    $config->appTimezone = 'America/Chicago';

    $results = $this->withConfig($config)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

如果你沒有提供這個設定，將使用應用程式的 app 設定檔案。

**withRequest($request)**

這個方法允許你提供一個 **IncomingRequest** 實體來滿足你的測試需求：

::

    $request = new CodeIgniter\HTTP\IncomingRequest(new Config\App(), new URI('http://example.com'));
    $request->setLocale($locale);

    $results = $this->withRequest($request)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

如果你沒有提供這個實體，那麼將會自動產生一個 IncomingRequest 實體，以及預設的應用程式值傳遞到你的控制器中。

**withResponse($response)**

你可以傳遞一個 **Response** 實體給這個方法：

::

    $response = new CodeIgniter\HTTP\Response(new Config\App());

    $results = $this->withResponse($response)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

如果你沒有提供這個實體，那麼將會自動產生一個 Response 實體，以及預設的應用程式值傳遞到你的控制器中。

**withLogger($logger)**

你可以傳遞 **Logger** 實體至這個方法：

::

    $logger = new CodeIgniter\Log\Handlers\FileHandler();

    $results = $this->withResponse($response)
                    ->withLogger($logger)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

如果你沒有提供這個實體，那麼將會自動產生一個 Logger 實體，以及預設的組態設定值傳遞到你的控制器中。

**withURI($uri)**

這個方法允許你提供一個新的 URI ，模擬使用者端在執行這個控制器時的 URL 。如果你需要在控制器中檢查 URL 區段的話，這將會是個很有用的功能。這個方法唯一的參數是代表有效 URI 的字串：

::

    $results = $this->withURI('http://example.com/forums/categories')
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

為了避免例外發生，在測試的過程中提供 URI 是一個很好的做法。

**withBody($body)**

這個方法允許你提供一個自定義 body 。在測試 API 控制器時，當你需要設定一個 JSON 值作為 body 的時候後，這個功能將會非常有用處。唯一的參數是代表請求主體的字串：

::

    $body = json_encode(['foo' => 'bar']);

    $results = $this->withBody($body)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

驗證響應
=====================

``ControllerTestTrait::execute()`` 會回傳一個 ``TestResponse`` 的實體。有關如何在測試案例中利用這個類別執行其他斷言和驗證的相關資訊，請參閱 `測試響應` <response.html>`_ 。

過濾器測試
==============

與控制器測試類似，框架提供了一些工具來幫助你建立用於 :doc:`過濾器 </incoming/filters>` 的測試，以及在你的專案中利用路由使用它們。

輔助特性
----------------

就像是控制器測試，你需要在你的測試案例中引入 ``FilterTestTrait`` 來啟用輔助特性提供的功能。

::

    <?php

    namespace CodeIgniter;

    use CodeIgniter\Test\CIUnitTestCase;
    use CodeIgniter\Test\FilterTestTrait;

    class FilterTestCase extends CIUnitTestCase
    {
        use FilterTestTrait;
    }

組態
-------------

因為與控制器測試的邏輯重疊， ``FilterTestTrait`` 被設計為與 ``ControllerTestTrait`` 一起工作，所以在同一個類別上，將同時需要這個特性存在。一旦上述特性被引入， ``CIUnitTestCase`` 將檢查類別中的 ``setUp`` 方法，替你的測試準備所有需要的元件。如果你需要一個特殊的組態，你可以在呼叫支援方法之前改變下列屬性。

* ``$request`` 預設 ``IncomingRequest`` 服務的預備版本
* ``$response`` 預設 ``ResponseInterface`` 服務的預備版本
* ``$filtersConfig`` 預設的 ``ConfigFilters`` 配置（注意：探索是由 ``Filters`` 處理的，所以這不包括模組的別名）。
* ``$filters`` 使用上述三個元件的 ``CodeIgniter\Filters\Filters`` 實體
* ``$collection`` 是 ``RouteCollection`` 的預備版本，其中包括 ``Config\Routes`` 的探索

預設組態通常會符合你的測試，因為它最接近「即時」專案，但如果你想模擬一個過濾器在某個沒有使用過濾器的路由意外觸發，你可以將這麼設定：

::

    class FilterTestCase extends CIUnitTestCase
    {
        use FilterTestTrait;

        protected function testFilterFailsOnAdminRoute()
        {
            $this->filtersConfig->globals['before'] = ['admin-only-filter'];

            $this->assertHasFilters('unfiltered/route', 'before');
        }
    ...

檢查路由
---------------

第一個輔助方法是 ``getFiltersForRoute()`` ，它將模擬你所提供的路由，並回傳所有過濾器（按照別名）的清單，這些過濾器將執行在你所設定的位置（「before」 或 「after」），無需實際執行任何控制器或路由程式。比起控制器和 HTTP 測試將有更多的效能優勢。

.. php:function:: getFiltersForRoute($route, $position)

    :param    string    $route: 被檢查的 URL 
    :param    string    $position: 被檢查的過濾器方法（「before」 或 「after」）
    :returns:    將執行的每個過濾器的別名
    :rtype:    string[]

    使用範例
    
    ::

        $result = $this->getFiltersForRoute('/', 'after'); // ['toolbar']

呼叫過濾器方法
----------------------

組態中所提到的屬性都是為了保證最好的效能，並且不受其他測試的干擾與影響。下一個輔助方法將使用這些屬性回傳一個能夠被呼叫的匿名函數，藉此安全地測試你的過濾器程式並檢查結果。

.. php:function:: getFilterCaller($filter, $position)

    :param    FilterInterface|string    $filter: 過濾器實體、類別或別名
    :param    string    $position: 被執行的過濾器方法「before」 或 「after」
    :returns:   用於執行模擬過濾器事件的可呼叫匿名函數
    :rtype:    Closure

    使用範例
    
    ::

        protected function testUnauthorizedAccessRedirects()
        {
            $caller = $this->getFilterCaller('permission', 'before');
            $result = $caller('MayEditWidgets');

            $this->assertInstanceOf('CodeIgniter\HTTP\RedirectResponse', $result);
        }

    請注意， ``Closure`` 接受何種參數的傳入，這些參數會被傳遞給你的過濾器方法。

斷言
----------

除了上面的輔助方法 ``FilterTestTrait`` 還附帶了一些斷言用於簡化你的測試方法。

**assertFilter()** 方法用於檢查你所給予的路由是否使用某個過濾器（透過別名）：

::

    // Make sure users are logged in before checking their account
    $this->assertFilter('users/account', 'before', 'login');

**assertNotFilter()** 方法用於檢查你所給予的路由是否沒有使用某個過濾器（透過別名）：

::

    // Make sure API calls do not try to use the Debug Toolbar
    $this->assertNotFilter('api/v1/widgets', 'after', 'toolbar');

**assertHasFilters()** 方法用於檢查你所給予的路由是否至少設定了一個過濾器：

::

    // Make sure that filters are enabled
    $this->assertHasFilters('filtered/route', 'after');

**assertNotHasFilters()** 方法檢查你所給予的路由是否沒有設定過濾器：

::

    // Make sure no filters run for our static pages
    $this->assertNotHasFilters('about/contact', 'before');
