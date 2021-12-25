##################
API 響應特性
##################

很多現代化 PHP 開發需要建立 API ，無論是為單一頁面應用程式（ SPA ）提供資料，還是做為一個獨立的產品，都需要建立 API 。 CodeIgniter 提供一個 API 響應的特性，可以和任何控制器一起使用，讓你簡單地建立常見的響應類型，不需要記住 HTTP 狀態代碼或應該回傳哪些響應類型。

.. contents::
    :local:
    :depth: 2

*************
使用範例
*************

下方的範例將表現一種控制器的常見使用模式：

::


    <?php namespace App\Controllers;

    use CodeIgniter\API\ResponseTrait;

    class Users extends \CodeIgniter\Controller
    {
        use ResponseTrait;

        public function createUser()
        {
            $model = new UserModel();
            $user  = $model->save($this->request->getPost());

            // Respond with 201 status code
            return $this->respondCreated();
        }
    }

在這個範例中，回傳一個 201 HTTP 狀態碼為，並帶有一般的狀態訊息： "Created" ，對於常見的需求，都有相應的方法可以使用：

::

    // 一般響應
    $this->respond($data, 200);

    // 故障響應
    $this->fail($errors, 400);

    // 項目已建立成功
    $this->respondCreated($data);

    // 項目已刪除成功
    $this->respondDeleted($data);

    // 命令已執行，無須響應
    $this->respondNoContent($message);

    // 使用者無權限
    $this->failUnauthorized($description);

    // 禁止使用
    $this->failForbidden($description);

    // 找不到資源
    $this->failNotFound($description);

    // 資料未通過驗證
    $this->failValidationError($description);

    // 資源已經存在
    $this->failResourceExists($description);
    
    // 資源已經被刪除
    $this->failResourceGone($description);

    // 使用者提出過多請求
    $this->failTooManyRequests($description);

***********************
處理響應類型
***********************

當你透過下列任何一種方法傳遞資料時，他們將會根據以下條件來決定結果會被格式化為何種資料型別：


* 如果 $data 是一個字串，那麼它將會被視為 HTML 發送回使用者端。
* 如果 $data 是一個陣列，它會根據控制器的 ``$this->format`` 規定進行格式化。如果這個值為空，那麼它將會嘗試與客戶端所要求的類型進行內容協商。如果在 **Config/Format.php** 沒有指定其他的屬性，則預設為 JSON 即是以 ``$supportedResponseFormats`` 屬性為主。

若要定義所使用的屬性輸出格式器，請編輯 **Config/Format.php** 檔案。其中的 ``$supportedResponseFormats`` 屬性包含著一系列的 mime 類型，你的應用程式可以自動地格式化響應類型。在預設的情形下，系統會自動格式化 XML 與 JSON 響應：

::

        public $supportedResponseFormats = [
            'application/json',
            'application/xml'
        ];

這是一個用於 :doc:`內容協商 </incoming/content_negotiation>`  過程中，用來決定應該回傳哪種類型的響應的陣列。如果使用者要求的內容，沒有辦法比對到你所支援的內容，那麼將會選用這個陣列中第一個格式作為回傳依據。

接下來，你需要定義用於格式化資料陣列的類別，這必須是一個完全合格的類別名稱，並且這個類別必須實作 **CodeIgniter\\Format\\FormatterInterface** 這個介面。Formatters 變數將會倚賴你產生它所支援的 JSON 與 XML 。 

::

    public $formatters = [
        'application/json' => \CodeIgniter\Format\JSONFormatter::class,
        'application/xml'  => \CodeIgniter\Format\XMLFormatter::class
    ];

因此，若在 **Accept** 標頭中請求 JSON 格式的資料，而你在實作時程式時透過 ``respond*`` 或 ``fail*`` 方法回傳的資料將會被 **CodeIgniter\\API\\JSONFormatter** 類別進行格式化，產生 JSON 資料發還給使用者端。

類別參考
***************
.. php:method:: setResponseFormat($format)

    :param string $format: 要回傳的響應類型，可以是 ``json`` 或 ``xml`` 。

    這定義了在響應中格式化陣列時要使用的格式，如果你替 ``$format`` 宣告了一個 ``null`` 空值，那麼它將透過內容協商自動確定一個回傳方式。

::

    return $this->setResponseFormat('json')->respond(['error' => false]);

.. php:method:: respond($data[, $statusCode=200[, $message='']])

    :param mixed  $data: 要回傳給使用者端的資料，可以是字串或是陣列。
    :param int    $statusCode: 要回傳的 HTTP 狀態碼，預設為 200 。
    :param string $message: 自訂回傳的 "reason" 訊息。

    在 api 特性中，其他的方法都會基於這個方法是回傳響應給使用者端。

    ``$data`` 元素可以是一個字串或陣列，在預設的情形下，字串將以 HTML 的形式回傳，而陣列將透過  json_encode 執行後已 JSON 形式回傳，除非 :doc:`內容協商  </incoming/content_negotiation>` 決定以不同的格式進行回傳。

    如果你傳遞了 ``$message`` 字串，它將會被用來替代響應狀態的標準 IANA 原因代碼。但不是每個使用者端都會遵照自訂的代碼，並且會自動使用與狀態碼相符的 IANA 原因代碼。

    .. note:: 由於它會在目前的 Response 實體上設定狀態代碼與 body ，所以這應該是你的程式中最後執行的方法。

.. php:method:: fail($messages[, int $status=400[, string $code=null[, string $message='']]])

    :param mixed $messages: 一個包含遭遇的錯誤訊息的字串或字串組成的陣列。
    :param int   $status: 要回傳的 HTTP 狀態碼，預設為 400 。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: 依據使用者喜好的形式進行 multi-part 響應。

    fail 是適用於表達錯誤響應的通用方法，所有 fail 相關的方法也都基於它。
    
    ``$messages`` 元素是一個字串或字串組成的陣列。

    ``$status`` 參數是應該回傳 HTTP 狀態碼。

    由於許多 API 使用自訂錯誤碼會更合適，所以可以在第三個參數中傳入一個自訂的錯誤代碼，如果沒有值，那麼它將會和 ``$status`` 相同。

    如果你傳遞了 ``$message`` 字串，它將會被用來替代響應狀態的標準 IANA 原因代碼。但不是每個使用者端都會遵照自訂的代碼，並且會自動使用與狀態碼相符的 IANA 原因代碼。

    $response 響應是一個陣列，其中包含兩元素 ``error`` 以及 ``messages`` 。 ``error`` 元素包含了錯誤的狀態碼， ``messages`` 元素包含錯誤訊息的陣列。響應的內容看起來就像是這樣：

    ::

	    $response = [
	        'status'   => 400,
	        'code'     => '321a',
	        'messages' => [
	            'Error message 1',
	            'Error message 2'
	        ]
	    ];

.. php:method:: respondCreated($data = null[, string $message = ''])

    :param mixed  $data: 要回傳給使用者端的資料，可以是字串或是陣列。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    設定新的資源成功創建後適當的狀態碼，通常是 201 ：

    ::

	    $user = $userModel->insert($data);
	    return $this->respondCreated($user);

.. php:method:: respondDeleted($data = null[, string $message = ''])

    :param mixed  $data: 要回傳給使用者端的資料，可以是字串或是陣列。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    設定這個 API 呼叫後的結果，為刪除了一個資源時適當的狀態碼，通常是 200 。

    ::

	    $user = $userModel->delete($id);
	    return $this->respondDeleted(['id' => $id]);

.. php:method:: respondNoContent(string $message = 'No Content')

    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當指令被伺服器成功執行後，但沒有具體意義的回傳可以給予使用者端時，設定適當的狀態碼，通常是 204 。

    ::

	    sleep(1);
	    return $this->respondNoContent();

.. php:method:: failUnauthorized(string $description = 'Unauthorized'[, string $code=null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當使用者未被授權或授權狀態不正確時，設定適當的狀態碼，通常是 401 。

    ::

	    return $this->failUnauthorized('Invalid Auth token');

.. php:method:: failForbidden(string $description = 'Forbidden'[, string $code=null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    與 failUnauthorized  不同，當請求的 API 從不被允許造訪時，應該使用這個方法。 Unauthorized 意味著鼓勵使用者端用不同的證書再試一次，但 Forbidden 代表著使用者端不應該再嘗試造訪，並不會因為造訪方式的不同而有任何改變，通常狀態代碼為 403 。

    ::

    	return $this->failForbidden('Invalid API endpoint.');

.. php:method:: failNotFound(string $description = 'Not Found'[, string $code=null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當無法照到所請求的資源時，設定適當的狀態碼，通常為 404 。

    ::

    	return $this->failNotFound('User 13 cannot be found.');

.. php:method:: failValidationError(string $description = 'Bad Request'[, string $code=null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當使用者發送的資料沒有辦法通過驗證規則時，設定適當的狀態碼，通常為 400 。

    ::

    	return $this->failValidationError($validation->getErrors());

.. php:method:: failResourceExists(string $description = 'Conflict'[, string $code=null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當使用者想要創建的資源已經存在時，設定適當的狀態代碼，通常為 409 。

    ::

    	return $this->failResourceExists('A user already exists with that email.');

.. php:method:: failResourceGone(string $description = 'Gone'[, string $code=null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當被請求的資源因為被刪除或不再提供，設定適當的狀態碼，通常為 410 。

    ::

    	return $this->failResourceGone('That user has been previously deleted.');

.. php:method:: failTooManyRequests(string $description = 'Too Many Requests'[, string $code=null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當使用者端呼叫 API 次數過多時使用。這可能源自於某種形式的節流或是速率限制，狀態碼通常是 400 。

    ::

    	return $this->failTooManyRequests('You must wait 15 seconds before making another request.');

.. php:method:: failServerError(string $description = 'Internal Server Error'[, string $code = null[, string $message = '']])

    :param string  $description: 顯示給使用者的錯誤訊息。
    :param string $code: 自訂的 API 專用錯誤代碼。
    :param string $message: 自訂回傳的 "reason" 訊息。
    :returns: Response 物件的 send() 方法的數值。

    當伺服器出現錯誤時設定適當狀態碼。

    ::

    	return $this->failServerError('Server error.');
