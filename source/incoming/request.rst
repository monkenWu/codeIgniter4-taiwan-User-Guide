Request 類別
****************************************************

請求類別是 HTTP 請求的物件導向表現形式。這意味著它可以用於傳入請求（例如來自瀏覽器的請求），以及發出請求，（例如從應用程式發到第三方應用程式）。
這個類別提供了它們共同需要的功能，但是這兩者也都有各自繼承 Request 類別，然後分別添加特定的功能。

想了解更多請到 :doc:`IncomingRequest 類別 </incoming/incomingrequest>` 和
:doc:`CURLRequest 類別 </libraries/curlrequest>` 。

類別參考
============================================================

.. php:class:: CodeIgniter\\HTTP\\Request

    .. php:method:: getIPAddress()

        :returns: 可以偵測到的使用者的 IP 地址，否則為 NULL ，如果 IP 地址無效，則回傳 0.0.0.0
        :rtype:   string

        可以偵測到的使用者的 IP 地址，否則為 NULL ，如果 IP 地址無效，則回傳 0.0.0.0::

            echo $request->getIPAddress();

        .. important:: 此方法會根據 ``App->proxyIPs`` 的設定，來回傳 HTTP_X_FORWARDED_FOR、 HTTP_CLIENT_IP、HTTP_X_CLIENT_IP 或H TTP_X_CLUSTER_CLIENT_IP。
            
    .. php:method:: isValidIP($ip[, $which = ''])

        :param    string $ip: IP 地址
        :param    string $which: IP 協議 ('ipv4' 或是 'ipv6')
        :returns: IP 有效回傳 true，否則回傳 false
        :rtype:   bool

        傳入一個 IP 地址，根據 IP 是否有效回傳 true 或 false。

        .. note:: $request->getIPAddress() 自動檢測 IP 地址是否有效。

            ::

                if ( ! $request->isValidIP($ip))
                {
                    echo 'Not Valid';
                }
                else
                {
                    echo 'Valid';
                }

        第二個參數可選，可以為 'ipv4' 或 'ipv6'，預設這兩種格式皆會檢查。

    .. php:method:: getMethod([$upper = FALSE])

        :param bool $upper: 以大寫還是小寫回傳方法名，TRUE 表示大寫
        :returns: HTTP 請求方法
        :rtype: string

        回傳 ``$_SERVER['REQUEST_METHOD']``，並且轉換字母到指定大寫或小寫。
        ::

            echo $request->getMethod(TRUE); // Outputs: POST
            echo $request->getMethod(FALSE); // Outputs: post
            echo $request->getMethod(); // Outputs: post

    .. php:method:: setMethod($method)

        :param string $upper: 設定請求得方法，用於偽裝請求。
        :returns: HTTP 請求方法
        :rtype: Request

    .. php:method:: getServer([$index = null[, $filter = null[, $flags = null]]])

        :param    mixed     $index: 變數名稱
        :param    int       $filter: 要使用的過濾器類型，完整列表 `見此 <https://www.php.net/manual/en/filter.filters.php>`__.
        :param    int|array $flags: 要使用的過濾器的 ID，完整列表 `見此 <https://www.php.net/manual/en/filter.filters.flags.php>`__.
        :returns: $_SERVER 值，如果不存在則回傳 NULL。
        :rtype:   mixed

        該方法與 :doc:`IncomingRequest 類別 </incoming/incomingrequest>` 中的 ``post()``、``get()`` 和 ``cookie()`` 方法相同。只是它只獲取 getServer 的資料 (``$_SERVER``)::

            $request->getServer('some_data');

        要回傳多個 ``$_SERVER`` 值的陣列，請將所有的鍵值以陣列傳遞。
        ::

            $require->getServer(['SERVER_PROTOCOL', 'REQUEST_URI']);

    .. php:method:: getEnv([$index = null[, $filter = null[, $flags = null]]])

        :param    mixed     $index: 變數名稱
        :param    int       $filter: 要使用的過濾器類型，完整列表 `見此 <https://www.php.net/manual/en/filter.filters.php>`__.
        :param    int|array $flags: 要使用的過濾器的 ID，完整列表 `見此 <https://www.php.net/manual/en/filter.filters.flags.php>`__.
        :returns: $_ENV 值，如果不存在則回傳 NULL。
        :rtype:   mixed

         該方法與 :doc:`IncomingRequest 類別 </incoming/incomingrequest>` 中的 ``post()``、``get()`` 和 ``cookie()`` 方法相同。只是它只獲取 getEnv 的資料 (``$_ENV``)::

            $request->getEnv('some_data');

        要回傳多個 ``$_ENV`` 值的陣列，請將所有的需要的鍵值以陣列傳遞。
        ::

            $require->getEnv(['CI_ENVIRONMENT', 'S3_BUCKET']);

    .. php:method:: setGlobal($method, $value)

        :param    string $method: 方法名稱
        :param    mixed  $value:  需要被加入的資料
        :returns: HTTP 請求方法
        :rtype:	Request

        允許手動設定 PHP 全域的值，如 $_GET、$_POST 等。

    .. php:method:: fetchGlobal($method [, $index = null[, $filter = null[, $flags = null]]])

        :param    string    $method: 輸入過濾器常數
        :param    mixed     $index: 值的名稱
        :param    int       $filter: 要使用的過濾器類型，完整列表 `見此 <https://www.php.net/manual/en/filter.filters.php>`__.
        :param    int|array $flags: 要使用的過濾器的 ID，完整列表 `見此 <https://www.php.net/manual/en/filter.filters.flags.php>`__.
        :rtype:   mixed

        從全域中獲取一個或多個物件，如 cookie、get、post 等，可以選擇在檢索時透過過濾器對輸入進行過濾。
