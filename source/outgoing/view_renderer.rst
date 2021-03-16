#############
視圖渲染
#############

.. contents::
    :local:
    :depth: 2

使用視圖渲染
***************************

``view()`` 是一個方便取得 ``renderer`` 服務實體的函數，設置資料並且渲染出此視圖。然而這通常是你所想要的，你可能會更直接的發現你使用它的次數：
::
	$view = \Config\Services::renderer();

或者，如果你不使用 ``View`` 類別當成你的預設渲染器，則可以直接實體化它：
::

	$view = new \CodeIgniter\View\View();

.. important:: 你應該在控制器內創建服務，如果你需要從程式庫內存取 View 類別，你應該在你的程式庫的建構內設置依賴。

然而你可以使用它提供三種標準方法內的任何一種：
**render(viewpath, options, save)**, **setVar(name, value, context)** 和 **setData(data, context)**.

它能做什麼
============

此 ``View`` 類別處理常規儲存在應用程式視圖路徑內的 HTML/PHP 腳本，將視圖參數提取到 PHP 變數後，就可以在腳本內中訪問。
這意味著你的視圖變數的命名必須是合法的 PHP 變數名稱。

此視圖類別在內部使用關聯陣列來積累視圖參數直到你呼叫 ``render()``。這代表了你的參數（或者變數）的名稱必須是唯一的，否則之後的變數設置將會覆蓋掉之前的。

這還會影你在腳本內不同內文的轉義參數值。你將必須為了每個轉義的值賦予唯一個參數名稱。

對於值為一個不附加任何含意的陣列參數。你可以依據自己的 PHP 程式碼適當地處理陣列。

方法鏈
===============

`setVar()` 和 `setData()` 方法是可被鏈接的，你可以將多個不同呼叫組合在一起：
::

	$view->setVar('one', $one)
	     ->setVar('two', $two)
	     ->render('myView');

轉義資料
=============

當你將資料傳遞給 ``setVar()`` 和 ``setData()`` 函數時，你可以選擇轉義你的資料以防止跨站腳本攻擊，你可以傳遞所需的語境以轉義數據。有關語境描述，請參見下文。

如果你不想轉義你的資料，你可以傳遞 `null` 或 `raw` 作為最終參數給每個函數：
::

	$view->setVar('one', $one, 'raw');

如果你選擇不轉義資料，或者你在物件實體內傳遞，你可以在帶有 ``esc()`` 函數的視圖內手動地轉義資料。則第一個參數就是要轉義的字串。而第二個參數是用於轉義數據的內容（見下文）：
::

	<?= esc($object->getStat()) ?>

轉義內容
-----------------

在預設情況下，``esc()`` 以及 ``setVar()`` 和 ``setData()`` 函數會假定你想轉義的資料在規範的 HTML 中使用。但是，如果資料打算在 Javascript，CSS 或 href 的屬性中使用，則需要使用不同的轉義規則才能生效。你可以傳入語境名稱作為第二個參數。有效的語境為html、js、css、url 和 attr：
::

	<a href="<?= esc($url, 'url') ?>" data-foo="<?= esc($bar, 'attr') ?>">Some Link</a>

	<script>
		var siteName = '<?= esc($siteName, 'js') ?>';
	</script>

	<style>
		body {
			background-color: <?= esc('bgColor', 'css') ?>
		}
	</style>

查看渲染器選項
=====================

可以將幾個選項傳遞給 ``render()`` 或 ``renderString()`` 方法：

-   ``cache`` - 儲存視圖的結果的時間（以秒為單位）；忽略 renderString()
-   ``cache_name`` - 用於儲存/獲取快取視圖結果的 ID ；預設值為 viewpath；
		忽略 renderString()
-   ``saveData`` - 如果應保留視圖資料參數以供後續調用，則為 true 

.. note:: 介面定義的 ``saveData`` 必須為布林值，但實作類（如下面的View）可能會將其擴展為包含 ``null`` 值。

類別參考
***************

.. php:class:: CodeIgniter\\View\\View

	.. php:method:: render($view[, $options[, $saveData=false]])
                :noindex:

		:param  string       $view: 視圖來源的檔案名稱
		:param  array        $options: 選項陣列，作為一對鍵/值
		:param  boolean|null $saveData: 如果為 true，將保存資料以供其他任何呼叫使用。如果為 false，將在渲染視圖後清除資料。如果為 null，則使用 config 設置。
		:returns: 所選視圖的渲染文字
		:rtype: string

		根據文件名稱和任何以設置的資料來建構輸出：
		::

			echo $view->render('myview');

	.. php:method:: renderString($view[, $options[, $saveData=false]])
                :noindex:

		:param  string       $view: 要呈現視圖的內容，例如從數據庫檢索的內容
		:param  array        $options: 選項陣列，作為一對鍵/值
		:param  boolean|null $saveData: 如果為 true，將保存資料以供其他任何呼叫使用。如果為 false，將在渲染視圖後清除資料。如果為 null，則使用 config 設置。
		:returns: 所選視圖的渲染文字
		:rtype: string

		根據文件名稱和任何以設置的資料來建構輸出：
		::

			echo $view->renderString('<div>My Sharona</div>');

		這可以用於顯示可能已存儲在資料庫中的內容，但你需要注意到這是一個潛在的安全漏洞，並且你 **必須** 驗證任何此類別的數據，並可能適當地對其進行轉義！

	.. php:method:: setData([$data[, $context=null]])
                :noindex:

		:param  array   $data: 選項陣列，作為一對鍵/值
		:param  string  $context: 用於數據轉義的內文。
		:returns: 渲染器，用於方法鏈接
		:rtype: CodeIgniter\\View\\RendererInterface.

		一次設置多條視圖數據：
		::

			$view->setData(['name'=>'George', 'position'=>'Boss']);

		支持的轉義的內文：html，css，js，url，attr或raw。
		如果 'raw', 就不會發生轉義。

		每次呼叫都會添加物件至正在累積的資料陣列中，直到呈現視圖為止。

	.. php:method:: setVar($name[, $value=null[, $context=null]])
                :noindex:

		:param  string  $name: 視圖資料變數的名稱
		:param  mixed   $value: 視圖資料的值
		:param  string  $context: 用來轉義的內文
		:returns: 渲染器，用於方法鏈接
		:rtype: CodeIgniter\\View\\RendererInterface.

		設置單個視圖資料::

			$view->setVar('name','Joe','html');

		支持的轉義的內文：html，css，js，url，attr或raw。
		如果 'raw', 就不會發生轉義。

		每次呼叫都會添加物件至正在累積的資料陣列中，直到呈現視圖為止。
