#########################
使用 CodeIgniter 的模型
#########################

.. contents::
    :local:
    :depth: 2

模型
======

模型提供一種與資料庫中特定資料表交互的方法，模型也內建輔助方法，讓你在與資料庫資料表交互的過程中有標準的方法可以呼叫，包括查找記錄、更新記錄，刪除記錄等等。

存取模型
================

模型通常會儲存在 ``app/Models`` 目錄下，通常它們會有一個與它們所在位置相同的命名空間，比如這個命名空間： ``namespace App\Models`` 。

你可以透過創造一個新的實體或是使用 ``model()`` 輔助函數來造訪類別中的模型。

::

    // 手動建立新類別
    $userModel = new \App\Models\UserModel();

    // 使用模型函數建立新類別
    $userModel = model('App\Models\UserModel', false);

    // 使用模型函數共享模型實體
    $userModel = model('App\Models\UserModel');

    // 使用你所提供的資料庫連接建立共享實體
    // 如果沒有傳入命名空間那它將會搜索所有的命名空間
    // 系統將會嘗試定位 UserModel 類別
    $db = db_connect('custom');
    $userModel = model('UserModel', true, $db);

CodeIgniter 的 Model
=======================

CodeIgniter 支援了模型類別，它提供了一些很好的功能，包括：

- 自動連接資料庫
- 基本的 CRUD 方法
- 模型內驗證
- 自動分頁
- 更多更多方法

這個類別提供了一個很強鍵的基礎，你可以在這個基礎上建立自己的模型，讓你可以快速地建構出應用程式的模型層。

建立你的模型
===================

若你需要利用 CodeIgniter 的模型，你只需要創建一個新的模型類別，並且使其繼承 ``CodeIgniter\Model`` ：

::

        <?php namespace App\Models;

        use CodeIgniter\Model;

	class UserModel extends Model
	{

	}

這個空的類別提供了對資料庫連接、查詢生成器，和一些額外的便捷方法的訪問。

連接資料庫
--------------------------

當類別被首次實體化後，如果沒有向建構方法傳遞資料庫的連結實體，那麼它將會自動連接到組態設定中預設的資料庫群組。你可以透過在類別中添加 DBGroup 屬性來修改每個模型會使用到的資料庫設定群組。這樣可以讓模型中任何對 ``$this->db`` 的呼叫引用，都是透過你所設定適合的連接執行。

::

    <?php namespace App\Models;

    use CodeIgniter\Model;

	class UserModel extends Model
	{
		protected $DBGroup = 'group_name';
	}

你可以把 "group_name" 替換成資料庫組態設定檔案中定義的資料庫群組名稱。

設定你的模型
----------------------

模型類別有幾個設定選項，可以透過設定這些選項讓「類別」方法無縫地為你工作。前兩個是所有 CRUD 需求都會用到的屬性，決定著我們需要使用什麼資料表，以及如何找到所需的記錄。

::

        <?php namespace App\Models;

        use CodeIgniter\Model;

	class UserModel extends Model
	{
		protected $table      = 'users';
		protected $primaryKey = 'id';

		protected $returnType = 'array';
		protected $useSoftDeletes = true;

		protected $allowedFields = ['name', 'email'];

		protected $useTimestamps = false;
		protected $createdField  = 'created_at';
		protected $updatedField  = 'updated_at';
		protected $deletedField  = 'deleted_at';

		protected $validationRules    = [];
		protected $validationMessages = [];
		protected $skipValidation     = false;
	}

**$table**

指定這個模型主要配合的資料表，這只適用於模型內建的 CRUD 方法，並不會限制你在你自己的查詢內一定得用這個表。

**$primaryKey**

你所選擇的資料表中資料記錄的唯一識別符號，它不一定要與資料庫中資料表的主鍵欄位相同，而是你在使用像是 ``find()`` 這種方法時，模型可以知道要將指定值以哪個欄位進行搜索。

.. note:: 所有模型必須指定一個 $primaryKey ，以使所有功能可以正常工作。

**$returnType**

模型提供的 CRUD 方法將幫助你減少工作量，並自動回傳結果資料，而不適普的的資料庫結果物件。這個設定允許你宣告回傳的資料類型。你可以鍵入 array 或是 object ，或者可以與結果物件的 getCustomResultObject() 方法一起使用完整的類別名稱。

**$useSoftDeletes**

如果這個值為 true ，那麼任何 delete 方法的呼叫都會在資料庫中修改 ``deleted_at`` 欄位，而不是直接刪除該筆資料。當資料可能在其他地方被引用時，這個功能可以替你將資料保存下來，也可以作為「資源回收桶」，讓被刪除的物件有被還原的可能，甚至你也可以將其保留下來做為未來安全性追蹤的依據。若是資料被設為刪除，你還是想呼叫到這筆資料，則必須在 find() 方法前先呼叫 withDeleted() 方法，否則 find() 方法只會回傳未被刪除的資料。

若你要使用這個功能，你需要在資料庫中建立型別為 DATETIME 或 INTEGER 的欄位，其名稱必須在 $dateFormat 成員屬性中定義，這個成員變數的值必須與資料庫的欄位名稱相同。而 $dateFormat 的預設名稱為 ``deleted_at`` 。

**$allowedFields**

在這個陣列中被記錄的欄位名稱都將在使用保存、插入，或更新方法期間被允許，而沒有被記錄的欄位名稱將被丟棄。這有助於防止將未處理的表單資訊直接傳遞給模型處理時，導致的自動綁定漏洞。

**$useTimestamps**

這個屬性的型別為布林，它決定了在執行插入與更新的方法時，是否會自動更新時間戳。如果為 true 將以 $dateFormat 屬性所指定格式，產出目前的時間記錄並存在的固定的欄位中。這個功能需要在資料庫中以適當的型別建立 "created_at" 以及 "updated_at" 欄位。

**$createdField**

指定使用哪個資料庫欄位來保存資料在創建時的時間戳，請將其留空並避免更新這個欄位（即使啟動了 useTimestamps 功能）。

**$updatedField**

指定使用哪個資料庫欄位來保存資料在更新時的時間戳，請將其留空並避免更新這個欄位（即使啟動了 useTimestamps 功能）。

**$dateFormat**

這個屬性將與 $useTimestamps 與 $useSoftDeletes 一起運作，確保正確的日期被插入到資料庫中。在預設的情形下，這個值會創建 DATETIME 型別的值，而這個屬性可以設定的選項為： datetime 、date 、int （ PHP 時間戳）。

**$validationRules**

這個屬性將會是驗證程式庫的 :ref:`validation-array` （如何保存規則）條目中所描述的驗證用陣列，或是驗證群組名稱的字串，下面將會更詳細地闡述。

**$validationMessages**

你將在這個屬性中儲存，驗證過程中你所設定的 :ref:`validation-custom-errors` （自訂錯誤消息）的陣列，下面將會有更詳細地闡述。

**$skipValidation**

這個屬性決定在進行 ``更新`` 與 ``插入`` 的過程中，是否會跳過驗證。預設值為 false ，這代表若沒有另外賦予值，模型將始終進行驗證。這個屬性主要由 ``skipValidation()`` 方法使用，你也可以將這個屬性改為 ``true`` ，讓模型永遠不要進行驗證。

**$beforeInsert**
**$afterInsert**
**$beforeUpdate**
**$afterUpdate**
**afterFind**
**afterDelete**

這些陣列允許你宣告需要執行的回呼方法，並在你指定的事件發生時執行。

資料作業
=================

尋找資料
------------

在尋找資料方面，模型提供了幾個函數來對資料表進行基礎的 CRUD 工作，包括：find()、insert()、 update()、 delete() 等等。

**find()**

以傳入的主鍵搜索資料，將會回傳一筆符合的資料：

::

	$user = $userModel->find($user_id);

你的查詢將會以 $returnType 指定的資料型別回傳。

你可以透過傳入一個以主鍵組成的陣列，來取得多筆資料：

::

	$users = $userModel->find([1,2,3]);

如果沒有傳入任何參數，那麼將會回傳這個模型所指定的資料表中所有的記錄。雖然表示的函數名稱沒有像 findAll() 一樣這麼直覺，但效果是相同的。

**findColumn()**

回傳 null 或是一個具有索引的欄位結果陣列。

::

 	$user = $userModel->findColumn($column_name);

$column_name 應該要是單個欄位的名稱，若否你則會獲得 DataException 這個例外的拋出。

**findAll()**

回傳所有結果。

::

	$users = $userModel->findAll();

在呼叫這個方法之前，可以根據自己的需求從中間插入查詢生成器的語法，來修改這個查詢。

::

	$users = $userModel->where('active', 1)
	                   ->findAll();

你也可以傳入兩個參數，分別代表偏移與限制。

::

	$users = $userModel->findAll($limit, $offset);

**first()**

將一定會回傳第一筆結果，這個功能最好與查詢生成器結合使用。

::

	$user = $userModel->where('deleted', 0)
	                  ->first();

**withDeleted()**

如果 $useSoftDeletes 為 true ，那麼 find() 方法將會以 "deleted_at IS NOT NULL" 這個條件執行資料庫查詢，不會回傳任何被假性刪除的記錄。當然，你若是需要這筆資料，就要在 find() 方法以前使用這個方法：

::

	// Only gets non-deleted rows (deleted = 0)
	$activeUsers = $userModel->findAll();

	// Gets all rows
	$allUsers = $userModel->withDeleted()
	                      ->findAll();

**onlyDeleted()**

withDeleted() 方法將會回傳已經刪除與未刪除的記錄，而這個方法將會修改下一個生效的 find() 方法，使它只會回傳被假性刪除的資料。

::

	$deletedUsers = $userModel->onlyDeleted()
	                          ->findAll();

儲存資料
-----------

**insert()**

這個方法允許你傳入一個鍵值陣列，使你可以在資料庫中插入一筆新資料。你所傳入的陣列的鍵必須與資料庫的欄位名稱相符，而每個鍵的值則是你所需要儲存的資料。

::

	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->insert($data);

**insertBatch()**

若是你需要一次插入多筆資料進資料庫中，你需要傳入一個包含上一個條目 insert() 使用的陣列的陣列。

::

	$data = [
		[
			'username' => 'darth',
			'email'    => 'd.vader@theempire.com'
		],
		[
			'username' => 'amos',
			'email'    => 'a.vader@theempire.com'
		]
	];

	$userModel->insertBatch($data);

**update()**

更新資料庫的現有記錄，第一個參數是你目標更新資料的 $primaryKey ，而第二個參數則是一個鍵值陣列。陣列的鍵必須與資料表中的欄位名稱相符，而每個鍵的值則是你所要更新的資料。

::

	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->update($id, $data);

透過傳入一個以主鍵組成的陣列作為第一個參數，可以只用一次呼叫更新多筆記錄。

::

    $data = [
		'active' => 1
	];

	$userModel->update([1, 2, 3], $data);

當你出現一些額外的需求時，你可以把傳入的參數留白，這個方法便會成為查詢生成器的更新指令一樣，讓你可以進行額外的驗證、事件等功能。

::

    $userModel
        ->whereIn('id', [1,2,3])
        ->set(['active' => 1])
        ->update();

**save()**

這個功能它封裝了 insert() 與 update() 兩個方法。它依據在資料表中是否找的到你所傳入的 $primaryKey 主鍵，自動處理你所需要的是插入或是更新。

::

	// 定義 model 屬性
	$primaryKey = 'id';

	// 執行 insert()
	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->save($data);

	// 如果找的到你所傳入的主健，將會執行 update() 
	$data = [
		'id'       => 3,
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];
	$userModel->save($data);

save() 方法還可以傳入一個物件並自動取得這個物鍵的公開屬性和保護屬性，然後將它們保存成相應的陣列，傳入到插入或更新的方法中。這種方式允許你使用簡潔的實體類別，它表示的是一個物件類型的單一實體。比如使用者、部落格文章、作業等。這個類別負責維護圍繞著物件本身的商業邏輯，例如：以某種方法格式化元素等。它不應該有任何將資料儲存到資料庫的邏輯，最簡單的使用方式如下所示：

::

	namespace App\Entities;

	class Job
	{
		protected $id;
		protected $name;
		protected $description;

		public function __get($key)
		{
			if (property_exists($this, $key))
			{
				return $this->$key;
			}
		}

		public function __set($key, $value)
		{
			if (property_exists($this, $key))
			{
				$this->$key = $value;
			}
		}
	}

一個對應實體類別的最簡模型可能會像這個樣子：

::

        use CodeIgniter\Model;

	class JobModel extends Model
	{
		protected $table = 'jobs';
		protected $returnType = '\App\Entities\Job';
		protected $allowedFields = [
			'name', 'description'
		];
	}

這個模型使用 ``jobs`` 資料表來運作，並將所有結果以 ``App\Entities\Job`` 的一個實體回傳。當你需要將這個記錄儲存到資料庫時，你需要撰寫自定方法，使用模型提供的 ``save()`` 方法檢查類別、獲取公開屬性與私有屬性並將它們儲存到資料庫中。

::

	// 獲取你所指定的 Job 實體
	$job = $model->find(15);

	// 執行一些改變
	$job->name = "Foobar";

	// 儲存改變
	$model->save($job);

.. note:: 如果你發現自己需要使用實體來建構程式，CodeIgniter 提供了一個內建的 :doc:`實體類別 </models/entities>` ，它提供了幾個方便的功能，讓開發實體變得更加簡單。

刪除資料
-------------

**delete()**

傳入主鍵作為參數，將刪除模型所指定的資料表符合的記錄。

::

	$userModel->delete(12);

如果模型的 $useSoftDeletes 為 true，則會將作為假性刪除判斷依據的 ``deleted_at`` 欄位寫入當下的日期和時間。你可以在第二個參數傳入 true 來強制執行永久刪除。

傳入以主鍵組成的陣列，可以一次刪除多個記錄。

::

    $userModel->delete([1,2,3]);

如果沒有傳入參數，就會像查詢生成器的刪除方法一樣，在這之前你會需要呼叫 where 相關的查詢生成器函數。

::

    $userModel->where('id', 12)->delete();

**purgeDeleted()**

透過完全清除軟性刪除的記錄來清理資料表（符合 "deleted_at IS NOT NULL" 這個條件）。

::

	$userModel->purgeDeleted();

驗證資料
---------------

對許多人來說，在模型中實作資料驗證，將會是減少程式碼重複的不二方法。模型類別提供在使用 ``insert()`` 、 ``update()`` 、 或 ``save()`` 方法保存在資料庫之前，自動讓所有資料進行驗證。

第一步，你需要在 ``$validationRules`` 這個類別屬性中，宣告需要應用驗證的欄位與規則。如果你想要使用自定的錯誤訊息，請將它們放在
``$validationMessages`` 陣列。

::

	class UserModel extends Model
	{
		protected $validationRules    = [
			'username'     => 'required|alpha_numeric_space|min_length[3]',
			'email'        => 'required|valid_email|is_unique[users.email]',
			'password'     => 'required|min_length[8]',
			'pass_confirm' => 'required_with[password]|matches[password]'
		];

		protected $validationMessages = [
			'email'        => [
				'is_unique' => 'Sorry. That email has already been taken. Please choose another.'
			]
		];
	}

另一種方式是透過函數將驗證訊息設定成欄位。

.. php:function:: setValidationMessage($field, $fieldMessages)

	:param	string	$field: 欄位名稱
	:param	array	$fieldMessages: 欄位錯誤訊息

	這個函數將設定欄位的錯誤訊息。

	使用範例：
	
	::

		$fieldName = 'name';
		$fieldValidationMessage = [
			'required'   => 'Your name is required here',
		];
		$model->setValidationMessage($fieldName, $fieldValidationMessage);

.. php:function:: setValidationMessages($fieldMessages)

	:param array $fieldMessages: 欄位訊息

	這個方法可以設定欄位訊息。

	使用範例：

	::

		$fieldValidationMessage = [
			'name' => [
				'required'   => 'Your baby name is missing.',
				'min_length' => 'Too short, man!',
			],
		];
		$model->setValidationMessages($fieldValidationMessage);

現在，每當你呼叫 ``insert()`` 、 ``update()`` ，或 ``save()`` 方法時，資料將會被自動驗證。如果驗證失敗，模型將會回傳 **false** 。你可以使用 ``error()`` 方法來存取驗證錯誤：

::

	if ($model->save($data) === false)
	{
		return view('updateUser', ['errors' => $model->errors()];
	}

這將回傳一個包含欄位名稱和相關錯誤訊息的陣列，可以用來在表單的頂部顯示所有錯誤，你也可以單獨顯示它們：

::

	<?php if (! empty($errors)) : ?>
		<div class="alert alert-danger">
		<?php foreach ($errors as $field => $error) : ?>
			<p><?= $error ?></p>
		<?php endforeach ?>
		</div>
	<?php endif ?>

如果你想在組態設定檔案中統一組織驗證用的規則以及錯誤訊息，只需將 ``$validationRules`` 設定為你所創建的規則群組名稱即可，就像這樣做：

::

	class UserModel extends Model
	{
		protected $validationRules = 'users';
	}

檢索驗證規則
---------------------------

你可以透過模型的 ``validationRules`` 屬性來檢索模型的驗證規則。

::

    $rules = $model->validationRules;

你也可以透過直接呼叫 getValidationRules() 方法，用選項來檢索這些規則的一個子集。

::

    $rules = $model->getValidationRules($options);

``$options`` 參數是一個鍵值陣列，包含著鍵為 "except" 或 "only" 的元素，這個鍵對應的數值則為你想查閱的欄位名稱陣列。

::

    // 取得除了 username 欄位以外的所有規則
    $rules = $model->getValidationRules(['except' => ['username']]);
    // 只取得 city 與 state 欄位的規則
    $rules = $model->getValidationRules(['only' => ['city', 'state']]);

驗證置換符號
-----------------------

模型提供了一個簡單的方法，可以根據你所傳入的資料替換掉規則的一部份。這聽請起來可以會有點艱深晦澀，但在 ``is_unique`` 驗證規則中相當便利。置換符號是以 $data 的形式傳入欄位（或是鍵值陣列）的名稱，這個陣列的 **值** 將會取代在目標字串中被大括弧包圍的相符欄位，這個例子應該可以充分說明這個功能：

::

    protected $validationRules = [
        'email' => 'required|valid_email|is_unique[users.email,id,{id}]'
    ];

在這個規則中，它規定除了具有置換符號的值相符的 id 記錄外，電子郵件的位置在資料庫中應該是唯一值，我們假設表單中的 POST 資料具有以下內容：

::

    $_POST = [
        'id' => 4,
        'email' => 'foo@example.com'
    ]

那麼 ``{id}`` 置換符號將被替換成數字 **4** ，因此被置換成下列的規則：

::


    protected $validationRules = [
        'email' => 'required|valid_email|is_unique[users.email,id,4]'
    ];

因此，當它所驗證的電子郵件是唯一值，就會忽略 ``id=4`` 的記錄。

這也可以用來在驗證的執行期間創建更動態的規則，你只需要注意傳入的動態鍵不要與表單的資料衝突即可。

保護欄位
-----------------

為了防止大規模的自動綁定攻擊，模型類別將 **要求** 你在 ``$allowedFields`` 類別屬性中列出，所有可以在插入和更新時更改的欄位名稱。除了這些之外的任何資料都將在進入資料庫之前被刪除，在保護時間戳或主鍵不被改變這件事情上，這是非常有用處的。

::

	protected $allowedFields = ['name', 'email', 'address'];

有時，你會發現你可能需要改變這些元素，這個需求通常會在測試、遷移，或資料填充的期間出現。在這種情形下，你可以開啟或關閉保護。

::

	$model->protect(false)
	      ->insert($data)
	      ->protect(true);

操作查詢生成器
--------------------------

你可以在任何需要的時候，使用模型資料庫連接所提供的查詢生成器的共享實體。

::

	$builder = $userModel->builder();

這個生成器已經在模型中的 $table 設定好了。

你還可以在同一個鏈式呼叫中優雅地混和使用查詢生成器方法與模型提供的 CRUD 方法。

::

	$users = $userModel->where('status', 'active')
			   ->orderBy('last_login', 'asc')
			   ->findAll();

.. note:: 你也可以無縫地造訪模型地資料庫連接：
	
	::
	
	$user_name = $userModel->escape($name);

轉換回傳型別
----------------------------

你可以透過模型的類別屬性 $returnType 來指定使用 find() 方法時的回傳資料型別。但有時你可能會希望以不同的格式回傳資料，模型提供了一些方法允許你這麼做。

.. note:: 這些方法只會改變下一次執行 find() 方法呼叫時的回傳型別，之後它將回復為預設值。

**asArray()**

下一次的 find() 方法將以鍵值陣列的形式回傳：

::

	$users = $userModel->asArray()->where('status', 'active')->findAll();

**asObject()**

下一次的 find() 將會把資料轉換為標準物件或自訂類別的實體回傳。

::

	// 回傳標準物件
	$users = $userModel->asObject()->where('status', 'active')->findAll();

	// 回傳自訂類別實體
	$users = $userModel->asObject('User')->where('status', 'active')->findAll();

處理大量資料
--------------------------------

有的時候，你可能會需要處理大量的資料，說不定會有記憶體不足的問題。為了簡單化這個問題，你可以使用 chunk() 方法來獲取更小的資料塊，然後再進行工作。第一個參數是單個資料塊中被檢索的行數，第二個參數是一個匿名陣列，它將會呼叫每一行的資料。

這最好在排程工作、資料匯出與其他大型任務中使用。

::

	$userModel->chunk(100, function ($data)
	{
		// do something.
		// $data is a single row of data.
	});

模型事件
============

在模型的執行的過程中，可以指定幾個時機執行數個回呼函數。這些方法可以用於正規化資料、雜湊密碼，以及儲存相關實體等等。以下提供數種方法來影響不同時機的執行行為：**$beforeInsert** 、 **$afterInsert** 、 **$beforeUpdate** 、 **afterUpdate** 、 **afterFind** 、以及 **afterDelete** 。

定義回呼
------------------

首先，我們在模型中會創建一個新的類別方法來定義回呼。這個類別會接受一個 $data 陣列作為它的唯一參數。$data 陣列的確切內容會因事件的不同而相異，但總是會包含一個名為 **data** 的鍵，其中包含著傳遞給原始方法的主要資料——在插入或更新的方法下，這將是會被插入到資料庫的鍵值陣列。主要的陣列內容也會包含傳遞給其他方法的值，待會將會詳細的介紹。所有的回呼方法都必須回傳原始的 $data 陣列，這樣其他的回呼函數才能得到完整的訊息。

::

	protected function hashPassword(array $data)
	{
		if (! isset($data['data']['password']) return $data;

		$data['data']['password_hash'] = password_hash($data['data']['password'], PASSWORD_DEFAULT);
		unset($data['data']['password'];

		return $data;
	}

指定要運作的回呼
---------------------------

你可以透過將方法名稱添加到相應的類別屬性（ beforeInsert 或 afterUpdate 等），來指定何時該運作哪個回呼。你也可以在一個事件中加入多個回呼，他們將相繼被處理。當然也可以在多個事件中使用同一個回呼。

::

	protected $beforeInsert = ['hashPassword'];
	protected $beforeUpdate = ['hashPassword'];

事件參數
----------------

由於傳遞給每個回呼的確切資料存在著一些差異，下面將會詳列傳遞給每個事件的 $data 參數中的詳細內容：

id = 被更新的行的主键。
data = 被插入的键/值对。如果一个对象或Entity类被传递给insert方法，首先将其转换为数组。


================ =========================================================================================================
事件             $data 內容
================ =========================================================================================================
beforeInsert      **data** = 即將被插入的鍵值陣列。如果一個物件或是實體類別被傳遞給插入方法，則會先將其轉換成陣列。
afterInsert       **id** = 新記錄的主鍵，若插入失敗則為 0 。
                  **data** = 被插入進資料庫的鍵值陣列。
                  **result** = 透過查詢生成器使用 insert() 方法的結果。
beforeUpdate      **id** = 被更新的資料的主鍵。
                  **data** = 即將被更新的鍵值陣列，如果一個物件或實體類別被傳遞給插入方法，首先會先將其轉換成陣列。
afterUpdate       **id** = 被更新的資料主鍵。
                  **data** = 更新完成的鍵值陣列。
                  **result** = 透過查詢生成器使用 update() 方法的結果
afterFind         將因為 find 方法的不同而相異，請詳閱下方內容：
- find()          **id** = 被搜索的主鍵。
                  **data** = 搜索結果的資訊列，若沒有結果則為空。
- findAll()       **data** = 要查找的資料列數，如果沒有找到結果則為空。
                  **limit** = 要查找的列數。
                  **offset** = 搜索過程中要跳過的列數。
- first()         **data** = 搜索過程中找到的結果列。如果沒找到則為空。
beforeDelete      將因為 delete 方法的不同而相異，請詳閱下方內容：
- delete()        **id** = 即將被刪除的主鍵。
                  **purge** = 布林，是否被完全刪除或假性刪除。
afterDelete       將因為 delete 方法的不同而相異，請詳閱下方內容：
- delete()        **id** = 被刪除的主鍵。
                  **purge** = 布林，是否被完全刪除或假性刪除。
                  **result** = 查詢生成器呼叫 delete() 的結果。
                  **data** = 未使用。
================ =========================================================================================================

創建手動模型
=====================

你不需要繼承任何的特殊類別來替你的應用程式創建一個模型。你只需要得到一個資料庫連接實體即可。這樣你就能繞過 CodeIgniter 的模型替你預先制定的功能，創建一個完全由你定義的手動模型。

::

    <?php namespace App\Models;

	use CodeIgniter\Database\ConnectionInterface;

	class UserModel
	{
		protected $db;

		public function __construct(ConnectionInterface &$db)
		{
			$this->db =& $db;
		}
	}
