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
    respond($data, 200);
    // 故障響應
    fail($errors, 400);
    // 項目已建立成功
    respondCreated($data);
    // 項目已刪除成功
    respondDeleted($data);
    // 命令已執行，無須響應
    respondNoContent($message);
    // 使用者無權限
    failUnauthorized($description);
    // 禁止使用
    failForbidden($description);
    // 找不到資源
    failNotFound($description);
    // 資料未通過驗證
    failValidationError($description);
    // 資源已經存在
    failResourceExists($description);
    // 資源已經被刪除
    failResourceGone($description);
    // 使用者提出過多請求
    failTooManyRequests($description);

***********************
處理響應類型
***********************

當你透過下列任何一種方法傳遞資料時，他們將會根據以下條件來決定結果會被格式化為何種資料型別：


* 如果 $data 是一個字串，那麼它將會被視為 HTML 發送為使用者端
* **如果 $data 是一個陣列，它將會嘗試與客戶端所要求的類型進行內容協商，默認為 JSON**
    如果在 Config\API.php 沒有指定其他的屬性，則以 ``$supportedResponseFormats`` 屬性為主。
* If $data is an array, it will be formatted according to the controller's ``$this->format`` value. If that is empty
    it will try to negotiate the content type with what the client asked for, defaulting to JSON
    if nothing else has been specified within Config\API.php, the ``$supportedResponseFormats`` property.

若要定義所使用的屬性輸出格式器，請編輯 **Config/Format.php** 檔案。其中的 ``$supportedResponseFormats`` 屬性包含著一系列的 mime 類型，你的應用程式可以自動地格式化響應類型。在預設的情形下，系統會自動格式化 XML 與 JSON 響應：

::

        public $supportedResponseFormats = [
            'application/json',
            'application/xml'
        ];


This is the array that is used during :doc:`Content Negotiation </incoming/content_negotiation>` to determine which
type of response to return. If no matches are found between what the client requested and what you support, the first
format in this array is what will be returned.

Next, you need to define the class that is used to format the array of data. This must be a fully qualified class
name, and the class must implement **CodeIgniter\\Format\\FormatterInterface**. Formatters come out of the box that
support both JSON and XML::

    public $formatters = [
        'application/json' => \CodeIgniter\Format\JSONFormatter::class,
        'application/xml'  => \CodeIgniter\Format\XMLFormatter::class
    ];

So, if your request asks for JSON formatted data in an **Accept** header, the data array you pass any of the
``respond*`` or ``fail*`` methods will be formatted by the **CodeIgniter\\API\\JSONFormatter** class. The resulting
JSON data will be sent back to the client.

Class Reference
***************
.. php:method:: setResponseFormat($format)

    :param string $format The type of response to return, either ``json`` or ``xml``

    This defines the format to be used when formatting arrays in responses. If you provide a ``null`` value for
    ``$format``, it will be automatically determined through content negotiation.

::

    return $this->setResponseFormat('json')->respond(['error' => false]);


.. php:method:: respond($data[, $statusCode=200[, $message='']])

    :param mixed  $data: The data to return to the client. Either string or array.
    :param int    $statusCode: The HTTP status code to return. Defaults to 200
    :param string $message: A custom "reason" message to return.

    This is the method used by all other methods in this trait to return a response to the client.

    The ``$data`` element can be either a string or an array. By default, a string will be returned as HTML,
    while an array will be run through json_encode and returned as JSON, unless :doc:`Content Negotiation </incoming/content_negotiation>`
    determines it should be returned in a different format.

    If a ``$message`` string is passed, it will be used in place of the standard IANA reason codes for the
    response status. Not every client will respect the custom codes, though, and will use the IANA standards
    that match the status code.

    .. note:: Since it sets the status code and body on the active Response instance, this should always
        be the final method in the script execution.

.. php:method:: fail($messages[, int $status=400[, string $code=null[, string $message='']]])

    :param mixed $messages: A string or array of strings that contain error messages encountered.
    :param int   $status: The HTTP status code to return. Defaults to 400.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: A multi-part response in the client's preferred format.

    The is the generic method used to represent a failed response, and is used by all of the other "fail" methods.

    The ``$messages`` element can be either a string or an array of strings.

    The ``$status`` parameter is the HTTP status code that should be returned.

    Since many APIs are better served using custom error codes, a custom error code can be passed in the third
    parameter. If no value is present, it will be the same as ``$status``.

    If a ``$message`` string is passed, it will be used in place of the standard IANA reason codes for the
    response status. Not every client will respect the custom codes, though, and will use the IANA standards
    that match the status code.

    The response is an array with two elements: ``error`` and ``messages``. The ``error`` element contains the status
    code of the error. The ``messages`` element contains an array of error messages. It would look something like::

	    $response = [
	        'status'   => 400,
	        'code'     => '321a',
	        'messages' => [
	            'Error message 1',
	            'Error message 2'
	        ]
	    ];

.. php:method:: respondCreated($data = null[, string $message = ''])

    :param mixed  $data: The data to return to the client. Either string or array.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when a new resource was created, typically 201.::

	    $user = $userModel->insert($data);
	    return $this->respondCreated($user);

.. php:method:: respondDeleted($data = null[, string $message = ''])

    :param mixed  $data: The data to return to the client. Either string or array.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when a new resource was deleted as the result of this API call, typically 200.

    ::

	    $user = $userModel->delete($id);
	    return $this->respondDeleted(['id' => $id]);

.. php:method:: respondNoContent(string $message = 'No Content')

    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when a command was successfully executed by the server but there is no
    meaningful reply to send back to the client, typically 204.

    ::

	    sleep(1);
	    return $this->respondNoContent();

.. php:method:: failUnauthorized(string $description = 'Unauthorized'[, string $code=null[, string $message = '']])

    :param string  $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when the user either has not been authorized,
    or has incorrect authorization. Status code is 401.

    ::

	    return $this->failUnauthorized('Invalid Auth token');

.. php:method:: failForbidden(string $description = 'Forbidden'[, string $code=null[, string $message = '']])

    :param string  $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Unlike ``failUnauthorized``, this method should be used when the requested API endpoint is never allowed.
    Unauthorized implies the client is encouraged to try again with different credentials. Forbidden means
    the client should not try again because it won't help. Status code is 403.

    ::

    	return $this->failForbidden('Invalid API endpoint.');

.. php:method:: failNotFound(string $description = 'Not Found'[, string $code=null[, string $message = '']])

    :param string  $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when the requested resource cannot be found. Status code is 404.

    ::

    	return $this->failNotFound('User 13 cannot be found.');

.. php:method:: failValidationError(string $description = 'Bad Request'[, string $code=null[, string $message = '']])

    :param string  $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when data the client sent did not pass validation rules.
    Status code is typically 400.

    ::

    	return $this->failValidationError($validation->getErrors());

.. php:method:: failResourceExists(string $description = 'Conflict'[, string $code=null[, string $message = '']])

    :param string  $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when the resource the client is trying to create already exists.
    Status code is typically 409.

    ::

    	return $this->failResourceExists('A user already exists with that email.');

.. php:method:: failResourceGone(string $description = 'Gone'[, string $code=null[, string $message = '']])

    :param string  $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when the requested resource was previously deleted and
    is no longer available. Status code is typically 410.

    ::

    	return $this->failResourceGone('That user has been previously deleted.');

.. php:method:: failTooManyRequests(string $description = 'Too Many Requests'[, string $code=null[, string $message = '']])

    :param string  $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when the client has called an API endpoint too many times.
    This might be due to some form of throttling or rate limiting. Status code is typically 400.

    ::

    	return $this->failTooManyRequests('You must wait 15 seconds before making another request.');

.. php:method:: failServerError(string $description = 'Internal Server Error'[, string $code = null[, string $message = '']])

    :param string $description: The error message to show the user.
    :param string $code: A custom, API-specific, error code.
    :param string $message: A custom "reason" message to return.
    :returns: The value of the Response object's send() method.

    Sets the appropriate status code to use when there is a server error.

    ::

    	return $this->failServerError('Server error.');
