##########################
處裡 HTTP 請求
##########################

為了充分地使用 CodeIgniter，你需要對處理 HTTP 請求和響應的方式有基本的了解。對於所有想要成功的 Web 開發者來說，**必須** 要理解 HTTP 背後的概念。

本章的第一部分會簡單的的概述 HTTP ，接著我們會討論如何使用 CodeIgniter 來處理 HTTP 請求與響應。

甚麼是 HTTP ?
=============

HTTP 是兩台機器基於文本互相通信的一種協定。當瀏覽器請求頁面時，它會詢問伺服器是否可以獲取該頁面，然後，伺服器會準備好頁面並發送響應回發送請求的瀏覽器。就是這麼簡單，當然可以講得更複雜些，但基本上就是這樣。

HTTP 是用於描述交換協定的術語。它代表超文本傳輸協定（Hypertext Transfer Protocol）。當你在開發 web 應用程式時，你只需要專注在了解瀏覽器要求了什麼，並如何做出適當的響應。

HTTP 請求
-----------

當使用者端（瀏覽器，手機 APP 等）發送 HTTP 請求時，它會向伺服器發送一則文本訊息然後等待響應。

這個文本訊息會長得像這樣::

	GET / HTTP/1.1
	Host codeigniter.com
	Accept: text/html
	User-Agent: Chrome/46.0.2490.80

這則訊息包含了所有伺服器需要用來了解使用者端請求的資訊。比如它請求的 method（GET，POST，DELETE 等）以及它的 HTTP 版本。

這個請求還包含了許多可選的標頭，這些標頭可以包含各種資訊：例如使用者端希望內容顯示為哪種語言、使用者端接受的格式類型等等。如果你有需要的話，維基百科上有一篇文章，列出了 `所有的 HTTTP 頭欄位 <https://zh.wikipedia.org/wiki/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5>`_

HTTP 響應
------------

伺服器在收到請求後，你的 web 應用程式會處理這則訊息然後輸出一些結果。伺服器會將你的輸出結果綑綁為響應給使用者端的一部分。服務器對使用者端的響應訊息看起來會像這樣::

	HTTP/1.1 200 OK
	Server: nginx/1.8.0
	Date: Thu, 05 Nov 2015 05:33:22 GMT
	Content-Type: text/html; charset=UTF-8

	<html>
		. . .
	</html>

響應訊息告訴使用者端，伺服器正在使用的 HTTP 版本規範，以及響應狀態碼（200）。狀態碼對於使用者端而言，是具有特定涵義且標準化的代碼：它可以告訴使用者端響應成功（200），或者找不到頁面（404）等等。在 IANA 可以找到 
`完整的響應狀態碼列表 <https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml>`_。（`這裡有 MDN 的中文版狀態碼列表 <https://developer.mozilla.org/zh-TW/docs/Web/HTTP/Status>`_）

處理 HTTP 請求和響應
-----------------------------------

雖然 PHP 提供了原生與 HTTP 請求和響應進行交互的方式，但 CodeIgniter 像大多數框架一樣，將它們抽象化，讓你擁有一個一致、簡單的介面。:doc:`IncomingRequest 類 </incoming/incomingrequest>` 是 HTTP 請求的物件導向形式。它有著你所需要的一切::

	use CodeIgniter\HTTP\IncomingRequest;

	$request = new IncomingRequest(new \Config\App(), new \CodeIgniter\HTTP\URI());

	// the URI being requested (i.e. /about)
	$request->uri->getPath();

	// Retrieve $_GET and $_POST variables
	$request->getVar('foo');
	$request->getGet('foo');
	$request->getPost('foo');

	// Retrieve JSON from AJAX calls
	$request->getJSON();

	// Retrieve server variables
	$request->getServer('Host');

	// Retrieve an HTTP Request header, with case-insensitive names
	$request->getHeader('host');
	$request->getHeader('Content-Type');

	$request->getMethod();  // GET, POST, PUT, etc

request 類會在後台為你做很多工作，幫你省心。``isAJAX()`` 和 ``isSecure()`` 函數會自動檢查幾種不同的 method 來確定最後正確的回答。

.. note:: ``isAJAX()`` 方法取決於 ``X-Requested-With`` 標頭，在某些情況下，預設是不會通過 JavaScript（即 fetch）在 XHR 請求中發送的標頭。請參閱 :doc:`AJAX 請求 </general/ajax>` 部分，瞭解如何避免這個問題。

CodeIgniter 還提供了 :doc:`Response 類 </outgoing/response>`，它是 HTTP 響應的物件導向形式。它為你提供一種簡單而強大的方法來建構對客戶的響應::

  use CodeIgniter\HTTP\Response;

  $response = new Response();

  $response->setStatusCode(Response::HTTP_OK);
  $response->setBody($output);
  $response->setHeader('Content-type', 'text/html');
  $response->noCache();

  // Sends the output to the browser
  $response->send();

此外， :doc:`Response 類 </outgoing/response>` 還允許你處理 HTTP 快取層以獲得最佳性能。
