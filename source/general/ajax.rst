##############
AJAX 請求
##############

``IncomingRequest::isAJAX()`` 方法使用 ``X-Requested-With`` 標頭來定義這個請求是 XHR 請求還是普通請求。然而，最新的 JavaScript 實作方法（例如： fetch） 並不會再發送這個標頭，因此 ``IncomingRequest::isAJAX()`` 將變得不這麼可靠了，因為沒有這個標頭就無法定義請求是否為 XHR 。

為了解決這個問題，最有效的方式（截至目前為止）就是手動定義請求了。強制將訊息傳送給伺服器，這樣就能辨識出這個請求是 XHR 。

下面將會向你演示如何在 Fetch API 和其他的 JavaScript 程式庫中，強制發送 ``X-Requested-With`` 標頭的方法。

Fetch API
=========

.. code-block:: javascript

    fetch(url, {
        method: "get",
        headers: {
          "Content-Type": "application/json",
          "X-Requested-With": "XMLHttpRequest"
        }
    });


jQuery
======

對於 jQuery 這個程式庫來說，不需要特意傳送這個標頭。根據 `官方使用文件 <https://api.jquery.com/jquery.ajax/>`_ ，它是所有 ``$.ajax()`` 都會附帶的標準標頭。若是你不想冒險，還是想強制傳送這個標頭的話，按照下面的方式做就可以了：

.. code-block:: javascript

    $.ajax({
        url: "your url",
        headers: {'X-Requested-With': 'XMLHttpRequest'}
    });


VueJS
=====

在 VueJS 中，若是你只使用 Axios 來處理這種類型的請求的話，你只需要在 ``創建`` 的函數中加入以下程式碼。

.. code-block:: javascript

    axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';


React
=====

.. code-block:: javascript

    axios.get("your url", {headers: {'Content-Type': 'application/json'}})