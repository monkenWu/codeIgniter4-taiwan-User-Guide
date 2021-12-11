#########
入門
#########

CodeIgniter 的建立是為了讓框架測試和應用程式測試盡可能地簡單，它內建了 ``phpUnit`` 的支援，並提供了許多方便的輔助方法，在測試應用程式上將面面俱到且不費吹灰之力。

.. contents::
    :local:
    :depth: 2

*************
系統設定
*************

安裝 phpUnit
==================

CodeIgniter 的所有測試功能都基於 `phpUnit <https://phpunit.de/>`__ 。有兩種方式可以安裝 phpUnit 至你的系統中。

Composer
--------

我們推薦你使用 `Composer <https://getcomposer.org/>`__ 安裝在你的專案中。雖然可以全域安裝，但我們不建議你這麼做，因為隨著時間的推移，可能會導致與系統上的其他專案產生相容性問題。

請確認你的系統上已經安裝了 Composer 。在專案根目錄（包含應用程式和系統目錄的目錄）中，在命令列鍵入以下內容：

::

    > composer require --dev phpunit/phpunit

這將會安裝符合你目前 PHP 版本的 phpUnit 。一旦完成，你就可以透過鍵入這個指令來執行這個專案的所有測試：

::

    > ./vendor/bin/phpunit

Phar
----

另一種方法是從 `phpUnit <https://phpunit.de/getting-started/phpunit-7.html>`__ 官方網站上下載 .phar 檔案。這是一個獨立的檔案，應該放置在你的專案根目錄下。

************************
測試你的應用程式
************************

PHPUnit 組態設定
=====================

框架在專案根目錄下有一個 ``phpunit.xml.dist`` 檔案，這個檔案控制了框架本身的單元測試。如果你需要自訂自己的 ``phpunit.xml`` ，請直接將它覆蓋即可。

如果你需要對你的應用程式進行單元測試的話，你的 ``phpunit.xml`` 應該排除 ``system`` 資料夾，以及任何 ``vendor`` 或 ``ThirdParty`` 資料夾。

測試類別
==============

為了利用框架提供的附加工具，你的測試必須繼承 ``CIUnitTestCase`` 。在預設的情形下，所有的測試都應該儲存在 **tests/app** 目錄之中。

要測試新的程式庫 **Foo** 的話，你需要在 **tests/app/Libraries/FooTest.php** 中創建一個新檔案：

::

    <?php

    namespace App\Libraries;

    use CodeIgniter\Test\CIUnitTestCase;

    class FooTest extends CIUnitTestCase
    {
        public function testFooNotBar()
        {
            // ...
        }
    }

若是要測試模型的話，你可能會需要一個類似 ``tests/app/Models/OneOfMyModelsTest.php`` 這樣子的檔案：

::

    <?php

    namespace App\Models;

    use CodeIgniter\Test\CIUnitTestCase;

    class OneOfMyModelsTest extends CIUnitTestCase
    {
        public function testFooNotBar()
        {
            // ...
        }
    }

你可以創建任何符合你測試需要或風格的目錄結構。在替測試類別設定命名空間時，請記住， **app** 目錄是 ``App`` 命名空間的根目錄，所以你使用的任何類別都必須有相對於 ``App`` 正確的命名空間。

.. note:: 測試類別並不會嚴格要求你的命名空間，但妥善使用命明空間將對確保類別名稱不衝突這件事很有幫助。

如果你需要測試資料庫結果，你必須使用 `CIDatabaseTestClass <database.html>`_ 類別。

過渡環境
--------

絕大多數的測試都會需要一些準備工作才能如期運行，PHPUnit 的 ``TestCase`` 提供了四種方法用於測試前後的工作：

::

    public static function setUpBeforeClass(): void
    public static function tearDownAfterClass(): void
    public function setUp(): void
    public function tearDown(): void

靜態方法應該在測試案例開始的前後執行，而本地的方法則是在每個測試之間執行。如果你實作了上述任一特殊函數，也必須運行它的父函數，以保障擴充測試案例時不會干擾環境。

::

    public function setUp(): void
    {
        parent::setUp();
        helper('text');
    }

除了這些方法之外，若你需要在 `set up` 與 `tear down` 期間運行一些無參數方法， ``CIUnitTestCase`` 與提供了一個方便的屬性：

::

    protected $setUpMethods = [
        'mockEmail',
        'mockSession',
    ];

    protected $tearDownMethods = [];


預設情況下，這將處理侵入性服務模擬，但你的類別可以覆寫它們或自行提供：

::

    class OneOfMyModelsTest extends CIUnitTestCase
    {
        protected $tearDownMethods = [
            'purgeRows',
        ];

        protected function purgeRows()
        {
            $this->model->purgeDeleted()
        }

特性
------

強化測試的常見方法是使用特性來合併不同測試案例的過渡環境。　``CIUnitTestCase`` 將檢查所有類別特徵，並訓找以特徵命名的過渡方法。例如：如果你需要向某些測試案例加入身份認證，則可以使用 ``set up`` 方法來建立身份驗證特徵以偽裝使用者登入：

::

    trait AuthTrait
    {
        protected setUpAuthTrait()
        {
            $user = $this->createFakeUser();
            $this->logInUser($user);
        }
    ...

    class AuthenticationFeatureTest
    {
        use AuthTrait;
    ...

額外斷言
---------------------

``CIUnitTestCase`` 提供了額外的單元測試斷言，你可能會覺得這些功能很有用。

**assertLogged($level, $expectedMessage)**

你所期望記錄的實際內容是：

::

        $config = new LoggerConfig();
        $logger = new Logger($config);

        ... do something that you expect a log entry from
        $logger->log('error', "That's no moon");

        $this->assertLogged('error', "That's no moon");

**assertEventTriggered($eventName)**

你所期望觸發的事件實際上是：

::

    Events::on('foo', function($arg) use(&$result) {
        $result = $arg;
    });

    Events::trigger('foo', 'bar');

    $this->assertEventTriggered('foo');

**assertHeaderEmitted($header, $ignoreCase=false)**

你所期待的標頭或 cookie 實際發出的內容是：

::

    $response->setCookie('foo', 'bar');

    ob_start();
    $this->response->send();
    $output = ob_get_clean(); // in case you want to check the actual body

    $this->assertHeaderEmitted("Set-Cookie: foo=bar");

.. note:: 這個測試案例應該在 PHPunit 中作為 `單獨的程序執行 <https://phpunit.readthedocs.io/en/7.4/annotations.html#runinseparateprocess>`_ 。

**assertHeaderNotEmitted($header, $ignoreCase=false)**

你所期待沒有發出這個標頭或 cookie ：

::

    $response->setCookie('foo', 'bar');

    ob_start();
    $this->response->send();
    $output = ob_get_clean(); // in case you want to check the actual body

    $this->assertHeaderNotEmitted("Set-Cookie: banana");

.. note:: 這個測試案例應該在 PHPunit 中作為 `單獨的處理程序執行 <https://phpunit.readthedocs.io/en/7.4/annotations.html#runinseparateprocess>`_ 。

**assertCloseEnough($expected, $actual, $message=\\'\\', $tolerance=1)**

對於延長的執行時間測試來說，判斷你所預期時間與實際時間的相差是否在你規定的公差範圍內：

::

    $timer = new Timer();
    $timer->start('longjohn', strtotime('-11 minutes'));
    $this->assertCloseEnough(11 * 60, $timer->getElapsedTime('longjohn'));

透過上述的設定，可以讓實際時間限制為 660 或 661 秒。

**assertCloseEnoughString($expected, $actual, $message='', $tolerance=1)**

對於延長的執行時間測試來說，將你所預期時間與實際時間的相差，在格式化為字串後，判斷是否在你規定的公差範圍內：

::

    $timer = new Timer();
    $timer->start('longjohn', strtotime('-11 minutes'));
    $this->assertCloseEnoughString(11 * 60, $timer->getElapsedTime('longjohn'));

透過上述的設定，可以讓實際時間限制為 660 或 661 秒。

存取保護或私有屬性
--------------------------------------

測試時，可以使用下述提到的 setter 與 getter 方法，來造訪以及測試類別中的 protected （保護）與  private（私有）方法與屬性。

**getPrivateMethodInvoker($instance, $method)**

你可以從類別外呼叫私有方法，這會回傳一個可以被你呼叫的函數。第一個參數是你所要測試的類別的一個實體，第二個參數是你所要呼叫的方法名稱。

::

    // Create an instance of the class to test
    $obj = new Foo();

    // Get the invoker for the 'privateMethod' method.
    $method = $this->getPrivateMethodInvoker($obj, 'privateMethod');

    // Test the results
    $this->assertEquals('bar', $method('param1', 'param2'));

**getPrivateProperty($instance, $property)**

你可以從一個類別的實體中，檢視一個私有或保護的屬性。第一個參數指的是需要測試的類別的實體，第二個參數是屬性的名稱。

::

    // Create an instance of the class to test
    $obj = new Foo();

    // Test the value
    $this->assertEquals('bar', $this->getPrivateProperty($obj, 'baz'));

**setPrivateProperty($instance, $property, $value)**

在某個類別的實體中，設定一個受保護的值。第一個參數指的是需要測試的類別的實體，第二個參數是待宣告值的屬性的名稱，第三個參數是你所要設定的值：

::

    // Create an instance of the class to test
    $obj = new Foo();

    // Set the value
    $this->setPrivateProperty($obj, 'baz', 'oops!');

    // Do normal testing...

服務的模擬測試
================

你可能會發現，你需要模擬 **app/Config/Services.php** 中某個定義好的服務，以限制你對於程式碼的測試範圍，並同時模擬服務的各種響應。在測試控制器和其他整合測試時更是如此。**服務類別** 提供了 ``injectMock()`` 和 ``reset()`` ，這兩個方法用於簡化這個過程。

**injectMock()**

這個方法允許你宣告服務類別將會回傳的準確實體。你可以使用這個方法來設定服務的屬性，使得它可以以特定的方式執行任務，或者使用測試模擬類別來替換服務。

::

    public function testSomething()
    {
        $curlrequest = $this->getMockBuilder('CodeIgniter\HTTP\CURLRequest')
                            ->setMethods(['request'])
                            ->getMock();
        Services::injectMock('curlrequest', $curlrequest);

        // Do normal testing here....
    }

第一個參數是你所要替換的服務，這個名稱必須與服務類別中函數的名稱完全一致。第二個參數是使用一個實體來替換掉它。

**reset()**

使用這個方法刪除了服務類別中的所有服務模擬類別，它將會恢復到原來的狀態。

**resetSingle(string $name)**

依名稱刪除單個服務的任何模擬與共享實體。

.. note:: 
    ``Email`` 和 ``Session`` 服務將預設模擬以防止侵入性測試行為。 若須阻止這些模擬，請從類別屬性中刪除它們的方法回呼： ``$setUpMethods = ['mockEmail', 'mockSession'];`` 。

模擬工廠實體
=========================

與服務相似，你可能會發現自己需要在測試期間提供一個預先組態的類別實體，這個實體將與 ``Factories`` 一起使用。你可以使用相同的 ``injectMock()`` 和 ``reset()`` 靜態方法（就像 **Services**），但它們需要一個額外的元件名稱做為前置參數：

::

    protected function setUp()
    {
        parent::setUp();

        $model = new MockUserModel();
        Factories::injectMock('models', 'App\Models\UserModel', $model);
    }

.. note::
    預設情形下，在每次測試之間會初始化所有元件工廠。修改測試用例的$setUpMethods 可以選擇是否需要將實體持久化。

串流過濾器
==============

**CITestStreamFilter** 提供了一些輔助函數作為替代方法。

你可能會需要測試一些難以測試的程式。有時，你需要獲取一個串流，例如 PHP 的 STDOUT 或 STDERR ，這個方法可能會以索幫助。 ``CITestStreamFilter`` 可以輔助你從你自你所選擇的串流獲取輸出。

以下範例將展示在測試案例中的使用方式：

::

    public function setUp()
    {
        CITestStreamFilter::$buffer = '';
        $this->stream_filter = stream_filter_append(STDOUT, 'CITestStreamFilter');
    }

    public function tearDown()
    {
        stream_filter_remove($this->stream_filter);
    }

    public function testSomeOutput()
    {
        CLI::write('first.');
        $expected = "first.\n";
        $this->assertEquals($expected, CITestStreamFilter::$buffer);
    }