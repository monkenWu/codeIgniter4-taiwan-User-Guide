靜態頁面
###############################################################################

**注:** 這個教學預設你已經下載了 CodeIgniter 並且在開發環境中 :doc:`部屬了框架 <../installation/index>` 。

你要做的第一件事就是設置一個 **控制器** 來處理靜態頁面。控制器是一個有助於指派工作的類別。它可以說是 Web 應用程式間的黏著劑。

舉個例子，當呼叫：

	``http://example.com/news/latest/10``

我們可以想像有個控制器名為「新聞（ news ）」，新聞呼叫的是 「最新的（latest）」這個方法（ method ）。這個方法可能會獲取 10 條新聞，並且將其呈現在畫面上。在 MVC 中，你經常會看到這子的 URL 映射模式：

	``http://example.com/[controller-class]/[controller-method]/[arguments]``

隨著 URL 的需求變得越來越複雜，這種方案可能會有所變化。但是現在，我們只需要先了解這些就好了。

讓我們製作第一個控制器
-------------------------------------------------------

在 **app/Controllers/Pages.php** 中創建一個有以下程式碼的檔案。

::

	<?php namespace App\Controllers;
	
	use CodeIgniter\Controller;

	class Pages extends Controller {

		public function index()
		{
			return view('welcome_message');
		}

		public function showme($page = 'home')
		{
		}
	}


你創建了一個名為 ``Pages`` 的類別，這個類別具有 ``showme`` 方法，這個方法將會被傳入 ``$page`` 引數。 還有一個 ``index()`` 方法，這與 **app/Controllers/Home.php** 這個控制器相同，將會默認顯示 CodeIgniter 歡迎頁面。

``Pages`` 繼承  ``CodeIgniter\Controller`` 類別。這代表， Pages 類別將可以呼叫 ``CodeIgniter\Controller`` 類別（ *system/Controller.php* ）中定義的方法與變數。

控制器是每個 Web 應用程式的 **請求中心** 。與任何 PHP 類別一樣，在控制器中必須使用 ``$this`` 呼叫。

現在，你已經創建了第一個方法，是時候製作一些用於 footer 與 header 的頁面樣板了。

在 **app/Views/templates/header.php** 路徑上創建含有下列程式碼的 header 檔案：

::

	<!doctype html>
	<html>
	<head>
		<title>CodeIgniter Tutorial</title>
	</head>
	<body>

		<h1><?= $title; ?></h1>

header 包含載入主要視圖之前輸出的基本 HTML 程式碼與標題。它還會輸出 ``$title`` 變數，等等我們將會在控制器對這個變數進行定義。接著，在 **app/Views/templates/footer.php** 路徑上創建含有下列程式碼的 footer 檔案：

::

		<em>&copy; 2019</em>
	</body>
	</html>

在控制器中新增邏輯
-------------------------------------------------------

剛才，你設定了一個包含 ``showme()`` 方法的控制器。這個方法將傳入一個引數，
這個引數是被載入的頁面的名稱。靜態頁面的主體將被儲存在 **app/Views/pages/** 目錄中。

在這個目錄中，創建名為 **home.php** 與 **about.php** 兩個檔案。在這些檔案中，輸入一些你喜歡的文字，並且保存它們。如果你不喜歡獨樹一格，你可以鍵入「 Hello World! 」。

為了載入這些頁面，你必須檢查請求的頁面是否實際存在，這些判斷邏輯是在創建 ``Pages`` 控制器後，於 ``showme()`` 方法中的主體。

::

	public function showme($page = 'home')
	{
		if ( ! is_file(APPPATH.'/Views/pages/'.$page.'.php'))
		{
		    // Whoops, we don't have a page for that!
		    throw new \CodeIgniter\Exceptions\PageNotFoundException($page);
		}

		$data['title'] = ucfirst($page); // Capitalize the first letter

		echo view('templates/header', $data);
		echo view('pages/'.$page, $data);
		echo view('templates/footer', $data);
	}

現在，當請求的頁面實際存在時，將載入該頁面（包含 header 與 footer ），並顯示給使用者。如果請求的頁面不存在，將顯示「 404 Page not found 」錯誤。

這個方法中的第一行將檢查頁面是否實際存在。 PHP 的原生 ``is_file()`` 函數用於檢查檔案是否位於預期的位置。拋出 ``PageNotFoundException`` 異常將會導致 CodeIgniter 顯示預設的錯誤頁面。

在 header 樣板中， ``$title`` 變數用於自訂頁面的標題。標題的值將在這個方法中進行定義。它不是將值直接宣告在變數之中，而是宣告成 ``$data`` 陣列中鍵值為 title 的元素。

最後我們得按照順序依序載入視圖，對於這個操作，我們使用 CodeIgniter 內建的 ``view()`` 方法。``view()`` 方法中的第二個引數用於將一些值傳遞給視圖。 ``$data`` 陣列中的每個值在傳遞給視圖後，將會被宣告為以鍵值命名的變數。所以控制器中的 ``$data['title']`` 將等價於視圖中的 ``$title`` 。

.. note:: 傳入 **view()** 函數的任何檔案名稱與目錄名稱都必須真實存在且完全一致，否則你的程式可能會在一些區分大小寫的系統平台上出現錯誤。

運作應用程式
-------------------------------------------------------

準備好進行測試了嗎？你不能將這個 app 運作在 PHP 內建的伺服器之中，因為它無法正確處理 ``public`` 資料夾下 ``.htaccess`` 檔案所提供的路徑規則。這個檔案中的規則主要是讓你在 URL 中省略 「 index.php/ 」。而 CodeIgniter 有自己的命令，你可以使用這個命令。

在命令列中移動到專案的根目錄，執行：
::

    php spark serve

這行指令將會把 Web 伺服器啟動在 8080 埠上，如果在瀏覽器中前往 ``localhost:8080`` ，你應該可以看到 CodeIgniter 的歡迎畫面。

現在，你可以在瀏覽器中嘗試多種 URL ，以查看上面製作的 `Pages` 究竟產生了甚麼......

- ``localhost:8080/pages`` 將顯示控制器中 `index` 方法的結果，也就是顯示 CodeIgniter 「 welcome 」 頁面，因為 `index` 是控制器的預設方法。

- ``localhost:8080/pages/index`` 也會顯示 CodeIgniter 「 welcome 」 頁面，因為我們明確的要求使用 index 方法。

- ``localhost:8080/pages/showme`` 將顯示剛才製作的「 home 」頁面，因為它是 `showme()` 方法所預設的 page 引數。

- ``localhost:8080/pages/showme/home`` 將顯示 「 home 」頁面 ，因為我們明確的要求了 page 的值。

- ``localhost:8080/pages/showme/about`` 因為我們明確的要求了 about ，將顯示你剛才製作的「 about 」頁面，

- ``localhost:8080/pages/showme/shop`` 將顯示「 404 - File Not Found 」錯誤畫面，因為 `app/Views/pages/shop.php` 並不存在。

路由
-------------------------------------------------------

控制器運作正常！

使用自訂的路由規則，你可以將任何 URL 映射到任何控制器和方法，並且跳出這個預設的路由約定： ``http://example.com/[controller-class]/[controller-method]/[arguments]``

讓我們試試看吧！打開 *app/Config/Routes.php* 這個路由設定檔，並查找其中「定義路由（ Route Definitions ）」的部分。

唯一沒有被註解的程式應該是這一行：
::

    $routes->get('/', 'Home::index');

這個指令指出，未指定任何內容的請求都應該由 ``Home`` 控制器的 ``index`` 方法進行處理。

我們緊接著在這一行程式下方新增下列程式：

::

	$routes->get('(:any)', 'Pages::showme/$1');

CodeIgniter 將從上至下依序讀取路由規則，並將請求導向至第一個匹配的規則。每一個規則都是左側的正規表示法，以及右側的控制器和方法名稱所組成（以斜線分隔）。當請求進入時， CodeIgniter 會找到第一個匹配項，並且呼叫適當的控制器與方法（可能會有引數），

有關路由的詳細資訊，請參閱 :doc:`URL 路由條目 </incoming/routing>`。

在這裡， ``$routes`` 陣列中的第二個規則是， **任何** 請求都會與萬用字元字串 ``(:any)`` 相匹配後，再引數傳遞給 ``Pages`` 類別的 ``view()`` 方法。

現在造訪 ``home`` 。它是不是正確將路由導向至控制器中的 ``showme()`` 方法呢？做得好！

你應該可以看到類似於以下內容的畫面:

.. image:: ../images/tutorial1.png
    :align: center
