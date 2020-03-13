# codeIgniter4-taiwan-User-Guide
CodeIgniter 4 Traditional Chinese(Taiwan) User Guide - codeigniter4 繁體中文(台灣)使用手冊翻譯計畫

[查閱最新使用手冊](https://monkenwu.github.io/codeIgniter4-taiwan-User-Guide/)

[查閱最新翻譯認領清單](https://github.com/monkenWu/codeIgniter4-taiwan-User-Guide/wiki/%E7%BF%BB%E8%AD%AF%E8%AA%8D%E9%A0%98%E6%B8%85%E5%96%AE)

## 風格指南
中文翻譯的文字排版請以 [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines) 為主。

### 參考
繁體中文翻譯請盡量遵守 [CodeIgniter3 使用手冊](https://codeigniter.org.tw/userguide3/) 的翻譯風格。

簡體中文版本的 [CodeIgniter4 使用手冊](https://github.com/CodeIgniter-Chinese/codeigniter4-user-guide) 也正在翻譯中，可以參考其翻譯但不接受直接以簡體中文轉換為繁體中文。請注意，中國與台灣在程式開發專業詞彙上的譯名大不相同。

### 提交規範

#### 若你想替翻譯盡一份心力，可以怎麼做？
1. fork 一份完整的程式碼。
2. 至[翻譯認領串](https://github.com/monkenWu/codeIgniter4-taiwan-User-Guide/issues/1)認領欲翻譯的檔案。
3. 在主線上產出一條屬於你的分支。
4. 翻譯結束後編譯一次預覽翻譯效果，並且重複校稿。
5. 在分支上 push 你所修改的檔案。
6. 回到 github 發出 pull requests。

#### 簽名
所有的提交的內容都必須要留下您的簽名，比如說：``John Doe <johndoe@example.com>``，其設定方式與你使用 git 的媒介有所不同，請務必設定完成後再進行提交。

## 如何部屬
CodeIgniter 使用手冊基於 Sphinx 進行撰寫，它可以管理文檔並且輸出成各種格式。頁面以人類可讀的 [eStructuredText](http://sphinx.pocoo.org/rest.html)
格式進行撰寫。

### 首要條件
Sphinx 需要 Python 的支援，如果你的作業系統是 OS X 或是 Ubuntu ，則內建了 Python 。你可以在 Terminal 中以不附帶任何參數的方式執行 ``Python`` ，它會載入並顯示你的 Python 版本。如果你使用的版本並非 2.7.* ，請額外或重新安裝一組。

### 安裝
1. 安裝 [easy_install](http://peak.telecommunity.com/DevCenter/EasyInstall#installing-easy-install) 
2. 執行 ``easy_install "sphinx==1.4.5"``
3. 執行 ``easy_install "sphinxcontrib-phpdomain==0.7.0"``
4. 執行 ``easy_install "jieba==0.42.1"``
4. 安裝 CI Lexer ，它的功能是在代碼範例中突出顯示 PHP 、 HTML 、 CSS ，以及 JavaScript 語法（請參閱　cilexer / README　）
5. 執行 ``cd <專案根目錄>``
6. 執行 ``make html`` 產出 HTML 。

### 編輯與創建文檔
所有原始檔案都存放在 ``source /`` 底下，你可以新增新的文檔或修改已有的文檔。就和修改程式碼一樣，我們建議新的內容都是以分支的方式新增，並且向此 Repository 發出  pull requests 。

### HTML在哪裡呢？
Sphinx 透過 ``source /`` 底下的文件自動生成使用者最終看到的網頁。當然，對於自動生成的 HTML 進行版本控制是沒有意義的，所以它並不在版本控制之下。你如果想要預覽翻譯後的效果，可以重新透過 Sphinx 自動產生新的 HTML 文件。首先，你必須先 `` cd <專案根目錄> ``  進入專案的根目錄，再執行上節所敘述的最後一個步驟  ``make html`` 。

執行步驟後，你將會看到編譯中的提示，待編譯成功後，自動產生的使用手冊以及圖片都會位於 ``build/html/`` 底下。 Sphinx 經過第一次編譯後，第二次再執行編譯將只針對修改的文件進行重新編譯，非常節省時間。但如果你想全部重新編譯的話，只需要刪除 ``build/`` 整個資料夾後重新編譯即可。
