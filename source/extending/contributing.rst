###########################
為 CodeIgniter 貢獻
###########################

CodeIgniter 是一個社群促使開發的專案，接受來自社群的程式碼與使用手冊的貢獻。這些貢獻以發出問題或 `拉取請求 <https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests>`_ 的形式在 GitHub 上的 `CodeIgniter4 儲存庫 <https://github.com/codeigniter4/CodeIgniter4>`_ 中提交。

發出問題是一種快速指出 BUG 的方法，如果你在 CodeIgniter 或 使用手冊發現了錯誤，那麼請你先檢查幾件事：

- 是否從未有人針對這個錯誤發出問題
- 這個問題是否已經被修復了（查看分支或尋找已經被關閉的問題）
- 是不是真的有很明顯的問題需要你去解決？

問題回報是非常用的方法，但更好的方案是透過分叉主儲存庫並提交修改到你自己的副本中，再回到 GitHub 發送拉取請求（這需要你使用 Git 版本控制系統）。

詳情請見儲存庫中的 `貢獻至 CodeIgniter4 <https://github.com/codeigniter4/CodeIgniter4/tree/develop/contributing>`_　部分。

*******
支援
*******

請注意， GitHub 儲存庫並不是回答一般問題的地方！如果你在使用某項功能遇到了非錯誤的困難，你可以：

- 在 `論壇 <http://forum.codeigniter.com/>`_ 上創建主題
- 在 `Slack <https://codeigniterchat.slack.com/>`_ 提出你的問題

如果你確定你的使用方法是否正確，或是你真的發現了 BUG ，煩請先至論壇上詢問。

********
安全性
********

你是否發現了 CodeIgniter 中的安全性問題？

請 *不要* 公開它們，請你發送電子郵件至 security@codeigniter.com ，或在 `HackerOne <https://hackerone.com/codeigniter>`_ 上的頁面傳送問題報告。

如果你真的發現了嚴重的漏洞，我們很願意將這個功勞歸功於你，並紀載至 `ChangeLog <../changelogs/index.html>`_ 上。

****************************
如何撰寫好的問題回報
****************************

使用一個具有描述性的主題，例如：解釋器程式庫無法正確解釋逗號（ parser library chokes on commas ），而不是一個模糊的標題：你的程式碼壞掉了（ your code broke ）。

在一個問題回報中只提出一個問題。

請列出你所知道發生問題的版本（例如： 4.0.1 ）或是組件為何（例如：解釋器）。

解釋一下你預期會有什麼結果，但卻發生了什麼問題。如果有的話也可以提供：錯誤訊息以及堆疊追蹤的資訊。

如果簡短的程式碼塊有助於你解釋錯誤的話，也請附上。你可以使用 pastebin 或 Dropbox 的工具，來幫助你處理較長的程式碼與螢幕截圖——不要將它們直接置於問題報告中。

如果你知道如何解決這個問題，你可以在自己的分叉與分支中這麼做，並且提交一個拉取請求給主儲存庫，而上面的問題報告也請附在拉取請求中。

如果你的問題報告能描述出重現問題的方法，那就太好了！如果你能包含重現問題的單元測試，那就更好了！因為這樣可以幫助修復問題人員可以有更明確的目標。