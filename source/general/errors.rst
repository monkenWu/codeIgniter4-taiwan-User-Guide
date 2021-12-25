##############
錯誤處理
##############

CodeIgniter 透過 Exceptions 將錯誤報告建構到你的系統中，包括 `SPL 異常集合 <https://www.php.net/manual/en/spl.exceptions.php>`_ 以及框架提供的一些自定義異常。根據你的環境設定，當一個錯誤或異常被拋出時，除非應用程式是在正式環境下執行，不然預設將會提示一個詳細的錯誤報告。在正式環境下，將會顯示一個通用的錯誤畫面，保持最佳的使用體驗。

.. contents::
    :local:
    :depth: 2

使用例外
================

本節是為新手程式設計師或是沒有使用過 exceptions 的開發者提供一個快速的概述。

「異常（exceptions）」指的是當某個異常被「拋出」時發生的簡單事件，這將會讓程式腳本停止當下的流程，然後執行錯誤處理程序，錯誤處理程序會顯示相應的錯誤頁面：

::

	throw new \Exception("Some message goes here");

如果你正在呼叫一個可能會拋出異常的方法，你可以使用 ``try/catch`` 來捕捉這個異常：

::

	try {
		$user = $userModel->find($id);
	} catch (\Exception $e) {
		die($e->getMessage());
	}

在這個例子中，我們將會捕捉所有類型的異常。如果 ``$userModel`` 拋出異常，則會被捕捉並執行 catch 內的程式碼。在這個例子中，腳本將會中止生命週期，回傳由 ``UserModel`` 所定義的錯誤訊息。

若是我們只想監視特定類型的異常，比如 UnknownFileException ，我們可以在 catch 參數中指定。任何被拋出且不是被捕捉的異常子類別，都將被傳遞給錯誤處理程序進行處理：

::

	catch (\CodeIgniter\UnknownFileException $e) {
		// do something here...
	}

這對於親自處理錯誤或在腳本結束之前執行清除很方便。如果你想讓錯誤處理程序正常執行，你可以在 catch 中拋出一個新的異常。

::

	catch (\CodeIgniter\UnknownFileException $e) {
		// do something here...

		throw new \RuntimeException($e->getMessage(), $e->getCode(), $e);
	}

組態設定
=============

在預設的情形下， Codeigniter 將顯示 ``development`` （開發） 和 ``testing`` （測試） 環境中所有的錯誤，而不會顯示錯誤在 ``production`` （上線） 環境中。你可以透過設定 ``.env`` 檔案中的  ``CI_ENVIRONMENT`` 來更改這個設定 。

.. important:: 寫入日誌這個行為並不會因為禁用錯誤報告而停止。

記錄例外
------------------

在預設情形下，除了「 404 找不到頁面」這個異常之外的所有異常都會被記錄下來。這可以透過設定 ``Config\Exceptions`` 的 **$log** 值來打開或關閉：

::

    class Exceptions
    {
        public $log = true;
    }

若要忽略其他的狀態碼，則可以在同一個檔案中設定忽略狀態碼：

::

    class Exceptions
    {
        public $ignoredCodes = [ 404 ];
    }

.. note:: 如果你當下的日誌設定沒有設定為記錄關鍵錯誤，那麼可能會無法記錄異常，所有的異常都會記錄為 **critical** （關鍵） 錯誤。

自定義例外
=================

以下是自定義的例外情況：

找不到頁面例外
---------------------

這是用來提示「 404 找不到頁面」的錯誤，當拋出時，系統將會顯示在 ``/app/views/errors/html/error_404.php`` 中找到的視圖。你應該為你的網站訂製所有錯誤的視圖。如果在 ``Config/Routes.php`` 中，你覆蓋了原有的 404 設定，那麼你的設定將會被呼叫，而不是標準的 404 頁面：

::

	if (! $page = $pageModel->find($id))
	{
		throw \CodeIgniter\Exceptions\PageNotFoundException::forPageNotFound();
	}

你可以將訊息傳遞到異常中，替換掉預設 404 所顯示的訊息。

設定例外
---------------

當組態設定類別中的值無效，或者設定類別的類型不對等的情快下，應該使用這個異常：

::

	throw new \CodeIgniter\Exceptions\ConfigException();

這個異常機會給予你一個 500 HTTP 狀態瑪，以及 3 退出碼 。

資料庫例外
-----------------

這個是個針對資料庫錯誤而拋出的異常，比如資料庫無法創建或暫時丟失時，就會拋出這個異常：

::

	throw new \CodeIgniter\Database\Exceptions\DatabaseException();

這個異常將會給予你一個 500 HTTP 狀態瑪，以及 8 退出碼 。

重新導向例外
-----------------

這是一個特殊狀況的例外，它允許你覆蓋所有響應的路由並強制重新導向到特定的路由或 URL ：

::

	throw new \CodeIgniter\Router\Exceptions\RedirectException($route);

``$route`` 可以是一個被宣告過的路由名稱、相對的 URL ，或者是完整的 URL 。你也可以給予一個重新導向碼來替換掉預設的 ( ``302`` , "temporary redirect") ：

::

	throw new \CodeIgniter\Router\Exceptions\RedirectException($route, 301);
