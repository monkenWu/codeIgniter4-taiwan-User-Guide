###############
資料庫事件
###############

資料庫類別包含一些 :doc:`事件 </extending/events>`，你可以利用這些事件來了解關於資料庫執行時，正在發生的事情或資訊。可以利用這些事件來搜集資訊並做分析和報告。 :doc:`偵錯與除錯應用程式 </testing/debugging>` 就是利用這些事件搜集查詢後的資訊並顯示在工具列上。

.. contents::
    :local:
    :depth: 2

==========
事件
==========

**DBQuery**

當執行新的查詢後，無論成功與否，都會觸發這個事件。唯一參數是當前語法的 :doc:`查詢 </database/queries>` 實體。
你可以使用這個事件在 STDOUT 中顯示所有查詢，記錄在文件中，甚至創建工具進行自動查詢分析，幫助你找出可能遺失的索引或查詢速度慢的原因等。範例參考如下：

::

    // In Config\Events.php
    Events::on('DBQuery', 'CodeIgniter\Debug\Toolbar\Collectors\Database::collect');

    // Collect the queries so something can be done with them later.
    public static function collect(CodeIgniter\Database\Query $query)
    {
        static::$queries[] = $query;
    }
