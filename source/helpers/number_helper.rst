################
Number 輔助函數
################

Number 輔助函數包含可以幫助你以語言環境感知方式處理數據資料的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

載入此輔助函數
===================

此輔助函數可以利用以下的程式碼載入：

::

	helper('number');

當事情不太對勁的時候
====================

當 PHP 的全域化與本地化的邏輯無法針對給定的語言環境與選項處理你提供的值，它將會拋出 ``BadFunctionCallException()``。

可用的功能
===================

以下敘述的功能皆可用：

.. php:function:: number_to_size($num[, $precision = 1[, $locale = null])

    :param	mixed	$num: byte 數量
    :param	int	$precision: 浮點數精度
    :returns:	格式化的數據大小字符串，如果提供的值不是數字，則為false
    :rtype:	string

    基於檔案大小將數字格式化為 byte 並加上合適的後綴。例如::

        echo number_to_size(456); // 回傳 456 Bytes
        echo number_to_size(4567); // 回傳 4.5 KB
        echo number_to_size(45678); // 回傳 44.6 KB
        echo number_to_size(456789); // 回傳 447.8 KB
        echo number_to_size(3456789); // 回傳 3.3 MB
        echo number_to_size(12345678912345); // 回傳 1.8 GB
        echo number_to_size(123456789123456789); // 回傳 11,228.3 TB

    選填的第二個參數讓你可以設定結果的精度::

	    echo number_to_size(45678, 2); // 回傳 44.61 KB

    選填的第三個參數讓你可以指定在產生數字時應該被使用的地區，並且可以影響他的格式化
    如果沒有指定地區，該 Request 將會被解析並從 headers 或是根據 app-default 中找尋一個合適的地區::

        // 產生 11.2 TB
        echo number_to_size(12345678912345, 1, 'en_US');
        // 產生 11,2 TB
        echo number_to_size(12345678912345, 1, 'fr_FR');

    .. note:: 由此功能產生的文字語言可以在 *language/<your_lang>/Number.php* 中找到。

.. php:function:: number_to_amount($num[, $precision = 1[, $locale = null])

    :param	mixed	$num: 欲格式化的數字
    :param	int	$precision: 浮點數的精度
    :param  string $locale: 要用來格式化的區域
    :returns:	一個人類可讀的字串版本，如果提供的值不是數字將回傳 false。
    :rtype:	string

    將數字轉換為人能理解的版本，像是 **123.4 trillion**
    數字最大可以到四千萬。例如::

        echo number_to_amount(123456); // 回傳 123 thousand
        echo number_to_amount(123456789); // 回傳 123 million
        echo number_to_amount(1234567890123, 2); // 回傳 1.23 trillion
        echo number_to_amount('123,456,789,012', 2); // 回傳 123.46 billion

    選填的第二個參數允許你設定結果的精度::

        echo number_to_amount(45678, 2); // 回傳 45.68 thousand

    選填的第三個參數允許指定地區::

        echo number_to_amount('123,456,789,012', 2, 'de_DE'); // 回傳 123,46 billion

.. php:function:: number_to_currency($num, $currency[, $locale = null])

    :param mixed $num: 欲格式化的數字
    :param string $currency: 貨幣種類，諸如 USD，EUR
    :param string $locale: 用來格式化的地區
    :param integer $fraction: 小數點後的位數
    :returns: 該數字做為設置區域貨幣的合適價錢
    :rtype: string

    將數字轉換為常見的貨幣形式，諸如 USD，EUR，GBP::

        echo number_to_currency(1234.56, 'USD');  // 回傳 $1,234.56
        echo number_to_currency(1234.56, 'EUR');  // 回傳 €1,234.56
        echo number_to_currency(1234.56, 'GBP');  // 回傳 £1,234.56
        echo number_to_currency(1234.56, 'YEN');  // 回傳 YEN1,234.56

.. php:function:: number_to_roman($num)

    :param string $num: 欲轉換的數字
    :returns: 從給定參數轉換的羅馬數字
    :rtype: string|null

    將一個數字轉換為羅馬數字::

        echo number_to_roman(23);  // 回傳 XXIII
        echo number_to_roman(324);  // 回傳 CCCXXIV
        echo number_to_roman(2534);  // 回傳 MMDXXXIV

    此功能處理從 1 到 3999 的數字。
    任何超出範圍的數值傳入它將會回傳 null 。