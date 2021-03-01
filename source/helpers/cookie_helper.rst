###############
Cookie 輔助函數
###############

Cookie 輔助函數檔案包含幫助使用 cookie 的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

載入此輔助函數
===================

此輔助函數可以利用以下的程式碼載入：

::

	helper('cookie');

可以使用的功能
===================

以下的功能都是可用的:

.. php:function:: set_cookie($name[, $value = ''[, $expire = ''[, $domain = ''[, $path = '/'[, $prefix = ''[, $secure = false[, $httpOnly = false]]]]]]])

	:param	mixed	$name: Cookie名稱 *或* 所有可用於此函數的所有參數的關聯陣列
	:param	string	$value: Cookie 值
	:param	int	$expire: 到期前的秒數
	:param	string	$domain: Cookie 域 （通常會是: .yourdomain.com）
	:param	string	$path: Cookie 路徑
	:param	string	$prefix: Cookie 名稱前綴
	:param	bool	$secure: 是否僅透過 HTTPS 傳送 cookie
	:param	bool	$httpOnly: 是否對 JavaScript 隱藏 cookie
	:rtype:	void

	這個輔助函數功能給你更友善的語法來設定瀏覽器 cookie。
	請參閱 :doc:`回應庫 </outgoing/response>` 對應它的使用說明,
	由於此函數是 ``Response::setCookie()`` 的一種。

.. php:function:: get_cookie($index[, $xssClean = false])

	:param	string	$index: Cookie 名稱
	:param	bool	$xss_clean: 是否對回傳值使用 XSS 過濾
	:returns:	cookie 值或如果沒找到的 NULL
	:rtype:	混合的

	此輔助函數功能提供更友善的語法以取得 cookie。
	由於此功能的動作與 ``IncomingRequest::getCookie()`` 相似，
	請參照 :doc:`傳入請求庫 </incoming/incomingrequest>` 
	以找到更詳細的使用說明。除非它同時也會預置你可能預先設定在你的 
	*app/Config/App.php* 檔中的 ``$cookiePrefix``

.. php:function:: delete_cookie($name[, $domain = ''[, $path = '/'[, $prefix = '']]])

	:param	string	$name: Cookie 名稱
	:param	string	$domain: Cookie 網域 (通常會是: .yourdomain.com)
	:param	string	$path: Cookie 路徑
	:param	string	$prefix: Cookie 名稱前綴
	:rtype:	void

	讓你刪除一個 cookie。除非你有設定一個自訂的路徑
	或其他值，否則你只需要 cookie 的名稱就足夠了。
	::

		delete_cookie('name');

	此功能與 ``set_cookie()`` 幾乎相同，除了它沒有值與倒數參數。
	你可以在第一個參數提交一個陣列的值，或是你可以設定離散的參數。
	::

		delete_cookie($name, $domain, $path, $prefix);