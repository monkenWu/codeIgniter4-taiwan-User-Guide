事件
#####################################

CodeIgniter 的事件功能提供一種方法，可以在不侵入核心檔案的情形下，進入和修改框架內部的工作。當 CodeIgniter 運作時，它會遵循著一定的執行順序，然而在某些情況下，你可能會想在執行過程的特定階段執行一些動作。例如：你需要在載入控制器的前後執行自己撰寫的腳本，亦或是你想在其他地方觸發自己的腳本。

事件採用 *發布/訂用* 模式運作，在腳本執行的過程中某個時刻會觸發一個事件，其他腳本就可以透過事件類別來 "訂用" 這個事件，讓它知道自己觸發該事件前會執行一些動作。

事件賦能
===============

事件始終處於可以使用的狀態，並且全域可用。

定義一個事件
=================

大部分的事件是在 **app/Config/Events.php** 檔案中定義，你可以利用事件類別的 ``on()`` 方法訂用到某個事件。第一個參數需傳入要訂用事件名稱，第二個參數是觸發該事件時將執行呼叫的參數：

::

	use CodeIgniter\Events\Events;

	Events::on('pre_system', ['MyClass', 'MyFunction']);

在這個例子中，每當 **pre_controller** 事件被執行時， ``MyClass`` 將被創建實體，並運作 ``MyFunction`` 方法。請注意，第二個參數需要讓 PHP 識別並且 `可以被呼叫 <https://www.php.net/manual/en/function.is-callable.php>`_ ：

::

	// Call a standalone function
	Events::on('pre_system', 'some_function');

	// Call on an instance method
	$user = new User();
	Events::on('pre_system', [$user, 'some_method']);

	// Call on a static method
	Events::on('pre_system', 'SomeClass::someMethod');

	// Use a Closure
	Events::on('pre_system', function(...$params)
	{
		. . .
	});

設定屬性
------------------

由於許多方法都可以被訂用到同一個事件中，所以你還需要一個方法定義這些方法被呼叫的順序。你可以透過傳入一個優先級作為 ``on()`` 方法的第三個參數實現這個功能。數值較低者則會先被執行，1 為最高優先級，並且較低的值沒有任何限制：

::

    Events::on('post_controller_constructor', 'some_function', 25);

任何具有相同優先級的訂用者都將會按照宣告生效的順序執行。

我們事先宣告了三個常數可以供你使用，這些常數設定了一些範圍，你可不使用這些常數，但它們將會幫助你增加程式碼的可讀性：

::

	define('EVENT_PRIORITY_LOW', 200);
	define('EVENT_PRIORITY_NORMAL', 100);
	define('EVENT_PRIORITY_HIGH', 10);

一旦排序後，所有的訂用者都會按照順序執行，如果任何訂用者回傳了一個布林 false ，那麼訂用者的執行將會暫停。

發布自己的活動
==========================

事件程式庫讓你可以簡單地創建事件。若需要使用這個功能，只需要在 **Events** 中呼叫 ``trigger()`` 方法即可：

::

	\CodeIgniter\Events\Events::trigger('some_event');

你可以透過加入任何數量的參數作為附加參數傳遞給訂用者，訂用者將按照宣告的順序取得這些參數：

::

	\CodeIgniter\Events\Events::trigger('some_events', $foo, $bar, $baz);

	Events::on('some_event', function($foo, $bar, $baz) {
		...
	});

模擬事件
=================

在測試的過程中，你可能不會希望事件真的被啟動了，例如：因為每天發送幾百封電子郵件既慢又會產生反效果。你可以告訴事件類別使用 ``simulate()`` 方法來模擬事件的運作。當它為 **true** 時，所有事件都將在觸發方法期間跳過，不過其它功能都會照常運作。

::

    Events::simulate(true);

你可以透過傳入 false 停止模擬：

::

    Events::simulate(false);

事件點
============

以下將列出 CodeIgniter 的核心程式碼中可以使用的事件點：

* **pre_system** 在系統執行初期就被呼叫，此時僅載入了基本類別以及事件類別，未執行路由或其他進程。
* **post_controller_constructor** 在控制器被實體化後及任何方法被呼叫前，立即呼叫。
* **post_system** 在最終渲染的頁面被發送到瀏覽器後、在系統執行結束後，在最終化的資料被發送到瀏覽器後呼叫。
