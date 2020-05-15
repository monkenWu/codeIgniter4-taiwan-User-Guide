=====================
誘捕系統類別
=====================

如果已開啟在 ``Application\Config\Filters.php`` 的檔案，誘捕系統類別就可以確定機器人何時發請求到 CodeIgniter4 的應用程式。這可以透過將表單欄位附加到所有表單來完成
，並且這個欄位是被人們藏起來但可以被機器人查看到。當資料被輸入進這個欄位時，就可以假設這個請求是來自機器人，你就可以拋出一個 ``HoneypotException``。

.. contents::
    :local:
    :depth: 2

啟用誘捕系統類別
=====================

啟用誘捕系統類別，必須對 ``app/Config/Filters.php`` 進行更改。只需要取消啟用誘捕系統類別來自 ``$globals`` 陣列的註解，像是：

::

    public $globals = [
            'before' => [
                'honeypot'
                // 'csrf',
            ],
            'after'  => [
                'toolbar',
                'honeypot'
            ]
        ];

一個啟用誘捕系統類別的過濾器的樣本是同捆的，像是  ``system/Filters/Honeypot.php`` ，如果並不適當，更改你的 ``app/Filters/Honeypot.php`` 
，並且在適當的設定下更改 ``$aliases`` 。

自定義誘捕系統類別
=====================

誘捕系統類別是可以被自定義的。可以在 ``app/Config/Honeypot.php`` 或者 ``.env`` 中設定。

* ``hidden`` - 使用 true|false 來控制是否能看見誘捕系統類別的欄位；預設為 ``true``
* ``label`` - 誘捕系統類別的欄位的 HTML label，預設為 'Fill This Field'
* ``name`` - 用來命名 HTML 表單欄位的樣板；預設為 'honeypot'
* ``template`` - 來自被用來使用誘捕系統類別的欄位樣板；預設為 '<label>{label}</label><input type="text" name="{name}" value=""/>'
