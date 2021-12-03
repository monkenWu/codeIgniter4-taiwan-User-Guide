################
URI 路由設定
################

.. contents::
    :local:
    :depth: 2

通常一個 URL 字串與他相應的控制器 類別 / 方法的關係是一對一的。
URI 中的區段規則應該要遵守以下樣式：

    example.com/class/method/id/

然而，在某些實體中，你可能會想要把這個關係重新映射，使得一個不同 類別 / 方法可以被不同地呼叫
而不是與 URL 單一呼應的那樣。

舉例來說，我們假設你想要你的 URL 們有這個樣式： ::

    example.com/product/1/
    example.com/product/2/
    example.com/product/3/
    example.com/product/4/

通常第二個 URL 的區段是保留給方法名稱的，但是在這個範例中取而代之的是一個 product ID。
為了克服這個問題，CodeIgniter 允許你重新映射 URI 處理器。

設定你自己的路由規則
==============================

路由規則都被定義在 **app/Config/Routes.php** 檔案中。在裡面你會看到它
創造了一個 RouteCollection 類別的實體，允許你可以指定你自己的路由標準。
路由可以使用佔位符或正規表達式來指定。

一個路由會單純的在左邊讀取 URI，然後再將其映射到右邊的控制器與方法中，伴隨著許多應該被傳輸進控制器中的參數。
而控制器和方法應該被以相同的方式列出，因此它應該是一個靜態方法，中間再用雙冒號去把它的全命名空間跟方法隔開，
例如 ``Users::list``。如果那個方法含有需要傳遞的參數，那參數們會被列在方法名稱的後面，並以正斜線隔開。 ::

    // Calls the $Users->list()
    Users::list
    // Calls $Users->list(1, 23)
    Users::list/1/23

佔位符
============

一個典型的路由應該長得像這樣： ::

    $routes->add('product/(:num)', 'App\Catalog::productLookup');

在一個路由中，第一個參數會包要被配對的 URI，而第二個參數則要包含需要被重新映射到的目標位置。
在上面的範例中，如果文字 "product" 在第一個 URL 段落被發現，第二個段落發現一個數字，
那麼取而代之的將會是 "App/Catalog" 類別以及 "productLookup" 方法。

佔位符是單純的字串，代表著一個正規表示式。在映射的過程中，這些佔位符將會被政則表達式的值所去帶。
他們主要是為了可讀性而存在。

下列的佔位符都可以被使用在你的路由中：

============ ===========================================================================================================
佔位符        描述
============ ===========================================================================================================
(:any)       會比對此段落到 URI 末端的所有字符。有可能會包含多 URI 段落。
(:segment)   會比對任何除了被前斜線 （/）改變單一段落結構以外的字符。
(:num)       會比對任何數字。
(:alpha)     會比對任何由字母字符組合的字串。
(:alphanum)  會比對任何由字母字符組合或數字的字串，又或者任何兩者的組合。
(:hash)      與 **(:segment)** 相同，但又可以被用來輕鬆檢視哪些路由使用了散列值的 id。
============ ===========================================================================================================

.. note:: **{locale}** 不能被當作佔位符或路由的一部分使用，因為它是被保留給 :doc:`localization </outgoing/localization>` 使用的。

範例
========

這裡有一些基礎的路由映射範例。

在第一段包含 "journals" 的 URL 會被重新映射到 "App/Blogs" 類別，
而它的預設方法通常會是 ``index()``： ::

    $routes->add('journals', 'App/Blogs');

URL 段落包含 "blog/joe" 將會被重新映射到 “\Blogs” 類別以及使用 “users” 方法。
其 ID 將會被設定為 “34”： ::

    $routes->add('blog/joe', 'Blogs::users/34');

將 “product” 作為第一段落的 URL，會把任何第二段落的路由重新映射到 “\Catalog” 類別以及使用 “productLookup” 方法： ::

    $routes->add('product/(:any)', 'Catalog::productLookup');

將 “product” 作為第一段落的 URL，且第二段落是數字的路由將會被重新映射到 “\Catalog” 類別
以及使用 “productLookupByID” 方法並傳入後面跟著的參數給該方法： ::

    $routes->add('product/(:num)', 'Catalog::productLookupByID/$1');

注意一個單獨的 ``(:any)`` 如果存在將會比對多個 URL 段落。例如以下路由： ::

    $routes->add('product/(:any)', 'Catalog::productLookup/$1');

它將會比對 product/123，product/123/456，product/123/456/789 等等。
控制器中的實作應該將最大參數考慮進去 ::

    public function productLookup($seg1 = false, $seg2 = false, $seg3 = false) {
        echo $seg1; // 會是所有範例中的 123
        echo $seg2; // 第一範例中錯誤，以及第二、第三範例中的 456
        echo $seg3; // 第一、第二範例中錯誤，以及第三範例中的 789
    }

如果比對了多個段落不是你的目標行為， 應該在定義路由的時候使用 ``(:segment)`` 。
以上面的範例 URL 作為範例 ::

    $routes->add('product/(:segment)', 'Catalog::productLookup/$1');

如此一來只會符合 product/123 並對其他範例產生 404 錯誤。

.. warning:: 雖然 ``add()`` 方法很便利，我們仍然建議持續使用以 HTTP 動詞為基礎的路由，
    原因如下，因為它更加的安全。如果你使用 :doc:`CSRF protection </libraries/security>` ，它並未保護 **GET**
    請求。如果在 ``add()`` 方法中指定的 URI 可以被 GET 方法存取，那麼 CSRF 保護
    將不會起作用。

.. note:: 使用 HTTP 動詞為基礎的路由也會提供些微的效能提升，因為
    只有符合當前方法的路由會被儲存，使得在比對的時候需要
    掃描更少的路由。

自定義佔位符
===================

你可以創造你自己的佔位符來設定你自己的路由檔案，完整地客製化體驗與可讀性。

你可以使用 ``addPlaceholder`` 方法新增佔位符。第一個參數是佔位符的名稱字串。
第二個參數是要取代進去的正規表示式樣式。
這必須在你新增路由之前呼叫 ::

    $routes->addPlaceholder('uuid', '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}');
    $routes->add('users/(:uuid)', 'Users::show/$1');

正規表示式
===================

如果你偏好以正規表示式來定義你的路由映射規則。任何合格的正規表示式
都是被允許的，向後參照也不例外。

.. important:: 注意：如果你使用向後參照你必須使用  “$” 而不是  “\\” 。
    典型的 RegEx 路由應該看起來像這樣 ::

    $routes->add('products/([a-z]+)/(\d+)', 'Products::show/$1/id_$2');

在上面的範例中，一個類似  products/shirts/123 的 URI反而會呼叫 ``Products`` 控制器類別中
的 ``show`` 方法， 並將原始的第一、第二段落當作參數傳遞給它。

使用正規表達式，你也可以抓取一個包含前斜線 (‘/’) 的段落，通常可以
代表多個段落中間的分隔號。

舉例來說，如果一個使用者存取你網頁中一個受密碼保護的區域，並且你希望可以將它們重新導向到登入前的同一畫面，你應該會覺得這個範例很實用 ::

    $routes->add('login/(.+)', 'Auth::login/$1');

對於那些不知道正規表達式且想要更加認識他們的人，
`regular-expressions.info <https://www.regular-expressions.info/>`_ 應該會是一個很好的起點。

.. important:: 注意: 你也可以用正規表達式混合比對萬用字元。

匿名函數
========

你可以使用匿名函數作為路由映射的目標位置。這個功能會在
使用者造訪該 URI 的時候被執行。這對執行小任務或者顯示簡單畫面內容都是很方便的 ::

    $routes->add('feed', function () {
        $rss = new RSSFeeder();

        return $rss->feed('general');
    });

映射多個路由
=======================

即便 add() 方法簡單易用，一次與多個路由互動可謂更加方便，為此我們使用 ``map()`` 方法。
除了為你個人需要增加的路由呼叫 ``add()`` 方法以外，你可以用一個路由的陣列然後
把它當作第一個參數傳入 ``map()`` 方法 ::

    $routes = [];
    $routes['product/(:num)'] = 'Catalog::productLookupById';
    $routes['product/(:alphanum)'] = 'Catalog::productLookupByName';

    $collection->map($routes);

重新導向路由
==================

任何存在夠久的網站一定會有頁面的遷移。你可以透過 ``addRedirect()`` 方法
指定重新導向到其他路由。第一個參數是舊路由的 URI 樣式。
第二個參數可以是重新導向的新路由，或者是已命名的路由的名稱。
第三個參數是隨著重新導向被送出的 的狀態碼。預設的值是 ``302`` ，它代表著暫時重新導向且在大部分的
狀況下都建議這麼做 ::

    $routes->add('users/profile', 'Users::profile', ['as' => 'profile']);

    // Redirect to a named route
    $routes->addRedirect('users/about', 'profile');
    // Redirect to a URI
    $routes->addRedirect('users/about', 'users/profile');

如果一個重新導向路由在讀取頁面的時候被比對到，使用者會立即在控制器讀取之前被重新導向到一個新頁面。

將路由分組
===============

你可以使用 ``group()`` 方法將你的路由根據一個通用名稱進行分組。該群組名稱會成為一個段落，
顯示位階優先於群組中的路由定義。這使得你可以減輕建立一組擁有共同開頭字串的額外路由的打字負擔
例如建立一個管理者區域的時候 ::

    $routes->group('admin', function ($routes) {
        $routes->add('users', 'Admin\Users::index');
        $routes->add('blog', 'Admin\Blog::index');
    });

這會用 "admin" 來當作 'users' 和 'blog' 的 URI 前綴，讓 URI 變得像 ``/admin/users`` 和 ``/admin/blog``。

如果你需要指派選項到一個群組中像是 `namespace <#assigning-namespace>`_, 請在回呼之前處理 ::

    $routes->group('api', ['namespace' => 'App\API\v1'], function ($routes) {
        $routes->resource('users');
    });

這會將 ``/api/users`` URI 的指向到 ``App\API\v1\Users`` 的資源路由。

你也可以為一組路由指定一個 `filter <filters.html>`_ 。這會永遠在
控制器之前後執行。這在處理認證或 api 紀錄時特別的方便 ::

    $routes->group('api', ['filter' => 'api-auth'], function ($routes) {
        $routes->resource('users');
    });

過濾器的值必須符合 **app/Config/Filters.php** 中定義的別名。

如果你需要在群組中嵌套群組以建立更好的組織，這也是可行的 ::

    $routes->group('admin', function ($routes) {
        $routes->group('users', function ($routes) {
            $routes->add('list', 'Admin\Users::list');
        });
    });

這會將路由導向到 ``admin/users/list``。注意傳遞到外部 ``group()`` 的選項（例如 ``namespace`` 和 ``filter``）
將不會與內部 ``group()`` 的選項合併。

某些時候，你可能想要使用過濾器或其他像是命名空間、子網域等路由設定而把路由分組。
在你沒有一定得對群組增加前綴的時候，你可以傳遞
一個空字串作為前綴，你群組中的路由會根據給定的路由設定做映射
就好像路由群組不存在一樣。

環境限制
========================

你可以創造僅會在特定環境中可視的一組路由。讓你可以創造
只有開發人員可以在本地機器中使用的工具，且在測試或產品伺服器中無法取用。
這只能透過 ``environment()`` 方法做到。第一個參數是環境的名稱。 Any
任何用這個匿名函數定義的路由只能在指定環境下被取用 ::

    $routes->environment('development', function ($routes) {
        $routes->add('builder', 'Tools\Builder::index');
    });

反向路由
===============

你可以在反向路由定義控制器和方法，以及任何參數，指定一個連結應該何去何從，
並讓路由查找他現在的映射為何者。讓你不用更動應用程式的程式碼就改變路由定義。
通常會被用在視圖中以創造連結。

舉例來說，如果你有一個想要連結的圖片庫，你可以使用 ``route_to()`` 輔助函數
來取得你當前應該使用的路由。第一個參數是完全合規的控制器與方法，
中間用雙冒號 （::）隔開，很像撰寫初始路由本身的樣子。應該要傳遞給路由的任何其他參數 
都需在後續傳入 ::

    // 路由定義為:
    $routes->add('users/(:num)/gallery(:any)', 'App\Controllers\Galleries::showUserGallery/$1/$2');

    // 產生到 user ID 15， gallery 12 的對應路由
    // 生成： /users/15/gallery/12
    <a href="<?= route_to('App\Controllers\Galleries::showUserGallery', 15, 12) ?>">View Gallery</a>

使用命名錄由
==================

你可以對路由命名使得你的應用程式更不脆弱。這會給路由一個可以被呼叫的名稱，
即便路由定義被改變了，你應用程式中用 ``route_to`` 建立的所有連結
不需要做任何改變就仍會有作用。一個路由透過 ``as`` 選項傳入的名字就可以被命名 ::

    // 路由被定義為：
    $routes->add('users/(:num)/gallery(:any)', 'Galleries::showUserGallery/$1/$2', ['as' => 'user_gallery']);

    // 產生到 user ID 15， gallery 12 的對應路由
    // 生成： /users/15/gallery/12
    <a href="<?= route_to('user_gallery', 15, 12) ?>">View Gallery</a>

這也有著讓視圖更加的可讀的額外好處。

在路由中使用 HTTP 動詞
==========================

你可以在路由規則中使用 HTTP 動詞（請求方法）來定義你的路由規則。這在建立 RESTFUL 應用程式時
特別實用。你可以使用任何的標準 HTTP 動詞（GET，POST，PUT，DELETE 等等）。
每個動詞有它自己的可用方法 ::

    $routes->get('products', 'Product::feature');
    $routes->post('products', 'Product::feature');
    $routes->put('products/(:num)', 'Product::feature');
    $routes->delete('products/(:num)', 'Product::feature');

你可以提供多個動詞以陣列的方式傳入到 ``match`` 方法讓路由去做映射 ::

    $routes->match(['get', 'put'], 'products', 'Product::feature');

僅命令列可用的路由
========================

你可以製作只有命令列可以使用的路由，且網頁瀏覽器不可取用，透過 ``cli()`` 方法即可達成。
在製作 cronjobs 或 CLI-only 工具很實用。任何以 HTTP 動詞為基礎的路由方法也會是 CLI 不可用的。
但若是以 ``any()`` 方法製作的路由仍會是命令列可以存取的 ::

    $routes->cli('migrate', 'App\Database::migrate');

全域選項
==============

所有製作路由的方法（add, get, post, `resource <restful.html>`_ 等等）可以用一個選項陣列
來調整所產生的路由，甚至限制它們。 ``$options`` 陣列永遠都是最後一個引數 ::

    $routes->add('from', 'to', $options);
    $routes->get('from', 'to', $options);
    $routes->post('from', 'to', $options);
    $routes->put('from', 'to', $options);
    $routes->head('from', 'to', $options);
    $routes->options('from', 'to', $options);
    $routes->delete('from', 'to', $options);
    $routes->patch('from', 'to', $options);
    $routes->match(['get', 'put'], 'from', 'to', $options);
    $routes->resource('photos', $options);
    $routes->map($array, $options);
    $routes->group('name', $options, function ());

使用過濾器
----------------

你可以改變特定路由的行為，透過提供過濾器在控制器前後執行的方式達成。在處理認證或 api 紀錄時特別好用。
過濾器的值可以是字串或由字串組成的陣列：

* 符合 **app/Config/Filters.php** 中定義的別名。
* 過濾器類別名稱。

請見 `Controller filters <filters.html>`_ 尋找更多設定過濾器的設定。

.. Warning:: 如果你在 **app/Config/Routes.php** 中給路由設定過濾器
    （不是在 **app/Config/Filters.php** 中），我們建議停用自動映射。
    當自動映射被啟用中，有可能會通過不同的 URL 取用到
    控制器而不是你設定的路由，
    這種狀況下你指定的路由就不會被用到。
    請見 :ref:`use-defined-routes-only` 以停用自動映射。

**別名過濾器**

你可以指定 **app/Config/Filters.php** 中的一個別名作為過濾器的值 ::

    $routes->add('admin',' AdminController::index', ['filter' => 'admin-auth']);

你也可以提供要傳給別名過濾器的 ``before()`` 和 ``after()`` 方法的引數 ::

    $routes->add('users/delete/(:segment)', 'AdminController::index', ['filter' => 'admin-auth:dual,noreturn']);

**類別名稱過濾器**

你可以指定一個類別名稱的過濾器作為過濾器的值 ::

    $routes->add('admin',' AdminController::index', ['filter' => \App\Filters\SomeFilter::class]);

**多個過濾器**

.. important:: *多個過濾器* 預設不可用。因為他會破壞向後兼容的特性。如果你想要使用它，你要自行設定。請在 :doc:`/installation/upgrade_415`  *一個路由多個過濾器* 中尋找更多細節。

你可以指定一個陣列作為過濾器的值 ::

    $routes->add('admin',' AdminController::index', ['filter' => ['admin-auth', \App\Filters\SomeFilter::class]]);

指派命名空間
-------------------

因為預設的命名空間會被作為產生的控制器的前置（詳情如下），你也可以使用　``namespace``　選項
來指定一個不同的命名空間作為選項陣列使用。該值應該是你想要修改的命名空間　::

    // 指定路由到 \Admin\Users::index()
    $routes->add('admin/users', 'Users::index', ['namespace' => 'Admin']);

新的命名空間只會在呼叫製作單一路由的任意方法時被應用，像是 get，post 等等。
對於任何製作多個路由的方法，新的命名空間會附著在該功能產生的所有路由上，
或者在 ``group()`` 的情況下，所有的路由會在使用匿名函數的時候產生。

主機名稱的限制
-----------------

你可以對函數進行路由群組的限制，限制只有你的應用程式的特定網域或子網域可用
通過將「主機名稱」選項以及你想要限制啟用的域名作為選項陣列傳遞來啟用 ::

    $collection->get('from', 'to', ['hostname' => 'accounts.example.com']);

這個範例只會允許主機名稱符合 "accounts.example.com" 的主機存取。
在 "example.com" 的主網站下就無法存取。

子網域的限制
-------------------

當 ``subdomain`` 的選項存在，系統將會限制路由只對該子網域可用。
該路由只會有在子網域名稱是正在查看應用程式的子網域的時候才會去匹配路由 ::

    // 限制 media.example.com
    $routes->add('from', 'to', ['subdomain' => 'media']);

你可以規定任何子網域，透過設定子網域值為星字號（*）。如果你正在瀏覽不具有子網域的 URL ，你就不會被匹配路由 ::

    // 限制任何子網域
    $routes->add('from', 'to', ['subdomain' => '*']);

.. important:: 系統並不是完美的，你應該在上正式環境之前測試你指定的域名。
    大部分域名都會正常的執行，除了某些邊緣情況，特別是域名本身有「點」的
    （不是被用來分開後綴或 www 的）會有導致誤報的潛在危機。

位移對應到的參數
---------------------------------

你可以通過 ``offset`` 選項以任意數值在你的路由中位移對應到的參數, 以該數值作為段落中偏移的數字。

這在開發以第一段作為版本號的 API 的時候特別好用。它也可以
被用在第一個引數是語言字串的時候 ::

    $routes->get('users/(:num)', 'users/show/$1', ['offset' => 1]);

    // Creates:
    $routes['users/(:num)'] = 'users/show/$2';

.. _priority:

映射處理中的隊列
----------------------

在與模組互動的時候，如果應用程式中的路由含有萬用字元會是一個問題。
會造成模組的路由不被正確地處理。
你可以透過 ``priority`` 選項降低路由處理的優先權解決這個問題。
參數可以使用正整數和零。 "priority" 越高的數字，在處裡隊列中排的越後面 ::

    // 首先你要啟用排序
    $routes->setPrioritize();

    // App\Config\Routes
    $routes->add('(.*)', 'Posts::index', ['priority' => 1]);

    // Modules\Acme\Config\Routes
    $routes->add('admin', 'Admin::index');

    // "admin" 路由現在會在萬用字元路由之前被處理。


如果要停用這個功能，你必須傳入 ``false`` 到該方法中。 ::

    $routes->setPrioritize(false);

.. note:: 預設狀況下，所有路由的優先全都是 0，
    負數會被解讀為絕對值。


路由設定選項
============================

RoutesCollection 類別提供多種選項以設定所有路由，且可以根據你應用程式的需求進行修改。
這些選項在 **app/Config/Routes.php** 的最上面可以找到。

預設命名空間
-----------------

把控制器配對給路由的時候，路由器會把預設密名空間的值加到指定路由控制器的前端。
預設狀況下，這個值是空值，讓每個路由可以指派控制器的完整名稱 ::

    $routes->setDefaultNamespace('');

    // Controller 為 \Users
    $routes->add('users', 'Users::index');

    // Controller 為 \Admin\Users
    $routes->add('users', 'Admin\Users::index');

如果你的控制器沒有明確的指定命名空間，你就不需要變更這裡。如果你給你的控制器命名空間，
那你可以改變這邊以減少你的打字負擔 ::

    $routes->setDefaultNamespace('App');

    // Controller 為 \App\Users
    $routes->add('users', 'Users::index');

    // Controller 為 \App\Admin\Users
    $routes->add('users', 'Admin\Users::index');

預設控制器
------------------

當一個使用者造訪你網站的根部 （也就是說 example.com），除非一個路由明確的位網站而存在
否則要使用哪一個控制器是皆是由 ``setDefaultController()`` 方法中的值所指定
此方法的預設值是 ``Home``
與 ``/app/Controllers/Home.php`` 中的控制器一致 ::

    // example.com 映射到 app/Controllers/Welcome.php
    $routes->setDefaultController('Welcome');

預設控制器也會在沒有找到對應路由的時候被使用， 且 URI 會被指到控制器的路徑中。
舉例說明，如果使用者造訪 ``example.com/admin``，若是在 ``/app/Controllers/admin/Home.php`` 中找到一個控制器，該控制器就會被使用。

預設方法
--------------

這與預設控制器設定相似，但它是被用來決定控制器與 URI 相符合時卻沒有方法段落存在的時候該使用哪個預設方法。
預設值為 ``index``。

在這範例中，如果使用者想要造訪 example.com/products，且一個 Products 控制器存在的狀況下，
``Products::listAll()`` 方法就會被執行 ::

    $routes->setDefaultMethod('listAll');

解析 URI 中的破折號
--------------------

這個選項讓你夠自動地將控制器與方法 URI 段落中的破折號 (‘-‘) 取代為下底線，
因此如果你有需要的話可以省去額外的路由進入點。 This is required because the
因為破折號並不是一個合規的類別或方法名稱字元且會在試著使用的時候造成致命錯誤，所以這是必須的 ::

    $routes->setTranslateURIDashes(true);

.. _use-defined-routes-only:

只使用有定義的路由
-----------------------

當沒有找到與定義相對應的 URI，系統會像上面所描述的一般，試圖在控制器與方法中
尋找對應的 URI 。你可以停用這個自動匹配，並將 ``setAutoRoute()`` 選項設為 false，去限制只能用你定義的路由 ::

    $routes->setAutoRoute(false);

.. warning:: 如果你使用 :doc:`CSRF protection </libraries/security>`，它並未保護 **GET**
    請求。如果你的 URI 可以被 GET 方法呼叫，那麼 CSRF 保護將不會起作用。

404 複寫
------------

當沒有頁面符合當前的 URI，系統將會顯示一個通用的 404 視圖。你可以透過指定
``set404Override()`` 選項以改變發生的事情。它的值可以是合規的類別／方法組合，就像在任何路由中的樣子，亦或是一個匿名函數 ::

    // 將會執行 App\Errors 類別中的 show404 方法
    $routes->set404Override('App\Errors::show404');

    // 將會顯示一個自定義視圖
    $routes->set404Override(function ()
    {
        echo view('my_errors/not_found.html');
    });


根據優先權處理路由
----------------------------

啟用或停用路由的優先權處理隊列。降低優先權的方法是被定義在路由選項中的。
預設為停用。這個功能會影響所有的路由。
可在 :ref:`priority` 中找到使用降低優先權的範例。 ::

    // 啟用
    $routes->setPrioritize();

    // 停用
    $routes->setPrioritize(false);
