#####################
使用實體類別
#####################

CodeIgniter 支援使用實體類別作為資料庫的第一類物件，同時讓它們保持可選擇是否使用。它們通常作為儲存庫模式的一部份，如果這更符合你的需求，也可以直接與 :doc:`模型 </models/model>` 一起使用。

.. contents::
    :local:
    :depth: 2

實體用法
============

就其核心而言，實體類別只是一個表示單個資料庫列的類別。它具有類別屬性來表示資料庫的列，並提供額外的方法來實現該列的商業邏輯。不過它的核心特徵是不能保存自己，因為這是模型或資料庫類別的工作。也就是說，如果保存物件的方式有任何改變，就不必修改使用到這個物件的所有應用程式。這使得你可以在快速雛形階段使用 JSON 或 XML 檔案來儲存物件，當你證明了這個概念可行時，再輕鬆地切換到資料庫模式。

我們來看看這個簡單的使用者實體，以及我們如何使用這個實體的說明。

假設你有一個名為 ``users`` 的資料庫資料表，它的綱目如下：

::

    id          - integer
    username    - string
    email       - string
    password    - string
    created_at  - datetime

建立實體類別
-----------------------

現在，我們將新建一個實體類別。因為在框架中沒有預設的位置來儲存這些類別，而它與現有的資料夾結構不相容，所以我們新建一個新的 **app/Entities** 目錄，並在其中新建 **app/Entities/User.php** 檔案，並包含以下內容：

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        //
    }


雖然待會我們將讓它變得更加有用，但在最簡單的情形下，這就是你唯一要做的。

建立模型
----------------

首先，我們新建一個　 **app/Models/UserModel.php** 模型檔案，讓我們與它進行交互：

::

    <?php namespace App\Models;

    use CodeIgniter\Model;

    class UserModel extends Model
    {
        protected $table         = 'users';
        protected $allowedFields = [
            'username', 'email', 'password'
        ];
        protected $returnType    = 'App\Entities\User';
        protected $useTimestamps = true;
    }

這個模型使用了資料庫中的 ``users`` 資料表進行所有活動。並且我們設定 ``$allowedFields`` 屬性來闡述所有允許被改變的外部欄位。``id`` 、 ``created_at`` 與 ``updated_at`` 等欄位是由類別或資料庫自動處理的，所以它並不在我們允許被改變的欄位名單中。最後，我們將 ``$returnType`` 宣告為實體類別的命名空間，這可以讓模型上所有成功的資料庫查詢產生資料的回傳，都會回傳我們所定義的使用者實體類別的實體（ instead ），而不是單純的物件或是陣列。

使用實體類別
-----------------------------

現在，你準備好了前置工作，請以操作其他類別的方式操作實體類別：

::

    $user = $userModel->find($id);

    // Display
    echo $user->username;
    echo $user->email;

    // Updating
    unset($user->username);
    if (! isset($user->username)
    {
        $user->username = 'something new';
    }
    $userModel->save($user);

    // Create
    $user = new App\Entities\User();
    $user->username = 'foo';
    $user->email    = 'foo@example.com';
    $userModel->save($user);

你可能已經注意到了，使用者實體類別並沒有為資料列設定任何屬性，但你仍然可以把它們作為公開屬性存取。在基本類別中， **CodeIgniter\Entity** 替你解決了這個問題，它還擁有 **isset()** 與 **unset()** 檢查屬性的能力，並追蹤物件新建或從資料庫中提取物件來比對那些資料列已被更改。

當 User 實體類別被傳遞給模型的 **save()** 方法時，它會自動讀取實體內的屬性，判斷這是次的 save() 是插入新記錄還是更新現有記錄，並將資料更新到被  **$allowedFields**  允許的欄位中。

快速填充屬性
--------------------------

實體類別還提供了一個方法 ``fill()`` ，它可允許你將一個鍵值陣列傳入其中，用來填充實體類別的屬性。陣列中的任何屬性都將被設定在實體中，但是，當透過模型保存實體內容時，只有 $allowedFields 中允許的欄位會被實際儲存在資料庫中，所以你可以在實體上儲存額外的資料，而不並擔心不相干的欄位會被存入資料庫。

::

    $data = $this->request->getPost();

    $user = new App\Entities\User();
    $user->fill($data);
    $userModel->save($user);

你也可以在建構函數中傳遞資料，在實體化（ Instantiation ）實體類別的過程中，資料會透過 `fill()` 方法傳遞資料。

::

    $data = $this->request->getPost();

    $user = new App\Entities\User($data);
    $userModel->save($user);

處理商業邏輯
=======================

雖然上述的範例很方便，但它們並不能幫助任何商業邏輯的執行。基本的實體類別實作了一些聰明的 ``__get()`` 與 ``__set()`` 方法，這些方法將會檢查特殊方法並使用它們，避免直接使用屬性，從而允許你強制實行所需的商業邏輯或資料轉換。

下面將提到如何更新 User 實體，並提供了如何使用的範例：

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;
    use CodeIgniter\I18n\Time;

    class User extends Entity
    {
        public function setPassword(string $pass)
        {
            $this->attributes['password'] = password_hash($pass, PASSWORD_BCRYPT);

            return $this;
        }

        public function setCreatedAt(string $dateString)
        {
            $this->attributes['created_at'] = new Time($dateString, 'UTC');

            return $this;
        }

        public function getCreatedAt(string $format = 'Y-m-d H:i:s')
        {
            // Convert to CodeIgniter\I18n\Time object
            $this->attributes['created_at'] = $this->mutateDate($this->attributes['created_at']);

            $timezone = $this->timezone ?? app_timezone();

            $this->attributes['created_at']->setTimezone($timezone);

            return $this->attributes['created_at']->format($format);
        }
    }

首先要注意的是我們所新增的方法名稱，對於每個方法，基本實體類別希望你將 snake_case 資料列名稱轉換為 PascalCase ，並以 ``set`` 與 ``get`` 作為前綴。每當你使用了直接語法（例如： $user->email ）設定或檢索類別屬性時，這些方法就會被自動呼叫。這些方法不需要是公開的，除非你想從其他的類別中呼叫它們，例如： ``created_at`` 類別屬性將可以透過 ``setCreatedAt()`` 與 ``getCreatedAt()`` 這兩個方法存取。

.. note:: 上述功能只在試圖從類別外部存取才會起作用，任何類別內部的方法必須直接呼叫 ``setX()`` 以及 ``getX()`` 方法。

在 ``setPassword()`` 方法中，我們能夠保證密碼是被雜湊過的。

在 ``setCreatedAt()`` 方法中，我們將從模型中接受到的字串轉換成一個 DateTime 物件，保證我們為 UTC 時區，這樣就能輕易轉換檢視器目前的時區。在 ``getCreatedAt()`` 方法中，它會將時間轉換為應用程式目前時區的格式化字串。

雖然實作的過程很簡單，但透過這些例子則表明，使用實體類別可以提供一個極度靈活的方式來執行商業邏輯，並創建讓人愉悅使用的物件。

::

    // 自動雜湊密碼，兩者的作用是相同的
    $user->password = 'my great password';
    $user->setPassword('my great password');

資料映射
============

在你的職業生涯中，很多時候你可能會遇到這樣子的狀況：應用程式的用途發生了變化，資料庫中原來的資料列名稱的意義發生改變。或者是，你發現了你的程式碼風格偏向使用駝峰式命名的類別屬性，而你的資料庫卻要求你使用 snake_case （每個單字間以下底線分隔）進行命名。這些時候都可以透過實體類別，輕鬆地進行映射處理。

透過一個例子來示範，想像一下你有一個簡單的使用者實體，它在整個應用程式中被使用：

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $attributes = [
            'id' => null,
            'name' => null,        // Represents a username
            'email' => null,
            'password' => null,
            'created_at' => null,
            'updated_at' => null,
        ];
    }

你的老闆突然告訴你，現在沒有人使用 "usernames" 了啦，我需要你將它改成電子信箱登入。但他還表示，她希望可以對應用程式進行個人化設定，因此他想要你改變名稱欄位的用途，讓 ``name`` 欄位用來表示使用者全名，而不是像以前那樣。為了保持整潔以，並確保這個欄位在資料庫中繼續保持著某種意義，你需要使用資料庫遷移，並將欄位重新命名為 ``full_name`` 。

先別想這個讓人為難的例子，我們現在有兩個選項可以修正使用者類別。我們可以將類別屬性從 ``$name`` 改成 ``$full_name`` ，但這需要修改整個應用程式才行。反之，我們可以簡單地將資料庫中的 ``full_name`` 欄位映射到 ``$name`` 屬性，就可以完成對實體的修改。

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $attributes = [
            'id' => null,
            'name' => null,        // Represents a username
            'email' => null,
            'password' => null,
            'created_at' => null,
            'updated_at' => null,
        ];

        protected $datamap = [
            'full_name' => 'name'
        ],
    }

透過在 ``$datamap`` 陣列中加入我們新的資料庫欄位名稱，等於是告訴類別說：「資料庫中的資料列應該透過甚麼屬性進行存取」。陣列中的鍵是資料庫中的資料列名稱，值則是要它所映射的類別屬性。

在這個範例中，當模型在使用者類別上設定 ``full_name`` 欄位時，實際上是將這個值賦值至 ``$name`` 屬性，所以可以透過 ``$user->name`` 來進行存取。這個值仍然可以使用 ``$user->full_name`` 進行存取，因為模型需要透過這個來得到資料並將它儲存在資料庫中。但要注意， ``unset`` 與 ``isset`` 只對映射到的 ``$name`` 屬性起作用，而不是對原始名稱 ``full_name`` 起作用。 

修改器
========

資料修改器
-------------

在預設的情形下，實體類別將會在設定或檢索時將命名為 `created_at` 、 `updated_at` ， 以及 `deleted_at` 的欄位轉換為 :doc:`時間與日期程式庫 </libraries/time>` 的實體（instances），這個程式庫將以一種不變的、當地語系化的方式提供大量有用的方法。

你可以透過將名稱添加到 **options['dates']** 陣列來定義那些屬性會被自動轉換：

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $dates = ['created_at', 'updated_at', 'deleted_at'];
    }

現在，上述提到的任何一個屬性被你囊括在陣列中，正如 **app/Config/App.php** 設定的那樣，它們將使用應用程式的所在時區，並被轉換成一個時間與日期程式庫的實體：

::

  $user = new App\Entities\User();

    // 轉換為時間實體
    $user->created_at = 'April 15, 2017 10:30:00';

    // 現在可以使用任何使間與日期程式庫的方法:
    echo $user->created_at->humanize();
    echo $user->created_at->setTimezone('Europe/London')->toDateString();

型別轉換
----------------

你可以指定在實體中 **成員** 屬性應該強制被轉換成你指定的資料型別，這個選項應該是一個鍵值陣列，其中的鍵是屬性名稱，值是它應該要被強制轉換成的資料型別。強制轉換只在取值時影響，並不會轉換在實體或資料庫中的永久值。屬性可以強制轉換為下列數種資料型別：**integer** 、  **float** 、  **double** 、  **string** 、  **boolean** 、  **object** 、  **array** 、  **datetime** ， 以及 **timestamp**。在屬性前加入問號，可將其標註為 nullable ，例如： **?string** 或 **?integer** 。
　
例如：你有一個具有 **is_banned** 屬性的使用者實體，你可以把它轉換為 boolean ：

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $casts = [
            'is_banned' => 'boolean',
            'is_banned_nullable' => '?boolean'
        ],
    }

Array/Json 轉換
------------------

Array/Json 的轉換對於儲存序列化的陣列或 json 欄位相當有用，當轉換為：

* **array** ，它們將自動取消序列化。
* **json** ，它們將自動設定為 json_decode($value,false) 的值。
* **json-array** ，它們將自動設定為 json_decode($value, true) 的值。

而讀取屬性的數值時，不像其他的資料型別你可以將屬性投射到：

* **array** 強制型別轉換序列化。
* **json** 與 **json-array** 強制轉換將在設定時對數值使用 json_encode 函數。

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $casts => [
            'options' => 'array',
		    'options_object' => 'json',
		    'options_array' => 'json-array'
        ];
    }

    $user    = $userModel->find(15);
    $options = $user->options;

    $options['foo'] = 'bar';

    $user->options = $options;
    $userModel->save($user);

檢查類別屬性是否變更
-------------------------------

你可以檢查一個實體的屬性在創建後始否發生了變化，這個方法唯一的參數就是你所想檢查的屬性名稱：

::

    $user = new User();
    $user->hasChanged('name');      // false

    $user->name = 'Fred';
    $user->hasChanged('name');      // true

或者省略這個參數，將會檢查整個實體是否發生了變化

::

    $user->hasChanged();            // true
