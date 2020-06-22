#####
視圖
#####

.. contents::
    :local:
    :depth: 2

視圖簡單來說就是一個網頁，或者是一個網頁的片段，比如頁眉、頁腳，與側邊欄位等等。事實上，如果你需要這種類型的網頁結構，「視圖」即可讓你靈活地將一個或多個視圖檔案嵌入到其他視圖檔案中。

視圖永遠不會被直接呼叫，它們必須由控制器載入。請注意，在 MVC 框架中，控制器是指揮者的角色，它負責獲取特定的視圖。如果你還沒有閱讀過 :doc:`控制器 </incoming/controllers>` 條目，那麼可以在繼續視圖的學習前先去閱讀一下。

使用你在控制頁面中創建的範例控制器，我們來替它加上一個視圖。

新建一個視圖
===============

使用你的編輯器或開發環境，創建一個 ``BlogView.php`` 檔案，並將以下內容置入其中：

::

	<html>
        <head>
            <title>My Blog</title>
        </head>
        <body>
            <h1>Welcome to my Blog!</h1>
        </body>
	</html>

然後，把這個檔案儲存在你的 **app/Views** 目錄下。

顯示一個視圖
=================

若需要載入和顯示一個特定的視圖檔案，你得使用下列函數：

::

	echo view('name');

函數中的傳入值 *name*  是視圖的檔案名稱。

.. important:: 如果等省去掉檔案的副檔名，那麼視圖將會自動加上 .php 的副檔名。

現在，打開你之前製作的 ``Blog.php`` 控制器檔案，將 echo 語法替換成視圖函數：

::

	<?php namespace App\Controllers;

        class Blog extends \CodeIgniter\Controller
	{
		public function index()
		{
			echo view('BlogView');
		}
	}

如果你使用上一個範例的 URL 造訪你的網站，你應該會看到新的視圖。這個 URL 視類似於這樣的：

::

	example.com/index.php/blog/

.. note:: 雖然所有範例都直接以 echo 的方式顯示視圖，但你也可以利用回傳的方式輸出視圖，它將會被附加到任何被捕獲的輸出中。

載入多個視圖
======================

CodeIgniter 將智慧地處理一個控制器中，對多個 ``view()`` 的呼叫。如果有一個以上的 ``view()`` 被呼叫，它們將會被串接在一起。例如，你可能希望有一個標題頁眉、選單視圖、內容視圖以及頁腳視圖，實作時看起來會像這樣：

::

	<?php namespace App\Controllers;

	class Page extends \CodeIgniter\Controller
	{
		public function index()
		{
			$data = [
				'page_title' => 'Your title'
			];

			echo view('header');
			echo view('menu');
			echo view('content', $data);
			echo view('footer');
		}
	}

上面的範例中，有使用到「動態添加資料」的功能，你會在待會出現的內容閱讀到相關的資訊。

載入在子目錄中的視圖
====================================

如果你喜歡透過組織，將視圖檔案分類在子目錄中，你也可以這麼做。當你這麼儲存你的視圖時，呼叫載入方法就必須將所要載入的目錄名稱也一起寫進來，例如：

::

	echo view('directory_name/file_name');

視圖的命名空間
================

你可以將視圖儲存在一個自訂命名的 **View** 目錄下，並以使用命名空間相同的方式載入它們。雖然 PHP 並不支援讓非類別以外的檔案擁有命名空間一樣的載入特性，但 CodeIgniter 提供了這個功能，你可以將你的視圖檔案以類似模組的方式進行管理，方便重用與佈局。

如果你的 ``Blog`` 目錄在 :doc:`自動載入器 </concepts/autoloader>` 中設定了一個位於 ``Example\Blog`` 下的 PSR-4 的映射。那麼，你就可以使用像一般存取視圖檔案一樣的方式，以命名空間的特性去載入它們。依照這個範例，你只需在視圖的名稱前加上正確的命名空間，就可以從 **/blog/views** 中載入 **BlogView** ：

::

  echo view('Example\Blog\Views\BlogView');

快取視圖
=============

你可以透過在第三個參數中傳入一個帶有 ``cache`` 選項的陣列，來命令 ``view`` 進行快取，並設定快取的秒數：

::

    // 快取視圖 60 秒
    echo view('file_name', $data, ['cache' => 60]);

在預設的情形下，視圖將使用與視圖檔案本身相同的名稱進行快取。你也可以在設定的陣列中傳入 ``cache_name`` 與你所想使用的快取名稱來取代這個預設值：

::

    // 快取視圖 60 秒
    echo view('file_name', $data, ['cache' => 60, 'cache_name' => 'my_cached_view']);

替視圖加入動態資料
===============================

資料是透過視圖函數的第二個參數從控制器傳入到你所開啟的視圖的。下面將演示一個範例：

::

	$data = [
		'title'   => 'My title',
		'heading' => 'My Heading',
		'message' => 'My Message'
	];

	echo view('blogview', $data);

讓我們延續你之前製作過的範例繼續試試，在你所製作控制器檔案中加入這段程式碼：

::

	<?php namespace App\Controllers;

	class Blog extends \CodeIgniter\Controller
	{
		public function index()
		{
			$data['title']   = "My Real Title";
			$data['heading'] = "My Real Heading";

			echo view('blogview', $data);
		}
	}

現在，打開你的視圖檔案，把標籤的內容改成由控制器傳入的陣列中，元素鍵名相對應的變數：

::

	<html>
        <head>
            <title><?= $title ?></title>
        </head>
        <body>
            <h1><?= $heading ?></h1>
        </body>
	</html>

然後，用瀏覽器重新進入此頁面，你應該就會看到原本的文字內容被變數的內容替換了。

傳入的資料，只能在該次呼叫的視圖中使用。如果你在一個請求中，多次呼叫 `view` 函數，你必須傳遞所需的資料到每個視圖中。這樣就能防止視圖間的資料不會汙染到對方。如果你想讓傳入到視圖中的資料持久化，你可以在第三個參數中傳入 `saveData` 選項來啟動：

::

	$data = [
		'title'   => 'My title',
		'heading' => 'My Heading',
		'message' => 'My Message'
	];

	echo view('blogview', $data, ['saveData' => true]);

此外，如果你希望視圖函數的預設功能是在呼叫之間保存資料，你可以在 **app/Config/Views.php** 的設定檔案中，將 ``$saveData`` 變數設定為 **true** 。

建立迴圈
==============

你傳入給視圖檔案的資料陣列並不限於簡單的變數，你也可以傳遞多維陣列，它可以用於以迴圈執行。例如，你從資料庫中取出數筆資料，資料通常會是多維陣列的形式。

這裡有一個簡單的範例，將以下程式碼加入到你的控制器中：

::

	<?php namespace App\Controllers;

	class Blog extends \CodeIgniter\Controller
	{
		public function index()
		{
			$data = [
				'todo_list' => ['Clean House', 'Call Mom', 'Run Errands'],
				'title'     => "My Real Title",
				'heading'   => "My Real Heading"
			];

			echo view('blogview', $data);
		}
	}

現在，打開你的視圖檔案新建一個迴圈：

::

	<html>
	<head>
		<title><?= $title ?></title>
	</head>
	<body>
		<h1><?= $heading ?></h1>

		<h3>My Todo List</h3>

		<ul>
		<?php foreach ($todo_list as $item):?>

			<li><?= $item ?></li>

		<?php endforeach;?>
		</ul>

	</body>
	</html>
