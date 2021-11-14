################
資料庫填充 
################

資料庫填充是一種將現有資料添加到資料庫的簡單方法，它在開發的過程中非常有用。當你需要以測試資料填充資料庫，並以這個資料作為開發依據時，此功能極為有效。但它的效果不亞於此，資料庫填充可以包含一些你不想包含在資料遷移中的靜態資料，例如：國家、地理編碼表、事件，或設定訊息等等。

資料庫填充是個簡單的類別，必須有一個 **run()** 方法，並繼承 ``CodeIgniter\Database\Seeder`` 。在 **run()** 方法中，這個類別可以創建各種形式或你所需要的資料。你可以透過 ``$this->db`` 與 ``$this->forge`` 分別存取資料庫和連結資料庫建構。資料庫填充檔案必須儲存在 **app/Database/Seeds** 目錄底下，文件名稱需與類別名稱相同。

::

        <?php namespace App\Database\Seeds;

    class SimpleSeeder extends \CodeIgniter\Database\Seeder
    {
        public function run()
        {
            $data = [
                'username' => 'darth',
                'email'    => 'darth@theempire.com'
            ];

            // 簡單查詢
            $this->db->query("INSERT INTO users (username, email) VALUES(:username:, :email:)", $data);

            // 使用查詢生成器
            $this->db->table('users')->insert($data);
        }
    }

巢套填充器
===============

資料填充器可以使用 **call()** 方法互相呼叫，這樣就能輕鬆建立一個執行中心且任務各自獨立的填充器組織。

::

    namespace App\Database\Seeds;

    use CodeIgniter\Database\Seeder;

    class TestSeeder extends Seeder
    {
        public function run()
        {
            $this->call('UserSeeder');
            $this->call('CountrySeeder');
            $this->call('JobSeeder');
        }
    }

你還可以在透過在 **call()** 方法中使用完整的命名空間類別名稱，這樣你就可以利用自動載入器找到任何地方的資料填充器，這對於多模組化的程式專案是一大福音。

::

    public function run()
    {
        $this->call('UserSeeder');
        $this->call('My\Database\Seeds\CountrySeeder');
    }

Using Faker
===========

If you want to automate the generation of seed data, you can use
the `Faker library <https://github.com/fakerphp/faker>`_.

To install Faker into your project::

    > composer require --dev fakerphp/faker

After installation, an instance of ``Faker\Generator`` is available in the main ``Seeder``
class and is accessible by all child seeders. You must use the static method ``faker()``
to access the instance.

::

    <?php

    namespace App\Database\Seeds;

    use CodeIgniter\Database\Seeder;

    class UserSeeder extends Seeder
    {
        public function run()
        {
            $model = model('UserModel');

            $model->insert([
                'email'      => static::faker()->email,
                'ip_address' => static::faker()->ipv4,
            ]);
        }
    }

使用填充器
=============

你可以透過資料庫設定類別取得資料填充器的實體：

::

    $seeder = \Config\Database::seeder();
    $seeder->call('TestSeeder');

以命令列執行填充
--------------------

如果你不想建立一個專門的控制器處理資料填充的需求，也可以使用命令列執行資料填充，它是 CLI 中遷移工具的一部份：

::

    > php spark db:seed TestSeeder

Creating Seed Files
-------------------

Using the command line, you can easily generate seed files.

::

    > php spark make:seeder user --suffix
    // Output: UserSeeder.php file located at app/Database/Seeds directory.

You can supply the **root** namespace where the seed file will be stored by supplying the ``--namespace`` option::

    > php spark make:seeder MySeeder --namespace Acme\Blog

If ``Acme\Blog`` is mapped to ``app/Blog`` directory, then this command will generate ``MySeeder.php`` at ``app/Blog/Database/Seeds`` directory.

Supplying the ``--force`` option will overwrite existing files in destination.