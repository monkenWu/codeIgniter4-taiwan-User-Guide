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

    <?php namespace App;

    use CodeIgniter\Test\FeatureTestCase;

    class TestFoo extends FeatureTestCase
    {
        public function setUp()
        {
            parent::setUp();
        }

        public function tearDown()
        {
            parent::tearDown();
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

    $result = $this->withRoutes($routes)
        ->get('users');

每個 routes 是一個包含三個元素的陣列，含有 HTTP 動詞（或是 add ）、指定的 URI 與路由的目的地。

設定會談值
----------------------

你可以使用 ``withSession()`` 方法來設定自訂的會談值，以便在測試中使用。這個方法需要傳入一個鍵值陣列，當這個方法被呼叫時，這些鍵值陣列將會對應到 PHP 的 $_SESSION 超全域變數中。這對於測試身份驗證來說很方便。 

::

    $values = [
        'logged_in' => 123
    ];

    $result = $this->withSession($values)
        ->get('admin');

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
    $result = $this->withBodyFormat('json')
        ->post('users', $userInfo);

    //Your controller can then get the parameters passed in with:
    $userInfo = $this->request->getJson();

Setting the Body
----------------

You can set the body of your request with the ``withBody()`` method. This allows you to format the body how you want
to format it. It is recommended that you use this if you have more complicated xml's to test. This will also not set
the Content-Type header for you so if you need that, you can set it with the ``withHeaders()`` method.

測試響應
====================

一旦你執行了 ``call()`` 函數並且獲得了回傳的結果後，你就可以在測試使用一些新的斷言。

.. note:: Response 物件在 ``$result->response`` 是公開屬性，如果有其他需要，你也可使用這個實體執行其他斷言。

響應狀態測試
------------------------

**isOK()**

證明這個響應是成功的響應 ，回傳一個布林值 true 或 false 。主要檢查 HTTP 狀態是否在 200 至 300 這個範圍之間。

::

    if ($result->isOK())
    {
        ...
    }

**assertOK()**

這個斷言僅使用了 **isOK()** 來測試響應結果。

::

    $this->assertOK();

**isRedirect()**

判斷響應是否為重新導向，回傳一個布林值 true 或 false 

::

    if ($result->isRedirect())
    {
        ...
    }

**assertRedirect()**

斷言這個響應是否是一個「重新導向響應」 的實體。

::

    $this->assertRedirect();

**assertStatus(int $code)**

斷言回傳的 HTTP 狀態碼與傳入的 $code 相符。

::

    $this->assertStatus(403);


會談斷言
------------------

**assertSessionHas(string $key, $value = null)**

斷言此時的會話中存在所指定鍵。如果你也傳入了 $value ，也會斷言你所指定的鍵的值是否與傳入的 $value 相符。

::

    $this->assertSessionHas('logged_in', 123);

**assertSessionMissing(string $key)**

斷言目前的會談不包含指定的 $key 。

::

    $this->assertSessionMissin('logged_in');


標頭斷言
-----------------

**assertHeader(string $key, $value = null)**

斷言響應中存在著一個名為 **$key** 的標頭。如果你也傳入了 **$value** ，也會斷言這個值是相符的。

::

    $this->assertHeader('Content-Type', 'text/html');

**assertHeaderMissing(string $key)**

斷言響應中不存在著名稱為 **$key** 的標頭。

::

    $this->assertHeader('Accepts');



Cookie 斷言
-----------------

**assertCookie(string $key, $value = null, string $prefix = '')**

斷言響應中存在著名為 **$key** 的 Cookie ，如果你也傳入了 **$value** ，也會判定這些值是否相符。如果你需要的話，也可以設定 cookie 的前綴，透過傳遞第三個參數作為判斷依據。

::

    $this->assertCookie('foo', 'bar');

**assertCookieMissing(string $key)**

斷言響應中不存在名為 **$key** 的 Cookie 。

::

    $this->assertCookieMissing('ci_session');

**assertCookieExpired(string $key, string $prefix = '')**

斷言名為 **$key** 的 Cookie 確實存在，但已經過期了。如果你需要的話，也可以設定 cookie 的前綴，透過傳遞第二個參數作為設定值。

::

    $this->assertCookieExpired('foo');


DOM 斷言
--------------

你可以利用下列的斷言執行測試，檢閱特定的元素與文字等內容是否存在於響應的 body 之中。

**assertSee(string $search = null, string $element = null)**

斷言文字與 HTML 存在於在頁面上。這個斷言可以指的是全體文字，或具體成搜索一個標記，例如指定 Clase 、 type 或 id 。

::

    // 斷言 Hello World 存在於頁面中
    $this->assertSee('Hello World');
    // 斷言存在著內容為 Hello World 的 h1 標籤
    $this->assertSee('Hello World', 'h1');
    // 斷言存在著包含 Hello World 的元素，並且它為 .notice Class 中的成員。
    $this->assertSee('Hello World', '.notice');
    // 斷言存在著包含 Hello World 的元素，並且它的 id 被宣告為 title  。
    $this->assertSee('Hellow World', '#title');

**assertDontSee(string $search = null, string $element = null)**

斷言的結果與 **assertSee()** 方法完全相反。

::

    // 斷言 Hello World 不存在於頁面中
    $results->dontSee('Hello World');
    // 斷言 Hello World 不存在於任何 h1 標籤中
    $results->dontSee('Hello World', 'h1');

**assertSeeElement(string $search)**

類似於 **assertSee()** 但它只斷言特定元素是否存在，並不會檢查任何文字內容。

::

    // 斷言 notice Class 在頁面上存在任何成員元素
    $results->seeElement('.notice');
    // 斷言頁面上具有 id 為 title 的元素
    $results->seeElement('#title')

**assertDontSeeElement(string $search)**

類似於 **assertSee()** ，但它只斷言一個元素是否不存在於頁面，它不檢查特定文字內容。

::

    // 斷言頁面不存在任何 id 為 title 的元素
    $results->dontSeeElement('#title');

**assertSeeLink(string $text, string $details=null)**

使用 **$text** 來斷言頁面上出現了帶有指定字串的超連接：

::

    // 斷言有一個文字為 Upgrade Account 的超連結存在於頁面
    $results->seeLink('Upgrade Account');
    // 斷言有一文字為 Upgrade Account 且它正好是 upsell class 成員的超連結
    $results->seeLink('Upgrade Account', '.upsell');

**assertSeeInField(string $field, string $value=null)**

斷言你所傳入的標籤與內容元素真實存在：

::

    // 斷言存在著名為 user 且值為 John Snow 的輸入
    $results->seeInField('user', 'John Snow');
    // 斷言陣列內的輸入
    $results->seeInField('user[name]', 'John Snow');

使用 JSON 
-----------------

響應經常會是 JSON 格式的回傳，特別是在呼叫 API 方法時。以下提供可以幫助你測試響應的方法。

**getJSON()**

這個

這個方法將以自串的形式回傳響應的 body ：

::

    // 響應 body 像是這樣:
    ['foo' => 'bar']

    $json = $result->getJSON();

    // 獲得的 $json 像是這樣:
    {
        "foo": "bar"
    }
 
.. note:: 需要注意的是， JSON 字串會漂亮地輸出在結果中。

**assertJSONFragment(array $fragment)**

斷言 $fragment 存在於 JSON 響應中，它不需要符合整個 JSON 值。

::

    // 響應 body 像是這樣:
    [
        'config' => ['key-a', 'key-b']
    ]

    // 將回傳 true
    $this->assertJSONFragment(['config' => ['key-a']);

.. note:: 這僅僅是使用了 phpUnit 的 `assertArraySubset() <https://phpunit.readthedocs.io/en/7.2/assertions.html#assertarraysubset>`_ 方法進行比較。

**assertJSONExact($test)**

類似於 **assertJSONFragment()** 但會檢閱整個 JSON 響應以確保結果精準地符合。

使用 XML
----------------

**getXML()**

如果你的應用程式會回傳 XML ，你可以使用這個方法檢閱它。