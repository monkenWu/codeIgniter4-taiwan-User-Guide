############
安裝指引
############

CodeIgniter4可以通過多種方式安裝：手動安裝、 `Composer <https://getcomposer.org>`_　安裝，或是使用　`Git <https://git-scm.com/>`_　安裝，選擇最適合你的方式。

- 如果你希望以 CodeIgniter3 聞名的：「下載後直接運作」這種安裝方式，請選擇手動安裝。
- 如果你的專案預期會使用多種不同的程式庫，並打算透過 PHP Composer 進行管理，那麼我們建議你使用 Composer 部屬 CodeIgniter4 環境。
- 如果你想對 CodeIgniter4 做出貢獻，那麼 Git 安裝會是最適合你的方式。

.. toctree::
    :titlesonly:

    installing_manual
    installing_composer
    installing_git
    running
    upgrading
    troubleshooting
    repositories

不論你使用哪一種安裝方式部屬 CodeIgniter4 ，你都可以線上閱讀 `中文使用手冊 <https://monkenwu.github.io/codeIgniter4-taiwan-User-Guide/>`_ 。

.. note:: 在使用 CodeIgniter 4 之前，請確定你所運行的環境符合我們的 :doc:`系統需求 </intro/requirements>` ，特別是最低 PHP 版本以及啟用對應的 PHP 擴充模組。比如說：在 ``php.ini`` 中取消掉 curl 和 intl 的註解才能啟用這些擴充模組。