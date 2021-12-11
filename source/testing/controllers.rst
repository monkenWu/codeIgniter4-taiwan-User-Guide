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

``ControllerTestTrait::execute()`` returns an instance of a ``TestResponse``. See `Testing Responses <response.html>`_ on
how to use this class to perform additional assertions and verification in your test cases.

Filter Testing
==============

Similar to Controller Testing, the framework provides tools to help with creating tests for
custom :doc:`Filters </incoming/filters>` and your projects use of them in routing.

The Helper Trait
----------------

Just like with the Controller Tester you need to include the ``FilterTestTrait`` in your test
cases to enable these features::

    <?php

    namespace CodeIgniter;

    use CodeIgniter\Test\CIUnitTestCase;
    use CodeIgniter\Test\FilterTestTrait;

    class FilterTestCase extends CIUnitTestCase
    {
        use FilterTestTrait;
    }

Configuration
-------------

Because of the logical overlap with Controller Testing ``FilterTestTrait`` is designed to
work together with ``ControllerTestTrait`` should you need both on the same class.
Once the trait has been included ``CIUnitTestCase`` will detect its ``setUp`` method and
prepare all the components needed for your tests. Should you need a special configuration
you can alter any of the properties before calling the support methods:

* ``$request`` A prepared version of the default ``IncomingRequest`` service
* ``$response`` A prepared version of the default ``ResponseInterface`` service
* ``$filtersConfig`` The default ``Config\Filters`` configuration (note: discovery is handle by ``Filters`` so this will not include module aliases)
* ``$filters`` An instance of ``CodeIgniter\Filters\Filters`` using the three components above
* ``$collection`` A prepared version of ``RouteCollection`` which includes the discovery of ``Config\Routes``

The default configuration will usually be best for your testing since it most closely emulates
a "live" project, but (for example) if you wanted to simulate a filter triggering accidentally
on an unfiltered route you could add it to the Config::

    class FilterTestCase extends CIUnitTestCase
    {
        use FilterTestTrait;

        protected function testFilterFailsOnAdminRoute()
        {
            $this->filtersConfig->globals['before'] = ['admin-only-filter'];

            $this->assertHasFilters('unfiltered/route', 'before');
        }
    ...

Checking Routes
---------------

The first helper method is ``getFiltersForRoute()`` which will simulate the provided route
and return a list of all Filters (by their alias) that would have run for the given position
("before" or "after"), without actually executing any controller or routing code. This has
a large performance advantage over Controller and HTTP Testing.

.. php:function:: getFiltersForRoute($route, $position)

    :param    string    $route: The URI to check
    :param    string    $position: The filter method to check, "before" or "after"
    :returns:    Aliases for each filter that would have run
    :rtype:    string[]

    Usage example::

        $result = $this->getFiltersForRoute('/', 'after'); // ['toolbar']

Calling Filter Methods
----------------------

The properties describe in Configuration are all set up to ensure maximum performance without
interfering or interference from other tests. The next helper method will return a callable
method using these properties to test your Filter code safely and check the results.

.. php:function:: getFilterCaller($filter, $position)

    :param    FilterInterface|string    $filter: The filter instance, class, or alias
    :param    string    $position: The filter method to run, "before" or "after"
    :returns:    A callable method to run the simulated Filter event
    :rtype:    Closure

    Usage example::

        protected function testUnauthorizedAccessRedirects()
        {
            $caller = $this->getFilterCaller('permission', 'before');
            $result = $caller('MayEditWidgets');

            $this->assertInstanceOf('CodeIgniter\HTTP\RedirectResponse', $result);
        }

    Notice how the ``Closure`` can take input parameters which are passed to your filter method.

Assertions
----------

In addition to the helper methods above ``FilterTestTrait`` also comes with some assertions
to streamline your test methods.

The **assertFilter()** method checks that the given route at position uses the filter (by its alias)::

    // Make sure users are logged in before checking their account
    $this->assertFilter('users/account', 'before', 'login');

The **assertNotFilter()** method checks that the given route at position does not use the filter (by its alias)::

    // Make sure API calls do not try to use the Debug Toolbar
    $this->assertNotFilter('api/v1/widgets', 'after', 'toolbar');

The **assertHasFilters()** method checks that the given route at position has at least one filter set::

    // Make sure that filters are enabled
    $this->assertHasFilters('filtered/route', 'after');

The **assertNotHasFilters()** method checks that the given route at position has no filters set::

    // Make sure no filters run for our static pages
    $this->assertNotHasFilters('about/contact', 'before');
