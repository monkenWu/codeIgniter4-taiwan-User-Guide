##############
命令列程式庫
##############

CodeIgniter 的命令列程式庫使創建互動式命令列腳本變得簡單，它包括：

* 提示使用者更多資訊
* 使你撰寫的文字在終端機中色彩斑斕
* Beeping (be nice!)
* 在長時間的任務中顯示進度條
* 自動替長文字換行，以符合視窗大小

.. contents::
    :local:
    :depth: 2

初始化類別
======================

你不宣要建立一個命令列程式庫的實體，因為它所有的方法都是靜態的，你只需要在你的控制器上方使用 ``use`` 語句來引入它即可。

::

    <?php

    namespace App\Controllers;

    use CodeIgniter\CLI\CLI;

    class MyController extends \CodeIgniter\Controller
    {
        // ...
    }

當你第一次載入檔案時，類別會自動初始化。

取得使用者輸入
===========================

有時，你會需要向使用者詢問更多的訊息，原因可能是，他們沒有提供指令需要的必要參數，或者是腳本在寫入檔案前發現檔案已存在，需要在覆寫前確認——這些需求都可以使用 ``prompt()`` 方法來處理。

你可以透過傳入問題字串作為第一個參數：

::

	$color = CLI::prompt('What is your favorite color?');

你可以提供一個預設的答案，如果使用者在傳遞第二個參數的過程中採用預設值來輸入，那麼將使用你所提供的答案：

::

	$color = CLI::prompt('What is your favorite color?', 'blue');

你可以透過在第二個參數中傳入一個允許的答案陣列來限制你可以接受的回答：

::

	$overwrite = CLI::prompt('File exists. Overwrite?', ['y','n']);

最後，你可以將你的驗證規則作為第三個參數傳入：

::

	$email = CLI::prompt('What is your email?', null, 'required|valid_email');

驗證規則也可以用陣列語法撰寫：

::

    $email = CLI::prompt('What is your email?', null, ['required', 'valid_email']);

**promptByKey()**

提示預設回答（選項），有時選項需要描述或太過複雜而無法單純透過數值來選擇。而 ``promptByKey()`` 能夠讓使用者透過鍵值而不是數值來選擇一個選項：

::

    $fruit = CLI::promptByKey('These are your choices:', ['The red apple', 'The plump orange', 'The ripe banana']);

    //These are your choices:
    //  [0]  The red apple
    //  [1]  The plump orange
    //  [2]  The ripe banana
    //
    //[0, 1, 2]:

你也可以自行命名鍵值：

::

    $fruit = CLI::promptByKey(['These are your choices:', 'Which would you like?'], [
        'apple' => 'The red apple',
        'orange' => 'The plump orange',
        'banana' => 'The ripe banana'
    ]);

    //These are your choices:
    //  [apple]   The red apple
    //  [orange]  The plump orange
    //  [banana]  The ripe banana
    //
    //Which would you like? [apple, orange, banana]:

最後，你可以將 :doc:`驗證 </libraries/validation>` 規則作為第三個參數傳遞給回答的輸入，能夠被接受的回答會自動地限制在傳遞的選項中。

提供回饋
==================

**write()**

我們為你提供了幾種方法來向使用者輸出回饋訊息，這可以是簡單的單一狀態更新，也可以是一個複雜的訊息表格，這些需求都可以完整的輸出到使用者的終端機視窗。其中，最核心的方法就屬 ``write()`` 方法了，它把需要輸出的字串作為第一個被傳入的參數：

::

	CLI::write('The rain in Spain falls mainly on the plains.');

你可以透過傳入一個顏色的名稱來做為第二個參數，這可以改變文字的顏色：

::

	CLI::write('File created.', 'green');

這可以用來區分訊息的狀態，或者用不同的顏色來創建頁眉，你甚至可以透過在第三個參數中傳入顏色的名稱來設定背景顏色：

::

	CLI::write('File overwritten.', 'light_red', 'dark_gray');

以下為可以使用的顏色：

* black
* dark_gray
* blue
* dark_blue
* light_blue
* green
* light_green
* cyan
* light_cyan
* red
* light_red
* purple
* light_purple
* light_yellow
* yellow
* light_gray
* white

以下為可以使用的背景顏色：

* black
* blue
* green
* cyan
* red
* yellow
* light_gray
* magenta

**print()**

``print()`` 函數與 ``write()`` 函數相同，使是它不強制在前後加上換行。相反的，它會將內容輸出到螢幕上游標的位置。這使得你可以在同一行中輸出多個項目，並從不同的呼叫中持續輸出。當你想顯示一些狀態、做一些事情，然後在同一列中輸出 "Done" 時，這個方法特別有用：

::

    for ($i = 0; $i <= 10; $i++) {
        CLI::print($i);
    }


**color()**

雖然 ``write()`` 指令會把一行字輸出到終端機上，並以 EOL 符號結束，但你也可以用 ``color()`` 方法製作一個字串片段，這時就不會強制執行 EOL 。這樣你就可以在同一列上創建多個輸出。或者，你可以在 ``write()`` 內部使用 ``color()`` 方法，就能輸出一個不同顏色的字串了：

::

	CLI::write("fileA \t". CLI::color('/path/to/file', 'white'), 'yellow');

這個例子會在視窗寫入一行文字， ``fileA`` 是黃色的，然後 ``/path/to/file`` 是白色的。

**error()**

如果你需要輸出錯誤，你應該使用有著錯誤語意的 ``error()`` 方法。這個方法會與 ``write()`` 以及 ``color()`` 一樣，將淺紅色的文字寫到 STDERR 而不是 STDOUT 。如果你的腳本正在監視錯誤，那麼這個方法將會十分有用，因為這樣他們就不用去篩選所有的訊息，只需要篩選實際的錯誤訊息。你可以像使用 ``write()`` 方法一樣使用它：

::

	CLI::error('Cannot write to file: ' . $file);

**wrap()**

這個指令將會把字串在當前的行上輸出，然後將其換行到新的行上設定長度。當顯示一個帶有著描述選項的列表時，這個功能會很有用，因為你一定會想讓訊息乖乖地待在視窗中，而不是超出螢幕大小：

::

	CLI::color("task1\t", 'yellow');
	CLI::wrap("Some long description goes here that might be longer than the current window.");

在預設的情形下，字串將在終端機的實際寬度下換行。 Windows 目前並沒有提供確定視窗大小的方法，所以我們將預設為單行 80 個字元。如果要將寬度限制為較短的寬度，以便確定符合視窗的話，則可以將最大長度作為第二個參數傳入方法。這將在最接近長度邊緣的單字進行換行，避免單字被切一半這件事發生：

::

	// 限制單行文字為最大 20 字寬
	CLI::wrap($description, 20);

你可能會有標題、檔案名稱、工作名稱記錄在左側，並且右側有包含其說明文字的需求。在預設的情形下，右方文字的換行將會回到視窗的左側，將沒辦法允許你這麼排版。在這種情況下，你可以傳入多個空格，在第一行之後填充每一行，這樣你就能在左方獲得清晰的邊緣列：

::

    $titles = [
        'task1a',
        'task1abc',
    ];
    $descriptions = [
        'Lorem Ipsum is simply dummy text of the printing and typesetting industry.',
        "Lorem Ipsum has been the industry's standard dummy text ever since the",
    ];

    // Determine the maximum length of all titles
    // to determine the width of the left column
    $maxlen = max(array_map('strlen', $titles));

    for ($i = 0; $i < count($titles); $i++) {
        CLI::write(
            // Display the title on the left of the row
            substr(
                $titles[$i] . str_repeat(' ', $maxlen + 3),
                0,
                $maxlen + 3
            ) .
            // Wrap the descriptions in a right-hand column
            // with its left side 3 characters wider than
            // the longest item on the left.
            CLI::wrap($descriptions[$i], 40, $maxlen + 3)
        );
    }

會建立起下列內容：

.. code-block:: none

    task1a     Lorem Ipsum is simply dummy
               text of the printing and typesetting
               industry.
    task1abc   Lorem Ipsum has been the industry's
               standard dummy text ever since the

**newLine()**

``newLine()`` 方法將向使用者輸出空行，它並不接受任何參數的傳入：

::

	CLI::newLine();

**clearScreen()**

你可以使用 ``clearScreen()`` 方法清除目前視窗內的內容，在大多數版本的 Windows 內，這將簡單地插入 40 行的空行，因為 Windows 不支援這個功能。但 Windows10 的 bash 集成將可以改變這一點：

::

	CLI::clearScreen();

**showProgress()**

如果你有一個長期執行的任務，你想讓使用者了解最新的進度，你可以使用 ``showProgress()`` 方法來表示進度，這個方法顯示的內容如下：

.. code-block:: none

	[####......] 40% Complete

這個進度方塊的動畫效果非常好。

要使用它的話，請在第一個參數中輸入目前的步驟，並將總步驟數當成第二個參數傳入。完成的百分比以及顯示的長度將根據這兩個數字決定。完成後，傳入 false 進度條就會被刪除。

::

    $totalSteps = count($tasks);
    $currStep   = 1;

    foreach ($tasks as $task) {
        CLI::showProgress($currStep++, $totalSteps);
        $task->run();
    }

    // Done, so erase it...
    CLI::showProgress(false);

**table()**

::

	$thead = ['ID', 'Title', 'Updated At', 'Active'];
	$tbody = [
		[7, 'A great item title', '2017-11-15 10:35:02', 1],
		[8, 'Another great item title', '2017-11-16 13:46:54', 0]
	];

	CLI::table($tbody, $thead);

.. code-block:: none

	+----+--------------------------+---------------------+--------+
	| ID | Title                    | Updated At          | Active |
	+----+--------------------------+---------------------+--------+
	| 7  | A great item title       | 2017-11-16 10:35:02 | 1      |
	| 8  | Another great item title | 2017-11-16 13:46:54 | 0      |
	+----+--------------------------+---------------------+--------+

**wait()**

等待一定的秒數，可以選擇等待訊息以及等待按鍵。

::

    // 將會等待你所指定的時間，並顯示倒數計時
    CLI::wait($seconds, true);
    // 繼續顯示訊息，等待輸入
    CLI::wait(0, false);
    // 等待你所指定的時間
    CLI::wait($seconds, false);