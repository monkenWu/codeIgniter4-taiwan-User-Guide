*******************
內容協商
*******************

內容協商是一種用來根據使用者端和伺服器端可處理的資源類型，來決定回傳給客戶端哪種類型的內容的機制。該機制可用來決定客戶端是想要 HTML 還是想要 JSON ，一個圖片是應該以 JPG 還是以 PNG 格式回傳，或者支持哪種類型的壓縮方法等。這些決策是通過分析四個不同的請求頭，而這些請求頭里支持多個帶有優先級的值選項。手動對這些值選項進行優先級配對通常是比較傷腦的，因此 CodeIgniter 提供了 Negotiator 來處理以上的過程。

=================
載入類別
=================

你可以通過 Service 類別來手動載入一個該類別的實體::

    $negotiate = \Config\Services::negotiator();

以上操作會獲取所有的請求實體並自動將其注入到 Negotiator （協商，下同）類別中。

該類別並不需要主動載入。而是通過請求的 ``IncomingRequest`` 實體來進行訪問。儘管你並不能通過這一過程直接訪問該實體，但你可以通過 ``negotiate()`` 方法來輕鬆的呼叫它的所有方法::

    $request->negotiate('media', ['foo', 'bar']);

當通過該方法訪問實體時，第一個參數是你需要匹配內容的類型，第二個是所支持的類型值構成的陣列。

===========
協商
===========

本節中，我們將討論四種可以用來協商的類型，並展示如何通過上述兩種方法來進行內容協商。

媒體
=====

第一層首先要看的就是「媒體」協商。該協商方式是通過 Accept 請求頭進行的，並且是可用的請求頭中最為複雜的類型之一。一個常見的例子就是使用者端告訴伺服器端其所需要的資料格式，而這種操作在 API 中最為常見。例如，一個使用者端可能從一個 API 端點請求以 JSON 編碼的資料::

    GET /foo HTTP/1.1
    Accept: application/json

該伺服器需要提供一個支持該內容類型的列表。在這個例子中，API 可能需要回傳像原生 HTML ，JSON 或者是 XML 格式的資料。而該列表應根據使用者端偏好的順序回傳::

    $supported = [
        'application/json',
        'text/html',
        'application/xml'
    ];

    $format = $request->negotiate('media', $supported);
    // 或是
    $format = $negotiate->media($supported);

在這個例子中，使用者端和伺服器端的協商一致，將資料以 JSON 的格式回傳，因此 'json' 就會從協商方法中回傳。預設情況下，如果沒有配對到，在 $support 陣列中的第一個元素就會被回傳。儘管在某些情況下，你可能會強制要求伺服器端嚴格得匹配格式。因此如果你將 ``true`` 作為最後的參數傳入時，在配對不到時就會回傳空字串::

    $format = $request->negotiate('media', $supported, true);
    // or
    $format = $negotiate->media($supported, true);

語言
========

另一個常見的用法就是決定回傳內容的語言。如果你執行的是一個單語言網站，該功能顯然並沒有什麼影響。但是如果對於那些提供多語言內容的網站來說，該功能就會變得非常有用，基於瀏覽器將通常會在 ``Accept-Language`` 請求頭中發送偏好的語言類型::

    GET /foo HTTP/1.1
    Accept-Language: fr; q=1.0, en; q=0.5

在這個例子中，瀏覽器偏好法語，並次偏好英語。如果你的網站支持英語或德語，那麼你就會如下操作::

    $supported = [
        'en',
        'de'
    ];

    $lang = $request->negotiate('language', $supported);
    // 或是
    $lang = $negotiate->language($supported);

本例中，en 將作為當前語言回傳。如果沒有產生配對，就會回傳 $supported 陣列的第一個元素，因此該元素將會一直作為偏好語言。

編碼
========

``Accept-Encoding`` 請求頭包含了使用者端所期望接收到的字符集，用於確定使用者端支持哪種類型的壓縮方式::

    GET /foo HTTP/1.1
    Accept-Encoding: compress, gzip

你的 web 伺服器將會定義可以使用的壓縮類型。某些伺服器，例如 Apache ,只支持了 **gzip**::

    $type = $request->negotiate('encoding', ['gzip']);
    // 或是
    $type = $negotiate->encoding(['gzip']);

想了解更多的話，請查閱 `維基百科 <https://en.wikipedia.org/wiki/HTTP_compression>`_.

字符集
=============

所期待的字符集類型會通過 ``Accept-Charset`` 請求頭來傳值::

    GET /foo HTTP/1.1
    Accept-Charset: utf-16, utf-8

預設情況下，如果沒有配對到的話就會回傳 **utf-8**::

    $charset = $request->negotiate('charset', ['utf-8']);
    // 或是
    $charset = $negotiate->charset(['utf-8']);
