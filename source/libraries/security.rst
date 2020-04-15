##############
安全性類別
##############

安全性類別包含許多方法，有助於保護你的網站免於遭到跨站請求偽造（Cross-Site Request Forgery）攻擊。



.. contents::
    :local:
    :depth: 2

*******************
加載程式庫
*******************

如果你對加載程式庫的唯一興趣是為了處理 CSRF 保護，那麼你將永遠不需要加載它，因為它做為一個過濾器運行，不需要手動操作。

如果你發現有需要直接訪問的情況，你可以透過服務文件（Services file）加載它::

	$security = \Config\Services::security();

*********************************
跨站請求偽造（CSRF）
*********************************

你可以透過修改 **app/Config/Filters.php** 開啟 CSRF 的保護功能。並在全域啟用 `CSRF` 過濾器::

	public $globals = [
		'before' => [
			//'honeypot'
			'csrf'
		]
	];

選擇的 URI 可以從 CSRF 保護中被白名單化（例如：API 端點期待外部 POST 的內容）。你可以在過濾器中添加這些 URI 作為例外狀況::

	public $globals = [
		'before' => [
			'csrf' => ['except' => ['api/record/save']]
		]
	];

也支持正規表示式（與大小寫無關）::

    public $globals = [
		'before' => [
			'csrf' => ['except' => ['api/record/[0-9]+']]
		]
	];

如果你使用 :doc:`表單輔助函數 <../helpers/form_helper>`，那麼
:func:`form_open()` 會自動在你的表單中插入一個隱藏的 csrf 字段。如果沒有，你可以使用 ``csrf_token()``
和 ``csrf_hash()`` 函數。
::

	<input type="hidden" name="<?= csrf_token() ?>" value="<?= csrf_hash() ?>" />

此外，你可以使用 ``csrf_field()`` 方法來產生隱藏的輸入欄位::

	// Generates: <input type="hidden" name="{csrf_token}" value="{csrf_hash}" />
	<?= csrf_field() ?>

當發送一個 JSON 請求時，CSRF 令牌也可以作為被傳遞的參數之一。
下一個傳遞 CSRF 令牌的方法是一個特殊的 Http 標頭，它的名稱可以透過函數 ``csrf_header()`` 來實現。

此外，你可以使用 ``csrf_meta()`` 方法來方便地產生 meta 標籤::

	// Generates: <meta name="{csrf_header}" content="{csrf_hash}" />
	<?= csrf_meta() ?>

檢查 CSRF 令牌可用性的順序如下:

1. ``$_POST`` 陣列
2. Http 標頭
3. ``php://input`` （JSON 請求） - 請記得，這種方法是最慢的，因為我們必須先對 JSON 進行解碼，然後再進行編碼

令牌可以在每次提交時重新產生（預設），也可以在 CSRF cookie 整個生命週期中保持不變。預設重新產生令牌提供了更好的安全性，但是也可能會導致可用性問題，例如其他令牌會變得無效（導覽歷程記錄上一頁或下一頁、多個分頁視窗、非同步操作等）。你可以透過編輯以下設定參數來改變此特性。
::

	public $CSRFRegenerate  = true;

當請求沒有通過 CSRF 驗證檢查時，預設情況下將會重新導向上一頁，你可以設置一個 ``error`` 的即時訊息，向終端使用者顯示該訊息，這提供了比瀏覽器崩潰更好的使用者體驗。可以透過編輯 **app/Config/App.php** 中的 ``$CSRFRedirect`` 值來關閉::

	public $CSRFRedirect = false;

即使重新導向值為  **true**，AJAX 呼叫也不會重新導向，但是會引發錯誤....。

*********************
其他有用的方法
*********************

你不需要直接使用安全性類別中大部分的方法。以下是一些與 CSRF 無關的方法。

**sanitizeFilename()**

嘗試檔案名稱消毒，以防止企圖遍歷目錄和其他安全威脅，這對於經由使用者輸入提供的文件特別有用。第一個參數是路徑消毒。

如果允許使用者的輸入可以包含相對路徑，例如 file/in/some/approved/folder.txt，可以將第二個可選參數 $relative_path 設置為 true。
::

	$path = $security->sanitizeFilename($request->getVar('filepath'));
