##################
加密服務
##################

.. important:: 千萬不要使用這個或其他加密程式庫來做密碼的儲存！密碼必須被雜湊加密，並且你應該透過 PHP 的
	`Password Hashing extension <https://www.php.net/password>`_ 來做這件事。

這個加密服務提供雙向對稱資料加密（秘密金鑰）。這個服務將實體化和／或初始化一個加密一個 **處理程序** 以適應你的參數，如下所述。

加密服務處理程序必須實作出 CodeIgniter 的樣本 ``EncrypterInterface``。使用一個適當的 PHP 延伸密碼或者第三方程式庫可能會需要你的服務器安裝額外軟體和／或可能
需要明確地在你的PHP實體做啟用。

當前支援以下 PHP　擴充模組：

- `OpenSSL <https://www.php.net/openssl>`_

這不是完整的密碼解決方案。如果你需要更多功能，像是：公鑰加密，我們建議你考慮直接使用 OpenSSL 或另一個 `Cryptography Extensions <https://www.php.net/manual/en/refs.crypto.php>`_。
更全面的軟體包，像是 `Halite <https://github.com/paragonie/halite>`_ 
（基於libsodium構建的O-O軟體包）也是另一種可能性。

.. note:: 由於已經移除了對 ``MCrypt`` 擴充模組的支援，所以從從 PHP 7.2 開始不建議使用。


.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

.. _usage:

****************************
使用加密服務
****************************

像是 CodeIgniter 其他服務，他可以透過 ``Config\Services`` 被載入
::

    $encrypter = \Config\Services::encrypter();

假若你已經設置你的開始金鑰 （參考 :ref:`configuration`），加密以及解密資料很簡單 - 將適當的字串傳遞給 ``encrypt()`` 和／或 ``decrypt()`` 方法
::

	$plainText = 'This is a plain-text message!';
	$ciphertext = $encrypter->encrypt($plainText);

	// Outputs: This is a plain-text message!
	echo $encrypter->decrypt($ciphertext);

就這麼簡單！加密程式庫將完成每一個所有必要的工作為整個過程提供開箱及用的加密安全性，你不必擔心。

.. _configuration:

設置程式庫
=======================

上面的範例使用在 ``app/Config/Encryption.php`` 。

這裡只有兩個設定:

======== ===============================================
選項      可能的值 (括號中為默認值)
======== ===============================================
key      Encryption key starter
driver   Preferred handler (OpenSSL)
======== ===============================================

你可以透過傳遞你自己對 ``Services`` 的呼叫來替換設置文件的設置。
這個 ``config`` 變數必須是 `Config\\Encryption` 類別的實體或是 `CodeIgniter\\Config\\BaseConfig` 的延伸模組。
::

    $config         = new Config\Encryption();
    $config->key    = 'aBigsecret_ofAtleast32Characters';
    $config->driver = 'OpenSSL';

    $encrypter = \Config\Services::encrypter($config);


預設行為
================

默認情況下，加密程式庫使用OpenSSL的處理程序。該加密處理程序使用 AES-256-CTR 演算法，你設定的 *key* 和 SHA512 HMAC 身分驗證。

設置你的加密金鑰
===========================

你的加密金要 **必須** 在使用中的加密演算法允許的範圍內。
像是 AES-256 ，要在 256 個 bits 或者 32 bytes （字元）內。

金鑰應盡可能的隨機，並且 **不得** 為常規本文字串，也不輸出雜湊函數等。要創造適當的金鑰，你可以使用加密程式庫內的 ``createKey()`` 方法。
::

	// $key will be assigned a 32-byte (256-bit) random key
	$key = Encryption::createKey(32);

金鑰可以被儲存在 *app/Config/Encryption.php* ，或你可以設計自己的儲存機制，並在加密／解密時動態傳遞金鑰。

要將保存到 *app/Config/Encryption.php* ，請打開你的檔案並設置
::

	public $key = 'YOUR KEY';

編碼金鑰或結果
------------------------

你會注意到 ``createKey()`` 方法輸出一個複製貼上可能會損壞的難處理二進位的資料，所以你可以使用 ``bin2hex()`` ， ``hex2bin()`` 或 Base64 編碼來更方便的使用金鑰，
像是
::

	// Get a hex-encoded representation of the key:
	$encoded = bin2hex(Encryption::createKey(32));

	// Put the same value in your config with hex2bin(),
	// so that it is still passed as binary to the library:
	$key = hex2bin(<your hex-encoded key>);

你可能會發現到相同的技術也可以使用在多個加密結果
::

	// Encrypt some text & make the results text
	$encoded = base64_encode($encrypter->encrypt($plaintext));

加密程序說明
========================

OpenSSL
-------------

長期以來， `OpenSSL <https://www.php.net/openssl>`_ 擴充套件一直是 PHP 標準的一部分。

CodeIgniter 的 OpenSSL 處理程序就是使用 AES-256-CTR 加密。

你所設定提供的 *key* 用來得到另外兩組，一組用於加密則另一組用來認證。
這透過已知技術 `HMAC-based Key Derivation Function <https://en.wikipedia.org/wiki/HKDF>`_ （HKDF） 來實現作為基於 HMAC 的金鑰產生函數。

訊息長度
==============
一個加密的資串通常長於原本的純文本字符串（取決於密碼）。

這受密碼演算法本身，初始化向量（IV）以及 HMAC 演算法的影響。此外，加密訊息也經過 Base64 編碼，因此無論使用甚麼字符都可以安全的儲存以及傳輸。

選擇資料儲存機制時，請留意這個資訊。
像是：Cookies 只能保存 4K 的訊息。

使用加密服務
=====================================

代替（或除了）使用 :ref:`usage` 中所描述得 ``Services`` ，你可以直接創造一個 "Encrypter" ，或者更改現有實體的設置。
::

    // create an Encrypter instance
    $encryption = new \Encryption\Encryption();

    // reconfigure an instance with different settings
    $encrypter = $encryption->initialize($config);

記住， ``$config`` 必須是 `Config\Encryption` 類別的實體或擴展 `CodeIgniter\Config\BaseConfig` 。


***************
類別參考
***************

.. php:class:: CodeIgniter\\Encryption\\Encryption

	.. php:staticmethod:: createKey($length)

		:param	int	$length: 輸出長度
		:returns:	具有指定長度的偽隨機密碼密鑰，失敗則為FALSE
		:rtype:	string

		透過作業系統來源中獲取隨機數據來創建加密金鑰（i.e. /dev/urandom）。


	.. php:method:: initialize($config)

		:param	BaseConfig	$config: 設定參數
		:returns:	CodeIgniter\\Encryption\\EncrypterInterface instance
		:rtype:	CodeIgniter\\Encryption\\EncrypterInterface
		:throws:	CodeIgniter\\Encryption\\Exceptions\\EncryptionException

		初始化（設定）程式庫來使用不同的設定。

		範例::

			$encrypter = $encryption->initialize(['cipher' => '3des']);

		請參考 :ref:`configuration` 部分詳細的訊息。

.. php:interface:: CodeIgniter\\Encryption\\EncrypterInterface

	.. php:method:: encrypt($data, $params = null)

		:param	string	$data: 數據加密
		:param		$params: 設定參數（金鑰）
		:returns:	加密資料或失敗時為FALSE
		:rtype:	string
		:throws:	CodeIgniter\\Encryption\\Exceptions\\EncryptionException

		加密輸入數據並回傳加密後的資料。

                如果您將參數作為第二個參數傳遞，如果 ``$params`` 是陣列，則 ``key`` 元素將是此操作的開始鍵；或者起始鍵可以作為字串傳遞。

		範例::

			$ciphertext = $encrypter->encrypt('My secret message');
			$ciphertext = $encrypter->encrypt('My secret message', ['key' => 'New secret key']);
			$ciphertext = $encrypter->encrypt('My secret message', 'New secret key');

	.. php:method:: decrypt($data, $params = null)

		:param	string	$data: 要解密的資料
		:param		$params: 設定參數（金鑰）
		:returns:	解密資料或失敗時為FALSE
		:rtype:	string
		:throws:	CodeIgniter\\Encryption\\Exceptions\\EncryptionException

		解密輸入資料並回傳解密後的資料。

                如果您將參數作為第二個參數傳遞，如果 ``$params`` 是陣列，則 ``key`` 元素將是此操作的開始鍵；或者起始鍵可以作為字串傳遞。

		範例::

			echo $encrypter->decrypt($ciphertext);
			echo $encrypter->decrypt($ciphertext, ['key' => 'New secret key']);
			echo $encrypter->decrypt($ciphertext, 'New secret key');
