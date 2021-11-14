###################
自訂命令列介面指令
###################

雖然像其他路由一樣使用命令列指令很方便，但你可以會發現有的時候你需要一些不同的東西，命令列指令說不定就可以幫助到你。它們是一些簡單的類別，不需要定義路由，這使得它們非常適合用於建構工具。開發者可以透過它處理資料庫遷移、資料庫填充、檢查排程工作狀態，甚至是替公司建構出自訂的程式碼生成器來簡化工作。

.. contents::
    :local:
    :depth: 2

****************
運作指令
****************

定位在根目錄下（就是擁有 **/app** 與 **/system** 的那個目錄），指令就能在命令列中運作。 CodeIgniter 已經提供了一個內建腳本 **spark** ，可以用於執行命令列指令： 

::

    > php spark

當你不指定任何指令時，會顯示一個簡單的提示資訊，並提供一個可用的指令列表，你應該以指令的名稱作為第一參數來運作你想使用的指令：

::

    > php spark migrate

有些時候指令需要額外的參數，這些參數應該直接在指令後用空格隔開：

::

    > php spark db:seed DevUserSeeder

You may always pass ``--no-header`` to suppress the header output, helpful for parsing results::

    > php spark cache:clear --no-header

對於 CodeIgniter 所提供的指令，如果你沒有鍵入它所需要的參數，你將會被提示正確運作指令應該需要的要求：

::

    > php spark migrate:version
    > Version?

Calling Commands
================

Commands can also be ran from within your own code. This is most often done within a controller for cronjob tasks,
but they can be used at any time. You do this by using the ``command()`` function. This function is always available.

::

    echo command('migrate:create TestMigration');

The only argument is string that is the command called and any parameters. This appears exactly as you would call
it from the command line.

All output from the command that is ran is captured when not run from the command line. It is returned from the command
so that you can choose to display it or not.

******************
使用提示指令
******************

你可以使用下列的提示指令來獲得在命令列介面中任何指令的提示資訊：

::

    > php spark help db:seed

Use the **list** command to get a list of available commands and their descriptions, sorted by categories.
You may also use ``spark list --simple`` to get a raw list of all available commands, sorted alphabetically.

*********************
建立新的指令
*********************

若是你在開發中需要，那麼建立新的指令真的非常容易。每個新的指令都必須繼承 ``CodeIgniter\CLI\BaseCommand`` 類別，並且實作 ``run()`` 方法。

你應該要在你的新指令類別檔案中實現下列屬性，以及這些指令的使用方式，這樣它就能在命令列指令列表中出現：

* ``$group``: 字串，用來描述該指令在指令表中被歸類的群組。例如：資料庫 ``Database`` 。
* ``$name``:  描述指令名稱的字串，例如： ``migrate:create`` 。
* ``$description``: 簡單描述指令效果的字串，例如：建立一個新的遷移檔案。
* ``$usage``: 描述指令使用方式的字串，例如： ``migrate:create [migration_name] [Options]`` 。
* ``$arguments``: 描述每個指令參數的鍵值陣列，例如：``'name' => 'The migration file name'`` 。
* ``$options``: 描述每個指令選項的鍵值陣列，例如：``'-n' => 'Set migration namespace'``

**提示列表或提示指令所顯示的內容將會從上述的屬性中產出。**

檔案位置
=============

你的指令檔案必須儲存在一個名為 **Commands** 的資料夾下，但是這個資料夾可以位於 :doc:`自動載入器 </concepts/autoloader>` 可以找的到的任何地方。這個目錄可以位於 **/app/Commands** ，也位於專案開發資料夾中你所自訂的目錄，比如說： **Acme/Commands** 。

.. note:: 當指令被執行時，完整的 CodeIgniter 命令列環境將被載入，這樣就可以獲得環境訊息、路徑訊息，以及任何在製作控制器時你所使用的工具。

範例指令
==================

讓我們透過一個示範指令來演示功能。這個指令唯一的作用是產出應用程式本身的基本資訊報告。首先，先至 **/app/Commands/AppInfo.php** 創建一個含有以下程式碼的檔案：

::

    <?php

    namespace App\Commands;

    use CodeIgniter\CLI\BaseCommand;
    use CodeIgniter\CLI\CLI;

    class AppInfo extends BaseCommand
    {
        protected $group       = 'demo';
        protected $name        = 'app:info';
        protected $description = 'Displays basic application information.';

        public function run(array $params)
        {
            // ...
        }
    }

如果以下了 **list** 指令，你會看到新的指令已經被列在 ``demo`` 群組下，仔細地瞧瞧，你應該會發現這十分容易。 ``$group`` 屬性的作用是將這個指令與其它指令組織起來，告訴你它應該被列在哪個群組底下。

而 ``$name`` 則是這個指令可以被呼叫的名稱，唯一的要求就是不可以含有空格，並且所有字元必須在命令列中有效。不過，按照慣例，指令應該都要是小寫的，在指令名稱中使用冒號進行更進一步的分組，這有助於防止指令名稱的衝突 。

最後的屬性 ``$description`` 則是一個簡單的字串，將在指令 **列表** 中顯示，它應該要能好好地描述指令的效果。

run()
-----

``run()`` 方法是運作指令時會呼叫的方法，``$params`` 陣列是指令名稱後可以接著使用的參數列表：

::

    > php spark foo bar baz

那麼 **foo** 就是指令的名稱， ``$params`` 陣列則是：

::

    $params = ['bar', 'baz'];

你也可以透過 :doc:`命令列程式庫 </cli/cli_library>` 存取，但 $params 已經替你從使用者輸入的字串中提煉出了已經定義好的參數，你可以透過 $params 中記錄的參數自訂腳本的行為。

我們的演示用指令有一個 ``run`` 方法，就像這樣：

::

    public function run(array $params)
    {
        CLI::write('PHP Version: '. CLI::color(phpversion(), 'yellow'));
        CLI::write('CI Version: '. CLI::color(\CodeIgniter\CodeIgniter::CI_VERSION, 'yellow'));
        CLI::write('APPPATH: '. CLI::color(APPPATH, 'yellow'));
        CLI::write('SYSTEMPATH: '. CLI::color(SYSTEMPATH, 'yellow'));
        CLI::write('ROOTPATH: '. CLI::color(ROOTPATH, 'yellow'));
        CLI::write('Included files: '. CLI::color(count(get_included_files()), 'yellow'));
    }

*****************
BaseCommand 類別
*****************

所有的指令都必須繼承 ``BaseCommand`` 類別，這個類別有幾個食用的方法，在創建自己的指令時你應該要熟悉這些方法。這個類別也有一個 :doc:`日誌記錄器 </general/logging>` ，你可以透過 **$this->logger** 呼叫它。

.. php:class:: CodeIgniter\\CLI\\BaseCommand

    .. php:method:: call(string $command[, array $params=[] ])

        :param string $command: 另一個要呼叫的指令名稱
        :param array $params: 向這個指令提供額外的參數。

        這個方法允許你在執行當前指令時呼叫其他指令：

        ::

        $this->call('command_one');
        $this->call('command_two', $params);

    .. php:method:: showError(\Exception $e)

        :param Exception $e: 用於錯誤報告的例外拋出。

        一種便捷的方法，保值一致且清晰的錯誤輸出給命令列介面：

        ::

            try {
                . . .
            } catch (\Exception $e) {
                $this->showError($e);
            }
    .. php:method:: showHelp()

        一個顯示指令提示的方法：（用法、參數、描述，和選項）

    .. php:method:: getPad($array, $pad)

        :param array    $array: 鍵值陣列
        :param integer  $pad: 填充空間

        計算鍵值陣列輸出的內距的方法。這個內距可以用來在命令列介面中輸出一個格式化的表格：

        ::

            $pad = $this->getPad($this->options, 6);

            foreach ($this->options as $option => $description) {
                CLI::write($tab . CLI::color(str_pad($option, $pad), 'green') . $description, 'yellow');
            }

            // 輸出就像這樣
            -n                  Set migration namespace
            -r                  override file
