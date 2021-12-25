######################
模擬系統類別
######################

框架內的一些元件提供了類別的模擬版本，讓你可以在測試中使用。在測試執行的過程中，這些類別可以取代正常的類別。通常它們會提供額外的斷言，在測試執行的過程中是否正確動作（或沒有動作）。這可能是檢查資料被正確快取，電子郵件被正確發送，等等。

.. contents::
    :local:
    :depth: 1

快取
=====

你可以用 ``mock()`` 方法來模擬快取，只需使用 ``CacheFactory`` 作為參數進行傳入。

::

    $mock = mock(CodeIgniter\Cache\CacheFactory::class);

這將回傳一個 ``CodeIgniter/TestMockMockCache`` 的實體，你可以直接使用它。同時，它也會將模擬插入到服務類別中，也就是說，在接下來你的程式對 ``service('cache')`` 或 ``ConfigServices::cache()`` 的任何呼叫都將使用模擬的類別。

當你在一個檔案中的多個測試方法中使用了這個方法時，你應該在測試的 ``setUp()`` 過程中呼叫 ``clean()`` 或 ``bypass()`` 方法，以保證測試案例執行時能夠有乾淨的環境。

附加方法
------------------

你可以用 ``bypass()`` 方法指揮模擬的快取處理程式不做任何快取。這將模擬使用虛擬的處理程式，並保證你的測試不依賴於快取的資料。

::

    $mock = mock(CodeIgniter\Cache\CacheFactory::class);
    // Never cache any items during this test.
    $mock->bypass();

可用斷言
--------------------

模擬類別提供了以下斷言，你可以在測試期間呼叫使用：

::

    $mock = mock(CodeIgniter\Cache\CacheFactory::class);

    // 斷言一個名為 $key 的快取存在
    $mock->assertHas($key);
    // 斷言名為 $key 的斷言存在並且數值為 $value
    $mock->assertHasValue($key, $value);
    // 斷言名為 $key 的快取不存在
    $mock->assertMissing($key);
