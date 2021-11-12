#############################
從 3.x 升級至 4.x
#############################

CodeIgniter 4 是一個完全重寫的框架，它不具備向後相容的特性。所以，轉移與重構應用將比升級應用將更加合適。若你完成應用轉移，從任意版本的 CodeIgniter 4 升級到下一個版本的 CodeIgniter 將會非常簡單。

與 CodeIgniter 3 相比，保持了「精益、平衡、簡單」的理念，但實踐上卻存在著很大的區別。

在升級的工作上，並沒有類似於「十二步驟」的檢查表。相反的，你必須利用
:doc:`安裝指引 </installation/index>` 在新的專案資料夾中取得全新的 CodeIgniter 4 ，接著再開始轉換整合你的應用組件，我們將指出你可能會需要考慮的因素。

並不是所有的 CI3 程式庫都移植或重構成 CI4 版本！關於最新的支援清單，你可以參閱 `CodeIgniter 4 路線圖 <https://forum.codeigniter.com/forum-33.html>`_　論壇分類中的 up-to-date 文章。

在開始專案的轉移之前， **請詳細閱讀使用指南** ！ 

**下載**

- CI4 仍然可以下載後直接運行的方式使用，即利用 zip 或 tarball 進行部屬。專案文件中包含完整的使用指南（在 `doc` 資料夾中）。

- 也可以使用 Composer 進行安裝。

**命名空間**

- CI4 是針對 PHP7.2+ 版本所設計的，除了 helpers 以外的所有類別都使用了命名空間的特性。

**Model, View and Controller**

- CodeIgniter is based on the MVC concept. Thus, the changes on the Model, View and Controller
  are one of the most important things you have to handle.
- In CodeIgniter 4, models are now located in ``app/Models`` and you have to add the lines
  ``namespace App\Models;`` along with ``use CodeIgniter\Model;`` right after the opening php tag.
  The last step is to replace ``extends CI_Model`` with ``extends Model``.
- The views of CodeIgniter 4 have been moved ``to app/Views``. Furthermore, you have to change
  the syntax of loading views from ``$this->load->view('directory_name/file_name')`` to
  ``echo view('directory_name/file_name');``.
- Controllers of CodeIgniter 4 have to be moved to ``app/Controllers;``. After that,
  add ``namespace App\Controllers;`` after the opening php tag.
  Lastly, replace ``extends CI_Controller`` with ``extends BaseController``.
- For more information we recommend you the following upgrade guides, which will give
  you some step-by-step instructions to convert the MVC classes in CodeIgniter4:

.. toctree::
    :titlesonly:

    upgrade_models
    upgrade_views
    upgrade_controllers

**類別載入**

- 不再有 CodeIgniter「超級物件」 的存在，框架組件的將會作為屬性，動態載入至控制器之中。

- 類別只在需要它的地方才進行實作，組件統一由 ``Services`` 進行管理。

- 類別載入器將自動以 PSR4 的規則，在 ``App`` （應用程式）和 ``CodeIgniter`` （系統）這兩個頂級命名空間進行類別定位；支援　Composer 自動載入，甚至是類別檔案在沒有宣告命名空間的情形下，也可以透過假設的方式，根據它們位於的文件夾，找到正確的模型與程式庫。

- 為了支援你熟悉的應用程式結構，你可以透過設定類別載入器來達成。例如使用：「 `HMVC <https://zh.wikipedia.org/wiki/HMVC>`_ 」風格。

**程式庫**

- 你的應用程式類別依然可以載入在 ``app/Libraries`` 之中的類別，但這些類別也可以儲存在別處。

- CI3 的 ``$this->load->library(x);`` 已棄用。現在，在你所想要使用的組件內，你可以按照命名空間的定義，使用 ``$this->x = new X();`` 實作你想使用的程式庫類別。

**輔助函數**

- 輔助函數與以舊版本相比有些簡化但大致相同。

**事件**

- 鉤子（ Hooks ）現在已被事件（Events）替代。

- CI3 的 ``$hook['post_controller_constructor']`` 已被棄用。現在，你可以使用命名空間下的 ``CodeIgniter\Events\Events;`` 的 ``Events::on('post_controller_constructor', ['MyClass', 'MyFunction']);`` 靜態方法達成一樣的效果。

- 事件功能始終啟用，並且全域可用。

**擴充框架**

- 在擴充、更換核心類別上，你不再需要使用 ``core`` 資料夾保存諸如 ``MY_...`` 的檔案。

- 在程式庫資料夾下，某些類別用於繼承或取代框架中原有的功能。現在，你不再需要以 ``MY_x`` 來命名。

- 你可以在任何需要的地方創建這個類別，只需要在 ``app/Config/Services.php`` 中添加合適服務方法（ service methods ），來替換掉默認組件。

Upgrading Libraries
===================

- Your app classes can still go inside ``app/Libraries``, but they don’t have to.
- Instead of CI3’s ``$this->load->library(x);`` you can now use ``$this->x = new X();``,
  following namespaced conventions for your component.
- Some libraries from CodeIgniter 3 no longer exists in Version 4. For all these
  libraries, you have to find a new way to implement your functions. These
  libraries are `Calendaring <http://codeigniter.com/userguide3/libraries/calendar.html>`_,
  `FTP <http://codeigniter.com/userguide3/libraries/ftp.html>`_,
  `Javascript <http://codeigniter.com/userguide3/libraries/javascript.html>`_,
  `Shopping Cart <http://codeigniter.com/userguide3/libraries/cart.html>`_,
  `Trackback <http://codeigniter.com/userguide3/libraries/trackback.html>`_,
  `XML-RPC /-Server <http://codeigniter.com/userguide3/libraries/xmlrpc.html>`_,
  and `Zip Encoding <http://codeigniter.com/userguide3/libraries/zip.html>`_.
- CI3's `Input <http://codeigniter.com/userguide3/libraries/input.html>`_ corresponds to CI4's :doc:`IncomingRequest </incoming/incomingrequest>`.
- CI3's `Output <http://codeigniter.com/userguide3/libraries/output.html>`_ corresponds to CI4's :doc:`Responses </outgoing/response>`.
- All the other libraries, which exist in both CodeIgniter versions, can be upgraded with some adjustments.
  The most important and mostly used libraries received an Upgrade Guide, which will help you with simple
  steps and examples to adjust your code.

.. toctree::
    :titlesonly:

    upgrade_configuration
    upgrade_database
    upgrade_emails
    upgrade_encryption
    upgrade_file_upload
    upgrade_html_tables
    upgrade_localization
    upgrade_migrations
    upgrade_pagination
    upgrade_responses
    upgrade_routing
    upgrade_security
    upgrade_sessions
    upgrade_validations
    upgrade_view_parser

.. note::
    More upgrade guides coming soon
