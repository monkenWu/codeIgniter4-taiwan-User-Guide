新聞頁面
###############################################################################

在上一個條目中，我們撰寫了引用靜態頁面的類別，也介紹了框架的一些基本概念。我們透過新增自訂路由規則獲得更加整潔的 URL 。是時候引入動態內容並且開始操作資料庫了！

建立要使用的資料庫
-------------------------------------------------------

CodeIgniter 假設你使用了所支援資料庫，如 :doc:`系統需求 </intro/requirements>` 所提到的。在本教學中，我們提供了基於 MySQL 資料庫的 SQL 程式碼。我們也假設你有一個用於資料庫管理的軟體（ mysql 、 MySQL Workbench 或 phpMyAdmin ）。

你需要創建一個這個教學需要使用到的資料庫，然後在 CodeIgniter 設定使用它。

使用資料庫管理軟體，連接到資料庫並且執行下列 SQL 命令（ MySql ），以及添加一些合適的測試資料。以下提到的將是創建 SQL 資料表所需的語句，但你應該要知道，在熟悉了 CodeIgniter 之後，就可以使用程式設計的方式完成這個操作。你可以閱讀有關於 :doc:`遷移 <../dbmgmt/migration>` 與 :doc:`資料填充 <../dbmgmt/seeds>` 的資訊，以便於創建更有用的資料庫設定。

::

    CREATE TABLE news (
        id int(11) NOT NULL AUTO_INCREMENT,
        title varchar(128) NOT NULL,
        slug varchar(128) NOT NULL,
        body text NOT NULL,
        PRIMARY KEY (id),
        KEY slug (slug)
    );

註：slug 是在 Web 發布內容中，面相使用者與 SEO 用於友好地識別與描述資源的簡短文字。

測試資料可能會類似於：

::

    INSERT INTO news VALUES 
    (1,'Elvis sighted','elvis-sighted','Elvis was sighted at the Podunk internet cafe. It looked like he was writing a CodeIgniter app.'),
    (2,'Say it isn\'t so!','say-it-isnt-so','Scientists conclude that some programmers have a sense of humor.'),
    (3,'Caffeination, Yes!','caffeination-yes','World\'s largest coffee shop open onsite nested coffee shop for staff only.');

連接資料庫
-------------------------------------------------------

請打開在佈署 CodeIgniter 時所創建的環境設定檔 ``.env`` ，在設定檔中取消資料庫相關設定的註解，並且依據要連入的資料庫進行相關設定。

::

    database.default.hostname = localhost
    database.default.database = ci4tutorial
    database.default.username = root
    database.default.password = root
    database.default.DBDriver = MySQLi

設定模型
-------------------------------------------------------

資料庫查詢不該在控制器中實作，而是在模型中進撰寫，以便日後可以輕鬆地重用它們。模型是資料庫與其他資料儲存媒介，檢索、插入和更新資料的地方。它們讓你能夠對資料進行存取。

打開 **app/Models/** 目錄後，新建名為 **NewsModel.php** 且包含下列程式碼的的檔案。在這之前，請確定你已經正確設定了資料庫，就像 :doc:`這裡 <../database/configuration>` 所說的。

::

    <?php

    namespace App\Models;

    use CodeIgniter\Model;

    class NewsModel extends Model
    {
        protected $table = 'news';
    }

這些程式瑪與前一個條目的控制器程式碼類似，這個新創建的模型繼承了 ``CodeIgniter\Model`` 類別以及載入了資料庫函數庫。與資料庫類別的溝通可以呼叫 ``$this->db`` 物件達成。

現在，資料庫與模型已經設定完畢，你需要一種方法從資料庫獲取所有的新聞內容。因此，我們使用 CodeIgniter :doc:`查詢產生器（ Query Builder ） <../database/query_builder>` 類別，這其中包含著資料庫的抽象層。透過查詢產生器我們可以輕鬆撰寫查詢，並使他們在 :doc:`所有支援的資料庫系統 <../intro/requirements>` 穩定工作。模型類別還允許使用查詢產生器輕鬆建立查詢，並提供了一些工具來簡化資料處理的流程。將下列程式碼加入剛才我們創建的模型中。

::

    public function getNews($slug = false)
    {
        if ($slug === false) {
            return $this->findAll();
        }

        return $this->where(['slug' => $slug])->first();
    }

這個程式碼包含兩個不同的查詢，你可以得到所有的新聞紀錄，或者是按條獲取。你可能已經注意到，在執行查詢之前我們尚未進行變數處理。 :doc:`查詢產生器 <../database/query_builder>` 會替您執行這個操作，使你的資料庫查詢更加安全。

此處所使用的兩種方法 ``findAll()`` 與 ``first()`` 由 Model 類別提供。它們藉由我們在 **NewsModel** 類別中設定的 ``$table`` 屬性知道要使用的資料表。它們是輔助方法，使用查詢產生器在當前的資料表上執行命查詢，並且以你所選擇的格式回傳結果陣列。在這個範例中， ``findAll()`` 回傳一個物件陣列。

顯示新聞
-------------------------------------------------------

現在，查詢已經撰寫完成，模型應該要綁定在向使用者顯示新聞項目的視圖。這可以在之前創建的 ``Pages`` 控制器中完成，但為了清楚起見，我們定義一個新的控制器。在 *app/Controllers/News.php* 路徑上使用以下程式碼創建新的 ``News`` 控制器檔案。 

::

    <?php

    namespace App\Controllers;

    use App\Models\NewsModel;
    use CodeIgniter\Controller;

    class News extends Controller
    {
        public function index()
        {
            $model = model(NewsModel::class);

            $data['news'] = $model->getNews();
        }

        public function view($slug = null)
        {
            $model = model(NewsModel::class);

            $data['news'] = $model->getNews($slug);
        }
    }


看看程式碼，你可能會發現與我們之前建立的文件有相似之處。首先，它繼承了 CodeIgniter 核心類別 ``Controller`` ，這個類別提供了幾個輔助方法，並且確保你可以使用當前的 ``Request`` 與 ``Response`` 物件。以及將運作資訊保存在伺服器的 ``Logger`` 類別。

接下來， ``model()`` 函數被用於建立 NewsModel 實體。這是一個輔助函數，你可以在 :doc:`這裡 </general/common_functions>` 閱讀到更多說明。若你不想這麼使用，你也可以寫成 ``$model = new NewsModel();`` 。

 ``$slug`` 變數在第二個方法中傳遞給模型，而模型也使用 slug 回傳相應的新聞。

現在，控制器透過我們的模型檢索資料，但尚未顯示任何資料。接下來我們得將這些資料傳遞給視圖。將  ``index()`` 修改成向下面這樣。

::

    public function index()
    {
        $model = model(NewsModel::class);

        $data = [
            'news'  => $model->getNews(),
            'title' => 'News archive',
        ];

        echo view('templates/header', $data);
        echo view('news/overview', $data);
        echo view('templates/footer', $data);
    }

上面的程式碼從模型獲得所有的新聞紀錄後，將它分配給變數。標題的值被宣告在 ``$data['title']`` 之中，所有資料都會傳送給視圖。現在需要創建一個視圖來呈現新聞畫面，在 **app/Views/news/overview.php** 中創建擁有以下程式碼的檔案。

::

    <h2><?= esc($title) ?></h2>

    <?php if (! empty($news) && is_array($news)): ?>

        <?php foreach ($news as $news_item): ?>

            <h3><?= esc($news_item['title']) ?></h3>

            <div class="main">
                <?= esc($news_item['body']) ?>
            </div>
            <p><a href="/news/<?= esc($news_item['slug'], 'url') ?>">View article</a></p>

        <?php endforeach; ?>

    <?php else: ?>

        <h3>No News</h3>

        <p>Unable to find any news for you.</p>

    <?php endif ?>

.. note::
    我們再次使用了 ``esc()`` 來防止 XSS 攻擊，但這次我們將「url」作為 ``esc()`` 的第二個參數。這是因為攻擊的模式是不同的，取決於輸出使用的語境。你可以在 :doc:`這裡 </general/common_functions>` 閱讀到更多的資訊。

在這裡每個專案將會透過迴圈產生並且顯示給使用者。你可以看到我們在 PHP 中撰寫了樣板，並且讓它與 HTML 混合。如果你更喜歡使用 樣板語法，則可以使用 CodeIgniter 的 :doc:`視圖器解析器 </outgoing/view_parser>` 。或置入任何你喜歡的解析器進入 CodeIgniter 。

新聞概述的畫面已經完成，但顯示單個新聞的頁面我們還沒做完。剛才我們創建建的模型中有個方法可以輕鬆地用於這個需求。你只需要向控制器新增一些程式碼，並且創建新的視圖。讓我們回到 ``News`` 控制器，並且更新 ``view()`` 方法。

::

    public function view($slug = null)
    {
        $model = model(NewsModel::class);

        $data['news'] = $model->getNews($slug);

        if (empty($data['news'])) {
            throw new \CodeIgniter\Exceptions\PageNotFoundException('Cannot find the news item: ' . $slug);
        }

        $data['title'] = $data['news']['title'];

        echo view('templates/header', $data);
        echo view('news/view', $data);
        echo view('templates/footer', $data);
    }


為了要取得特定的新聞項目，我們得向 ``getNews()`` 方法傳遞 ``$slug`` 變數。剩下的工作只剩在 **app/Views/news/view.php** 中創下擁有下列程式碼的視圖。

::

    <h2><?= esc($news['title']) ?></h2>
    <p><?= esc($news['body']) ?></p>

路由
-------------------------------------------------------

由於前面創建了萬用字元路由規則，因此你需要額外的路由來查看剛剛創建的控制器。新增以下設定到路由設定檔（ **app/config/routes.php** ），這樣可以確保瀏覽器請求可以送到 ``News`` 控制器。而不是直達 ``Pages`` 控制器。第一行路由將會把 slug 傳遞到 ``News`` 控制器的 ``view()`` 方法。

::

	$routes->get('news/(:segment)', 'News::view/$1');
	$routes->get('news', 'News::index');
	$routes->get('(:any)', 'Pages::showme/$1');

將瀏覽器指向你的新聞頁面，即造訪 ``localhost:8080/news`` 。你應該可以看到新聞的清單，每條新聞都有一個連結，它將只顯示一篇文章。

.. image:: ../images/tutorial2.png
    :align: center