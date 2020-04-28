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

The first thing to notice is the name of the methods we've added. For each one, the class expects the snake_case
column name to be converted into PascalCase, and prefixed with either ``set`` or ``get``. These methods will then
be automatically called whenever you set or retrieve the class property using the direct syntax (i.e. $user->email).
The methods do not need to be public unless you want them accessed from other classes. For example, the ``created_at``
class property will be accessed through the ``setCreatedAt()`` and ``getCreatedAt()`` methods.

.. note:: This only works when trying to access the properties from outside of the class. Any methods internal to the
    class must call the ``setX()`` and ``getX()`` methods directly.

In the ``setPassword()`` method we ensure that the password is always hashed.

In ``setCreatedAt()`` we convert the string we receive from the model into a DateTime object, ensuring that our timezone
is UTC so we can easily convert the viewer's current timezone. In ``getCreatedAt()``, it converts the time to
a formatted string in the application's current timezone.

While fairly simple, these examples show that using Entity classes can provide a very flexible way to enforce
business logic and create objects that are pleasant to use.

::

    // Auto-hash the password - both do the same thing
    $user->password = 'my great password';
    $user->setPassword('my great password');

資料映射
============

At many points in your career, you will run into situations where the use of an application has changed and the
original column names in the database no longer make sense. Or you find that your coding style prefers camelCase
class properties, but your database schema required snake_case names. These situations can be easily handled
with the Entity class' data mapping features.

As an example, imagine you have the simplified User Entity that is used throughout your application::

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

Your boss comes to you and says that no one uses usernames anymore, so you're switching to just use emails for login.
But they do want to personalize the application a bit, so they want you to change the name field to represent a user's
full name now, not their username like it does currently. To keep things tidy and ensure things continue making sense
in the database you whip up a migration to rename the `name` field to `full_name` for clarity.

Ignoring how contrived this example is, we now have two choices on how to fix the User class. We could modify the class
property from ``$name`` to ``$full_name``, but that would require changes throughout the application. Instead, we can
simply map the ``full_name`` column in the database to the ``$name`` property, and be done with the Entity changes::

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

By adding our new database name to the ``$datamap`` array, we can tell the class what class property the database column
should be accessible through. The key of the array is the name of the column in the database, where the value in the array
is class property to map it to.

In this example, when the model sets the ``full_name`` field on the User class, it actually assigns that value to the
class' ``$name`` property, so it can be set and retrieved through ``$user->name``. The value will still be accessible
through the original ``$user->full_name``, also, as this is needed for the model to get the data back out and save it
to the database. However, ``unset`` and ``isset`` only work on the mapped property, ``$name``, not on the original name,
``full_name``.

修改器
========

資料修改器
-------------

By default, the Entity class will convert fields named `created_at`, `updated_at`, or `deleted_at` into
:doc:`Time </libraries/time>` instances whenever they are set or retrieved. The Time class provides a large number
of helpful methods in an immutable, localized way.

You can define which properties are automatically converted by adding the name to the **options['dates']** array::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $dates = ['created_at', 'updated_at', 'deleted_at'];
    }

Now, when any of those properties are set, they will be converted to a Time instance, using the application's
current timezone, as set in **app/Config/App.php**::

    $user = new App\Entities\User();

    // Converted to Time instance
    $user->created_at = 'April 15, 2017 10:30:00';

    // Can now use any Time methods:
    echo $user->created_at->humanize();
    echo $user->created_at->setTimezone('Europe/London')->toDateString();

型別轉換
----------------

You can specify that properties in your Entity should be converted to common data types with the **casts** property.
This option should be an array where the key is the name of the class property, and the value is the data type it
should be cast to. Casting only affects when values are read. No conversions happen that affect the permanent value in
either the entity or the database. Properties can be cast to any of the following data types:
**integer**, **float**, **double**, **string**, **boolean**, **object**, **array**, **datetime**, and **timestamp**.
Add a question mark at the beginning of type to mark property as nullable, i.e. **?string**, **?integer**.

For example, if you had a User entity with an **is_banned** property, you can cast it as a boolean::

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

Array/Json casting is especially useful with fields that store serialized arrays or json in them. When cast as:

* an **array**, they will automatically be unserialized,
* a **json**, they will automatically be set as an value of json_decode($value, false),
* a **json-array**, they will automatically be set as an value of json_decode($value, true),

when you read the property's value.
Unlike the rest of the data types that you can cast properties into, the:

* **array** cast type will serialize,
* **json** and **json-array** cast will use json_encode function on

the value whenever the property is set::

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

You can check if an Entity attribute has changed since it was created. The only parameter is the name of the
attribute to check::

    $user = new User();
    $user->hasChanged('name');      // false

    $user->name = 'Fred';
    $user->hasChanged('name');      // true

Or to check the whole entity for changed values omit the parameter::

    $user->hasChanged();            // true
