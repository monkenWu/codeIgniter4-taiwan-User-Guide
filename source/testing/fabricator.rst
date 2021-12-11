####################
產生測試資料
####################

通常，你會需要替你的應用程式提供範例資料以執行測試。 ``Fabricator`` 類別使用 fzaninotto 所開發的 `Faker 程式庫 <https://github.com/fzaninotto/Faker//>`_ ，以此將模型變成亂數資料的產生器。使你能夠在資料填充或測試案例中，使用資料偽裝器來為你的單元測試提供偽裝資料。

.. contents::
    :local:
    :depth: 2

支援的模型
================

``Fabricator`` 支援任何繼承了框架核心 ``CodeIgniter\Model`` 的模型。你可以使用自訂模型並實作 ``CodeIgniter\Test\Interfaces\FabricatorModel`` 介面。

::

    class MyModel implements CodeIgniter\Test\Interfaces\FabricatorModel

.. note:: 
    除了方法之外，這個介面還宣告了一些模型的必要屬性，相關細節請參閱介面的程式碼。

載入資料偽裝器
===================

在最基本的條件下，資料偽裝器將使用模型作為基本操作：

::

    use App\Models\UserModel;
    use CodeIgniter\Test\Fabricator;

    $fabricator = new Fabricator(UserModel::class);

這個參數可以是模型名稱的字串，也可以是模型的實體。

::

    $model = new UserModel($testDbConnection);

    $fabricator = new Fabricator($model);

定義格式器
===================

Faker 向格式器請求來產生資料，在沒有定義格式器的情況下， ``Fabricator`` 將嘗試根據欄位名稱以及他所表示的模型屬性來猜測最適合的格式，並返回到 ``$fabricator->defaultFormatter`` 。如果你的欄位名稱與常用格式器相對應，又或者你不太在乎欄位的內容，沒有定義格式器可能不會帶來什麼壞處。但在大多數的情形下，你會希望指定一個格式器作為建構函數的第二個參數：

::

    $formatters = [
        'first'  => 'firstName',
        'email'  => 'email',
        'phone'  => 'phoneNumber',
        'avatar' => 'imageUrl',
    ];

    $fabricator = new Fabricator(UserModel::class, $formatters);

你還可以使用 ``setFormatters()`` 方法，在初始化資料偽裝器後改變它的格式器。

**進階格式化**

有時，格式器的預設回傳是不夠的。Faker 提供大多數格式器的參數，讓你能夠進一步限制隨機資料的範圍。資料偽裝器將檢查並使用你所傳入的模型中宣告的 ``fake()`` 方法，在這個方法裡你將可以準確地描述偽裝的資料所呈現的是什麼樣子：

::

  class UserModel
    {
        public function fake(Generator &$faker)
        {
            return [
                'first'  => $faker->firstName,
                'email'  => $faker->email,
                'phone'  => $faker->phoneNumber,
                'avatar' => Faker\Provider\Image::imageUrl(800, 400),
                'login'  => config('Auth')->allowRemembering ? date('Y-m-d') : null,
            ];
        }

請注意，這個例子的前三個資料與之前所使用的格式器是相同的。但對於 ``avatar`` 我們請求了一個不同於預設的圖像大小， ``login`` 使用基於應用程式組態的條件。這兩種情況都不能使用 ``$formatters`` 。此時你可能會希望將測試資料的定義與正是的模型分開管理，因此在你的測試 ``support`` 資料夾中定義一個子類別是一個很好的做法：

::

    namespace Tests\Support\Models;

    class UserFabricator extends \App\Models\UserModel
    {
        public function fake(&$faker)
        {

本土化
============

Faker 支援許多不同的地區設定。請查閱 Faker 文件以確定哪些提供者支援你的地區。在建構函數的第三個參數中你可以指定區域設定：

::

    $fabricator = new Fabricator(UserModel::class, null, 'fr_FR');

如果未指定地區，它將使用在 **app/Config/App.php** 中的地區設定作為 ``defaultLocale`` 。你可以使用資料偽裝器的 ``getLocale()`` 方法檢查區域設定。

偽裝資料
===============

當你正確初始化了一個資料偽裝器，就可以很輕鬆地使用 ``make()`` 產生測試資料：

::

    $fabricator = new Fabricator(UserFabricator::class);
    $testUser   = $fabricator->make();
    print_r($testUser);

你會得到像是這樣的東西：

::

    array(
        'first'  => "Maynard",
        'email'  => "king.alford@example.org",
        'phone'  => "201-886-0269 x3767",
        'avatar' => "http://lorempixel.com/800/400/",
        'login'  => null,
    )

你也可以透過給予一個數量取得更多結果：

::

    $users = $fabricator->make(10);

``make()`` 的回傳型別會參考模型的定義，但你也可以直接使用以下方法強制使用一個型別：

::

    $userArray  = $fabricator->makeArray();
    $userObject = $fabricator->makeObject();
    $userEntity = $fabricator->makeObject('App\Entities\User');

從 ``make()`` 回傳的結果可以在測試中使用或插入至你的資料庫中。另外， ``Fabricator`` 包含 ``create()`` 指令，能夠替你插入資料至資料庫，由於模型回呼、資料庫格式，或主鍵與時間戳記等特殊鍵值的關係， ``create()`` 所回傳的結果可能會與 ``make()`` 不同。你可能會得到類似於下面的回傳：

::

    array(
        'id'         => 1,
        'first'      => "Rachel",
        'email'      => "bradley72@gmail.com",
        'phone'      => "741-241-2356",
        'avatar'     => "http://lorempixel.com/800/400/",
        'login'      => null,
        'created_at' => "2020-05-08 14:52:10",
        'updated_at' => "2020-05-08 14:52:10",
    )

與 ``make()`` 相似，你可以透過提供一個數量來插入並回傳一個由物件組成的陣列：

::

    $users = $fabricator->create(100);


最後，有時你可能需要使用完整的資料庫物件進行測試，但實際上你並沒有使用資料庫。 ``create()`` 可以透過第二個參數來模擬資料庫物件，回傳帶有上述額外資料庫欄位的物件，而不需要實際接觸資料庫：

::

    $user = $fabricator(null, true);

    $this->assertIsNumeric($user->id);
    $this->dontSeeInDatabase('user', ['id' => $user->id]);

指定測試資料
====================

資料能夠自動產生真的很便利，但你可能會希望在不影響格式化組態的情況下，替你的測試提供特定的欄位，而不是替每個變體建立一個新的資料提供者。這時，你就可以使用 ``setOverrides()`` 來覆寫指定欄位的數值：

::

    $fabricator->setOverrides(['first' => 'Bobby']);
    $bobbyUser = $fabricator->make();

現在，使用 ``make()`` 或 ``create()`` 產生的任何資料一定會以 "Bobby" 做為 ``first`` 欄位的內容：

::

    array(
        'first'  => "Bobby",
        'email'  => "latta.kindel@company.org",
        'phone'  => "251-806-2169",
        'avatar' => "http://lorempixel.com/800/400/",
        'login'  => null,
    )

    array(
        'first'  => "Bobby",
        'email'  => "melissa.strike@fabricon.us",
        'phone'  => "525-214-2656 x23546",
        'avatar' => "http://lorempixel.com/800/400/",
        'login'  => null,
    )

可以傳入第二個參數到 ``setOverrides()`` 方法中，用來控制覆寫的動作是持續覆寫還是僅覆寫單次。

::

    $fabricator->setOverrides(['first' => 'Bobby'], $persist = false);
    $bobbyUser = $fabricator->make();
    $bobbyUser = $fabricator->make();

請注意，在第一次的結果回傳後，資料偽裝器將會停止覆寫。

::

    array(
        'first'  => "Bobby",
        'email'  => "belingadon142@example.org",
        'phone'  => "741-857-1933 x1351",
        'avatar' => "http://lorempixel.com/800/400/",
        'login'  => null,
    )

    array(
        'first'  => "Hans",
        'email'  => "hoppifur@metraxalon.com",
        'phone'  => "487-235-7006",
        'avatar' => "http://lorempixel.com/800/400/",
        'login'  => null,
    )

如果你並未提供第二個參數，在預設的情型下數值將不會改變。

測試輔助函數
============

通常，你只需要一個用於測試的一次性偽裝物件。測試輔助函數提供了 ``fake($model, $overrides, $persist = true)`` 函數來幫忙你達成這個需求：

::

    helper('test');
    $user = fake('App\Models\UserModel', ['name' => 'Gerry']);

這相當於你這麼做：

::

    $fabricator = new Fabricator('App\Models\UserModel');
    $fabricator->setOverrides(['name' => 'Gerry']);
    $user = $fabricator->create();

如果你只是需要一個偽裝物件，而不會將它儲存到資料庫中，你也可以將 false 傳入至 persist 參數中。

資料表計數
============

通常你的偽裝資料將會依賴於其他偽裝資料， ``Fabricator`` 能夠讓你計算每個資料表建立的偽裝項目的數量，你可以參考以下範例：

你的專案擁有使用者與群組，在你的測試案例中，你想用不同規模的群組來搭建各種場景，所以你利用 ``Fabricator`` 建立了一堆群組。現在，在你想替這些群組建立偽裝的使用者，但不想把他們分配到部存在的群組 ID ，你的模型中的偽裝方法可能會如下所示：

::

    class UserModel
    {
        protected $table = 'users';

        public function fake(Generator &$faker)
        {
            return [
                'first'    => $faker->firstName,
                'email'    => $faker->email,
                'group_id' => rand(1, Fabricator::getCount('groups')),
            ];
        }

現在，新的使用者的建立將保證是有效群組的一份子：　``$user = fake(UserModel::class);``

``Fabricator`` 會在內部處理計數，但你也可以存取下列靜態方法來進行控制：

**getCount(string $table): int**

回傳特定資料表的目前數值（預設為 0 ）。

**setCount(string $table, int $count): int**

手動設定特定資料表的數值，例如，你在沒有使用資料偽裝器的情況下，建立了一些測試項目，但你仍希望把這個動作算入最終計數中。

**upCount(string $table): int**

將特定資料表的數值加 1 ，並回傳新的數值。（這被 ``Fabricator::create()`` 方法所使用）。

**downCount(string $table): int**

將特定資料表的數值減 1 併回傳新的數值，例如，如果你刪除了一個偽裝項目但想要追蹤更改。

**resetCounts()**

重置所有計數。 在測試案例之間呼叫它是個好選擇（儘管使用了  ``CIUnitTestCase::$refresh = true`` 會自動執行）。
