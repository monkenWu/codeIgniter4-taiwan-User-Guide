訪問請求
*********************

訪問請求類別提供物件導向表示法來呈現來自客戶端（如瀏覽器）的 HTTP 請求。
它繼承並可以存取任何來自 :doc:`Request </incoming/request>` 和 :doc:`Message </incoming/message>` 的方法 ，除此之外，也可使用下列方法。

.. contents::
    :local:
    :depth: 2

存取請求
---------------------

如果當前類別是 ``CodeIgniter\Controller`` 的子類別，並且可以作為類別屬性來訪問，則該請求類別的實體以下提供給您::

    <?php

    namespace App\Controllers;

    use CodeIgniter\Controller;

    class UserController extends Controller
    {
        public function index()
        {
            if ($this->request->isAJAX()) {
                // ...
            }
        }
    }

如果您不在控制器內，但仍需存取應用程式的訪問物件，則您可透過 :doc:`Services class </concepts/services>` 獲取該物件的副本::

    $request = \Config\Services::request();

但是，如果該類別不是控制器，則最好將請求作為依賴傳入，如此您可將其作為類別屬性來保存::


    <?php

    use CodeIgniter\HTTP\RequestInterface;

    class SomeClass
    {
        protected $request;

        public function __construct(RequestInterface $request)
        {
            $this->request = $request;
        }
    }

    $someClass = new SomeClass(\Config\Services::request());

判斷請求的型態
------------------------

一個請求有可多種的型態，包括 AJAX 請求或者來自命令列的請求。您可以透過 ``isAJAX()`` 和 ``isCLI()`` 方法來判斷其型態::

    // Check for AJAX request.
    if ($request->isAJAX()) {
        // ...
    }

    // Check for CLI Request
    if ($request->isCLI()) {
        // ...
    }

.. note:: ``isAJAX()`` 方法取決於 ``X-Requested-With`` 標頭，
    在某些情況下，該標頭預設不會是通過 JavaScript（即 Fetch）在 XHR 請求中發送的。
    有關避免此問題的方法，請查閱 :doc:`AJAX Requests </general/ajax>`。

您可以透過 ``method()`` 方法來檢查該請求代表何種 HTTP 方法::

    // Returns 'post'
    $method = $request->getMethod();

在預設情況下，此方法會回傳一個小寫字串(即「get」，「post」，等等)。您可以透過將此呼叫放入 ``str_to_upper()`` 將回傳值轉為大寫字串::

    // Returns 'GET'
    $method = str_to_upper($request->getMethod());

您也可以透過 ``isSecure()`` 方法來檢查該請求是否來自 HTTPS 連線::

    if (! $request->isSecure()) {
        force_https();
    }

檢索輸入
----------------

您可以透過請求物件中的 $_SERVER，$_GET，$_POST，和 $_ENV 來檢索輸入，資料不會被自動過濾，並且會回傳請求中傳入的原始輸入資料。
相較於直接存取它們（如 $_POST[『something』]），透過下列內建方法存取最大的好處是如果該資料不存在，則會回傳 null ，並且可以過濾數據。這使您可以更方便地使用這些資料，而不用在每次使用前都需要測試它們是否存在。
換句話說，在正在情況下您有可能會有下列程式碼::

    $something = isset($_POST['foo']) ? $_POST['foo'] : null;

透過 CodeIgniter 的內建方法，您可以簡單地這樣做::

    $something = $request->getVar('foo');

``getVar()`` 方法會從 $_REQUEST 中拉取並回傳任何來自於 $_GET， $POST，或 $_COOKIE 的資料。
雖然如此作法很方便，但您通常會需要使用更具體的方法，例如::

* ``$request->getGet()``
* ``$request->getPost()``
* ``$request->getServer()``
* ``$request->getCookie()``

此外，以下提供一些實用的方法用於檢索 $_GET 或 $_POST 中的資訊，並且可以讓您保有選擇檢索順序的能力::

* ``$request->getPostGet()`` - checks $_POST first, then $_GET
* ``$request->getGetPost()`` - checks $_GET first, then $_POST

**獲取 JSON 資料**

您可以使用 ``getJSON()`` 方法來獲取 php://input 中以 JSON 串流表示的內容。

.. note::  這無法檢查傳入的資料是否為合法的 JSON，在使用此方法前，您應該要確保傳入的資料是 JSON。

::

    $json = $request->getJSON();

在預設情況下，這會將 JSON 資料中的任何物件作為物件回傳。如果您想要將其轉為關聯陣列，則將 ``true`` 作為第一個參數傳入。

第二個和第三個參數對應到的是 `json_decode <https://www.php.net/manual/en/function.json-decode.php>`_ PHP 函數中的 ``depth`` 和 ``options`` 引數。

如果傳入的請求中的 ``CONTENT_TYPE`` 標頭被設為 「application/json」，則您也可以使用 ``getVar()`` 來獲取 JSON 串流。
在這情況下使用 ``getVar()`` 總是會回傳物件。

**獲取 JSON 中特定資料**

您可以透過將您想要的資料名稱作為變數傳傳入 ``getVar()`` 中來從 JSON 串流中獲取特定的資料。
或者您可以使用「點」符號來獲取 JSON 中那些不是根等級的資料。

::

    //With a request body of:
    {
        "foo": "bar",
        "fizz": {
            "buzz": "baz"
        }
    }
    $data = $request->getVar('foo');
    //$data = "bar"

    $data = $request->getVar('fizz.buzz');
    //$data = "baz"


如果您想要的結果是一個關聯陣列而不是物件，則您可以使用 ``getJsonVar()`` 並且將 ``true`` 作為第二個參數傳入。
此函數在您不能保證傳入的請求有正確的 ``CONTENT_TYPE`` 標頭時，仍可以使用。

::

    //With the same request as above
    $data = $request->getJsonVar('fizz');
    //$data->buzz = "baz"

    $data = $request->getJsonVar('fizz', true);
    //$data = ["buzz" => "baz"]

.. note:: 有關更多「點」符號的資訊，請查閱 ``Array`` 輔助函數中的 ``dot_array_search()``。

**檢索原始資料（PUT，PATCH，DELETE）**

最後，您可以透過 ``getRawInput()`` 來獲得 php://input 中內容的原始串流::

    $data = $request->getRawInput();

以下寫法可以檢索資料並且將其轉成陣列。例如::

    var_dump($request->getRawInput());

    [
        'Param1' => 'Value1',
        'Param2' => 'Value2'
    ]

**過濾輸入資料**

為了維持您的應用程式的安全，您將希望過濾所有您要存取的輸入。您可以將要使用的過濾器型態作為第二個參數傳入這些方法。
原生的 ``filter_var()`` 函數可以被用來過濾。請前往查閱 PHP 手冊中的 `valid
filter types <https://www.php.net/manual/en/filter.filters.php>`_ 列表。

過濾一個 POST 變數會看起來像這樣::

    $email = $request->getVar('email', FILTER_SANITIZE_EMAIL);

除了 ``getJSON()`` 方法之外，上述所有提到的方法皆可以將過濾器型態作為第二個參數傳入。

檢索標頭
------------------

您可以透過 ``headers()`` 方法來存取任何和請求一起送過來的標頭。此方法會回傳包含所有標頭的陣列，此陣列的鍵為標頭的名稱，而值為
``CodeIgniter\HTTP\Header`` 的一個實體::

    var_dump($request->headers());

    [
        'Host'          => CodeIgniter\HTTP\Header,
        'Cache-Control' => CodeIgniter\HTTP\Header,
        'Accept'        => CodeIgniter\HTTP\Header,
    ]

如果您只需要單一標頭，您可以將標頭名稱傳入 ``header()`` 方法中。
如果存在的話，這會以名稱不區分大小寫的方式攫取特定的標頭物件。反之，它會回傳 ``null``::

    // these are all equivalent
    $host = $request->header('host');
    $host = $request->header('Host');
    $host = $request->header('HOST');

您總是可以使用 ``hasHeader()`` 來檢查這個請求中的特定標頭是否存在::

    if ($request->hasHeader('DNT')) {
        // Don't track something...
    }

如果您需要標頭所有的值以一行字串表示，可以使用 ``getHeaderLine()`` 方法::

    // Accept-Encoding: gzip, deflate, sdch
    echo 'Accept-Encoding: '.$request->getHeaderLine('accept-encoding');

如果您需要整個標頭，並且希望名稱和值在同一個字串中，只需將標頭轉換成字串::

    echo (string)$header;

URL 請求
---------------

您可以通過表示此請求的當前URI的 :doc:`URI </libraries/uri>` 物件的 ``$request->uri`` 屬性來檢索。
您可以將此物件轉換成字串來獲取當前請求的完整 URL::

    $uri = (string)$request->uri;

此物件讓您完全有能力從該請求中自行獲取任何部分::

    $uri = $request->uri;

    echo $uri->getScheme();         // http
    echo $uri->getAuthority();      // snoopy:password@example.com:88
    echo $uri->getUserInfo();       // snoopy:password
    echo $uri->getHost();           // example.com
    echo $uri->getPort();           // 88
    echo $uri->getPath();           // /path/to/page
    echo $uri->getQuery();          // foo=bar&bar=baz
    echo $uri->getSegments();       // ['path', 'to', 'page']
    echo $uri->getSegment(1);       // 'path'
    echo $uri->getTotalSegments();  // 3

您可以對當前的 URI 字串（相對於您的 baseURL 的路徑）使用 ``getPath()`` 和 ``setPath()`` 方法。
要注意的是，在 ``IncomingRequest`` 的共享實體中的這個相對路徑會被 :doc:`URL Helper </helpers/url_helper>` 函數使用，所以這是一個「欺騙」用於測試的請求的好方法。 ::

    class MyMenuTest extends CIUnitTestCase
    {
        public function testActiveLinkUsesCurrentUrl()
        {
            service('request')->setPath('users/list');
            $menu = new MyMenu();
            $this->assertTrue('users/list', $menu->getActiveLink());
        }
    }


已上傳的檔案
--------------

所有已上傳的檔案的資訊都可透過 ``$request->getFiles()`` 取回，並回傳一個 
:doc:`FileCollection </libraries/uploaded_files>` 實體。這有助於減少操作已上傳檔案的麻煩，
並且使用了最佳實踐來最小化所有安全性風險。
::

    $files = $request->getFiles();

    // Grab the file by name given in HTML form
    if ($files->hasFile('uploadedFile')) {
        $file = $files->getFile('uploadedfile');

        // Generate a new secure name
        $name = $file->getRandomName();

        // Move the file to it's new home
        $file->move('/path/to/dir', $name);

        echo $file->getSize('mb'); // 1.23
        echo $file->getExtension(); // jpg
        echo $file->getType(); // image/jpg
    }

你可以以基於 HTML 檔案輸入給予的檔案名稱取回一個單獨上傳的檔案::

    $file = $request->getFile('uploadedfile');

你可以以基於 HTML 檔案輸入給予的檔案名稱取回一個作為多個檔案上傳的一部分已上傳的相同名稱的檔案陣列::

    $files = $request->getFileMultiple('uploadedfile');

內容協商
-------------------

你可以透過請求 ``negotiate()`` 方法輕鬆的更改內容樣式::

    $language    = $request->negotiate('language', ['en-US', 'en-GB', 'fr', 'es-mx']);
    $imageType   = $request->negotiate('media', ['image/png', 'image/jpg']);
    $charset     = $request->negotiate('charset', ['UTF-8', 'UTF-16']);
    $contentType = $request->negotiate('media', ['text/html', 'text/xml']);
    $encoding    = $request->negotiate('encoding', ['gzip', 'compress']);

查看 :doc:`內容協商 </incoming/content_negotiation>` 以了解更多細節。

類別參考
===============

.. note:: 除了條列在這裡的方法，此類別也繼承了來自
    :doc:`Request 類別 </incoming/request>` 與 :doc:`Message 類別 </incoming/message>` 的方法。

由父類別提供的可用方法有:

* :meth:`CodeIgniter\\HTTP\\Request::getIPAddress`
* :meth:`CodeIgniter\\HTTP\\Request::isValidIP`
* :meth:`CodeIgniter\\HTTP\\Request::getMethod`
* :meth:`CodeIgniter\\HTTP\\Request::setMethod`
* :meth:`CodeIgniter\\HTTP\\Request::getServer`
* :meth:`CodeIgniter\\HTTP\\Request::getEnv`
* :meth:`CodeIgniter\\HTTP\\Request::setGlobal`
* :meth:`CodeIgniter\\HTTP\\Request::fetchGlobal`
* :meth:`CodeIgniter\\HTTP\\Message::getBody`
* :meth:`CodeIgniter\\HTTP\\Message::setBody`
* :meth:`CodeIgniter\\HTTP\\Message::appendBody`
* :meth:`CodeIgniter\\HTTP\\Message::populateHeaders`
* :meth:`CodeIgniter\\HTTP\\Message::headers`
* :meth:`CodeIgniter\\HTTP\\Message::header`
* :meth:`CodeIgniter\\HTTP\\Message::hasHeader`
* :meth:`CodeIgniter\\HTTP\\Message::getHeaderLine`
* :meth:`CodeIgniter\\HTTP\\Message::setHeader`
* :meth:`CodeIgniter\\HTTP\\Message::removeHeader`
* :meth:`CodeIgniter\\HTTP\\Message::appendHeader`
* :meth:`CodeIgniter\\HTTP\\Message::prependHeader`
* :meth:`CodeIgniter\\HTTP\\Message::getProtocolVersion`
* :meth:`CodeIgniter\\HTTP\\Message::setProtocolVersion`

.. php:class:: CodeIgniter\\HTTP\\IncomingRequest

    .. php:method:: isCLI()

        :returns: 要是請求是由命令列發起的則回傳 true，否則回傳 False。
        :rtype: bool

    .. php:method:: isAJAX()

        :returns: 要是請求是一個 AJAX 請求則回傳 true，否則回傳 False。
        :rtype: bool

    .. php:method:: isSecure()

        :returns: 要是請求是一個 HTML 請求則回傳 true，否則回傳 False。
        :rtype: bool

    .. php:method:: getVar([$index = null[, $filter = null[, $flags = null]]])

        :param  string  $index: 欲找尋的變數/值的名稱。
        :param  int     $filter: 要使用的過濾器類型。過濾器類型清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :param  int     $flags: 要使用的旗標。旗標清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.flags.php>`__。
        :returns:   要是沒有提供參數則回傳 $_REQUEST，否則回傳找到的 REQUEST 值，沒找到則回傳 null
        :rtype: mixed|null

        第一個參數會包含你在尋找的 REQUSET 項目的名稱::

            $request->getVar('some_data');

        要是你試圖取回的項目不存在則此方法會回傳 null。

        第二個可選參屬允許你透過 PHP 篩選運行資料。傳入想要的過濾器類型作為第二個參數::

            $request->getVar('some_data', FILTER_SANITIZE_STRING);

        回傳一個由所有 POST 項目組成的陣列不須呼叫任何參數。

        如果想要回傳所有的 POST 項目並且將它們傳過過濾器，將第一個參數設為 null 並將第二個參數設為你想使用的過濾器。::

            $request->getVar(null, FILTER_SANITIZE_STRING);
            // returns all POST items with string sanitation

        如果想要回傳一個由多個 POST 參數組成的陣列，將所有必要的鍵以陣列傳入::

            $request->getVar(['field1', 'field2']);

        相同的規則也可以應用在這裡，如果想要取回過濾器篩選後的參數，將第二個參數設為想要使用的過濾器類型::

            $request->getVar(['field1', 'field2'], FILTER_SANITIZE_STRING);

    .. php:method:: getGet([$index = null[, $filter = null[, $flags = null]]])

        :param  string  $index: 欲找尋的變數/值的名稱。
        :param  int     $filter: 要使用的過濾器種類。過濾器類型清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :param  int     $flags: 要使用的旗標。旗標清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.flags.php>`__。
        :returns:       要是沒有提供參數則回傳 $_GET，否則回傳找到的 GET 值，沒找到則回傳 null
        :rtype: mixed|null

        此方法與 ``getVar()`` 是完全相同的，差別在此方法是抓取 GET 資料。

    .. php:method:: getPost([$index = null[, $filter = null[, $flags = null]]])

        :param  string  $index: 欲找尋的變數/值的名稱。
        :param  int     $filter: 要使用的過濾器類型。過濾器類型清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :param  int     $flags: 要使用的旗標。旗標清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.flags.php>`__。
        :returns:       要是沒有提供參數則回傳 $_POST，否則回傳找到的 POST 值，沒找到則回傳 null
        :rtype: mixed|null

        此方法與 ``getVar()`` 是完全相同的，差別在此方法是抓取 POST 資料。

    .. php:method:: getPostGet([$index = null[, $filter = null[, $flags = null]]])

        :param  string  $index: 欲找尋的變數/值的名稱。
        :param  int     $filter: 要使用的過濾器類型。過濾器類型清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :param  int     $flags: 要使用的旗標。旗標清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.flags.php>`__。
        :returns:       要是沒有提供參數則回傳 $_POST，否則回傳找到的 POST 值，沒找到則回傳 null
        :rtype: mixed|null

        此方法在運作上和 ``getPost()`` 與 ``getGet()`` 大同小異，差別在於此方法結合了兩者。
        此方法會搜索 POST 與 GET 串流來取得資料，先搜尋過 POST 後，再搜尋 GET ::

            $request->getPostGet('field1');

    .. php:method:: getGetPost([$index = null[, $filter = null[, $flags = null]]])

        :param  string  $index: 欲找尋的變數/值的名稱。
        :param  int     $filter: 要使用的過濾器類型。過濾器類型清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :param  int     $flags: 要使用的旗標。旗標清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.flags.php>`__。
        :returns:       要是沒有提供參數則回傳 $_POST，否則回傳找到的 POST 值，沒找到則回傳 null
        :rtype: mixed|null

        此方法在運作上和 ``getPost()`` 與 ``getGet()`` 大同小異，差別在於此方法結合了兩者。
        此方法會搜索 POST 與 GET 串流來取得資料，先搜尋過 GET 後，再搜尋 POST ::

            $request->getGetPost('field1');

    .. php:method:: getCookie([$index = null[, $filter = null[, $flags = null]]])
        :noindex:

        :param    mixed    $index: COOKIE 名稱
        :param  int     $filter: 要使用的過濾器類型。過濾器類型清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :param  int     $flags: 要使用的旗標。旗標清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.flags.php>`__。
        :returns:        要是沒有提供參數則回傳 $_COOKIE，否則回傳找到的 COOKIE 值，沒找到則回傳 null
        :rtype:    mixed

        此方法和 ``getPost()`` 與 ``getGet()`` 是完全相同的，差別在此方法是抓取 cookie 資料。::

            $request->getCookie('some_cookie');
            $request->getCookie('some_cookie', FILTER_SANITIZE_STRING); // with filter

        如果想要回傳一個由多個 cookie 值組成的陣列，將所有必要的鍵以陣列傳入::

            $request->getCookie(['some_cookie', 'some_cookie2']);

        .. note:: 不同於 :doc:`Cookie Helper <../helpers/cookie_helper>`
            函數 :php:func:`get_cookie()`，此方法並不會預先設定你的 ``$config['cookie_prefix']`` 值。

    .. php:method:: getServer([$index = null[, $filter = null[, $flags = null]]])
        :noindex:

        :param    mixed    $index: 值的名稱
        :param  int     $filter: 要使用的過濾器類型。過濾器類型清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :param  int     $flags: 要使用的旗標。旗標清單請前往
                        `這裡 <https://www.php.net/manual/en/filter.filters.flags.php>`__。
        :returns:       回傳找到的 $_SERVER 項目的值，沒找到則回傳null
        :rtype:    mixed

        此方法和 ``getPost()`` 與 ``getGet()`` 與 ``getCookie()`` 是完全相同的，差別在此方法是抓取 getServer 資料 (``$_SERVER``)::

            $request->getServer('some_data');

        如果想要回傳一個由多個 ``$_SERVER`` 值組成的陣列，將所有必要的鍵值以陣列傳入。
        ::

            $request->getServer(['SERVER_PROTOCOL', 'REQUEST_URI']);

    .. php:method:: getUserAgent([$filter = null])

        :param  int $filter: 要使用的過濾器類型。過濾器類型清單請前往
                    `這裡 <https://www.php.net/manual/en/filter.filters.php>`__。
        :returns:  回傳在 SERVER 資料找到的 User Agent 字串，沒找到則回傳null。
        :rtype: mixed

        此方法回傳在 SERVER 資料中的 User Agent 字串::

            $request->getUserAgent();

    .. php:method:: getPath()

        :returns:        目前與 ``$_SERVER['SCRIPT_NAME']`` 相關的 URI 路徑
        :rtype:    string

        這是測定"當前 URI"最安全的方法，因為 ``IncomingRequest::$uri`` 可能不會意識到對於基本 URL 完整的應用程式組態設定。

    .. php:method:: setPath($path)

        :param    string    $path: 要被當成當前 URI 使用的相關路徑
        :returns:        這個傳入的請求
        :rtype:    IncomingRequest

        通常只用在測試，此方法允許你為當前的請求設定相關路徑值，而不是依賴 URI 檢測。這同時也會以新的路徑更新潛在的 ``URI`` 實體。
