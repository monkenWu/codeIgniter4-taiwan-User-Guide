###############
Array 輔助函式
###############

Array 輔助函數提供了多種功能用以簡化陣列更複雜的應用。他並不打算複製任何 PHP 現存的功能 - 除非它已經大大的簡化了它們的使用難度

.. contents::
    :local:

讀取此陣列輔助函式
===================

此陣列輔助函式可以透過以下程式碼讀取::

	helper('array');

可用的功能
===================

以下列舉的功能都可用:

..  php:function:: dot_array_search(string $search, array $values)

    :param  string  $search: dot-notation String 解釋了如何對陣列進行搜尋
    :param  array   $values: 欲搜尋的字串
    :returns: 在陣列中所搜尋到S的值，或是空
    :rtype: 混合型態

    此方法使你能夠使用 dot-notation 對陣列搜尋一個特定的值，並且允許使用「 * 」通配符。給定以下陣列 ::

        $data = [
            'foo' => [
                'buzz' => [
                    'fizz' => 11
                ],
                'bar' => [
                    'baz' => 23
                ]
            ]
        ]

    我們可以利用 「 foo.buzz.fizz 」 這個搜尋字串定位 fizz 的值。同樣的， baz 的值可以透過 「 foo.bar.baz 」 找到::

        // Returns: 11
        $fizz = dot_array_search('foo.buzz.fizz', $data);

        // Returns: 23
        $baz = dot_array_search('foo.bar.baz', $data);

    你可以利用星號取代任何的段落。當星號被發現，它會對所有的子節點做搜尋直到找出結果。如果你不知道你的值，或是你的值有一個數字的指標，這個功能是很方便的。::

        // Returns: 23
        $baz = dot_array_search('foo.*.baz', $data);
