**************************
偵錯與除錯應用程式
**************************

.. contents::
    :local:
    :depth: 2

================
替換 var_dump
================

雖然在替你的應用程式除錯上使用 XDebug 與良好的 IDE 是不可或缺的，但有的時候敏捷的 ``var_dump()`` 可能更適合你。 CodeIgniter 同捆了優秀的 `Kint <https://kint-php.github.io/kint/>`_ 偵錯工具，這將使 CodeIgniter 變得更好。 Kint 遠遠超出了你所使用的常用工具，它提供了許多替代資料，比如：將時間戳格式化成可以被識別的日期、將十六進位色碼顯示為顏色，或是讓陣列資料輸出成表格型態使你方便閱讀，等等的眾多功能。

啟用 Kint
=============

在預設的情形下， Kint 僅在 **開發** 與 **測試** 環境下得以開啟。你可以透過修改 index.php 主檔案的環境設定部分中的 ``$useKint`` 變數來開啟 Kint 。

::

    $useKint = true;

使用 Kint
==========

**d()**

``d()`` 方法將會把你所傳入的資料傳遞到螢幕上，並且允許腳本繼續執行。

::

    d($_SERVER);

**dd()**

這個方法與 ``d()`` 相同，它也同樣等於 ``dies()`` 。執行完這個行程式，後續的程式碼將不會繼續執行。

**trace()**

透過 Kint 獨特的 spin 功能，你可以回朔到當前的執行點：

::

    trace();

欲知更多請參閱 `Kint 的 git 頁面 <https://kint-php.github.io/kint//>`_ 。

=================
除錯工具列
=================

除錯工具列提供了關於當下所請求的頁面一目瞭然的訊息，包括基準測試的結果、已運作的資料庫查詢、請求，以及響應資料等。這些在你的開發階段都會非常有用，可以幫助你除錯與最佳化。

.. note:: 除錯工具列目前仍在開發中，尚有幾個計畫中的功能還未能實現。

啟動工具列
====================

在 *除了* 上線環境外的任何環境，這個工具列都是預設啟動的。只要定義了 CI_DEBUG 常數值為 true ，工具列就會顯示。這個常數是在啟動檔案中宣告的（ **app/Config/Boot/development.php** ），可以在啟動檔案中宣告它會在甚麼環境下進行顯示。

.. note:: The Debug Toolbar is not displayed when your ``baseURL`` setting (in **app/Config/App.php** or ``app.baseURL`` in **.env**) does not match your actual URL.

工具列本身是以 :doc:`後濾器 </incoming/filters>` 的形式顯示，你可以透過從 **app/Config/Filters.php** 檔案中刪除 ``$globals`` 這個屬性來阻止它的運作。

選擇顯示內容
---------------------

CodeIgniter 內建多個蒐集器，用於蒐集資料並顯示在工具列上頭。你可以輕易地製作自己的工具列來個自訂工具列。要確定哪些蒐集器目前是被顯示的，你可以前往 **app/Config/Toolbar.php** 組態設定檔案：

::

    public $collectors = [
        \CodeIgniter\Debug\Toolbar\Collectors\Timers::class,
        \CodeIgniter\Debug\Toolbar\Collectors\Database::class,
        \CodeIgniter\Debug\Toolbar\Collectors\Logs::class,
        \CodeIgniter\Debug\Toolbar\Collectors\Views::class,
        \CodeIgniter\Debug\Toolbar\Collectors\Cache::class,
        \CodeIgniter\Debug\Toolbar\Collectors\Files::class,
        \CodeIgniter\Debug\Toolbar\Collectors\Routes::class,
        \CodeIgniter\Debug\Toolbar\Collectors\Events::class,
    ];

若是有你不想顯示的蒐集器，註解掉它即可。你可以透過完全符合要求的類別名稱來加入自定義的蒐集器，接著我們將告訴你蒐集器會影響到標籤頁以及時間軸上顯示的訊息是哪些。

.. note:: 有些標籤頁只在他們具有內容時才會被顯示出來，比如說資料庫與日誌檔案。否則，他們將從螢幕上被隱藏，已換取更多的顯示空間。

CodeIgniter 所內建的蒐集器為：

* **計時器** 蒐集所有基準測試的資料，包括系統以及你的應用程式。
* **資料庫** 顯示所有資料庫連接所執行的查詢列表，以及它們所執行的時間。
* **日誌** 任何被記錄的訊息都會在這裡顯示，在長期運作或記錄許多項目的系統中，可能會造成記憶體問題，這個功能應該要禁用。
* **視圖** 將渲染視圖的時間顯示於時間軸上，並且在單獨的標籤頁中顯示傳遞給視圖的所有資料。
* **快取** 將顯示有關快取命中與未命中的訊息，以及執行時間。
* **檔案** 顯示在這個請求期間已載入的所有檔案列表。
* **路由** 顯示目前的路由和系統中定義的所有路由的相關訊息。
* **事件** 顯示在當下的請求期間載入所有事件的列表。

設定基準測試點
========================

為了讓分析工具編譯和顯示你的基準測試資料，你必須使用特定的語法來命名你的標記點。

請閱讀 :doc:`基準測試程式庫 </testing/benchmark>` 頁面中關於設定基準測試點的資訊。

創建自訂蒐集器
==========================

創建自定蒐集器是一個簡單的任務，你得創建一個新的類別，並以完整的命名空間命名，自動載入器才能夠找到它。它繼承了 ``CodeIgniter\Debug\Toolbar\Collectors\BaseCollector`` 類別，這個類別提供了許多你可以置換的方法，你必須根據你預想的工作方式來正確設定這些屬性。

::

    <?php

    namespace MyNamespace;

    use CodeIgniter\Debug\Toolbar\Collectors\BaseCollector;

    class MyCollector extends BaseCollector
    {
        protected $hasTimeline = false;

        protected $hasTabContent = false;

        protected $hasVarData = false;

        protected $title = '';
    }

對於任何想要在工具列的時間軸中顯示訊息的蒐集器， **$hasTimeline**  應該設定為 ``true`` 。如果為 ``true`` 則你需要實作 ``formatTimelineData()`` 方法來格式化並回傳資料進行顯示。

如果蒐集器想使用自訂的內容並顯示自己的標籤頁，**$hasTabContent** 應該要為 ``true`` 。你需要提供 ``$title`` ，實作 ``display()`` 方法來渲染出標籤的內如容。如果你想在標籤內容的標題右方顯示額外訊息，你可能需要實作出 ``getTitleDetails()`` 方法。

如果這個蒐集器想要把額外的資料加入到 ``Vars`` 標籤中， **$hasVarData** 必須為 ``true`` 。如果為 true ，則需要實作 ``getVarData()`` 方法。

**$title** 定義的是顯示在標籤上的標題。

顯示工具列標籤
------------------------

若想顯示一個工具列標籤，你必須：

1. 在 ``$title`` 中填寫作為工具列標題以及標籤 header 的文字。
2. 設定 ``$hasTabContent`` 為 ``true`` 。
3. 實作 ``display()`` 方法。
4. 實作 ``getTitleDetails()`` 方法（這是可選的）。

``display()`` 創建了在標籤頁中顯示的 HTML 。你不需要擔心標籤頁的標題，因為這是由工具列自動處理的。這個方法應該要回傳一個 HTML 字串。

``getTitleDetails()`` 方法應該回傳一個字串，這是顯示在標籤標題右方的字串。例如：資料庫標籤顯示的是所有連接的查詢總數，而檔案標籤顯示的是檔案總數的文字內容。

提供時間軸資料
-----------------------

若想提供需要在時間軸中顯示的訊息，你必須：

1. 設定 ``$hasTimeline`` 為 ``true`` 。
2. 實作 ``formatTimelineData()`` 方法。

``formatTimelineData()`` 方法必須回傳一個含有陣列的陣列，其格式化的方式必須使用時間軸可以正確排序的方式記錄，內部陣列必須包含以下資訊：

::

    $data[] = [
        'name'      => '',     // Name displayed on the left of the timeline
        'component' => '',     // Name of the Component listed in the middle of timeline
        'start'     => 0.00,   // start time, like microtime(true)
        'duration'  => 0.00,   // duration, like mircrotime(true) - microtime(true)
    ];

提供變數
--------------

若想將資料加入到 Vars 標籤中，你必須：

1. 設定 ``$hasVarData`` 為 ``true`` 。
2. 實現 ``getVarData()`` 方法。

``getVarData()`` 方法應該要回傳一個包含所要顯示的資訊的鍵值陣列，外部陣列的鍵名是 Vars 標籤的部分名稱：

::

    $data = [
        'section 1' => [
            'foo' => 'bar',
            'bar' => 'baz',
        ],
        'section 2' => [
            'foo' => 'bar',
            'bar' => 'baz',
        ],
     ];