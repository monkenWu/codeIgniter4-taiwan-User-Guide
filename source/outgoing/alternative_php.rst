###################################
用於視圖檔案的 PHP 語法
###################################

如果你沒有利用樣板引擎來簡化輸出，你將可以在 View 檔案中使用純 PHP 進行渲染。為了盡量減少這些檔案中的 PHP 程式碼，並讓它成為更容易被識別的區塊，建議你使用 PHP 替代語法來簡化控制結構以及 echo 語句。

替代 Echo
=================

通常的情況下，如果你需要使用 echo 或者是輸出一個變數，你可能會這麼做：

::

	<?php echo $variable; ?>

使用替代語法，你就能這麼簡化它：

::

	<?= $variable ?>

替代控制結構
==============================

在控制結構中，如：if 、for 、foreach ，以及 while ，也可以使用簡化的方式撰寫，我們來舉個 ``foreach`` 的簡化範例：

::

    <ul>

    <?php foreach ($todo as $item) : ?>

        <li><?= $item ?></li>

    <?php endforeach ?>

    </ul>

注意，簡化語法將省略大括弧。相反的，結構語句後頭緊接著的括弧被話成了冒號與 ``endforeach`` 。上面所列出的控制結構都有相應的結束於法： ``endif`` 、 ``endfor`` 、 ``endforeach`` ，以及 ``endwhile`` 。

另外，請注意在每個控制結構之後（除了最後一個結構以外）不要使用分號結束，要使用冒號，這一點非常重要！

這裡再舉個　``if``/``elseif``/``else``　的例子，請注意分號的使用方式：

::

    <?php if ($username === 'sally') : ?>

        <h3>Hi Sally</h3>

    <?php elseif ($username === 'joe') : ?>

        <h3>Hi Joe</h3>

    <?php else : ?>

        <h3>Hi unknown user</h3>

    <?php endif ?>
