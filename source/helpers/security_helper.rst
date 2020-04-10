##################
Security 輔助函式
##################

Security 輔助函式包含與安全相關的功能。

.. contents::
  :local:

讀取此輔助函式
===================

此輔助函式可以利用以下的程式碼讀取::

	helper('security');

可用的功能
===================

以下敘述的功能皆可用：

.. php:function:: sanitize_filename($filename)

	:param	string	$filename: 檔案名稱
    	:returns:	消毒過的檔案名稱
    	:rtype:	string

    	提供遍歷目錄的保護。

    	此功能是 ``\CodeIgniter\Security::sanitize_filename()`` 的一個別名。
	如果需要更多資訊，請見 :doc:`Security Library <../libraries/security>`。

.. php:function:: strip_image_tags($str)

	:param	string	$str: 輸入字串
    	:returns:	輸入字串不包含 image 標籤
    	:rtype:	string

    	這是一個可以從字串中隔除 image 標籤的安全功能。
    	它會把 image URL 留為普通文字。

    	例如::

		$string = strip_image_tags($string);

.. php:function:: encode_php_tags($str)

	:param	string	$str: 輸入字串
    	:returns:	安全格式化的字串
    	:rtype:	string

    	這是一個可以將 PHP 標籤轉為實體的安全功能。

	例如::

		$string = encode_php_tags($string);
