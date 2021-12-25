##############
CLI 產生器
##############

CodeIgniter4 現在內建了產生器，以方便建立控制器、模型、實體等樣板檔案。你也可以只用一個指令就能搭建出一套完整的檔案。

.. contents::
    :local:
    :depth: 2

************
簡介
************

當使用 ``php spark list`` 列出指令時，所有內建的產生器都位於 ``Generators`` 的命名空間中。要查看某個特定生成器的完整描述和使用資訊，請使用以下指令。

::

    > php spark help <generator_command>

你得把 ``<generator_command>`` 替換成要檢查的指令。

*******************
內建產生器
*******************

CodeIgniter4 預設提供了下列產生器。

make:command
------------

建立一個新的 spark 指令。

使用方式:
===========
::

    make:command <name> [選項]

參數:
=========
* ``name``: 指令類別的類別名稱 **[必須]**

選項:
========
* ``--command``: 在 Spark 中執行的指令名稱，預設為： ``command:name`` 。
* ``--group``: 指令的群組／命名空間。基本指令預設為 ``CodeIgniter`` ，產生器指令預設為 ``Generators`` 。
* ``--type``: 指令的類型，是 ``basic`` 指令還是 ``generator`` 指令，預設為 ``basic`` 。
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:config
-----------

建立一個新的設定檔案。

使用方式:
===========
::

    make:config <name> [選項]

參數:
=========
* ``name``: 設定類別的類別名稱 **[必須]**

選項:
========
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:controller
---------------

建立新的控制器檔案。

使用方式:
===========
::

    make:controller <name> [選項]

參數:
=========
* ``name``: 控制器的類別名稱 **[必須]**

選項:
========
* ``--bare``: 繼承 ``CodeIgniter\Controller`` 而不是 ``BaseController`` 。
* ``--restful``: 繼承 RESTful 資源。可選選項為 ``controller`` 與 ``presenter`` 。預設為 ``controller`` 。
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:entity
-----------

建立一個新的實體檔案。

使用方式:
===========
::

    make:entity <name> [選項]

參數:
=========
* ``name``: 實體類別的名稱。 **[必須]**

選項:
========
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:filter
-----------

建立一個過濾器檔案。

使用方式:
===========
::

    make:filter <name> [選項]

參數:
=========
* ``name``: 過濾器類別名稱。 **[必須]**

選項:
========
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:model
----------

新建新的模型檔案。

使用方式:
===========
::

    make:model <name> [選項]

參數:
=========
* ``name``: 模型的類別名稱。 **[必須]**

選項:
========
* ``--dbgroup``: 欲使用的資料庫群組。預設為 ``default`` 。
* ``--return``: 從 ``array`` 、 ``object`` ，以及 ``entity`` 設定回傳類型。預設為 ``array`` 。
* ``--table``: 提供資料表名稱。預設為類別名稱的複數。
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:seeder
-----------

建立一個新的資料填充檔案。

使用方式:
===========
::

    make:seeder <name> [選項]

參數:
=========
* ``name``: 資料填充的類別名稱。 **[必須]**

選項:
========
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:migration
--------------

建立一個新的遷移檔案。

使用方式:
===========
::

    make:migration <name> [選項]

參數:
=========
* ``name``: 遷移檔案的類別名稱。 **[必須]**

選項:
========
* ``--session``: 替資料庫工作階段產生遷移檔案。
* ``--table``: 替資料庫工作階段的資料表設定名稱，預設為 ``ci_sessions`` 。
* ``--dbgroup``: 為資料庫工作階段設定資料庫群組。預設為 ``default`` 群組。
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

make:validation
---------------

建立一個新的驗證檔案。

使用方式:
===========
::

    make:validation <name> [選項]

參數:
=========
* ``name``: 驗證檔案的類別名稱。 **[必須]**

選項:
========
* ``--namespace``: 依照根命名空間，預設為 ``APP_NAMESPACE`` 。
* ``--suffix``: 在產生的類別名稱上加上元件的後綴。
* ``--force``: 使用這個選項將可以強制覆蓋現有檔案。

.. note:: 
    你需要將產生器產出的程式碼放在子資料夾中嗎？打個比方，如果你想要建立一個控制器類別，並把它放在主資料夾 ``Controllers`` 的 ``Admin`` 子資料夾中。這時，你只需要在類別名稱前加上子資料夾，就像這樣： ``php spark make:controller admin/login`` 。那麼，這個指令將在 ``Controllers/Admin`` 子資料夾中建立起 ``Login`` 控制器，命名空間則為 ``App/Controllers/Admin`` 。

.. note:: 
    你的應用程式執行在模組上嗎？程式碼的產生將以根命名空間為預設的 ``APP_NAMESPACE`` 。如果你需要產生程式碼在模組的命名空間下，請在指令中使用 ``--namespace`` 選項，例如： ``php spark make:model blog --namespace Acme\Blog`` 。

.. warning::
    在使用 ``--namespace`` 選項時，請確認你所提供的命名空間是在 ``Config\Autoload`` 檔案中 ``$psr4`` 陣列所定義的有效命名空間。或在 Composer 的自動載入檔案中定義的有效命名空間。否則，程式碼的產生將被中斷。

.. warning:: 
    使用 ``migrate:create`` 來建立遷移檔案的方法即將被棄用，它將在未來的版本移除。 請使用 ``make:migration --session`` 代替它。另外，請使用 ``make:migration --session`` 來代替被棄用的 ``session:migration`` 。

****************************************
一套完整的庫存程式碼鷹架
****************************************

在我們的開發階段中，有時是依照組別來建立功能的，比如說：建立一個 *Admin* 組別。這個組別將會包含它所需要的控制器、模型、遷移檔案，甚至是實體。你可能會因為上述需求而在命令列逐一輸入每個產生器指令，並祈求如果有一個生成器指令來統一指揮它們就好了。

免煩惱！ CodeIgniter4 為此提供了 ``make:scaffold`` 指令，它把控制器、模型、實體、遷移，和資料填充器等產生器指令統一封裝在一塊兒。你只需要提供類別名稱，它會以此命名所有自動產生的類別。另外，每種產生器指令所支援的 **獨有選項** 都能夠被 scaffold 指令自動識別。

在你的終端機執行以下指令：

::

    php spark make:scaffold user

它將建立這些類別：

(1) ``App\Controllers\User``;
(2) ``App\Models\User``;
(3) ``App\Database\Migrations\<some date here>_User``; and
(4) ``App\Database\Seeds\User``.

若你打算在鷹架檔案中同時產生一個 ``Entity`` 類別時，你需在指令中提到 ``--return entity`` ，此時，它將被傳遞給模型產生器。

**************
產生器特性
**************

所有的產生器指令都必須使用 ``GeneratorTrait`` ，它將會提供在程式碼產生上所需的方法。
