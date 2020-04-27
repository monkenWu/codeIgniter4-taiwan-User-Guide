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

    // Create a new class manually
    $userModel = new App\Models\UserModel();

    // Create a new class with the model function
    $userModel = model('App\Models\UserModel', false);

    // Create a shared instance of the model
    $userModel = model('App\Models\UserModel');

    // Create shared instance with a supplied database connection
    // When no namespace is given, it will search through all namespaces
    // the system knows about and attempt to located the UserModel class.
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

An associative array of data is passed into this method as the only parameter to create a new
row of data in the database. The array's keys must match the name of the columns in a $table, while
the array's values are the values to save for that key::

	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->insert($data);

**update()**

Updates an existing record in the database. The first parameter is the $primaryKey of the record to update.
An associative array of data is passed into this method as the second parameter. The array's keys must match the name
of the columns in a $table, while the array's values are the values to save for that key::

	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->update($id, $data);

Multiple records may be updated with a single call by passing an array of primary keys as the first parameter::

    $data = [
		'active' => 1
	];

	$userModel->update([1, 2, 3], $data);

When you need a more flexible solution, you can leave the parameters empty and it functions like the Query Builder's
update command, with the added benefit of validation, events, etc::

    $userModel
        ->whereIn('id', [1,2,3])
        ->set(['active' => 1])
        ->update();

**save()**

This is a wrapper around the insert() and update() methods that handle inserting or updating the record
automatically, based on whether it finds an array key matching the $primaryKey value::

	// Defined as a model property
	$primaryKey = 'id';

	// Does an insert()
	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->save($data);

	// Performs an update, since the primary key, 'id', is found.
	$data = [
		'id'       => 3,
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];
	$userModel->save($data);

The save method also can make working with custom class result objects much simpler by recognizing a non-simple
object and grabbing its public and protected values into an array, which is then passed to the appropriate
insert or update method. This allows you to work with Entity classes in a very clean way. Entity classes are
simple classes that represent a single instance of an object type, like a user, a blog post, job, etc. This
class is responsible for maintaining the business logic surrounding the object itself, like formatting
elements in a certain way, etc. They shouldn't have any idea about how they are saved to the database. At their
simplest, they might look like this::

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

A very simple model to work with this might look like::

        use CodeIgniter\Model;

	class JobModel extends Model
	{
		protected $table = 'jobs';
		protected $returnType = '\App\Entities\Job';
		protected $allowedFields = [
			'name', 'description'
		];
	}

This model works with data from the ``jobs`` table, and returns all results as an instance of ``App\Entities\Job``.
When you need to persist that record to the database, you will need to either write custom methods, or use the
model's ``save()`` method to inspect the class, grab any public and private properties, and save them to the database::

	// Retrieve a Job instance
	$job = $model->find(15);

	// Make some changes
	$job->name = "Foobar";

	// Save the changes
	$model->save($job);

.. note:: If you find yourself working with Entities a lot, CodeIgniter provides a built-in :doc:`Entity class </models/entities>`
	that provides several handy features that make developing Entities simpler.

刪除資料
-------------

**delete()**

Takes a primary key value as the first parameter and deletes the matching record from the model's table::

	$userModel->delete(12);

If the model's $useSoftDeletes value is true, this will update the row to set ``deleted_at`` to the current
date and time. You can force a permanent delete by setting the second parameter as true.

An array of primary keys can be passed in as the first parameter to delete multiple records at once::

    $userModel->delete([1,2,3]);

If no parameters are passed in, will act like the Query Builder's delete method, requiring a where call
previously::

    $userModel->where('id', 12)->delete();

**purgeDeleted()**

Cleans out the database table by permanently removing all rows that have 'deleted_at IS NOT NULL'. ::

	$userModel->purgeDeleted();

驗證資料
---------------

For many people, validating data in the model is the preferred way to ensure the data is kept to a single
standard, without duplicating code. The Model class provides a way to automatically have all data validated
prior to saving to the database with the ``insert()``, ``update()``, or ``save()`` methods.

The first step is to fill out the ``$validationRules`` class property with the fields and rules that should
be applied. If you have custom error message that you want to use, place them in the ``$validationMessages`` array::

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

The other way to set the validation message to fields by functions,

.. php:function:: setValidationMessage($field, $fieldMessages)

	:param	string	$field
	:param	array	$fieldMessages

	This function will set the field wise error messages.

	Usage example::

            $fieldName = 'name';
            $fieldValidationMessage = array(
                            'required'   => 'Your name is required here',
                    );
            $model->setValidationMessage($fieldName, $fieldValidationMessage);

.. php:function:: setValidationMessages($fieldMessages)

	:param	array	$fieldMessages

	This function will set the field messages.

	Usage example::

            $fieldValidationMessage = array(
                    'name' => array(
                            'required'   => 'Your baby name is missing.',
                            'min_length' => 'Too short, man!',
                    ),
            );
            $model->setValidationMessages($fieldValidationMessage);

Now, whenever you call the ``insert()``, ``update()``, or ``save()`` methods, the data will be validated. If it fails,
the model will return boolean **false**. You can use the ``errors()`` method to retrieve the validation errors::

	if ($model->save($data) === false)
	{
		return view('updateUser', ['errors' => $model->errors()];
	}

This returns an array with the field names and their associated errors that can be used to either show all of the
errors at the top of the form, or to display them individually::

	<?php if (! empty($errors)) : ?>
		<div class="alert alert-danger">
		<?php foreach ($errors as $field => $error) : ?>
			<p><?= $error ?></p>
		<?php endforeach ?>
		</div>
	<?php endif ?>

If you'd rather organize your rules and error messages within the Validation configuration file, you can do that
and simply set ``$validationRules`` to the name of the validation rule group you created::

	class UserModel extends Model
	{
		protected $validationRules = 'users';
	}

檢索驗證規則
---------------------------

You can retrieve a model's validation rules by accessing its ``validationRules``
property::

    $rules = $model->validationRules;

You can also retrieve just a subset of those rules by calling the accessor
method directly, with options::

    $rules = $model->getValidationRules($options);

The ``$options`` parameter is an associative array with one element,
whose key is either "except" or "only", and which has as its
value an array of fieldnames of interest.::

    // get the rules for all but the "username" field
    $rules = $model->getValidationRules(['except' => ['username']]);
    // get the rules for only the "city" and "state" fields
    $rules = $model->getValidationRules(['only' => ['city', 'state']]);

驗證置換符號
-----------------------

The model provides a simple method to replace parts of your rules based on data that's being passed into it. This
sounds fairly obscure but can be especially handy with the ``is_unique`` validation rule. Placeholders are simply
the name of the field (or array key) that was passed in as $data surrounded by curly brackets. It will be
replaced by the **value** of the matched incoming field. An example should clarify this::

    protected $validationRules = [
        'email' => 'required|valid_email|is_unique[users.email,id,{id}]'
    ];

In this set of rules, it states that the email address should be unique in the database, except for the row
that has an id matching the placeholder's value. Assuming that the form POST data had the following::

    $_POST = [
        'id' => 4,
        'email' => 'foo@example.com'
    ]

then the ``{id}`` placeholder would be replaced with the number **4**, giving this revised rule::

    protected $validationRules = [
        'email' => 'required|valid_email|is_unique[users.email,id,4]'
    ];

So it will ignore the row in the database that has ``id=4`` when it verifies the email is unique.

This can also be used to create more dynamic rules at runtime, as long as you take care that any dynamic
keys passed in don't conflict with your form data.

保護欄位
-----------------

To help protect against Mass Assignment Attacks, the Model class **requires** that you list all of the field names
that can be changed during inserts and updates in the ``$allowedFields`` class property. Any data provided
in addition to these will be removed prior to hitting the database. This is great for ensuring that timestamps,
or primary keys do not get changed.
::

	protected $allowedFields = ['name', 'email', 'address'];

Occasionally, you will find times where you need to be able to change these elements. This is often during
testing, migrations, or seeds. In these cases, you can turn the protection on or off::

	$model->protect(false)
	      ->insert($data)
	      ->protect(true);

操作查詢生成器
--------------------------

You can get access to a shared instance of the Query Builder for that model's database connection any time you
need it::

	$builder = $userModel->builder();

This builder is already set up with the model's $table.

You can also use Query Builder methods and the Model's CRUD methods in the same chained call, allowing for
very elegant use::

	$users = $userModel->where('status', 'active')
			   ->orderBy('last_login', 'asc')
			   ->findAll();

.. note:: You can also access the model's database connection seamlessly::

			$user_name = $userModel->escape($name);

轉換回傳型別
----------------------------

You can specify the format that data should be returned as when using the find*() methods as the class property,
$returnType. There may be times that you would like the data back in a different format, though. The Model
provides methods that allow you to do just that.

.. note:: These methods only change the return type for the next find*() method call. After that,
			it is reset to its default value.

**asArray()**

Returns data from the next find*() method as associative arrays::

	$users = $userModel->asArray()->where('status', 'active')->findAll();

**asObject()**

Returns data from the next find*() method as standard objects or custom class intances::

	// Return as standard objects
	$users = $userModel->asObject()->where('status', 'active')->findAll();

	// Return as custom class instances
	$users = $userModel->asObject('User')->where('status', 'active')->findAll();

處理大量資料
--------------------------------

Sometimes, you need to process large amounts of data and would run the risk of running out of memory.
To make this simpler, you may use the chunk() method to get smaller chunks of data that you can then
do your work on. The first parameter is the number of rows to retrieve in a single chunk. The second
parameter is a Closure that will be called for each row of data.

This is best used during cronjobs, data exports, or other large tasks.
::

	$userModel->chunk(100, function ($data)
	{
		// do something.
		// $data is a single row of data.
	});

模型事件
============

There are several points within the model's execution that you can specify multiple callback methods to run.
These methods can be used to normalize data, hash passwords, save related entities, and much more. The following
points in the model's execution can be affected, each through a class property: **$beforeInsert**, **$afterInsert**,
**$beforeUpdate**, **afterUpdate**, **afterFind**, and **afterDelete**.

定義回呼
------------------

You specify the callbacks by first creating a new class method in your model to use. This class will always
receive a $data array as its only parameter. The exact contents of the $data array will vary between events, but
will always contain a key named **data** that contains the primary data passed to the original method. In the case
of the insert* or update* methods, that will be the key/value pairs that are being inserted into the database. The
main array will also contain the other values passed to the method, and be detailed later. The callback method
must return the original $data array so other callbacks have the full information.

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

You specify when to run the callbacks by adding the method name to the appropriate class property (beforeInsert, afterUpdate,
etc). Multiple callbacks can be added to a single event and they will be processed one after the other. You can
use the same callback in multiple events::

	protected $beforeInsert = ['hashPassword'];
	protected $beforeUpdate = ['hashPassword'];

事件參數
----------------

Since the exact data passed to each callback varies a bit, here are the details on what is in the $data parameter
passed to each event:

================ =========================================================================================================
Event            $data contents
================ =========================================================================================================
beforeInsert      **data** = the key/value pairs that are being inserted. If an object or Entity class is passed to the
                  insert method, it is first converted to an array.
afterInsert       **id** = the primary key of the new row, or 0 on failure.
                  **data** = the key/value pairs being inserted.
                  **result** = the results of the insert() method used through the Query Builder.
beforeUpdate      **id** = the primary key of the row being updated.
                  **data** = the key/value pairs that are being inserted. If an object or Entity class is passed to the
                  insert method, it is first converted to an array.
afterUpdate       **id** = the primary key of the row being updated.
                  **data** = the key/value pairs being updated.
                  **result** = the results of the update() method used through the Query Builder.
afterFind         Varies by find* method. See the following:
- find()          **id** = the primary key of the row being searched for.
                  **data** = The resulting row of data, or null if no result found.
- findAll()       **data** = the resulting rows of data, or null if no result found.
                  **limit** = the number of rows to find.
                  **offset** = the number of rows to skip during the search.
- first()         **data** = the resulting row found during the search, or null if none found.
beforeDelete      Varies by delete* method. See the following:
- delete()        **id** = primary key of row being deleted.
                  **purge** = boolean whether soft-delete rows should be hard deleted.
afterDelete       Varies by delete* method. See the following:
- delete()        **id** = primary key of row being deleted.
                  **purge** = boolean whether soft-delete rows should be hard deleted.
                  **result** = the result of the delete() call on the Query Builder.
                  **data** = unused.
================ =========================================================================================================


創建手動模型
=====================

You do not need to extend any special class to create a model for your application. All you need is to get an
instance of the database connection and you're good to go. This allows you to bypass the features CodeIgniter's
Model gives you out of the box, and create a fully custom experience.

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
