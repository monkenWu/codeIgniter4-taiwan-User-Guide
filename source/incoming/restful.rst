處理 RESTful 請求資源
#######################################################

.. contents::
    :local:
    :depth: 2

表現層狀態轉換（REST）是分佈式應用的一種架構風格，最早由 Roy Fielding 在他 2000 年的博士論文 `《Architectural Styles and
the Design of Network-based Software Architectures》 <https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm>`_ 中描述。這本可能讀起來會有點艱澀的，你可能會覺得 `《Richardson Maturity Model》 <https://martinfowler.com/articles/richardsonMaturityModel.html>`_ 是比較溫和的介紹。

REST 比大多數的軟體架構都有更多的解釋，也有更多的誤解，也許可以說，在一個架構中，你所符合的 Roy Fielding 的原則越多，你的應用就會被認為是最" RESTful "的。

CodeIgniter 透過它的資源路由和 `ResourceController`，可以讓你的資源輕鬆創建RESTful API。

資源路由
============================================================

您可以使用 ``resource()`` 方法為分散的資源快速創建少量的 RESTful 路由。這將為一個資源創建五個最常見的路由：創建一個新的資源，更新一個現有的資源，列出所有的資源，顯示一個單一的資源，以及刪除一個單一的資源。第一個參數是資源的名稱::

    $routes->resource('photos');

    // 同等於以下內容:
    $routes->get('photos/new',             'Photos::new');
    $routes->post('photos',                'Photos::create');
    $routes->get('photos',                 'Photos::index');
    $routes->get('photos/(:segment)',      'Photos::show/$1');
    $routes->get('photos/(:segment)/edit', 'Photos::edit/$1');
    $routes->put('photos/(:segment)',      'Photos::update/$1');
    $routes->patch('photos/(:segment)',    'Photos::update/$1');
    $routes->delete('photos/(:segment)',   'Photos::delete/$1');

.. note:: 上面排的這個順序是為了方便說明，而創建路由的實際順序是在 RouteCollection 中定義的，這是為了確保能正確的解析路由。

.. important:: 路由是按照指定的順序依序配對的，所以如果你在 get 'photos/poll' 上面有一個 photos 資源，那麼 show 路由會在 get 路由之前匹配。要解決這個問題，將 get 路由移到資源順序的前面，就會優先進行配對。

第二個參數接受一個選項陣列，可以用來修改生成的路由。雖然這些
路由是用來導向 API 的，但在這裡還有更多的方法可以使用，你可以透過傳入 'websafe' 選項來讓它生成 update 和 delete 方法，可和 HTML 表單一起使用::

    $routes->resource('photos', ['websafe' => 1]);

    // The following equivalent routes are created:
    $routes->post('photos/(:segment)/delete', 'Photos::delete/$1');
    $routes->post('photos/(:segment)',        'Photos::update/$1');

改變使用的控制器
--------------------------

你可以透過在選項中輸入 ``controller`` 和你想指定的控制器名稱來指定使用的控制器::

	$routes->resource('photos', ['controller' =>'App\Gallery']);

	// Would create routes like:
	$routes->get('photos', 'App\Gallery::index');

改變使用的置換符號
---------------------------

預設情況下，當需要資源 ID 時，會使用 ``segment`` 置換符號。但你也可以透過傳遞 ``placeholder`` 選項來改變它::

	$routes->resource('photos', ['placeholder' => '(:num)']);

	// Generates routes like:
	$routes->get('photos/(:num)', 'Photos::show/$1');

限制路由的生成
---------------------

你可以使用 ``only`` 選項來限制路由的生成。這應該是一個陣列或由逗號分隔的方法名稱列表，只有符合這些方法之一的路由才會被創造。其他的路由將被忽略::

	$routes->resource('photos', ['only' => ['index', 'show']]);

或是，你也可以用 ``except`` 選項來排除指定的路由。該選項會在``only`` 之後執行::

	$routes->resource('photos', ['except' => 'new,edit']);

有效的方法包括：index、show、create、update、new、edit 以及 delete。

資源路由
============================================================

`ResourceController` 為你的 RESTful API 提供了一個方便的進入點，它有著與上述資源路由相對應的方法。

擴展它，覆蓋 `modelName` 和 `format` 屬性，然後實作你想使用的方法::

	<?php namespace App\Controllers;

	use CodeIgniter\RESTful\ResourceController;

	class Photos extends ResourceController
	{

		protected $modelName = 'App\Models\Photos';
		protected $format    = 'json';

		public function index()
		{
			return $this->respond($this->model->findAll());
		}

                // ...
	}

這個路由會是::

    $routes->resource('photos');

表現層路由
============================================================

使用 ``presenter（）`` 方法，你可以用像創建資源控制器一樣的方式，快速創建一個表現層控制器。這為控制器方法創建路由，該方法將回傳你的視圖資源，或處理從這些視圖中提交的表單。

因為你可以使用傳統的控制器，所以不是非用它不可，但使用它你會比較方便。它的用法類似資源路由::


    $routes->presenter('photos');

    // 同等於以下內容:
    $routes->get('photos/new',                'Photos::new');
    $routes->post('photos/create',            'Photos::create');
    $routes->post('photos',                   'Photos::create');   // alias
    $routes->get('photos',                    'Photos::index');
    $routes->get('photos/show/(:segment)',    'Photos::show/$1');
    $routes->get('photos/(:segment)',         'Photos::show/$1');  // alias
    $routes->get('photos/edit/(:segment)',    'Photos::edit/$1');
    $routes->post('photos/update/(:segment)', 'Photos::update/$1');
    $routes->get('photos/remove/(:segment)',  'Photos::remove/$1');
    $routes->post('photos/delete/(:segment)', 'Photos::update/$1');

.. note:: 上面排的這個順序是為了方便說明，而創建路由的實際順序是在 RouteCollection 中定義的，這是為了確保能正確的解析路由。

你不會同時為資源和表現層控制器提供 `photos` 路由，你應該用個方法來區分它們，例如::

    $routes->resource('api/photo');
    $routes->presenter('admin/photos');


第二個參數接受一個選項陣列，可以用來修改生成的路由。

更改所使用的控制器
--------------------------

你可以透過 ``controller`` 選項來傳遞需要更改的控制器的名字來指定實際使用的控制器::

	$routes->presenter('photos', ['controller' =>'App\Gallery']);

	// 這會生成如下的路由:
	$routes->get('photos', 'App\Gallery::index');

更改所使用的置換符號
---------------------------

預設情況下，當需要資源 ID 時，我們使用 ``segment`` 置換符號。你可以透過 ``placeholder`` 選項來傳遞一個新的字串來指定新的置換符號::

	$routes->presenter('photos', ['placeholder' => '(:num)']);

	// 這會生成如下的路由:
	$routes->get('photos/(:num)', 'Photos::show/$1');

限制路由的生成
---------------------

你可以使用 ``only`` 選項來限制路由的生成。這應該是一個陣列或由逗號分隔的方法名稱列表，只有符合這些方法之一的路由才會被創造。其他的路由將被忽略::

	$routes->presenter('photos', ['only' => ['index', 'show']]);

或是，你也可以用 ``except`` 選項來排除指定的路由。該選項會在``only`` 之後執行::

	$routes->presenter('photos', ['except' => 'new,edit']);

有效的方法包括：index、show、create、update、new、edit 以及 delete。

資源表現器
============================================================

`ResourcePresenter` 提供了一個便捷的進入點，來為你輸出對應資源的視圖，而它同樣也可使用如同資源路由的方法來處理從這些視圖裡提交的表單。

繼承或覆蓋 `modelName` 屬性，並實作那些你需要呼叫的方法::

	<?php namespace App\Controllers;

	use CodeIgniter\RESTful\ResourcePresenter;

	class Photos extends ResourcePresenter
	{

		protected $modelName = 'App\Models\Photos';

		public function index()
		{
			return view('templates/list',$this->model->findAll());
		}

                // ...
	}

生成的路由會長這樣子::

    $routes->presenter('photos');

表現層／控制器比較
=============================================================

下表對比了分別用 ``resource()`` 和 ``presenter()`` 方法創建的預設路由以及對應的控制器方法。

================ ========= ====================== ======================== ====================== ======================
操作        	 方法       控制器路由             表現層路由                控制器方法             表現層方法
================ ========= ====================== ======================== ====================== ======================
**New**          GET       photos/new             photos/new               ``new()``              ``new()``
**Create**       POST      photos                 photos                   ``create()``           ``create()``
Create (alias)   POST                             photos/create                                   ``create()``
**List**         GET       photos                 photos                   ``index()``            ``index()``
**Show**         GET       photos/(:segment)      photos/(:segment)        ``show($id = null)``   ``show($id = null)``
Show (alias)     GET                              photos/show/(:segment)                          ``show($id = null)``
**Edit**         GET       photos/(:segment)/edit photos/edit/(:segment)   ``edit($id = null)``   ``edit($id = null)``
**Update**       PUT/PATCH photos/(:segment)                               ``update($id = null)`` 
Update (websafe) POST      photos/(:segment)      photos/update/(:segment) ``update($id = null)`` ``update($id = null)``
**Remove**       GET                              photos/remove/(:segment)                        ``remove($id = null)``
**Delete**       DELETE    photos/(:segment)                               ``delete($id = null)`` 
Delete (websafe) POST                             photos/delete/(:segment) ``delete($id = null)`` ``delete($id = null)``
================ ========= ====================== ======================== ====================== ======================
