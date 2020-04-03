################
CodeIgniter URLs
################

在預設的情況下， CodeIgniter 中的 URL 設計得友好於搜尋引擎與人類。 CodeIgniter 沒有使用 query-string 的方式處理 URL ，而是以 **基於區段** 的方式進行處理：

::

	example.com/news/article/my_article

URI 區段
============

按照 模型-視圖-控制器 的規則， URL 中的區段通常會這樣子表示：

::

	example.com/class/method/ID

1. 第一個區段通常是指被呼叫的控制器　**類別** 。
2. 第二個區段通常使指被呼叫類別下的 **方法** 。
3. 在這之後任何的區段，都將會成為傳遞給控制器的變數（方法的參數）。

在 :doc:`URI 函式庫 <../libraries/uri>` 與 :doc:`URL 輔助函數 <../helpers/url_helper>` 包含了一些函數可以便利地處理你的 URL 資料。除此之外，你的 URL 也可以使用 :doc:`URL 路由 </incoming/routing>` 功能重新映射，讓它變得更加靈活。

隱藏在路由中的 index.php
===========================

通常， **index.php** 檔案會在你的 URLs 中出現：

::

	example.com/index.php/news/article/my_article

如果你的伺服器支援 URL 重寫的功能，你可以通過 URL 重寫來輕易地隱藏這個檔案。不同伺服器的處理方式各不相同，我們將在這裡以兩個最常見的 Web 伺服器作為例子。

Apache 網頁伺服器
-----------------

Apache 必須啟用 *mod_rewrite* 擴充元件。如果成功啟用，你就可以使用帶有簡單規則 ``.htaccess`` 檔案。這裡以一個使用了「 negative 」方法的例子，除了被指定的項目外，所有的請求都會被重新定向：

.. code-block:: apache

	RewriteEngine On
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule ^(.*)$ index.php/$1 [L]

在這個例子中，除了已存在真實路徑的目錄與檔案外，任何的 HTTP 請求都會被視為對 index.php 進行請求。

.. note:: 這些特定的規則可能不會在所有伺服器設定上都有效。

.. note:: 請確保上述的規則中，排除了你所要使用到的靜態資源。

NGINX
-----

在 NGINX 下，你可以定義一個 location 程式塊，然後使用 ``try_files`` 指令，就可以得到與 Apache 設定檔相同的效果：

.. code-block:: nginx

	location / {
		try_files $uri $uri/ /index.php$is_args$args;
	}

首先，它會先尋找一個與 URL 相同的檔案或目錄（從根目錄和別名指令的設定下，建構出完整的檔案路徑），之後才會把請求與參數導向至 index.php 。