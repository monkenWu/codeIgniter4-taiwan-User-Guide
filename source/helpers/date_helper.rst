#############
Date 輔助函式
#############

Date 輔助函式檔案包含協助與日期運作的功能

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

讀取此輔助函式
===================

此輔助函式可以利用以下的程式碼讀取::

	helper('date');

可用的功能
===================

以下敘述的功能皆可用：

.. php:function:: now([$timezone = NULL])

	:param	string	$timezone: 時區
	:returns:	UNIX 時間戳
	:rtype:	int

	回傳當前時間做為一個 UNIX 時間戳，基於你的 config 檔案中所設定的「參考時間」，
	引用到伺服器的本地時間或任何 PHP 所支援的時區。
	如果你無意於設定你的 master 時間為任何其他 PHP 所支援的時區（如果你運行一個網站，你通常會允許每個使用者設定自己的時區）
	在 PHP 上使用 ``time()`` 功能並沒有好處。::

		echo now('Australia/Victoria');

	如果時間戳並不被支援，它將會根據 **time_reference** 設定回傳 ``time()`` 。

.. php:function:: timezone_select([$class = '', $default = '', $what = \DateTimeZone::ALL, $country = null])

	:param	string	$class: 可選擇是否要應用於選取範圍的類別
	:param	string	$default: 最初選擇的預設值
	:param	int	$what: DateTimeZone 類別常數 (請參考 `listIdentifiers <https://www.php.net/manual/en/datetimezone.listidentifiers.php>`_)
	:param	string	$country: 一個兩個字的 ISO 3166-1 相容的國家 / 地區代碼 (請參考 `listIdentifiers <https://www.php.net/manual/en/datetimezone.listidentifiers.php>`_)
	:returns:	預先格式化的 HTML 選擇欄位
	:rtype:	string

	產生一個 `選擇` 可用的時區的表單欄位 （可以根據你的選擇透過 `$what` 與 `$country` 過濾結果）
	你可以提供一個選項類別應用在欄位上以利簡化格式，還可以提供一個預設選取的值。
	::

		echo timezone_select('custom-select', 'America/New_York');

	許多先前在 CodeIgniter 3 內的 ``date_helper`` 許多功能已經在 CodeIgniter 4 中被移動到 ``I18n`` 模塊
