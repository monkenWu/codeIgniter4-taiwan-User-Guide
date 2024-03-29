認證方式 
#####################################

CodeIgniter 刻意不提供內建的認證或授權相關類別，因為已經有很多優秀的第三方模組可以提供這類服務。社群內也有許多資源可以輔助你撰寫屬於自己的功能，以下是推薦的實作方法，鼓勵模組、專案以及框架本身的開發者可以保持一致。

建議指南
===============

* 處理登入以及登出的操作應該在成功時觸發相應的 ``login`` 與 ``logout`` 事件。
* 定義 "目前使用者" 的模組應該定義函數 ``user_id()`` 來回傳使用者的唯一識別符號，對於 "目前沒有使用者" 登入應該回傳 ``null`` 。

符合上述建議的模組可以透過在 **composer.json** 中加入以下內容來表示相容性。

::

    "provide": {
        "codeigniter4/authentication-implementation": "1.0"
    },

你可以在 `Packagist <https://packagist.org/providers/codeigniter4/authentication-implementation>`_ 上查閱提供上述實作模組列表。
