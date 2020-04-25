###################
控制器測試
###################

透過一些新的輔助類別與 trait（特性），將會讓測試控制器變得更加便利。測試控制器時，你可以在控制器內執行程式碼，而不需要先運作整個應用程式 bootstrap （啟動）的過程。大多數時候，使用 `特性測試工具 <feature.html>`_ 會更加簡單，但這個功能將始終存在，以備不時之需。

.. note:: 因為整個框架沒有經過啟動，所以在某些特殊的狀況，你會無法使用這種方式測試控制器。

使用輔助特性
================

你只需要在測試中宣告使用 ``ControllerTester`` 特性，便可以在任何的基礎測試類別中使用它。

::

    <?php namespace CodeIgniter;

    use CodeIgniter\Test\ControllerTester;

    class TestControllerA extends \CIDatabaseTestCase
    {
        use ControllerTester;
    }

一旦包含了這個特性在你的測試類別中，你就可以開始設定環境，包括請求、響應類別、請求 body ，與 URI 等內容。當你用 ``controller()`` 方法指定你所要使用的 controller 時，請傳入該 controller 全稱的類別名稱。最後，使用 ``execute()`` 呼叫你所想要運作的方法名稱。

::

    <?php namespace CodeIgniter;

    use CodeIgniter\Test\ControllerTester;

    class TestControllerA extends \CIDatabaseTestCase
    {
        use ControllerTester;

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

在控制器中執行你所指定的方法，唯一的參數就是你所要運作的方法名稱：

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

這個方法允許你提供一個新的 URI ，模擬使用者端在運作這個控制器時的 URL 。如果你需要在控制器中檢查 URL 區段的話，這將會是個很有用的功能。這個方法唯一的參數是代表有效 URI 的字串：

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

當控制器被執行完畢時，將會回傳一個新的 **ControllerResponse** 實體，這個實體提供了許多有用的方法，也可以透過呼叫方法產生 Request 與 Response 。

**isOK()**

提供你一個簡單的檢查，證明這個響應是成功的響應。主要檢查 HTTP 狀態是否在 200 至 300 這個範圍之間。

::

    $results = $this->withBody($body)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

    if ($results->isOK())
    {
        . . .
    }

**isRedirect()**

檢查響應是否是重新導向：

::

    $results = $this->withBody($body)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

    if ($results->isRedirect())
    {
        . . .
    }

**request()**

你可以透過這個方法產生請求物件：

::

    $results = $this->withBody($body)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

    $request = $results->request();

**response()**

你可以透過這個方法產生響應物件（如果存在）：

::

    $results = $this->withBody($body)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

    $response = $results->response();

**getBody()**

你可以透過 **getBody()** 方法造訪響應的 body ，這個響應會被發送到使用者端。這個方法可以產生 HTML 或 JSON 響應。

::

    $results = $this->withBody($body)
                    ->controller(\App\Controllers\ForumController::class)
                    ->execute('showCategories');

    $body = $results->getBody();

響應輔助方法
-----------------------

你得到的響應包含著一些輔助方法，用於驗證響應中的 HTML 輸出。這些方法對於在測試中宣告斷言時非常有用。

**see()** 方法將會檢查你所傳入的字串是否存在於頁面本身，你也可以更加具體的描述它，確定他是否是某種標記的描述，例如： tag 、 class 或 id ：

::

    // 驗證 Hello World 是否存在於頁面中
    $results->see('Hello World');
    // 驗證是否有存在著 Hello World 的 h1 標籤
    $results->see('Hello World', 'h1');
    // 驗證是否有存在著包含 Hello World 的元素，並且它為 .notice Class 中的成員。
    $results->see('Hello World', '.notice');
    // 驗證是否存在著包含 Hello World 的元素，並且它的 id 被宣告為 title  。
    $results->see('Hellow World', '#title');

而 **dontSee()** 的方法則完全相反於 **see()** 方法：

::

    // 驗證 Hello World 不存在於頁面中
    $results->dontSee('Hello World');
    // 驗證 Hello World 不存在於任何 h1 標籤中
    $results->dontSee('Hello World', 'h1');

**seeElement()** 與 **dontSeeElement()** 和前面的方法非常類似，但它並不會去比對元素的值。相反的，它指是檢查頁面上的某個元素是否存在：

::

    // 驗至 notice Class 在頁面上是否有任何成員元素
    $results->seeElement('.notice');
    // 驗證頁面上是否有 id 為 title 的元素
    $results->seeElement('#title')
    // 驗證頁面上是否不存在任何 id 為 title 的元素
    $results->dontSeeElement('#title');

你可以使用 **seeLink()** 來確認頁面上出現了某個帶有指定字串的超連接：

::

    // 驗證是否有一個文字為 Upgrade Account 的超連結
    $results->seeLink('Upgrade Account');
    // 驗證是否有一個文字為 Upgrade Account 且它正好是 upsell class 成員的超連結
    $results->seeLink('Upgrade Account', '.upsell');

**seeInField()** 用於驗證你所傳入的標籤與內容元素是否存在：

::

    // 驗證是否存在著名為 user 且值為 John Snow 的輸入
    $results->seeInField('user', 'John Snow');
    // 驗證陣列內的輸入
    $results->seeInField('user[name]', 'John Snow');

最後，你可以使用 **seeCheckboxIsChecked()** 方法來檢查某個核取方塊是否被選中：

::

    // 驗證 class 為 foo 的成員核取方塊是否被選中
    $results->seeCheckboxIsChecked('.foo');
    // 驗證 id 為 bar 的核取方塊是否被選中
    $results->seeCheckboxIsChecked('#bar');
