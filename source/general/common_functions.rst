##############################
全域函數與常數
##############################

CodeIgniter 提供了一些全域函數與變數讓你在任何時候都可以使用，這些函數和變數不需要載入任何額外的程式庫或輔助函數。

.. contents::
    :local:
    :depth: 2


================
全域函數
================

服務存取器
=================

.. php:function:: cache ( [$key] )

    :param  string $key: 想要從快取項目中取出的快取名稱。（可選）
    :returns: 單獨指定的快取項目，或者是整個快取物件。
    :rtype: mixed

    如果沒有提供 $key ，將會回傳快取引擎的實體。如果提供了 $key ，則會回傳現在儲存在快取中的 $key 的值，如果沒有找到，則會回傳 null 。

    範例：
	
	::

     	$foo = cache('foo');
    	$cache = cache();

.. php:function:: cookie(string $name[, string $value = ''[, array $options = []]])

    :param string $name: Cookie name
    :param string $value: Cookie value
    :param array $options: Cookie options
    :rtype: ``Cookie``
    :returns: ``Cookie`` instance
    :throws: ``CookieException``

    Simpler way to create a new Cookie instance.

.. php:function:: cookies([array $cookies = [][, bool $getGlobal = true]])

    :param array $cookies: If ``getGlobal`` is ``false``, this is passed to ``CookieStore``'s constructor.
    :param bool $getGlobal: If ``false``, creates a new instance of ``CookieStore``.
    :rtype: ``CookieStore``
    :returns: Instance of ``CookieStore`` saved in the current ``Response``, or a new ``CookieStore`` instance.

    Fetches the global ``CookieStore`` instance held by ``Response``.

.. php:function:: env ( $key[, $default=null])

	:param string $key: 需要檢索的環境變數名稱
	:param mixed  $default: 如果沒有找到值，預設回傳的值。
	:returns: 環境變數，預設值或為 null 。
	:rtype: mixed

	用於檢索已經被設定過的環境變數，如果沒有找到，則會回傳一個預設值。將會是實際的布林值，而不是以字串表示。

	當與 .env 檔案結合使用時，這個函數會特別有用。用於設定環境本身的特定值，如資料庫設定、API 金鑰等等。

.. php:function:: esc ( $data, $context='html' [, $encoding])

	:param   string|array   $data: 預計被跳脫的資訊。
	:param   string   $context: 轉譯的內容，預設是 html 。
	:param   string   $encoding: 字串的編碼格式。
	:returns: 跳脫後的資料。
	:rtype: mixed

	在你的網頁中使用跳脫後的資訊，將可以防止 XSS 攻擊。這個函數使用 Laminas Escaper 程式庫來處理實際的資料過濾。

	如果 $data 是一個字串，那麼就會被轉譯後回傳。如果 $data 是一個陣列，那麼它就會針對陣列做循環，轉換調每個鍵/值的「值」。

	支援的內容值： html 、 js 、 css 、 url 、 attr 、 raw 、 null 。

.. php:function:: helper( $filename )

	:param   string|array  $filename: 要載入的輔助函數名稱，或者是含有多個輔助函數名稱的陣列。

	載入一個輔助函數檔案。

	詳情請參閱 :doc:`helpers` 頁面。

.. php:function:: lang($line[, $args[, $locale ]])

	:param string $line: 要檢索的文字行數。
	:param array  $args: 替代置換符號的資料陣列。
	:param string $locale: 指定要使用的語言環境，而非預設的語言環境。

	根據別名字串檢索特定的語言環境檔案。

	更多資訊，請參閱 :doc:`Localization </outgoing/localization>` 頁面。

.. php:function:: model($name [, $getShared = true [, &$conn = null ]])

    :param string                   $name:
    :param boolean                  $getShared:
    :param ConnectionInterface|null $conn:
    :returns: More simple way of getting model instances
    :rtype: mixed


.. php:function:: old( $key[, $default = null, [, $escape = 'html' ]] )

	:param string $key: 需要檢查的舊表單資料。
	:param mixed  $default: 如果 $key 不存在，則回傳預設值。
	:param mixed  $escape: `轉譯 <#esc>`_ 內容或使用 false 禁用。
	:returns: 定義 key 的值，或者使用預設值。
	:rtype: mixed

	提供一個簡單的方法，可以從已經提交的表單中造訪「舊的輸入資料」。

	範例：
	
	::

		// in controller, checking form submittal
		if (! $model->save($user))
		{
			// 'withInput' is what specifies "old data"
			// should be saved.
			return redirect()->back()->withInput();
		}

		// In the view
		<input type="email" name="email" value="<?= old('email') ?>">
		// Or with arrays
		<input type="email" name="user[email]" value="<?= old('user.email') ?>">

.. note:: 這個功能內建在 :doc:`表單輔助函數 </helpers/form_helper>` 之中，若是你不使用表單輔助函數，你才會需要用到這個功能。

.. php:function:: session( [$key] )

	:param string $key: 需要檢查的 Session 項目名稱。
	:returns: 如果沒有傳入 $key ，則會回傳 Session 物件的實體；如果有傳入 $key ，則尋找 Session 中是否有這個值後回傳，若找不到則為 null。 
	:rtype: mixed

	提供一個存取 Session 類別和檢索儲存值的便捷方法，更多訊息請參閱 :doc:`Sessions </libraries/sessions>` 頁面。

.. php:function:: timer( [$name] )

	:param string $name: 基準點的名稱。
	:returns: Timer 實體。
	:rtype: CodeIgniter\Debug\Timer

	提供快速造訪 Timer 類別的方法，你可以傳遞一個基準點名稱做為唯一引數。方法將從這個基準點開始計時，如果已經有一個帶有這個名子的 Timer 再執行，則會停止執行。

	範例::

		// Get an instance
		$timer = timer();

		// Set timer start and stop points
		timer('controller_loading');    // Will start the timer
		. . .
		timer('controller_loading');    // Will stop the running timer

.. php:function:: view ($name [, $data [, $options ]])

	:param   string   $name: 要載入的檔案名稱。
	:param   array    $data: 傳遞給視圖的 鍵/值 陣列。
	:param   array    $options: 將會傳遞給渲染類別的選項陣列。
	:returns: 輸出視圖。
	:rtype: string

	抓取目前的 RendererInterface 相容類別，並告訴它渲染所指定的視圖。提供便捷的方法，可以在控制器、程式庫與路由閉包中使用。

	在 `$options` 陣列中有一個選項可以使用，即 `saveData` 。它指定的資料將保持在同一請求多次呼叫的 `View()` 之中。在預設的情況下，這個視圖的資料將在顯示視圖後被棄用。

	提供 $option 陣列是為了方便第三方與 Twig 等程式庫的集成。

	範例::

		$data = ['user' => $user];

		echo view('user_profile', $data);

	更多詳情，請閱讀 :doc:`視圖 </outgoing/views>` 頁面。

.. php:function:: view_cell ( $library [, $params = null [, $ttl = 0 [, $cacheName = null]]] )

    :param string      $library:
    :param null        $params:
    :param integer     $ttl:
    :param string|null $cacheName:
    :returns: View cells are used within views to insert HTML chunks that are managed by other classes.
    :rtype: string

    For more details, see the :doc:`View Cells </outgoing/view_cells>` page.


其他功能
=======================

.. php:function:: app_timezone ()

    :returns: The timezone the application has been set to display dates in.
    :rtype: string

    Returns the timezone the application has been set to display dates in.

.. php:function:: csrf_token ()

	:returns: 當前 CSRF 權杖的名稱。
	:rtype: string

	回傳當前 CSRF 權杖的名稱。

.. php:function:: csrf_header ()

	:returns: 當前 CSRF 權杖的 header 名稱。
	:rtype: string

	當前 CSRF 權杖的 header 名稱。

.. php:function:: csrf_hash ()

	:returns: 當前 CSRF 的雜湊值。
	:rtype: string

	當前 CSRF 的雜湊值。

.. php:function:: csrf_field ()

	:returns: 一個帶有隱藏輸入的 HTML 字串，包含所有需要的 CSRF 訊息。
	:rtype: string

	回傳已經插入 CSRF 訊息的隱藏輸入：

	::

		<input type="hidden" name="{csrf_token}" value="{csrf_hash}">

.. php:function:: csrf_meta ()

	:returns: 一個包含 meta 標籤的 HTML 字串，包含所有需要的 CSRF 訊息。
	:rtype: string

	回傳一個已經插入了 CSRF 訊息的 meta 標籤：

	::

		<meta name="{csrf_header}" content="{csrf_hash}">

.. php:function:: force_https ( $duration = 31536000 [, $request = null [, $response = null]] )

	:param  int  $duration: 瀏覽器將連接到這個資源轉換成 HTTPS 的秒數。
	:param  RequestInterface $request: 目前 Request 物件的實體。
	:param  ResponseInterface $response: 目前 Response 物件的實體。

	檢查目前是否透過 HTTPS 造訪該頁面。如果是，將不動作。若否，則該使用者將會以 HTTPS 的形式被重新導向到當前 URL 。將會設定 HTTP Strict-Transport-Security 標頭，它將讓現代瀏覽器自動把 $duration 的所有 HTTP 請求修改為 HTTPS 請求。

.. php:function:: function_usable ( $function_name )

    :param string $function_name: Function to check for
    :returns: TRUE if the function exists and is safe to call, FALSE otherwise.
    :rtype: bool

.. php:function:: is_cli ()

	:returns: 如果該腳本是從命令列中執行，則回傳 TRUE 否則回傳 FALSE 。
	:rtype: bool

.. php:function:: is_really_writable ( $file )

    :param string $file: The filename being checked.
    :returns: TRUE if you can write to the file, FALSE otherwise.
    :rtype: bool

.. php:function:: log_message ($level, $message [, $context])

	:param   string   $level: 嚴重程度。
	:param   string   $message: 要記錄的訊息。
	:param   array    $context: 包含標籤與值的關聯陣列，在 $message 中被替換。
	:returns: 如果紀錄成功則為 TRUE ；如果紀錄失敗則為 FALSE 。
	:rtype: bool

	使用 **app/Config/Logger.php** 中定義的日誌處理程式來記錄訊息。
	
	級別可能是以下值之一： **emergency （緊急）** 、 **alert （提示）** 、 **critical （重要）** 、 **error （錯誤）** 、 **warning （警告）** 、 **notice （通知）** 、 **info （訊息）**，與 **debug （除錯）**.

	$context 可以用來替代訊息字串中的值，有關詳細資訊，請參閱 :doc:`日誌資訊 <logging>` 頁面。

.. php:function:: redirect( string $uri )

	:param  string  $uri: 使用者將被重新導向的目標 URL 。

	回傳一個 RedirectResponse 實體，允許使用者輕鬆創建重新定向：

	::

		// 返回上一頁
		return redirect()->back();

		// 前往特定的 URI
		return redirect()->to('/admin');

		// 前往 named/reverse-routed 的 URI
		return redirect()->route('named_route');

		// 在重新定向時保留舊的輸入值，這樣他們就可以被 `old()` 函數使用。
		return redirect()->back()->withInput();

		// 設定快閃訊息（Flash message）
		return redirect()->back()->with('foo', 'message');

		// Copies all cookies from global response instance
		return redirect()->back()->withCookies();

        // Copies all headers from the global response instance
        return redirect()->back()->withHeaders();

	當傳遞 URL 到函數中時，它會被視為反向路由請求，而不是 relative/full 的 URI ，處理方式與使用 redirect()->route() 相同 ：

	::

		// 前往 named/reverse-routed 的 URI
		return redirect('named_route');

.. php:function:: remove_invisible_characters($str[, $urlEncoded = TRUE])

	:param	string	$str: 輸入字串
	:param	bool	$urlEncoded: 是否要清除 URL 編碼的字元
	:returns:	Sanitized string
	:rtype:	string

	這個函數可以防止在 ASCII 字元間插入 NULL 字元，就像 Java\\0script 。

	範例：
	
	::

		remove_invisible_characters('Java\\0script');
		// 最後會輸出: 'Javascript'　字串

.. php:function:: route_to ( $method [, ...$params] )

	:param   string   $method: 命名路由的別名，或是要匹配的 控制器／方法 的名稱。
	:param   mixed   $params: 在路由中傳遞一個或多個要匹配的引數。

	根據命名的路由別名以及 控制器::方法 ，生成一個相對的 URI 組合。如果提供了引數，則引數將會生效。

	有關更多訊息，請見 :doc:`/incoming/routing` 頁面.

.. php:function:: service ( $name [, ...$params] )

	:param   string   $name: 要被載入的服務名稱。
	:param   mixed    $params: 傳遞給方法的一個或多個的引數。
	:returns: 指定的服務類別的實體。
	:rtype: mixed

	提供了對系統中定義的任何 :doc:`Services <../concepts/services>` 的方便存取。這將始終回傳一個共享的類別實體，因此無論在一次的請求中呼叫多少次，都只會創建一個類別實體。

	範例：
	
	::

		$logger = service('logger');
		$renderer = service('renderer', APPPATH.'views/');

.. php:function:: single_service ( $name [, ...$params] )

	:param   string   $name: 要載入的服務名稱。
	:param   mixed    $params: 一個或多個要傳遞給服務方法的引數。
	:returns: 指定的服務類別的實體。
	:rtype: mixed

	與上面描述的 **service()** 函數相同，但呼叫這個函數每次都會回傳一個新的類別實體，而 **service** 每次的回傳則是相同的實體。

.. php:function:: slash_item ( $item )

    :param string $item: Config item name
    :returns: The configuration item or NULL if the item doesn't exist
    :rtype:  string|null

    Fetch a config file item with slash appended (if not empty)

.. php:function:: stringify_attributes ( $attributes [, $js] )

	:param   mixed    $attributes: 字串、鍵值陣列或物件。
	:param   boolean  $js: 如果值不需要引號，則為 TRUE （ Javascript 風格 ）。
	:returns: 含有屬性 鍵／值 的字串，以逗號分開。
	:rtype: string

	輔助函數用於將字串、陣列或物件屬性串換成字串。

================
全域常數
================

以下的常數將可以在應用程式中的任何地方使用。

核心常數
==============

.. php:const:: APPPATH

	**app** 目錄的絕對路徑。

.. php:const:: ROOTPATH

	專案根目錄的絕對路徑，也就是 ``APPPATH`` 的上一層。

.. php:const:: SYSTEMPATH

	**system** 資料夾的絕對路徑。

.. php:const:: FCPATH

	存放前置（front）控制器的絕對路徑。

.. php:const:: WRITEPATH

	**writable** 目錄的絕對路徑。

時間常數
==============

.. php:const:: SECOND

	等於 1.

.. php:const:: MINUTE

	等於 60.

.. php:const:: HOUR

	等於 3600.

.. php:const:: DAY

	等於 86400.

.. php:const:: WEEK

	等於 604800.

.. php:const:: MONTH

	等於 2592000.

.. php:const:: YEAR

	等於 31536000.

.. php:const:: DECADE

	等於 315360000.
