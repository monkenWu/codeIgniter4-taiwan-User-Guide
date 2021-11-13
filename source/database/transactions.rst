############
交易
############

CodeIgniter 透過資料庫抽象化，讓你在可以在支援交易的安全資料表類型的資料庫中使用交易。在 MySQL 中，你需要運行 InnoDB 或 BDB 的資料表類型，而不是更常見的MyISAM。大多數的資料庫系統支援原生的交易功能。

如果你不熟悉交易，我們建議你在網路上瞭解你所使用的資料庫的交易。以下你的資訊都假設你對交易有一定程度的了解。

.. contents::
    :local:
    :depth: 2

CodeIgniter 的交易方式
======================================

CodeIgniter 使用常見的資料庫類別 ADODB ，它的處理過程非常相似於交易的處理方式。因為它將運作交易的過程簡單化了許多，所以我們選擇這個方式進行處理。在大多數的情況下，你只需要幾行程式碼就能完成交易。

一般而言，交易需要執行大量的工作，因為它需要你追蹤你的查詢，並根據查詢的成功或失敗來決定要提交或退回。相反地，我們實現了一個聰明的交易系統，該系統可以自動地完成所有操作（你也可以手動管理你的交易，但實際上並沒有比較好）。

執行交易
====================

要使用交易執行查詢，你需要使用 ``$this->db->transStart()`` 和 ``$this->db->transComplete()`` 兩個函數，參考以下範例：

::

	$this->db->transStart();
	$this->db->query('AN SQL QUERY...');
	$this->db->query('ANOTHER QUERY...');
	$this->db->query('AND YET ANOTHER QUERY...');
	$this->db->transComplete();

你可以在開始跟完成函數中執行任意數量的查詢，它們會根據你查詢的成功或失敗來提交或退回。

嚴格模式
===========

在預設情況下， CodeIgniter 執行所有的交易都是嚴格模式。在嚴格模式下，當你執行多個群組的交易，如果一個群組失敗或錯誤，則全部群組都會退回。如果關閉嚴格模式，每個群組都是獨立的，代表一個群組發生失敗或錯誤並不會影響其他的群組。

參考以下範例關閉嚴格模式：

::

	$this->db->transStrict(false);

錯誤處理
===============

如果你在 Config/Database.php 中開啟錯誤報告，當你的提交失敗時，你將會看見標準的錯誤訊息。如果除錯模式關閉，則可以管理你的錯誤，如下面的範例：

::

	$this->db->transStart();
	$this->db->query('AN SQL QUERY...');
	$this->db->query('ANOTHER QUERY...');
	$this->db->transComplete();

    if ($this->db->transStatus() === false) {
        // generate an error... or use the log_message() function to log your error
    }

禁用交易
======================

在預設情況下，交易是開啟的。如果你需要關閉交易，你可以使用 ``$this->db->transOff()`` 來關閉：

::

	$this->db->transOff();

	$this->db->transStart();
	$this->db->query('AN SQL QUERY...');
	$this->db->transComplete();

當交易被關閉時，你的查詢會自動被提交，就像沒有使用交易時的查詢一樣。

測試模式
=========

你可以選擇將交易系統轉換成 "測試模式" ，即使查詢有結果，它也會將查詢退回。
要使用測試模式，你只需要在 ``$this->db->transStart()`` 的第一個參數中傳入 TRUE：

::

	$this->db->transStart(true); // 查詢將會退回
	$this->db->query('AN SQL QUERY...');
	$this->db->transComplete();

手動執行交易
=============================

如果你想要手動執行交易，參考以下範例：

::

	$this->db->transBegin();

    $this->db->query('AN SQL QUERY...');
    $this->db->query('ANOTHER QUERY...');
    $this->db->query('AND YET ANOTHER QUERY...');

    if ($this->db->transStatus() === false) {
        $this->db->transRollback();
    } else {
        $this->db->transCommit();
    }


.. note:: 手動執行交易，請確認是使用 ``$this->db->transBegin()``，而 **不是** ``$this->db->transStart()`` 。
