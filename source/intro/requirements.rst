###################
系統需求
###################

需要至少 `PHP <http://php.net/>`_ 7.2 或更高的版本。 並且安裝了 
`*intl* 擴充模組 <http://php.net/manual/en/intl.requirements.php>`_
。

你的PHP伺服器必須啟用下列擴充模組： ``php-json`` 、 ``php-mbstring`` 、 ``php-mysqlnd`` 、 ``php-xml``。

若你需要使用 :doc:`CURLRequest </libraries/curlrequest>` 程式庫，你可能需要安裝 `libcurl <http://php.net/manual/en/curl.requirements.php>`_ 。

大部分的Web應用程式都需要資料庫，目前支援的資料庫如下：

  - MySQL (5.1+) 通過 *MySQLi* 驅動。
  - PostgreSQL 通過 *Postgre* 驅動。
  - SQLite3 通過 *SQLite3* 驅動。

並不是所有的驅動都在CodeIgniter4進行了轉換或重構。以下將列出特別的項目：

  - MySQL (5.1+) 通過 *pdo* 驅動。
  - Oracle 通過 *oci8* 以及 *pdo* 驅動。
  - PostgreSQL 通過　*pdo* 驅動。
  - MS SQL 通過 *mssql*、*sqlsrv* (適用於　2005 或更高的版本)，以及 *pdo* 驅動。
  - SQLite 通過 *sqlite* (version 2) 以及 *pdo* 驅動。
  - CUBRID 通過 *cubrid* 以及 *pdo* 驅動。
  - Interbase/Firebird 通過 *ibase* 以及 *pdo* 驅動。
  - ODBC 通過 *odbc* 以及 *pdo* 驅動 （您必須知道ODBC實際上是一個抽象層）。

