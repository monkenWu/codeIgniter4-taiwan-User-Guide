###################
資料庫遷移
###################

遷移是一種方便的管理模式，你可以有條不紊地改變你地資料庫。你當然可以手動編輯 SQL 片段，但你還得告訴開發人員們要記得運作這些片段。再下一次部屬時，你還得追蹤在正式伺服器上需要運作這些片段。

資料庫資料表 **遷移** 會追蹤那些資料遷移已經成功運作過了，你只要確定你的遷移已經到設定完畢，然後呼叫 ``$migration->latest()`` 將資料庫更新到最新的狀態。你也可以使用 ``$migration->setNamespace(null)->progess()`` 來包還所有命名空間中的遷移項目。

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

In this example some simple code is placed in **app/Controllers/Migrate.php**
to update the schema::

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
			  // Do something with the error here...
			}
		}

	}

*******************
命令列工具
*******************
CodeIgniter ships with several :doc:`commands </cli/cli_commands>` that are available from the command line to help
you work with migrations. These tools are not required to use migrations but might make things easier for those of you
that wish to use them. The tools primarily provide access to the same methods that are available within the MigrationRunner class.

**migrate**

Migrates a database group with all available migrations::

    > php spark migrate

You can use (migrate) with the following options:

- (-g) to chose database group, otherwise default database group will be used.
- (-n) to choose namespace, otherwise (App) namespace will be used.
- (-all) to migrate all namespaces to the latest migration

This example will migrate Blog namespace with any new migrations on the test database group::

    > php spark migrate -g test -n Blog

When using the `-all` option, it will scan through all namespaces attempting to find any migrations that have
not been run. These will all be collected and then sorted as a group by date created. This should help
to minimize any potential conflicts between the main application and any modules.

**rollback**

Rolls back all migrations, taking the database group to a blank slate, effectively migration 0::

  > php spark migrate:rollback

You can use (rollback) with the following options:

- (-g) to choose database group, otherwise default database group will be used.
- (-b) to choose a batch: natural numbers specify the batch, negatives indicate a relative batch
- (-f) to force a bypass confirmation question, it is only asked in a production environment

**refresh**

Refreshes the database state by first rolling back all migrations, and then migrating all::

  > php spark migrate:refresh

You can use (refresh) with the following options:

- (-g) to choose database group, otherwise default database group will be used.
- (-n) to choose namespace, otherwise (App) namespace will be used.
- (-all) to refresh all namespaces
- (-f) to force a bypass confirmation question, it is only asked in a production environment

**status**

Displays a list of all migrations and the date and time they ran, or '--' if they have not been run::

  > php spark migrate:status
  Filename               Migrated On
  First_migration.php    2016-04-25 04:44:22

You can use (status) with the following options:

- (-g) to choose database group, otherwise default database group will be used.

**create**

Creates a skeleton migration file in **app/Database/Migrations**.
It automatically prepends the current timestamp. The class name it
creates is the Pascal case version of the filename.

  > php spark migrate:create [filename]


You can use (create) with the following options:

- (-n) to choose namespace, otherwise (App) namespace will be used.

*********************
遷移參考
*********************

The following is a table of all the config options for migrations, available in **app/Config/Migrations.php**.

========================== ====================== ========================== =============================================================
偏好                       預設                   選項                       描述
========================== ====================== ========================== =============================================================
**enabled**                TRUE                   TRUE / FALSE               Enable or disable migrations.
**table**                  migrations             None                       The table name for storing the schema version number.
**timestampFormat**        Y-m-d-His\_                                        The format to use for timestamps when creating a migration.
========================== ====================== ========================== =============================================================

***************
類別參考
***************

.. php:class:: CodeIgniter\\Database\\MigrationRunner

	.. php:method:: findMigrations()

		:returns:	An array of migration files
		:rtype:	array

		An array of migration filenames are returned that are found in the **path** property.

	.. php:method:: latest($group)

		:param	mixed	$group: database group name, if null default database group will be used.
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		This locates migrations for a namespace (or all namespaces), determines which migrations
		have not yet been run, and runs them in order of their version (namespaces intermingled).

	.. php:method:: regress($batch, $group)

		:param	mixed	$batch: previous batch to migrate down to; 1+ specifies the batch, 0 reverts all, negative refers to the relative batch (e.g. -3 means "three batches back")
		:param	mixed	$group: database group name, if null default database group will be used.
		:returns:	TRUE on success, FALSE on failure or no migrations are found
		:rtype:	bool

		Regress can be used to roll back changes to a previous state, batch by batch.
		::

			$migration->batch(5);
			$migration->batch(-1);

	.. php:method:: force($path, $namespace, $group)

		:param	mixed	$path:  path to a valid migration file.
		:param	mixed	$namespace: namespace of the provided migration.
		:param	mixed	$group: database group name, if null default database group will be used.
		:returns:	TRUE on success, FALSE on failure
		:rtype:	bool

		This forces a single file to migrate regardless of order or batches. Method "up" or "down" is detected based on whether it has already been migrated. **Note**: This method is recommended only for testing and could cause data consistency issues.

	.. php:method:: setNamespace($namespace)

	  :param  string  $namespace: application namespace.
	  :returns:   The current MigrationRunner instance
	  :rtype:     CodeIgniter\Database\MigrationRunner

	  Sets the path the library should look for migration files::

	    $migration->setNamespace($path)
	              ->latest();
	.. php:method:: setGroup($group)

	  :param  string  $group: database group name.
	  :returns:   The current MigrationRunner instance
	  :rtype:     CodeIgniter\Database\MigrationRunner

	  Sets the path the library should look for migration files::

	    $migration->setNamespace($path)
	              ->latest();
