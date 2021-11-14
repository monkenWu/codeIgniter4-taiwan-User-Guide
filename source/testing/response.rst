#################
Testing Responses
#################

The ``TestResponse`` class provides a number of helpful functions for parsing and testing responses
from your test cases. Usually a ``TestResponse`` will be provided for you as a result of your
`Controller Tests <controllers.html>`_ or `HTTP Feature Tests <feature.html>`_, but you can always
create your own directly using any ``ResponseInterface``::

    $result = new \CodeIgniter\Test\TestResponse($response);
    $result->assertOK();

Testing the Response
====================

Whether you have received a ``TestResponse`` as a result of your tests or created one yourself,
there are a number of new assertions that you can use in your tests.

Accessing Request/Response
--------------------------

**request()**

You can access directly the Request object, if it was set during testing::

    $request = $results->request();

**response()**

This allows you direct access to the response object::

    $response = $results->response();

Checking Response Status
------------------------

**isOK()**

證明這個響應是成功的響應 ，回傳一個布林值 true 或 false 。主要檢查 HTTP 狀態是否在 200 至 300 這個範圍之間。

::

    if ($result->isOK()) {
        ...
    }

**assertOK()**

這個斷言僅使用了 **isOK()** 來測試響應結果。 **assertNotOK** 是這個斷言的反向用法。

::

    $result->assertOK();

**isRedirect()**

判斷響應是否為重新導向，回傳一個布林值 true 或 false 

::

    if ($result->isRedirect()) {
        ...
    }

**assertRedirect()**

斷言這個響應是否是一個「重新導向響應」 的實體。 **assertNotRedirect** 是這個斷言的反向用法。

::

    $result->assertRedirect();

**assertRedirectTo()**

Asserts that the Response is an instance of RedirectResponse and the destination
matches the uri given.
::

    $result->assertRedirectTo('foo/bar');

**getRedirectUrl()**

Returns the URL set for a RedirectResponse, or null for failure.
::

    $url = $result->getRedirectUrl();
    $this->assertEquals(site_url('foo/bar'), $url);

**assertStatus(int $code)**

斷言回傳的 HTTP 狀態碼與傳入的 $code 相符。

::

    $result->assertStatus(403);


Session Assertions
------------------

**assertSessionHas(string $key, $value = null)**

斷言此時的會話中存在所指定鍵。如果你也傳入了 $value ，也會斷言你所指定的鍵的值是否與傳入的 $value 相符。

::

    $result->assertSessionHas('logged_in', 123);

**assertSessionMissing(string $key)**

斷言目前的會談不包含指定的 $key 。

::

    $result->assertSessionMissin('logged_in');


Header Assertions
-----------------

**assertHeader(string $key, $value = null)**

斷言響應中存在著一個名為 **$key** 的標頭。如果你也傳入了 **$value** ，也會斷言這個值是相符的。

::

    $result->assertHeader('Content-Type', 'text/html');

**assertHeaderMissing(string $key)**

斷言響應中不存在著名稱為 **$key** 的標頭。

::

    $result->assertHeader('Accepts');


Cookie Assertions
-----------------

**assertCookie(string $key, $value = null, string $prefix = '')**

斷言響應中存在著名為 **$key** 的 Cookie ，如果你也傳入了 **$value** ，也會判定這些值是否相符。如果你需要的話，也可以設定 cookie 的前綴，透過傳遞第三個參數作為判斷依據。

::

    $result->assertCookie('foo', 'bar');

**assertCookieMissing(string $key)**

斷言響應中不存在名為 **$key** 的 Cookie 。

::

    $result->assertCookieMissing('ci_session');

**assertCookieExpired(string $key, string $prefix = '')**

斷言名為 **$key** 的 Cookie 確實存在，但已經過期了。如果你需要的話，也可以設定 cookie 的前綴，透過傳遞第二個參數作為設定值。

::

    $result->assertCookieExpired('foo');

響應輔助方法
-------------

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

DOM Assertions
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

這個方法將以自串的形式回傳響應的 body ：

::

    // 響應 body 像是這樣:
    ['foo' => 'bar']

    $json = $result->getJSON();

    // 獲得的 $json 像是這樣:
    {
        "foo": "bar"
    }

You can use this method to determine if ``$response`` actually holds JSON content::

    // Verify the response is JSON
    $this->assertTrue($result->getJSON() !== false)

.. note:: 需要注意的是， JSON 字串會格式化地輸出在結果中。

**assertJSONFragment(array $fragment)**

斷言 $fragment 存在於 JSON 響應中，它不需要符合整個 JSON 值。

::

    // 響應 body 像是這樣:
    [
        'config' => ['key-a', 'key-b']
    ]

    // 將回傳 true
    $this->assertJSONFragment(['config' => ['key-a']);

**assertJSONExact($test)**

類似於 **assertJSONFragment()** 但會檢閱整個 JSON 響應以確保結果精準地符合。

使用 XML
----------------

**getXML()**

如果你的應用程式會回傳 XML ，你可以使用這個方法檢閱它。