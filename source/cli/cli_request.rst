****************
CLI 請求類別
****************

如果有一個請求來自於命令列的呼叫，那麼這個請求的物件實際上就是一個 ``CLIRequest`` 。它的行為與傳統的 :doc:`請求 </incoming/request>` 相同，但為了方便起見，增加了一些存取器的方法。

====================
存取器
====================

**getSegments()**

回傳一個被視為路徑的一部份的命令列參數的陣列。

::

    // command line: php index.php users 21 profile -foo bar
    echo $request->getSegments();  // ['users', '21', 'profile']

**getPath()**

將重建的路徑作為字串回傳：

::

    // command line: php index.php users 21 profile -foo bar
    echo $request->getPath();  // users/21/profile

**getOptions()**

回傳被視為選項的命令列參數陣列：

::

    // command line: php index.php users 21 profile -foo bar
    echo $request->getOptions();  // ['foo' => 'bar']

**getOption($which)**

回傳被視為選項的特定命令列參數的值：

::

    // command line: php index.php users 21 profile -foo bar
    echo $request->getOption('foo');  // bar
    echo $request->getOption('notthere'); // NULL

**getOptionString()**

回傳選項的重建命令列字串：

::

    // command line: php index.php users 21 profile -foo bar
    echo $request->getOptionPath();  // -foo bar

Passing ``true`` to the first argument will try to write long options using two dashes::

    // php index.php user 21 --foo bar -f
    echo $request->getOptionString(); // -foo bar -f
    echo $request->getOptionString(true); // --foo bar -f
