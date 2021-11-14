####################
HTTP 特性測試
####################

特性測試允許你查驗你的應用程式進行單次呼叫的結果。它可能會回傳一個 Web 表單結果，或者是使用一個 API 端點。這是個很方便的功能，因為它讓你可以在測試的過程中，完整地模擬單個請求的生命週期，因此可以驗證路由是否正常工作、響應格式是否正確以及分析回傳結果等。

.. contents::
    :local:
    :depth: 2

建立測試用類別
==============

特性測試要求所有欲使用的測試類別都必須繼承 ``CodeIgniter\Test\FeatureTestCase`` 這個類別。由於你所繼承的 FeatureTestCase 類別也繼承了 `CIDatabaseTestCase <database.html>`_ 
這個類別，所以必須在執行你動作前先呼叫 ``parent::setUp()`` 與 ``parent::tearDown()`` ：

::

    <?php

    namespace App;

    use CodeIgniter\Test\DatabaseTestTrait;
    use CodeIgniter\Test\FeatureTestTrait;

    class TestFoo extends CIUnitTestCase
    {
        use DatabaseTestTrait, FeatureTestTrait;

        protected function setUp(): void
        {
            parent::setUp();

            $this->myClassMethod();
        }

        protected function tearDown(): void
        {
            parent::tearDown();

            $this->anotherClassMethod();
        }
    }

發送一個頁面請求
=================

本質而言， FeatureTestCase 僅僅允許你使用 ``call`` 方法在應用程式上呼叫端點並回傳結果。第一個參數是你所要使用的 HTTP 方法（最常見的是 GET 或 POST ），第二個參數是你所想要測試的網站路徑，第三個參數接受你傳入一個陣列，填充你所使用的 HTTP 動詞在 PHP 下的超全域變數——也就是說，在 **GET** 方法下 **$_GET** 將被填充，在 **POST** 方法下 **$_POST** 將被你填充。

::

    // 取得一個簡單頁面
    $result = $this->call('get', site_url());

    // 送出一個表單
    $result = $this->call('post', site_url('contact'), [
        'name' => 'Fred Flintstone',
        'email' => 'flintyfred@example.com'
    ]);

針對各式 HTTP 動詞也有提供相應的的速記方法，減少打字量之餘也可以使程式碼更加清晰：

::

    $this->get($path, $params);
    $this->post($path, $params);
    $this->put($path, $params);
    $this->patch($path, $params);
    $this->delete($path, $params);
    $this->options($path, $params);

.. note:: 並不是對每個 HTTP 動詞都會使用到 $params 陣列，但為了保持程式的一致，我們將預設把 $params 包含在內。

設定預設路由
------------------------

你可以透過傳入一個 routes 陣列至 ``withRoutes()`` 方法，來宣告一個自訂的路由集合。若是宣告了路由內容，將會覆蓋系統中現有的路由設定。

::

    $routes = [
       [ 'get', 'users', 'UserController::list' ]
     ];

    $result = $this->withRoutes($routes)->get('users');

每個 routes 是一個包含三個元素的陣列，含有 HTTP 動詞（或是 add ）、指定的 URI 與路由的目的地。

設定會談值
----------------------

你可以使用 ``withSession()`` 方法來設定自訂的會談值，以便在測試中使用。這個方法需要傳入一個鍵值陣列，當這個方法被呼叫時，這些鍵值陣列將會對應到 PHP 的 $_SESSION 超全域變數中。這對於測試身份驗證來說很方便。 

::

    $values = [
        'logged_in' => 123
    ];

    $result = $this->withSession($values)->get('admin');

    // Or...

    $_SESSION['logged_in'] = 123;

    $result = $this->withSession()->get('admin');

Setting Headers
---------------

You can set header values with the ``withHeaders()`` method. This takes an array of key/value pairs that would be
passed as a header into the call.::

    $headers = [
        'CONTENT_TYPE' => 'application/json'
    ];

    $result = $this->withHeaders($headers)->post('users');

繞過事件
----------------

在你的應用程式中，使用「事件」是非常方便的事情，但在測試中可能會出現問題，尤其是「發送郵件」這種事件。你可以透過 ``skipEvents()`` 方法告訴系統跳過任何事件：

::

    $result = $this->skipEvents()
        ->post('users', $userInfo);

Formatting The Request
-----------------------

You can set the format of your request's body using the ``withBodyFormat()`` method. Currently this supports either
`json` or `xml`. This will take the parameters passed into ``call(), post(), get()...`` and assign them to the
body of the request in the given format. This will also set the `Content-Type` header for your request accordingly.
This is useful when testing JSON or XML API's so that you can set the request in the form that the controller will expect.
::

    //If your feature test contains this:
    $result = $this->withBodyFormat('json')->post('users', $userInfo);

    //Your controller can then get the parameters passed in with:
    $userInfo = $this->request->getJson();

Setting the Body
----------------

You can set the body of your request with the ``withBody()`` method. This allows you to format the body how you want
to format it. It is recommended that you use this if you have more complicated xml's to test. This will also not set
the Content-Type header for you so if you need that, you can set it with the ``withHeaders()`` method.

測試響應
====================

``FeatureTestTrait::call()`` returns an instance of a ``TestResponse``. See `Testing Responses <response.html>`_ on
how to use this class to perform additional assertions and verification in your test cases.
