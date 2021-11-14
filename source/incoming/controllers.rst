###########
控制器
###########

控制器是你的應用程式的核心，它們決定了應該如何處理 HTTP 請求。

.. contents::
    :local:
    :depth: 2


什麼是控制器？
=====================

控制器是一個簡單的類別檔案，它可以使用與 URI 有關的命名方式。

以這個 URI 為例：

::

	example.com/index.php/helloworld/

在上面的例子中，CodeIgniter 會嘗試找到一個名為 Helloworld.php 的控制器並載入它。

**當一個控制器的名稱與 URI 的第一個區段匹配時，它將會被加載。**


讓我們試一下：Hello World!
==========================

讓我們創建一個簡單的控制器，這樣你就可以看到它的操作方式。使用你的文本編輯器，創建一個名為 Helloworld.php 的文件，然後把下面的程式碼放在裡面。

You will notice that the Helloworld Controller is extending the BaseController. you can also extend the CodeIgniter\Controller if you do not need the functionality of the BaseController.

The BaseController provides a convenient place for loading components and performing functions that are needed by all your controllers. You can extend this class in any new controller.

For security reasons be sure to declare any new utility methods as protected or private.:

::

    <?php

    namespace App\Controllers;

    class Helloworld extends BaseController
    {
        public function index()
        {
            echo 'Hello World!';
        }
    }

然後將文件保存到你的 **/app/Controllers/** 目錄下。

.. important:: 該文件必須命名為 "Helloworld.php"，大寫 "H"。


現在用類似於這樣的 URL 訪問你的網站：

::

	example.com/index.php/helloworld

如果你做對了，你應該會看到：

::

	Hello World!

.. important:: 控制器類別的名稱必須以大寫字母開頭，並且只有第一個字符可以是大寫。

這是有效的命名：

::

    <?php

    namespace App\Controllers;


    class Helloworld extends BaseController
    {

    }

這是 **無效** 的命名：

::

    <?php

    namespace App\Controllers;

    class helloworld extends BaseController
    {

    }

這是 **無效** 的命名：

::

    <?php

    namespace App\Controllers;

    class HelloWorld extends BaseController
    {

    }

另外，一定要確保你的控制器擴展了父控制器類別，這樣它才可以繼承所有的方法。

.. note::
    The system will attempt to match the URI against Controllers by matching each segment against
    folders/files in APPPATH/Controllers, when a match wasn't found against defined routes.
    That's why your folders/files MUST start with a capital letter and the rest MUST be lowercase.
    If you want another naming convention you need to manually define it using the
    :doc:`URI Routing <routing>` feature.

    Here is an example based on PSR-4: Autoloader::

        \<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>

        $routes->get('helloworld', 'App\Controllers\HelloWorld::index');

方法
=======

在上面的例子中，方法的名稱是 ``index()``。如果 URI 的第二個區段是空的，那麼 "index" 方法預設總會被載入。另一種顯示 "Hello World" 消息的方法是這樣的：

::

	example.com/index.php/helloworld/index/

**URI 的第二個區段決定了控制器中的哪個方法會被呼叫。**


讓我們來試一試，為你的控制器中添加一個新方法：

::

    <?php

    namespace App\Controllers;

    class Helloworld extends BaseController
    {
        public function index()
        {
            echo 'Hello World!';
        }

        public function comment()
        {
            echo 'I am not flat!';
        }
    }

現在載入下面的URL，查看 comment 方法：

::

	example.com/index.php/helloworld/comment/

你應該看到你的新內容了。

將 URI 區段傳給你的方法
====================================

如果你的 URI 包含兩個以上的區段，它們將會作為參數傳遞給你的方法。

例如，假設你的 URI 長得是這樣的：

::

	example.com/index.php/products/shoes/sandals/123

URI 的第三個區段和第四個區段（"sandals" 和 "123"）將傳入你的方法：

::

    <?php

    namespace App\Controllers;

    class Products extends BaseController
    {
        public function shoes($sandals, $id)
        {
            echo $sandals;
            echo $id;
        }
    }

.. important:: 如果你使用了 :doc:`URI 路由 <routing>`
	功能，傳遞給你的方法的區段，將是重定向的區段。

定義一個預設控制器
=============================

CodeIgniter 可以在被告知沒有 URI 時、或是被請求你的網站根網址時，CodeIgniter 會加載一個預設的控制器。讓我們用 Helloworld 控制器來試一下。

要指定一個預設控制器，請打開你的 **app/Config/Routes.php** 檔案並設定這個變數：

::

	$routes->setDefaultController('Helloworld');

其中 'Helloworld' 是你要使用的控制器類別的名稱。

在 **Routes.php** 下面的 "Route Definitions" 部分的幾行中，將下面這一行註解掉：

::

$routes->get('/', 'Home::index');

如果你現在瀏覽到你的網站時沒有指定任何的 URI 區段，你會看到 "Hello World" 訊息。

.. note:: 這一行 ``$routes->get('/', 'Home::index');`` 是你在打造你的產品應用中會想要使用的最佳化寫法。但為了示範，我們不想使用這個功能。``$routes->get()`` 在 :doc:`URI Routing <routing>` 中會進行解釋。

想了解的更加深入，可以參考 :doc:`URI 路由文件 <routing>` 中的「路由設定選項」章節。

呼叫重映射方法
======================

如上所述，URI 的第二段通常決定了要呼叫控制器中的哪個方法。 CodeIgniter 允許你透過使用 ``_remap()`` 方法來覆蓋這個行為：

::

    public function _remap()
    {
        // Some code here...
    }

.. important:: 如果你的控制器包含一個名為 _remap() 的方法，無論你的 URI 包含什麼，它 **都會被呼叫** 。它覆蓋了 URI 正常判斷該呼叫哪個方法的行為，允許你定義自己的路由規則方法。

被覆蓋的呼叫方法（通常是 URI 的第二段）將作為參數傳遞給 ``_remap()`` 方法：

::

    public function _remap($method)
    {
        if ($method === 'some_method') {
            return $this->$method();
        } else {
            return $this->default_method();
        }
    }


方法名稱後的任何額外的字段都會被傳遞到 ``_remap()`` 中。這些參數可以被傳遞到方法中，來模擬CodeIgniter的預設行為。

範例：

::

    public function _remap($method, ...$params)
    {
        $method = 'process_'.$method;

        if (method_exists($this, $method)) {
            return $this->$method(...$params);
        }

        throw \CodeIgniter\Exceptions\PageNotFoundException::forPageNotFound();
    }

私有方法
===============

在某些情況下，你可能希望某些方法不要被公開訪問。為了達到這個目的，只需將方法聲明為私有或保護方法。這樣就可以防止它被 URL 請求送達。舉個例子，如果你為 `Helloworld` 控制器定義了一個像這樣的方法：

::

    protected function utility()
    {
        // some code
    }

然後嘗試使用以下 URL 來訪問它，他將無法執行：

::

	example.com/index.php/helloworld/utility/

將您的控制器組織成子目錄
================================================

如果你正在構建一個大型的應用程式，你可能會希望將控制器分層組織或結構化為子目錄。CodeIgniter 可以讓你達成這個目的。

只需在主要的 *app/Controllers/* 目錄下創建子目錄，然後將你的控制器類別放進去。

.. important:: Folder names MUST start with an uppercase letter and ONLY the first character can be uppercase.

.. note:: 使用此功能時，URI的第一段必須要指定資料夾。例如，假設你有一個控制器位於這裡：

	::
		
		app/Controllers/products/Shoes.php

	要呼叫上面的控制器，你的 URI 會看起來像這樣：
		
	::

		example.com/index.php/products/shoes/show/123

你的每個子目錄都可能包含一個預設控制器，如果 URL 裡面 *只* 包含子目錄，那麼這個控制器就會被呼叫。只需在那裡放一個與你的 *app/Config/Routes.php* 檔案中指定的 "default_controller" 名稱相匹配的控制器即可。

CodeIgniter 還允許你使用 :doc:`URI Routing <routing>` 功能重新映射你的URI。

擁有的屬性
===================

你創建的每個控制器都應該擴展  ``CodeIgniter\Controller`` 類別。這個類別提供了幾個功能，所有的控制器都可以使用。

**請求物件**

應用程式的主要的 :doc:`請求實體 </incoming/request>` 總是可以作為一個類別屬性，``$this->request``。

**響應物件**

應用程式的主要的 :doc:`響應實體 </outgoing/response>` 總是可以作為一個類別屬性，``$this->response``。

**日誌物件**

應用程式的主要的 :doc:`日誌實體 <../general/logging>` 總是可以作為一個類別屬性，``$this->logger``。

**強制 HTTPS**

在所有的控制器中都有一個方便的方法，可以強制使用者透過 HTTPS 來訪問一個方法::

    if (! $this->request->isSecure()) {
        $this->forceHTTPS();
    }

預設情況下，在支持 HTTP 嚴格傳輸安全頭的現代瀏覽器中，這個呼叫應該強制瀏覽器將非 HTTPS 訪問轉換為 HTTPS 訪問一年。你可以透過傳入持續時間（秒）作為第一個參數來修改：

::

	if (! $this->request->isSecure()){
		$this->forceHTTPS(31536000);    // 一年
	}

.. note:: 有數個 :doc:`以時間定義的常數 </general/common_functions>` 可以讓你使用，像是 YEAR、MONTH 等。

輔助函數
---------------

你可以定義一個輔助函數的陣列作為類別屬性。每當控制器被載入時，這些輔助函數將自動載入到記憶體中，這樣你就可以在控制器的任何地方使用它們的方法：

::

    namespace App\Controllers;

    class MyController extends BaseController
    {
        protected $helpers = ['url', 'form'];
    }

驗證資料
======================

為了簡化資料驗證，控制器還提供了一個方便的方法 ``validate()``。該方法在第一個參數中接受一個規則陣列，在可選的第二個參數中，接受一個自定義的錯誤信息陣列，如果物件無效，則顯示自定義的錯誤訊息。在內部，這個方法使用控制器的 **$this->request** 實體來獲取要驗證的資料。:doc:`驗證函式庫文件 </libraries/validation>` 中有關於規則和訊息陣列格式的詳細資訊，以及可用的規則：

::

    public function updateUser(int $userID)
    {
        if (! $this->validate([
            'email' => "required|is_unique[users.email,id,{$userID}]",
            'name'  => 'required|alpha_numeric_spaces'
        ])) {
            return view('users/update', [
                'errors' => $this->validator->getErrors()
            ]);
        }

        // do something here if successful...
    }

如果你覺得在設定文件中保留規則更簡單，你可以把 $rules 陣列替換成  ``Config\Validation.php`` 中定義的組名：

::

    public function updateUser(int $userID)
    {
        if (! $this->validate('userRules')) {
            return view('users/update', [
                'errors' => $this->validator->getErrors()
            ]);
        }

        // do something here if successful...
    }

驗證也可以在模型中自動處理，但有時在控制器中進行驗證會更方便。具體到哪裡，由你自己決定.

.. note:: Validation can also be handled automatically in the model, but sometimes it's easier to do it in the controller. Where is up to you.

就是這樣！
==========

一言以蔽之，這些就是關於控制器的所有知識。
