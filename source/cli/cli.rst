############################
透過命命列運作
############################

除了透過瀏覽器中以 URL 呼叫應用程式的 :doc:`控制器 </incoming/controllers>` 外，你還可以透過命令列介面（ CLI ）來載入它們。

.. contents::
    :local:
    :depth: 2

什麼是命令列介面
================

命令列介面是一種基於文字與電腦交互的方式，更多資訊可以查看 `維基百科 <https://zh.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%95%8C%E9%9D%A2>`_ 。 

為何需要透過命令列介面運作
=============================

從命令列介面運作 CodeIgniter 的原因有很多，雖然它們並不是這麼顯而易見。

-  運作你的排程工作，不需要使用 *wget*  或 *curl* 。
-  透過檢查 :php:func:`is_cli()` 的回傳值，使你的排程工作無法在 URL 中載入。
-  製作交互式的任務，可以做一些特別的事情，比如：權限設定、修剪快取，以及運作備份等。
-  與其他語言進行整合，例如： 一個隨意的 C++ 腳本可以呼叫一個命令，然後在你的模型中運作這個程式碼。

讓我們試試： Hello World!
==========================

讓我們建立一個簡單的控制器，這樣你就可以看到操作的過程。使用你的文字編輯器，建立一個名為 Tools.php 的檔案，並在其中寫入以下程式碼：

::

    <?php

    namespace App\Controllers;

    use CodeIgniter\Controller;

    class Tools extends Controller
    {
        public function message($to = 'World')
        {
            echo "Hello {$to}!" . PHP_EOL;
        }
    }

接著，把這個檔案儲存在你的 **app/Controllers/** 目錄底下。

通常你會使用這樣的 URL 造訪你的網站：

::

    example.com/index.php/tools/message/to

但現在，我們打開 Mac/Linux 的 Terminal ，或者在 Windows 中打開 cmd ，然後定位至 CodeIgniter 的專案根目錄。

.. code-block:: bash

    $ cd /path/to/project/public
    $ php index.php tools message

如果你做得沒錯，你應該會看到螢幕上出現 *Hello World!* 的訊息。

.. code-block:: bash

    $ php index.php tools message "John Smith"

在這裡，我們使用與 URL 一樣的方式傳遞一個參數， "John Smith" 就會作為參數傳入到控制器方法中，你得輸出就會變為：

::

    Hello John Smith!

基本知識
==================

總而言之，這就是命令列中關於控制器的所有知識。記住，這只是一個普通的控制器，所以路由以及 ``_remap()`` 都能夠正常工作。

然而， CodeIgniter 提供了額外的工具，讓你可以更加愉快地建立命令列介面。包括只限於命令列的路由以及幫助你開發該路由工具的程式庫。

只限於命令列的路由
-------------------

在你的 **Routes.php** 檔案中，你可以像創建其他路由一樣，新增只有命令列介面可以存取的路由。你可以使用 ``cli()`` 方法取代 ``get()`` 或是 ``post()`` 方法，剩下的就和普通的路由定義完全一樣。

::

    $routes->cli('tools/message/(:segment)', 'Tools::message/$1');

更多路由相關資訊，請詳閱 :doc:`路由 </incoming/routing>` 頁面。

命令列程式庫
---------------

命令列程式庫讓你更簡單地使用 CLI 介面，它提供了彩色文字輸出到終端機畫面的簡單方法，更可以允許你向使用者提示訊息，使你能夠輕鬆建立靈活且智慧的工具。

更多詳情請見 :doc:`命令列程式庫 </cli/cli_library>` 頁面。