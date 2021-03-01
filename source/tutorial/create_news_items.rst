建立新聞
###############################################################################

現在，你已經知道如何使用 CodeIgniter 從資料庫中讀取資料，但尚未將任何資訊寫入到資料庫中。在本條目，你將擴充之前創建的新聞控制器與模型以完成此功能。

創建表單
-------------------------------------------------------

你需要製作出一個表單，藉由表單的輸入才能將資料存入資料庫。這代表你需要製作一個包含兩個欄位的表單，一個欄位用於標題、一個欄位用於文章內容。你將從模型中的標題派生出這個 slug 。請在 **app/Views/news/create.php** 新建一個包含下列程式碼的視圖。

::

    <h2><?= esc($title); ?></h2>

    <?= \Config\Services::validation()->listErrors(); ?>

    <form action="/news/create" method="post">
        <?= csrf_field() ?>

        <label for="title">Title</label>
        <input type="input" name="title" /><br />

        <label for="body">Text</label>
        <textarea name="body"></textarea><br />

        <input type="submit" name="submit" value="Create news item" />

    </form>

你可能會覺得 ``\Config\Services::validation()->listErrors()`` 這個函數看起來很陌生。它能用於表單驗證相關的錯誤報告。而 ``csrf_field()`` 函數創建了一個帶有 CSRF 令牌的隱藏輸入框，有助於你預防一些常見的攻擊。

回到你的 ``News`` 控制器。你將在這裡執行：檢查表單是否已經提交、提交的資料是否通過驗證，這兩項操作。我們使用 :doc:`表單驗證 <../libraries/validation>` 程式庫進行驗證。

::

    public function create()
    {
        helper('form');
        $model = new NewsModel();

        if (! $this->validate([
            'title' => 'required|min_length[3]|max_length[255]',
            'body'  => 'required'
        ]))
        {
            echo view('templates/header', ['title' => 'Create a news item']);
            echo view('news/create');
            echo view('templates/footer');

        }
        else
        {
            $model->save([
                'title' => $this->request->getVar('title'),
                'slug'  => url_title($this->request->getVar('title')),
                'body'  => $this->request->getVar('body'),
            ]);
            echo view('news/success');
        }
    }

上面的程式碼增加了很多功能，前幾行負責載入表單輔助函數與新聞模型。之後，透過輔助函數驗證 $_POST 欄位。在這種情形下，標題和文字欄位是必須的。

如上所述， CodeIgniter 具有強大的驗證程式庫，你可以在 :doc:`這裡 <../libraries/validation>` 閱讀更多關於這個程式庫的資訊。

繼續往下看，你可以看到一個條件式，檢查表單驗證是否成功運行。如果沒有成功，則顯示表單；；如果表單提交後，也通過了所有的規則，就會呼叫模型，把新聞的資料傳遞到模型中。這裡你會看到一個新的函數 url\_title() （由 :doc:`URL 輔助函數 <../helpers/url_helper>` 提供），它將你傳遞給它的字串使用破折號替代所有的空格，你便可以獲得一個漂亮的 slug ，這非常適合用於創建 URL 。

在這之後，就會載入一個視圖來顯示成功訊息，新建一個 **app/Views/news/success.php** 檔案並且寫入一些成功訊息。

你可簡單地這麼寫：

::

    News item created successfully. 

更新模型
-------------------------------------------------------

完成了上述工作，剩下的就是將模型給設定好，讓資料可以被正確保存。 ``save()`` 方法將根據主鍵來判斷是否需要插入訊息，若是這一筆資料已存在便會執行更新。在本例中，我們並沒有傳遞 id 給它，所以它將會插入一筆新的紀錄在 **news** 表。

但是，在默認的情形向，模型中的插入與更新方法並不會保存任何資料，因為它不知道那些欄位是可以被安全地更新。我們可以編輯模型，在 ``$allowedFields`` 屬性中宣告可更新地欄位列表。

::

    <?php

    namespace App\Models;

    use CodeIgniter\Model;

    class NewsModel extends Model
    {
        protected $table = 'news';

        protected $allowedFields = ['title', 'slug', 'body'];
    }


這個新的屬性現在包含了允許被更新的欄位，注意到我們省略了 ``id`` 嗎？這是因為你幾乎不需要這樣做，它在資料庫中是一個自動遞增的欄位。這有助於防止 Mass assignment vulnerability 漏洞的發生。如果你的模型正在處理你的時間戳，那麼你也應該將那些時間戳排除在外。

路由
-------------------------------------------------------

在你開始在你的 CodeIgniter 應用程式中添加新聞之前，你必須到 **app/Config/Routes.php** 這個設定檔添加額外的規則，這將可以確保 CodeIgniter 將 ``create`` 視為一個可執行的方法，而不是新聞的 slug 。你可以在 :doc:`這裡 </incoming/routing>` 閱讀更多有關路由的知識。

::

    $routes->match(['get', 'post'], 'news/create', 'News::create');
    $routes->get('news/(:segment)', 'News::view/$1');
    $routes->get('news', 'News::index');
    $routes->get('(:any)', 'Pages::view/$1');

現在將瀏覽器指向你的 CodeIgniter 開發環境，並前往 ``/news/create`` 這個URL添加一些新聞，就可以查看你所添加的不同頁面了！

.. image:: ../images/tutorial3.png
    :align: center
    :height: 415px
    :width: 45%

.. image:: ../images/tutorial4.png
    :align: center
    :height: 415px
    :width: 45%

恭喜你
-------------------------------------------------------

你剛剛完成了你第一個 CodeIgniter4 應用程式！

下圖顯示的是專案的 **app** 資料夾，你所創建的所有文件顯示成綠色字體。兩個你所修改的設定檔案（資料庫與路由設定檔）並沒有改變顏色。

.. image:: ../images/tutorial9.png
    :align: left
