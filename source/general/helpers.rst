################
輔助函數
################

輔助函數，顧名思義就是輔助你完成任務的助手。每個輔助函數檔案都是特定類別的功能集合。有輔助你創造連結的 **URL 輔助函數** 、創建表單元素的 **表單輔助函數** 、各種文字格式化的 **文字輔助函數** 、設定和讀取 Cookie 的 **Cookie 輔助函數**，以及替你處理檔案的 **檔案輔助函數** 等等功能。

.. contents::
    :local:
    :depth: 2

與大多數的框架不同，CodeIgniter 的輔助函數不是以物件導向的方式撰寫的。它們是簡單且過程式的函數（ procedural functions ）。每個輔助函數只執行一個特定的任務，並不會依賴其他函數。

因為 CodeIgniter 在預設的情況下並不會載入任何程式檔案，所以使用輔助函數的第一步就是得先載入它。一旦載入後，它將會在你的 :doc:`控制器 </incoming/controllers>` 與 :doc:`視圖 </outgoing/views>` 中變成全域可以使用的狀態。

輔助函數通常會存放在 **system/Helpers** 或 **app/Helpers** 路徑下。 CodeIgniter 首先會在 **app/Helpers** 路徑下尋找你所指定的輔助函數。如果不存在，才會在全域的 **system/Helpers/** 路徑下尋找。

載入輔助函數
================

載入一個輔助函數是一件非常簡單的事情，就下面這樣：

::

	helper('name');

其中的 **name** 是輔助函數的檔案名稱，並不包含 .php 副檔名或 helper 字詞。

例如，要載入 **Cookie 輔助函數** 它的檔案名稱是 **cookie_helper.php** ，你要載入它時則必須這麼做：

::

	helper('cookie');

如果你需要同時載入多個輔助函數，你可以將一個陣列傳入其中，所有陣列中有提到的檔案名稱都會被載入。

::

	helper(['cookie', 'date']);

只要在使用輔助函數之前載入它，它可以在你的控制器的任何地方載入（雖然這並不是個好方法，但你甚至可以在視圖檔案中進行載入）。你可以在控制器的建構中載入輔助函數，這樣它就能在任何函數中自動可用。或者，你也可以在需要輔助函數的特定函數中載入只有它可以使用的輔助函數。

.. note:: 上面所提到的輔助函數載入方法並不會回傳任何值，所以不要試圖藉由載入方法來宣告任何變數，如上所示地使用它就行了。

.. note::  URL 輔助函數會在任何時候自動載入，所以你不需要手動載入它。

從非標準位置載入輔助函數
-----------------------------------

輔助函數可以從 **app/Helpers** 以及 **system/Helpers** 以外的路徑載入，在 :doc:`自動載入檔案 <../concepts/autoloader>` 的 PSR-4 部分中所設定的命名空間可以找到預設以外的路徑。你可以在輔助函數的名稱前加上它所屬的命名空間，在這個命名空間中，載入器希望它位於 ``Helpers`` 子目錄裡，下面的範例可以幫助你解決這個問題。

在這個範例中，假設我們將所有與 Blog 相關的程式碼都歸類到自己的命名空中，就像 ``Example\Blog`` 這樣。這些檔案都放置在伺服器的 **/Modules/Blog/** 路徑中。因此，我們把 blog 模組的輔助函數檔案放在 **/Modules/Blog/Helpers/** 中，也就是把 **blog_helper** 儲存在 **/Modules/Blog/Helpers/blog_helper.php** 這個位置。在我們的控制器裡，我們可以用下述方式來載入輔助函數：

::

	helper('Modules\Blog\blog');

.. note:: 以這種方式載入的檔案，它所表示的命名空間並不是真正的命名空間，而是一種方便定位檔案的方式。

使用輔助函助
==============

一旦你載入了你想要使用的輔助函數檔案，你就可以像是呼叫內建 PHP 函數一樣呼叫它們。

例如，在你的一個視圖檔案中使用 ``anchor()`` 函數建立一個連接，你可以這麼做：

::

	<?php echo anchor('blog/comments', 'Click Here');?>

其中的 Click Here 是連結的名稱， blog/comments 則是希望可以連接到的控制器與方法的 URI 。

擴充輔助函數
===================

如果你想要「擴充」輔助函數，請在  **app/Helpers/** 資料夾下新建一個與現有輔助函數相同名子的檔案。

如果你只想要在現有的輔助函數中添加一兩個新功能，或者是改變一個特定的輔助函數下的功能的運作方式，那麼使用你的版本直接替換整個輔助函數就會顯得矯枉過正了。在這種需求下，最好是簡單地擴充輔助函數。

.. note:: 術語「擴充」的定義不太嚴謹，因為擴充函數有著過程式且離散的特性，不能在傳統程序化（programmatic）的意義上進行擴充。就本質上來說，「擴充」這個功能可以讓你獲得增加或替換輔助函數中方法的能力。

例如，為了擴充原生的 **陣列輔助函數** 你將新建一個名為 **app/Helpers/array_helper.php** 的檔案，你可以新增或覆蓋函數。

::

	// any_in_array() is not in the Array Helper, so it defines a new function
	function any_in_array($needle, $haystack)
	{
		$needle = is_array($needle) ? $needle : [$needle];

		foreach ($needle as $item)
		{
			if (in_array($item, $haystack))
			{
				return TRUE;
			}
	        }

		return FALSE;
	}

	// random_element() is included in Array Helper, so it overrides the native function
	function random_element($array)
	{
		shuffle($array);
		return array_pop($array);
	}

**helper()** 方法將會掃描所有在 **app/Config/Autoload.php** 中定義的 PSR-4 命名空間，並載入所有相同名稱的輔助函數。這樣就可以載入任何模組的輔助函數，以及你為某個應用程式專門建立的輔助函數。載入順序如下所示：

1. app/Helpers ：在這裡的檔案將會最先被載入。
2. {namespace}/Helpers ：所有的命名空間都是按照定義的順序循環載入。
3. system/Helpers ：基本檔案最後才會進行載入。

接下來呢？
=================

所有你可以使用的輔助函數都在輔助函數條目中條列著。細心查閱吧！看看它們如何使用。
