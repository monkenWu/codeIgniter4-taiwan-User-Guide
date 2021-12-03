##################
控制器過濾器
##################

.. contents::
    :local:
    :depth: 2

控制器過濾器讓你可以在控制器執行前或執行後進行操作。與 :doc:`事件 </extending/events>` 不同，你可以選擇過濾器將會應用到的特定 URI。傳入的過濾器可能會修改請求，而且過濾器可以作用在響應上，甚至是修改響應，有很大的靈活性及操作空間。這裡有一些常見的任務範例是可能會用到過濾器的：

* 在傳入的請求中執行 CSRF 防護
* 根據他們的角色限制你網站的區域
* 對特定的端點進行速率限制
* 顯示 "維修中" 的頁面
* 進行自動的內容協商
* 更多...

*****************
建立過濾器
*****************

過濾器是個實作了 ``CodeIgniter\Filters\FilterInterface`` 的簡單類別。它們包含了兩個方法： ``before()`` 與 ``after()``，分別保存在控制器運作之前與之後的程式碼。你的類別必須包含這兩個方法，不過若是你用不到它們，方法裡面可以留空。過濾器的框架類別看起來是::

    <?php

    namespace App\Filters;

    use CodeIgniter\HTTP\RequestInterface;
    use CodeIgniter\HTTP\ResponseInterface;
    use CodeIgniter\Filters\FilterInterface;

    class MyFilter implements FilterInterface
    {
        public function before(RequestInterface $request, $arguments = null)
        {
            // Do something here
        }

        public function after(RequestInterface $request, ResponseInterface $response, $arguments = null)
        {
            // Do something here
        }
    }

前濾器
==============

在任何的過濾器中，你可以回傳 ``$request`` 物件，它會替換當前的請求，讓你可以對其進行更改，並且在控制器執行時更改仍然是存在的。

因為過濾器是在控制器之前執行的，所以你可能會想要停止控制器的動作。此外，當你有一連串的過濾器時，你也可能會想在某些過濾器執行後，其後的過濾器才開始執行。你可以回傳 **任何的非空** 結果來輕鬆地做到這件事情。如果前濾器回傳了一個空的結果，則控制器或是此過濾器之後的過濾器仍然會被執行。回傳一個 ``Request`` 實體是個非空結果中的例外。在前濾器回傳它的話，只會替換目前的 ``$request`` 物件，並不會停止執行。

這個通常會用來處理重新導向，像是這個範例所示::

    public function before(RequestInterface $request, $arguments = null)
    {
        $auth = service('auth');

        if (! $auth->isLoggedIn()) {
            return redirect()->to(site_url('login'));
        }
    }

若回傳的是一個 ``Response`` 實體，響應會被傳回至客戶端，且腳本的執行將會停止。這對於實作速率限制的API很有幫助。參見在 :doc:`Throttler </libraries/throttler>` 的範例。

後濾器
=============

後濾器與前濾器幾乎相同，但是你只能夠回傳 ``$response`` 物件，而且不能夠停止腳本的執行。這確實允許你修改最終的輸出，或僅對其做一些操作。可以用在確保某些安全性相關的標頭有正確的設定，或是快取最終的輸出，又或是用敏感字詞過濾器過濾最終的輸出。

*******************
設定過濾器
*******************

一旦建立了你的過濾器，你需要去設定它們何時要開始運作。這是在 **app/Config/Filters.php** 中完成的。這個檔案包含了四個屬性，讓你能夠準確地設定過濾器要何時運作。

$aliases
========

``$aliases`` 陣列用於將一個簡單的名稱與一個或多個完全合格的類別名稱關聯起來，而這些類別名稱即為要運作的過濾器::

    public $aliases = [
        'csrf' => \CodeIgniter\Filters\CSRF::class,
    ];

別名具有強制性，如果你之後嘗試使用完整的類別名稱，系統將會拋出錯誤。以這種方式定義它們使更改使用的類別變得簡單許多。當你決定你需要切換至不同的驗證系統時這會很有幫助，因為你只需要切換過濾器的類別便可完成。你可以結合多個過濾器到一個別名裡，讓複雜的過濾器組易於使用::

    public $aliases = [
        'apiPrep' => [
            \App\Filters\Negotiate::class,
            \App\Filters\ApiAuth::class,
        ]
    ];

你應該要定義跟你的需求一樣多的別名。

$globals
========

第二部分允許你定義任何過濾器，且被應用在每個由此框架製作的請求中。你應該注意你用了多少個過濾器，因為在每個請求都運作過多的過濾器可能會影響性能。可以透過在 before 陣列或是 after 陣列中加入別名以指定過濾器::

    public $globals = [
        'before' => [
            'csrf',
        ],
        'after' => [],
    ];

有時，你會想應用一個過濾器在大多數的請求中，但希望幾個請求不受到影響。一個常見的例子是，你需要幾個 URI 不受 CSRF 防護過濾器的影響，使第三方網站的請求能夠命中一個或兩個特定的 URI，並且其餘的 URI 仍然受到保護。為了做到這件事，要在對應的別名添加有個鍵為 'except' 、值為一個 URI 的陣列作為值::

    public $globals = [
        'before' => [
            'csrf' => ['except' => 'api/*'],
        ],
        'after' => [],
    ];


在過濾器的設定中可以用到 URI 的地方，你都可以使用正規表達式，或像是這個範例，使用星號作為萬用字元去匹配在其後的所有字元。在這個範例中，任何以 ``api/`` 作為開頭的 URL 皆會免於 CSRF 防護的影響，但是網站中所有的表單皆會受到防護。如果你需要指定數個 URI，你可以使用 URI 樣式的陣列::

    public $globals = [
        'before' => [
            'csrf' => ['except' => ['foo/*', 'bar/*']],
        ],
        'after' => [],
    ];

$methods
========

你可以在特定 HTTP 方法的所有請求中使用過濾器，如 POST、GET、PUT 等。在這個陣列中，你須以小寫的方法名稱去指定他們。它的值是要運作的過濾器陣列。與 ``$globals`` 或 ``$filters`` 屬性不同，這些只會以前濾器的方式運作::

    public $methods = [
        'post' => ['foo', 'bar'],
        'get'  => ['baz'],
    ]

除了標準的 HTTP 方法，這裡也支援一個特別的情況： 'cli'。 'cli' 方法會應用在所有從命令列裡執行的請求。


$filters
========

這個屬性是一個過濾器別名的陣列。對每個別名，你可以指定過濾器要應用的 before 與 after 陣列，裡面包含了 URI 樣式的列表::

    public filters = [
        'foo' => ['before' => ['admin/*'], 'after' => ['users/*']],
        'bar' => ['before' => ['api/*', 'admin/*']],
    ];

過濾器的引數
=================

設定過濾器時，若是在設定路由，可能會傳送額外的引數給過濾器::

    $routes->add('users/delete/(:segment)', 'AdminController::index', ['filter' => 'admin-auth:dual,noreturn']);

在這個範例中， ``['dual', 'noreturn']`` 陣列會在 ``$arguments`` 中被傳遞給過濾器的 ``before()`` 與 ``after()`` 實作方法。

****************
提供的過濾器
****************

有三個過濾器與 CodeIgniter4 同綑在一起： ``Honeypot`` ， ``CSRF`` 與 ``DebugToolbar`` 。

.. note:: 過濾器會按照設定檔裡宣告的順序執行，但有一個例外與 ``DebugToolbar`` 有關，是它總是在最後一個執行。因為 ``DebugToolbar`` 要能夠註冊在其他過濾器發生的所有事情。
