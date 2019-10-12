Composer 安裝
###############################################################################

.. contents::
    :local:
    :depth: 1

你可以利用多種方式透過 Composer 安裝 CodeIgniter4 。

前兩種方式將說明如何以建立新專案為目標安裝 CodeIgniter4 ；第三種方式將告訴你如何把 CodeIgniter4 添加到現有的網頁應用程式之中。

**Note** ：如果你是用 Git 進行專案管理並且與他人共同開發，那麼 git ignored 將會自動忽略 ``vendor`` 資料夾，你必須要在 clone 你的儲存庫後執行 ``composer update`` ，將專案所依賴的程式庫下載至本地開發環境之中。

安裝穩定版本
============================================================

在 `Git 儲存庫 <https://github.com/codeigniter4/appstarter>`_ 中的 CodeIgniter 4 穩定版本將包含一個基本的框架與程式（預設歡迎畫面），Composer 將依賴 Git 儲存庫所發布的最新版本。

若你希望將 CodeIgniter4 使用在全新的產品開發專案中，你將適合這種安裝方式。

安裝與設定
-------------------------------------------------------

在專案的根目錄中執行以下指令： ::

    composer create-project codeigniter4/appstarter project-root -s rc

執行上述指令將會在目錄下新增一個「 project-root 」 資料夾。

如果你省略了上述指令中「 project-root 」參數，那麼 Composer 將會自動生成出「 appstarter 」資料夾，你可以依照自己的需求命名你的資料夾。

若是你不需要安裝 phpunit 及其所有的 Composer 依賴（ dependencies ），則必須在上述的指令中添加「 –no-dev 」。執行了這個指令， Composer 將會安裝單純的框架檔案，以及我們預先綑綁的三個所需的依賴程式庫。

下例將使用默認的專案資料夾名稱「 appstarter 」演示上述需求的安裝指令： ::

    composer create-project codeigniter4/appstarter -s rc --no-dev

往後若需要升及框架，你只需要按照下一小節「如何升級框架」來進行升級。

如何升級框架
-------------------------------------------------------

每當有新版本的 codeigniter4 發布時，請從專案的根目錄執行以下指令： ::

    composer update 

如果你在創建專案時使用了「 --no-dev 」，那麼在升級框架時你也必須做一樣的事：  ``composer update --no-dev`` 。

最後，你必須閱讀新版本的升級說明，並檢查 ``app/Config`` 資料夾中的文件是否因為升級有所改變。

優點
-------------------------------------------------------

安裝簡單；容易更新。

缺點
-------------------------------------------------------

在自動更新後，仍需檢查 ``app/Config`` 是否有因為升級而有所改變。

結構
-------------------------------------------------------

在安裝 CodeIgniter 4 後，這些資料夾將會部署進你的專案中：

- app, public, tests, writable 
- vendor/codeigniter4/framework/system
- vendor/codeigniter4/framework/app & public (更新後可以利用這裡比較 ``app/Config`` 是否有所不同。)

安裝開發中版本
============================================================

安裝與設定
-------------------------------------------------------

`CodeIgniter 4 開發中版本 <https://github.com/codeigniter4/devstarter>`_ 將包含一個基本的框架與程式（預設歡迎畫面），但其組成依賴於框架的開發（ develop ）分支，將會是未發布的版本。

這種安裝方式將適合希望把 CodeIgniter4 部屬在全新專案的開發者，你必須注意的是，開發中的版本將會有頻繁的更改或是功能的刪減，也有可能運作著未穩定的程式。

你可以閱讀  `開發中的使用者說明書（原文） <https://codeigniter4.github.io/CodeIgniter4/>`_，這與穩定發布版的使用者說明書不同，它將會明確地記載開發中的功能與使用說明。 

在專案的根目錄中執行以下指令： ::

    composer create-project codeigniter4/devstarter -s dev

透過這行指令的執行 Composer 將會自動生成出「 devstarter 」資料夾，你可以依照自己的需求命名你的資料夾。

你也可以提供自己的專案名稱作為第三個 Composer 參數，如果你不想包含 phpunit 及其所有的 Composer 依賴（ dependencies ），則必須在指令中添加「 –no-dev 」。滿足上述需求，你的指令可能會像下述例子： ::

    composer create-project codeigniter4/devstarter my-awesome-project -s dev --no-dev


如何升級框架
-------------------------------------------------------

每當有新版本的 codeigniter4 發布時，請從專案的根目錄執行以下指令： ::

    composer update 

如果你在創建專案時使用了「 --no-dev 」，那麼在升級框架時你也必須做一樣的事：  ``composer update --no-dev`` 。

請務必檢查「修改記錄檔（ changelog ）」，查看框架是否有任何足以影響你的應用程式的改動，請記住，最新的更改可能未記載於記錄檔之中。

最後，你必須閱讀新版本的升級說明，並檢查 ``app/Config`` 資料夾中的文件是否因為升級有所改變。

優點
-------------------------------------------------------

安裝簡單；容易更新；最新版本。

缺點
-------------------------------------------------------

我們將不保證框架的穩定性，升級後出現任何的問題將是你的責任。

在自動更新後，仍需檢查 ``app/Config`` 是否有因為升級而有所改變。

結構
-------------------------------------------------------

在安裝 CodeIgniter 4 後，這些資料夾將會部署進你的專案中：

- app, public, tests, writable 
- vendor/codeigniter4/framework/system
- vendor/codeigniter4/framework/app & public (更新後可以利用這裡比較 ``app/Config`` 是否有所不同。)

將 CodeIgniter4 部屬到現有專案中
============================================================

與「手動安裝」 `CodeIgniter 4 <https://github.com/codeigniter4/framework>`_ 相同，你也可以透過 Composer 下載最新版本再自行部屬。

你可以在框架中的 ``app`` 資料夾開發你的應用程式，而 ``public`` 資料夾則是放置網站的公開檔案。

在你的專案根目錄執行以下指令： ::

    composer require codeigniter4/framework @rc

與前兩個 Composer 安裝方法相同，你可以通過「 --no-dev 」引數，省略安裝 phpunit 及其 Composer 依賴，

安裝
-------------------------------------------------------

將 app 、 publuc 、 tests 以及 writable 資料夾從 ``vendor/codeigniter4/framework`` 複製到你的專案根目錄中。

並且，也將 ``env`` 、 ``phpunit.xml.dist`` 以及 ``spark`` 檔案，從 ``vendor/codeigniter4/framework`` 複製到你的專案根目錄中。

為了讓框架得以載入正確的 system 資料夾，你必須調整 ``app/Config/Paths.php`` 中 $systemDirectory 變數的路徑字串，指向 ``vendor/codeigniter/framework`` 的相對位置。

如何升級框架
-------------------------------------------------------

每當有新版本的 codeigniter4 發布時，請從專案的根目錄執行以下指令： ::

    composer update 

如果你在創建專案時使用了「 --no-dev 」，那麼在升級框架時你也必須做一樣的事：  ``composer update --no-dev`` 。

最後，你必須閱讀新版本的升級說明，並檢查 ``app/Config`` 資料夾中的文件是否因為升級有所改變。

優點
-------------------------------------------------------

相對簡單的安裝；容易更新。

缺點
-------------------------------------------------------

在自動更新後，仍需檢查 ``app/Config`` 是否有因為升級而有所改變。

結構
-------------------------------------------------------

在執行完上述流程後，這些資料夾將會部署進你的專案中：

- app, public, tests, writable 
- vendor/codeigniter4/framework/system

安裝多語言系統提示
============================================================

如果在開發的過程中你想在查閱繁體中文或非英語語系的系統提示，可以用下列方式將它們加入到你的專案之中。

在你的專案根目錄執行以下指令： ::

    composer require codeigniter4/translations @beta

每當你執行 ``composer update`` 後，多語言的系統提示也將隨著框架一同升級。
