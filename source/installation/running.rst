運作你的應用程式
###############################################################################

.. contents::
    :local:
    :depth: 1

CodeIgniter 4 應用程式可透過多種不同的方式運作，託管在 Web 伺服器上，使用虛擬化或 CodeIgniter 命令列工具進行測試。本條目將會介紹如何使用每種技術，並且說明其中的優缺點。

如果你初次使用 CodeIgniter ，請閱讀使用手冊的 :doc:`入門 </intro/index>` 條目，了解如何建構動態的 PHP 應用程式。享受吧！

初始組態與設定
=================================================

#. 使用文字編輯器打開 **app/Config/App.php** 這個檔案並設定基本的 URL 。如果你需要更多的靈活性，可以將 ``.env`` 檔中的 baseURL 設定為  **app.baseURL="http://example.com"** 。

#. 如果需要使用資料庫，請使用文字編輯器打開 **app/Config/Database.php** 這個檔案並設定資料庫。或者可以在 ``.env`` 中進行這些設定。

在正式（ production ）環境中，我們可能會禁用 PHP 錯誤報告或任何僅用於開發中的功能。在 CodeIgniter 中，可以透過設定環境變數來達成，在 :doc:`環境變數 </general/environments>` 條目上有更全面的描述。無設定任何環境變數的應用程式將會直接套用正式環境的相關設定。若需要使用框架提供的除錯工具，應將環境設定為開發 ``ENVIRONMENT`` 。

.. note:: 如果你將在 Web 伺服器（例如：Apache 或 Nginx ）上運作你的網站，則需要修改專案內 ``writable`` 資料夾許可權，讓 Web 伺服器得以使用具有權限的使用者或帳戶進行寫入。

本地開發伺服器
=================================================

CodeIgniter 4 附帶了本地開發伺服器，利用了 PHP 內建的 Web 伺服器和 CodeIgniter 路由。你可以在專案根目錄下透過命令列啟動它::

    php spark serve

這個命令將會啟動伺服器，你現在可以在瀏覽器中前往 http://localhost:8080 查看應用程式。

.. note:: 內建的開發伺服器限用於開發環境，它絕不應該在正式伺服器上使用。

如果需要在伺服器上運作特別的網址名稱，而不是簡單的本機 localhost，需要先將主機添加到 ``hosts`` 檔案中。這個檔案的路徑會根據作業系統而各不相同，所有類 Unix 的系統（包括 OSX ）通常都會將這個檔案放置在 **/etc/hosts** 。
（譯者註： Windows 的 Host 檔案放置在 **C:\\WINDOWS\\system32\\drivers\\etc\\hosts** 之中）

你可以透過三種選項自訂本地開發伺服器：

- 你可以使用 CLI 中的 ``-host`` 選項將應用程式運作在你所設定的 host 上::

    php spark serve -host example.dev

- 在預設的情形下，伺服器將在 8080 埠上運作。但你可能會需要運做多個網站，或者這個埠已經被其他應用程式佔用。這時，可以使用 CLI 中 ``-port`` 選項::

    php spark serve -port 8081

- 你還可以使用 CLI 中的 ``-php`` 選項指定你所要使用的 PHP 版本，這個選項所需的值是 PHP 執行檔的檔案路徑::

    php spark serve -php /usr/bin/php7.6.5.4

部屬在 Apache 伺服器
=================================================

CodeIgniter4 應用程式通常會部屬在網頁伺服器中。通常 Apache的 ``httpd`` 是標準的平台，在手冊中我們預設你使用它。

Apache 與許多平台綁定在一起，你也可以選擇使用 `Bitnami <https://bitnami.com/stacks/infrastructure>`_ 所綑綁的資料庫引擎、 PHP 與 Apache。

.htaccess
-------------------------------------------------------

「 mod_rewrite 」這個模組可以用於隱藏 URL 中的「 index.php 」。

請確認 rewrite 模組在 Apache 設定檔中啟用，例如：``apache2/conf/httpd.conf``::

    LoadModule rewrite_module modules/mod_rewrite.so

同時，你也要確認檔案根目錄的 <Directory> 元素啟用 AllowOverride 功能::

    <Directory "/opt/lamp/apache2/htdocs">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

虛擬伺服器
-------------------------------------------------------

我們推薦你在「虛擬伺服器」上運作你的應用程式。你可以為每個應用程式設定不同的別名。

請確認 虛擬伺服器（ virtual hosting ）模組在 Apache 設定檔中啟用，例如：``apache2/conf/httpd.conf``::

    LoadModule vhost_alias_module modules/mod_vhost_alias.so

在類 Unix 平台或 Windows 主機上的 host 檔案中添加主機別名，通常在 Unix 平台為 ``/etc/hosts`` Windows 平台則為 ``c:/Windows/System32/drivers/etc/hosts``，別名可以命名為 myproject.local 或 myproject.test 或任何自訂的別名

::

    127.0.0.1 myproject.local

在虛擬伺服器設定中幫應用程式新增  <VirtualHost> 元素。這個設定檔通常在 ``apache2/conf/extra/httpd-vhost.conf``::

    <VirtualHost *:80>
        DocumentRoot "/opt/lamp/apache2/htdocs/myproject/public"
        ServerName myproject.local
        ErrorLog "logs/myproject-error_log"
        CustomLog "logs/myproject-access_log" common
    </VirtualHost>

如果你的專案資料夾不在 Apache 文件根目錄，則 <VirtualHost> 需要再新增一層 <Directory> 元素來授權網頁伺服器對檔案的訪問。

測試
-------------------------------------------------------

完成了上述的設定後，你的網頁應用程式將可以透過 ``http://myproject.local`` 在瀏覽器中進行訪問。

每當改變設定的內容時，都需要重新啟動 Apache。

Hosting with Nginx
=================================================
Nginx is the second most widely used HTTP server for web hosting.
Here you can find an example configuration using PHP 7.3 FPM (unix sockets) under Ubuntu Server.

This configuration enables URLs without “index.php” in them and using CodeIgniter's “404 - File Not Found” for URLs ending with “.php”.

.. code-block:: nginx

    server {
        listen 80;
        listen [::]:80;

        server_name example.com;

        root  /var/www/example.com/public;
        index index.php index.html index.htm;

        location / {
            try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;

            # With php-fpm:
            fastcgi_pass unix:/run/php/php7.3-fpm.sock;
            # With php-cgi:
            # fastcgi_pass 127.0.0.1:9000;
        }

        error_page 404 /index.php;

        # deny access to hidden files such as .htaccess
        location ~ /\. {
            deny all;
        }
    }

使用 Vagrant 進行部屬
=================================================

即便你在不同的環境中進行開發，利用虛擬化技術在預計部屬的環境中，對網頁應用程式進行測試是個不錯的方法。

程式庫中附帶一個 ``VagrantFile.dist`` 設定檔，可以複製到 ``VagrantFile`` 並針對該系統進行訂製，例如：啟用對特定資料庫或快取引擎的訪問。

設定
-------------------------------------------------------

我們假設你已經為你的平台安裝了 `VirtualBox <https://www.virtualbox.org/wiki/Downloads>`_ 與  
`Vagrant <https://www.vagrantup.com/downloads.html>`_ 。

Vagrant 設定檔假設你的系統上設定了 `ubuntu/bionic64 Vagrant box 
<https://app.vagrantup.com/ubuntu/boxes/bionic64>`_ ::

    vagrant box add ubuntu/bionic64

測試
-------------------------------------------------------

設定完成後，你可以使用以下命令在 VM 中啟動網頁應用程式::

    vagrant up

你的網頁應用程式將可以在 ``http://localhost:8080`` 造訪，程式碼覆蓋率報告將位於 ``http://localhost:8081`` ，使用指南則位於 ``http://localhost:8082`` 。

Bootstrapping the App
=================================================

In some scenarios you will want to load the framework without actually running the whole
application. This is particularly useful for unit testing your project, but may also be
handy for using third-party tools to analyze and modify your code. The framework comes
with a separate bootstrap script specifically for this scenario: ``system/Test/bootstrap.php``.

Most of the paths to your project are defined during the bootstrap process. You may use
pre-defined constants to override these, but when using the defaults be sure that your
paths align with the expected directory structure for your installation method.
