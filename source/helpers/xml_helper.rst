#############
XML 輔助函數
#############

XML 輔助函數檔案包含協助與 XML 資料互動的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

載入此輔助函數
===================

此輔助函數可以利用以下的程式碼載入：

::

	helper('xml');

可用的功能
===================

以下敘述的功能皆可用：

.. php:function:: xml_convert($str[, $protect_all = FALSE])

	:param string $str: 要轉換的文字字串
	:param bool $protect_all: 是否保留所有看起來像是潛在實體而非只是數值化的實體的內容，例如 &foo;
	:returns: XML 轉換後的字串
	:rtype:	string

	獲取一個字串作為輸入並轉換以下列出的保留 XML 字元為實體：

	  - 連字號: &
	  - 小於與大於符號：< >
	  - 單個、兩個的引號：' "
	  - 破折號：-

	如果連字號是存在的數字實體，此功能就會忽略連字號， 像是：&#123;。 例如::

		$string = '<p>Here is a paragraph & an entity (&#123;).</p>';
		$string = xml_convert($string);
		echo $string;

	輸出:

	.. code-block:: html

		&lt;p&gt;Here is a paragraph &amp; an entity (&#123;).&lt;/p&gt;
