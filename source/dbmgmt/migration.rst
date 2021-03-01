###################
資料庫遷移
###################

遷移是一種方便的管理模式，它讓你可以有條不紊地改變你地資料庫。你當然可以選擇手0動編輯 SQL 片段，然後告訴開發人員們要記得運作這些片段。但在下一次部屬專案時，說不定你還得追蹤在正式伺服器上是否運作了這些片段。

而資料庫資料表 **遷移** 功能會自動追蹤哪些資料遷移已經成功運作過了，你只要確定你的遷移已經設定完畢，然後呼叫 ``$migration->latest()`` 將資料庫更新到最新的狀態。你也可以使用 ``$migration->setNamespace(null)->progess()`` 來包含所有命名空間中的遷移項目。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

********************
遷移檔案的命名方式
********************

每個遷移都是按照數字順序向前或向後運作，每個遷移都會使用創建遷移時的時間戳進行編號，格式為 **YYYYMMDDHHIISS** （例如： **20121031100537** ）。這有助於防止在團隊開發中發生編號衝突。

遷移檔案的前綴是遷移的編號，在下底後緊接著是具有一定描述性的遷移名稱。年份、月份以及日期可以使用破折號、下底線，或者完全不使用，就像：

* 20121031100537_add_blog.php
* 2012-10-31-100538_alter_blog_track_views.php
* 2012_10_31_100539_alter_blog_add_translations.php


******************
新建一個遷移檔案
******************

這是一個新網站的第一次遷移，它具有一個部落格系統。所有的遷移都在 **app/Database/Migrations/** 目錄下，名稱可能是 *20121031100537_add_blog.php* 。

::

	<?php namespace App\Database\Migrations;

	class AddBlog extends \CodeIgniter\Database\Migration {

		public function up()
		{
			$this->forge->addField([
				'blog_id'          => [
					'type'           => 'INT',
					'constraint'     => 5,
					'unsigned'       => TRUE,
					'auto_increment' => TRUE
				],
				'blog_title'       => [
					'type'           => 'VARCHAR',
					'constraint'     => '100',
				],
				'blog_description' => [
					'type'           => 'TEXT',
					'null'           => TRUE,
				],
			]);
			$this->forge->addKey('blog_id', TRUE);
			$this->forge->createTable('blog');
		}

		public function down()
		{
			$this->forge->dropTable('blog');
		}
	}

資料庫連接以及資料庫建構類別都可以透過 ``$this->db`` 以及 ``$this->forge`` 分別被你取用。

另外，你也可以使用命令列介面來產生一個骨架檔案，更多詳情請繼續閱讀下文。

外來鍵
============

當你的資料表具有外來鍵的特性時，在試圖刪除資料表或資料列等動作，遷移經常會出現問題。要在運作遷移時暫時繞過外來鍵的檢查，請在資料庫連接上使用 ``disableForeignKeyChecks()`` 以及 ``enableForeignKeyChecks()`` 方法。

::

    public function up()
    {
        $this->db->disableForeignKeyChecks();

        // 在這之間寫入你的遷移規則

        $this->db->enableForeignKeyChecks();
    }

資料庫群組
===============

遷移將只針對一個資料庫群組運作，如果你在 **app/Config/Database.php** 中定義了多個群組，那麼它將預設在組態設定檔案中指定的 ``$defaultGroup`` 上運作。或許有時，你需要不同資料庫的不同模式，你擁有一個資料庫用於記錄一般的站點訊息，而另一個資料庫用於任務關鍵資料的存取，你可以透過在遷移時設定 ``$DBGroup`` 來確保遷移只針對正確的資料庫群組運作，當然，這個名稱必須與資料庫群組的名稱完全一致。

::

    <?php namespace App\Database\Migrations;

    class AddBlog extends \CodeIgniter\Database\Migration
    {
        protected $DBGroup = 'alternate_db_group';

        public function up() { . . . }

        public function down() { . . . }
    }

命名空間
==========

資料遷移程式庫會自動掃描你在 **app/Config/Autoload.php** 中指定的所有命名空間，或者是從外部來源（如： Composer ）載入命名空間，它使用 ``$psr4`` 屬性找到相符的命名空間，並且將包括在 **Database/Migrations** 下找到的所有遷移。

每個命名空間都會有屬於自己的版本序列，這將幫助你升級降級每個模組（命名空間），而不影響到其他命名空間。

例如：我們假設自動載入組態設定檔案中定義了以下命名空間。

::

	$psr4 = [
		'App'       => APPPATH,
		'MyCompany' => ROOTPATH.'MyCompany'
	];

這將查找位於 **APPPATH/Database/Migrations** 以及 **ROOTPATH/MyCompany/Database/Migrations** 這兩個路徑的所有遷移，這使得在可重用的模組化程式碼套件中，加入遷移檔案變得更加簡單。

*************
使用範例
*************

在這個範例中，我們在 **app/Controllers/Migrate.php** 放置了一些簡單的程式碼來更新綱目：

::

    <?php namespace App\Controllers;

	class Migrate extends \CodeIgniter\Controller
	{

		public function index()
		{
			$migrate = \Config\Services::migrations();

			try
			{
			  $migrate->latest();
			}
			catch (\Exception $e)
			{
			  // 當例外拋出時可以做點什麼
			}
		}

	}

*******************
命令列工具
*******************

CodeIgniter 內建了幾個 :doc:`命令列指令 </cli/cli_commands>` ，讓你可以透過命令列來進行遷移。這些工具並不是使用遷移這個功能的必要條件，但對於那些擅長使用命令列的人來說，可能會讓遷移變得更加便利。這些工具主要是提供了對 MigrationRunner 類別的存取。

**migrate**

遷移功能可以使用在資料庫群組中。

::

    > php spark migrate

migrate 指令具有以下可用選項：

- ``-g`` 選擇資料庫群組，否則將使用預設資料庫群組
- ``-n`` 選擇命名空間，否則將使用 "App" 命名空間
- ``-all`` 將所有命名空間都升級至最新遷移

這個範例將遷移 BLOG 命名空間和測試資料庫群組上所有的新遷移。

::

    > php spark migrate -g test -n Blog

當使用 ``-all`` 選項，它將掃描所有的命名空間，試圖找到任何尚未運作過的遷移。這些遷移都將被記錄起來，再以創建日期進行排序後執行。這將有助於幫助任何應用程式與模組之間的淺在衝突。

**rollback**

退回所有遷移，將資料庫群組重新開始，設定有效遷移為 0 ：

::

  > php spark migrate:rollback

rollback 指令具有以下可用選項：

- ``-g`` 選擇資料庫群組，否則將使用預設資料庫群組
- ``-b`` 選擇批次，自然數為指定批次，負數為相對批次
- ``-f`` 強行透過確認問題，這只有在上線環境會詢問

**refresh**

更新資料庫狀態，先退回所有遷移然後重新執行遷移：

::


  > php spark migrate:refresh

refresh 指令具有以下可選選項：

- ``-g`` 選擇資料庫群組，否則將使用預設資料庫群組
- ``-n`` 選擇命名空間，否則將使用 "App" 命名空間
- ``-all`` 將所有命名空間都進行更新
- ``-f`` 強行透過確認問題，這只有在上線環境會詢問

**status**

顯示所有遷移列表和運作的日前與時間，如果沒有運作過將會顯示 "--" ：

::

  > php spark migrate:status
  Filename               Migrated On
  First_migration.php    2016-04-25 04:44:22

status 指令具有以下可選選項：

- ``-g`` 選擇資料庫群組，否則將使用預設資料庫群組

**make:migration**

在 **app/Database/Migrations** 中新建一個骨架檔案。它將自動以目前的時間戳命名，它所創建的類別名稱將會是檔案名稱的駝峰式命名版。

::

  > php spark migrate:create [filename]

You can use (make:migration) with the following options:

- ``-n`` - to choose namespace, otherwise the value of ``APP_NAMESPACE`` will be used.
- ``-force`` - If a similarly named migration file is present in destination, this will be overwritten.

*********************
遷移參考
*********************

下面將提到遷移相關的所有設定選項，可以在 **app/Config/Migrations.php** 找到它們。

========================== ====================== ========================== =============================================================
偏好                       預設                   選項                       描述
========================== ====================== ========================== =============================================================
**enabled**                TRUE                   TRUE / FALSE               啟動或關閉遷移
**table**                  migrations             None                       用於儲存綱目的版本號碼的資料表名稱。
**timestampFormat**        Y-m-d-His\_                                       用於創建遷移當下時間戳的格式。
========================== ====================== ========================== =============================================================

***************
類別參考
***************

.. php:class:: CodeIgniter\\Database\\MigrationRunner

	.. php:method:: findMigrations()

		:returns:	陣列或遷移檔案
		:rtype:	array

		回傳 **path** 屬性中找到的遷移檔案名稱陣列。

	.. php:method:: latest($group)

		:param	mixed	$group: 資料庫名稱，如果為 null 則會使用預設資料庫群組
		:returns:	成功為 TRUE ，失敗為 FALSE
		:rtype:	bool

		這將在一個命名空間（或所有的命名空間）中定位遷移，確定那些遷移還沒有被運作過，並按照它們的版本順序運作（不管位於哪個命名空間都將參與排序）。

	.. php:method:: regress($batch, $group)

		:param	mixed	$batch: 要遷移的到的批次，1 或大於 1 為指定批次，0 為恢復所有批次，負數則是指相對批次（例如 -3 為 "退回三個批次" ）
		:param	mixed	$group: 資料庫名稱，如果為 null 則會使用預設資料庫群組
		:returns:	成功為 TRUE ，失敗為 FALSE 或是未發現任何遷移
		:rtype:	bool

		Regress 可以用於逐批退回到以前的狀態。

		::

			$migration->batch(5);
			$migration->batch(-1);

	.. php:method:: force($path, $namespace, $group)

		:param	mixed	$path:  有效的遷移檔案路徑
		:param	mixed	$namespace: 遷移檔案的命名空間
		:param	mixed	$group: 資料庫名稱，如果為 null 則會使用預設資料庫群組
		:returns:	成功為 TRUE ，失敗為 FALSE
		:rtype:	bool
		
		強制將單個檔案進行遷移，不考慮順序或批次處理。根據是否已經遷移的方法， "up" 或 "down" 來檢測。
		
		.. note::  這個方法僅用於測試，可能會發生資料不一致的問題。

	.. php:method:: setNamespace($namespace)

		:param  string  $namespace: 應用程式命名空間
		:returns:   目前的 MigrationRunner 實體
		:rtype:     CodeIgniter\Database\MigrationRunner

		設定程式庫應該尋找的遷移檔案路徑：

		::

			$migration->setNamespace($path)
					->latest();

	.. php:method:: setGroup($group)

		:param  string  $group: 資料庫群組名稱
		:returns:   目前的 MigrationRunner 實體
		:rtype:     CodeIgniter\Database\MigrationRunner

		設定程式庫應該使用的資料庫群組名稱：

		::
		
			$migration->setGroup($group)
					->latest();