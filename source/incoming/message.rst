=============
HTTP 訊息
=============

Message 類別提供了 HTTP 訊息中請求和響應共有部分的介面，包括訊息主體、協定版本、處理標頭的程序和處理內容協商的方法。

這個類別是 :doc:`Request 類別 </incoming/request>` 跟 :doc:`Response 類別 </outgoing/response>` 的父類別。
因此，某些方法（如內容協商方法）可能僅適用於請求或響應其中之一，但他們都已經被包含在此類別裡以維持標頭方法的完整。


什麼是內容協商？
============================

從本質上講，內容協商只是HTTP規範的一部分，它允許單一資源提供多種類型的內容，允許用戶端請求最適合他們的數據類型。

一個典型的例子是無法顯示 PNG 檔的瀏覽器只能請求 GIF 或 JPEG 影像。
當 getServer 收到請求時，它會查看客戶端所請求的的文件類型並從其支援的圖像格式中選擇最佳匹配，在本例中可能選擇 JPEG 圖像來返回。

同樣的協商可以發生在四種型別的資料上：

* **媒體/文件型別** - 這可能是圖像格式，或 HTML vs. XML 或 JSON。
* **字元集** - 返回的文件應該設置為何種字元集。通常是 UTF-8 。
* **文件編碼** - 通常是在結果上使用的壓縮型別。
* **文件語言** - 對於支援多種語言的網站，這有助於確定要返回的語言。
  
***************
類別參考
***************

.. php:class:: CodeIgniter\\HTTP\\Message

    .. php:method:: getBody()

        :returns: 目前的訊息主體
        :rtype: 混和的

        回傳目前的訊息主體（如果已設置）。如果主體不存在，則返回 null 。
        ::

            echo $message->getBody();

    .. php:method:: setBody($data)

        :param  mixed  $data: 訊息的主體
        :returns: 訊息|響應實體，以允許方法被串聯起來
        :rtype: CodeIgniter\\HTTP\\Message|CodeIgniter\\HTTP\\Response

        設置當前請求的主體。

    .. php:method:: appendBody($data)

        :param  mixed  $data: 訊息的主體
        :returns: 訊息|響應實體，以允許方法被串聯起來
        :rtype: CodeIgniter\\HTTP\\Message|CodeIgniter\\HTTP\\Response

        將資料加到當前請求的主體中。

    .. php:method:: populateHeaders()

        :returns: 無

        掃描和分析在 SERVER 資料中找到的標頭，並將其存儲以供以後訪問。
        這被 :doc:`IncomingRequest 類別 </incoming/incomingrequest>` 使用來確保當前請求的標頭可用。

        標頭是以 ``HTTP_`` 開頭的任何 SERVER 數據，如 ``HTTP_HOST``。
        每條訊息都從其標準的大寫字母和底線格式轉換為 ucwords 和破折號格式。
        前面的 ``HTTP_`` 將從字串中刪除。因此， ``HTTP_ACCEPT_LANGUAGE`` 變成了 ``Accept-Language``。

    .. php:method:: headers()

        :returns: 所有找到的標頭的陣列
        :rtype: 陣列

        回傳所有找到的或先前設置的標頭文件的陣列。

    .. php:method:: header($name)

        :param  string  $name: 要檢索的標頭名稱
        :returns: 回傳單個標頭物件。如果存在多個同名的標頭，則將返回標頭物件的陣列
        :rtype: \CodeIgniter\\HTTP\\Header|array

        檢索一個當前訊息標頭的值。 ``$name`` 是標頭名稱但是不區分大小寫。
        當標頭按上述方式在內部轉換時，您可以使用任何類型的大小寫存取標頭。
        ::

            // These are all the same:
            $message->header('HOST');
            $message->header('Host');
            $message->header('host');

        如果標頭有多個值，則 ``getValue()`` 將以陣列形式返回。你可以使用 ``getValueLine()`` 方法來以字串形式檢索。
        ::

            echo $message->header('Accept-Language');

            // Outputs something like:
            'Accept-Language: en,en-US'

            echo $message->header('Accept-Language')->getValue();

            // Outputs something like:
            [
                'en',
                'en-US'
            ]

            echo $message->header('Accept-Language')->getValueLine();

            // Outputs something like:
            'en,en-US'

        你可以在第二個參數輸入過濾值來過濾標頭。
        ::

            $message->header('Document-URI', FILTER_SANITIZE_URL);


    .. php:method:: hasHeader($name)

        :param  string  $name: 你想要查看是否存在的標頭名稱
        :returns: 如果標頭名稱存在，則回傳 true，否則返回 false
        :rtype: 布林

    .. php:method:: getHeaderLine($name)

        :param  string $name: 要檢索的標頭名稱
        :returns: 表示標頭值的字串
        :rtype: 字串

        以字串形式返回標頭的值。此方法允許您在標頭具有多個值時輕鬆獲取標頭的字串表示形式。結果會適當地連接。
        ::

            echo $message->getHeaderLine('Accept-Language');

            // Outputs:
            en, en-US

    .. php:method:: setHeader($name, $value)

        :param string $name: 要為其設置值的標頭的名稱
        :param mixed  $value: 要將標頭設置為的值
        :returns: 當前的訊息|響應實體
        :rtype: CodeIgniter\\HTTP\\Message|CodeIgniter\\HTTP\\Response

        設置單個標頭的值。 ``$name`` 是不區分大小寫的標頭名稱。
        如果標頭不存在，則創建一個新的。 ``$value`` 可以是字串或字串陣列。
        ::

            $message->setHeader('Host', 'codeigniter.com');

    .. php:method:: removeHeader($name)

        :param string $name: 要移除的標頭的名稱
        :returns: 當前的訊息實體
        :rtype: CodeIgniter\\HTTP\\Message

        從訊息中刪除標頭。 ``$name`` 是不區分大小寫的標題。
        ::

            $message->removeHeader('Host');

    .. php:method:: appendHeader($name, $value)

        :param string $name: 要修改的標頭的名稱
        :param string  $value: 要添加到標頭的值
        :returns: 當前的訊息實體
        :rtype: CodeIgniter\\HTTP\\Message

        向現有標頭添加值。標頭必須已經是陣列，而不是單個字串。
        如果它是一個字串，那麼將拋出一個LogicException。
        ::

            $message->appendHeader('Accept-Language', 'en-US; q=0.8');

    .. php:method:: prependHeader($name, $value)

        :param string $name: 要修改的標頭的名稱
        :param string  $value: 要附加到標頭的值
        :returns: 當前的訊息實體
        :rtype: CodeIgniter\\HTTP\\Message

        在現有標頭前面附加一個值。標頭必須已經是陣列，而不是單個字串。
        如果它是一個字串，那麼將拋出一個 LogicException。
        ::

            $message->prependHeader('Accept-Language', 'en,');

    .. php:method:: getProtocolVersion()

        :returns: 當前 HTTP 協定版本
        :rtype: 字串

        返回訊息當前的 HTTP 協定。如果未設置任何設置，將返回 ``null`` 。 
        可接受的值為 ``1.0`` 、 ``1.1`` 和 ``2.0``。

    .. php:method:: setProtocolVersion($version)

        :param string $version: HTTP 協定版本
        :returns: 當前訊息實體
        :rtype: CodeIgniter\\HTTP\\Message

        設置此訊息使用的 HTTP 協定版本。有效值為 ``1.0`` 、 ``1.1`` 和 ``2.0``
        ::

            $message->setProtocolVersion('1.1');
