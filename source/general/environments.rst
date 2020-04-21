##############################
處理多種環境
##############################

開發者通常會希望在開發環境或正式環境中，得到不同的系統行為。例如：在開發應用程式時，錯誤輸出是一件很有用的事情，但在上線環境中可能會帶來安全隱憂。在開發環境中，你可能會希望載入一些正式環境中不會使用的額外的工具。

環境常數
========================

在預設的情形下，CodeIgniter 內建的環境常數將使用 ``$_SERVER['CI_ENVIRONMENT']`` 提供的值，否則默認為 production（上線模式）。可以根據你的伺服器的性質，設定多種不同的模式。

.env
----

設定變數最簡單的方式就是使用 :doc:`.env 檔案 </general/configuration>` 。

.. code-block:: ini

    CI_ENVIRONMENT = development

Apache
------

這個伺服器變數可以在你的 ``.htaccess`` 檔案與 Apache `設定檔案 <https://httpd.apache.org/docs/2.2/mod/mod_env.html#setenv>`_ 中進行設定。

.. code-block:: apache

    SetEnv CI_ENVIRONMENT development

nginx
-----

在 nginx 下，你必須透過 ``fastcgi_params`` 將環境變數傳遞給 `$_SERVER` 變數。儘管 env 在伺服器上可以正常運作，但它也可以不使用 env 在虛擬主機上工作。你需要修改你的伺服器設定，類似於這樣：

.. code-block:: nginx

	server {
	    server_name localhost;
	    include     conf/defaults.conf;
	    root        /var/www;

	    location    ~* \.php$ {
	        fastcgi_param CI_ENVIRONMENT "production";
	        include conf/fastcgi-php.conf;
	    }
	}

這個替代方法可以用於 nginx 或其他伺服器，你也可以完全刪除這個邏輯，並根據伺服器的 IP 位置設定常數。

除了會影響一些基本的框架行為外（請參考下一個條目），你還可以在你自己的開發中專案使用個常數來區分正在運作的環境為何。

引導檔案
----------

CodeIgniter 把與環境名稱相同的 PHP 腳本置於 **APPPATH/Config/Boot** 之下。這些檔案可以包含任何對於環境的自定義設定，無論是更新錯誤顯示設定、載入額外的開發者工具，還是其他的設定，這些檔案都會被系統自動載入。下列文件是在你部屬全新的框架時就創建的：

* development.php
* production.php
* testing.php

影響預設的框架行為
=====================================

在 CodeIgniter 系統中，有一些地方使用了 ENVIRONMENT 常數。本條目將會介紹預設的框架行為是如何受到影響的。

錯誤報告
---------------

將 ENVIRONMENT 常數設定為 development（開發） 會導致所有的 PHP 錯誤在發生時會直接渲染到瀏覽器。反之，將常數設定為 production（正式） 將禁止所有錯誤報告的輸出。在正式環境中禁用錯誤報告是一個良好的 :doc:`安全實踐 </concepts/security>` 。

組態設定檔案
-------------------

另外，你可以讓 CodeIgniter 載入特定環境的組態設定檔案。這非常適合管理不同環境下不同的 API 金鑰。關於這個設定的細節，將在 :doc:`組態設定檔案 </general/configuration>` 使用說明的「處理不同環境」條目中提到。