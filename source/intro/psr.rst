**************
PSR 規範
**************

`PHP-FIG <http://www.php-fig.org/>`_ 創立於 2009 年，透過統一的介面標準以及程式碼風格標準，提升框架與程式間的協同工作能力（ Interoperability ）。縱使 CodeIgniter 不是 FIG 的成員，但我們還是有相同的目標也採納了許多實作建議。這份文檔所列出的是我們在框架中所遵守的 PSR 規範以及如何實現。

**PSR-1 ：基本程式寫作標準（ Basic Coding Standard ）**

這份規範涵蓋了基本類別、方法，以及檔案命名標準。我們的 `開發風格指南 <https://github.com/codeigniter4/CodeIgniter4/blob/develop/contributing/styleguide.md>`_ 符合 PSR-1 ，並且在它的基礎上新增了自己的標準。

**PSR-2 ：程式碼風格指南（ Coding Style Guide ）**

我們的 `開發風格指南 <https://github.com/codeigniter4/CodeIgniter4/blob/develop/contributing/styleguide.md>`_ 遵循 PSR 建議，並且加上一套我們自己的程式碼風格約束。

**PSR-3 ：日誌記錄器（ Logger Interface ）**

CodeIgniter 的 :doc:`日誌記錄器（ 記錄 ） </general/logging>` 實作了這份 PSR 所提供的所有介面。

**PSR-4 ：自動載入標準（ Autoloading Standard ）**

這份 PSR 明確地描述 classes 該如何以符合規則的命名空間與檔案路徑進行自動載入。 :doc:`自動載入器（ Autoloader ） </concepts/autoloader>` 符合 PSR-4 的規範。

**PSR-6 ：快取介面標準（ Caching Interface ）**

雖然框架中的快取元件沒有遵守 PSR-6 或是 PSR-16，但 CodeIgniter4 團隊提供了一套單獨的轉接器作為補充模組。當然，我們推薦你直接使用 CodeIgniter4 原生的快取驅動，因為轉接器的開發僅僅是為了與第三方程式庫相容。若你需要了解更多訊息，可以造訪 `CodeIgniter4 Cache 儲存庫 <https://github.com/codeigniter4/cache>`_ 。

**PSR-7 ： HTTP訊息介面標準（ HTTP Message Interface ）**

這份 PSR 標準化了與 HTTP 的互動模式，縱使有許多概念已經成為 CodeIgniter HTTP 層的一部分，但 CodeIgniter 並不追求相容這份建議。

---

如果你發現框架中聲稱符合 PSR 規範的部分，並非按照正確的規範實作時，敬請告知，我們將對其進行修復，或者是至 github 送出拉取請求。
