###############
陣列輔助函數
###############

陣列輔助函數提供了多種功能用以簡化陣列更複雜的應用。他並不打算複製任何 PHP 現存的功能 - 除非它已經大大的簡化了它們的使用難度

.. contents::
    :local:

載入陣列輔助函數
===================

此陣列輔助函數可以透過以下程式碼載入：

::

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
    
    If the array key contains a dot, then the key can be escaped with a backslash::

        $data = [
            'foo' => [
                'bar.baz' => 23,
            ],
            'foo.bar' => [
                'baz' => 43,
            ],
        ];

        // Returns: 23
        $barBaz = dot_array_search('foo.bar\.baz', $data);
        // Returns: 43
        $fooBar = dot_array_search('foo\.bar.baz', $data);

..  php:function:: array_deep_search($key, array $array)

    :param  mixed  $key: The target key
    :param  array  $array: The array to search
    :returns: The value found within the array, or null
    :rtype: mixed

    Returns the value of an element with a key value in an array of uncertain depth

..  php:function:: array_sort_by_multiple_keys(array &$array, array $sortColumns)

    :param  array  $array:       The array to be sorted (passed by reference).
    :param  array  $sortColumns: The array keys to sort after and the respective PHP
                                 sort flags as an associative array.
    :returns: Whether sorting was successful or not.
    :rtype: bool

    This method sorts the elements of a multidimensional array by the values of one or
    more keys in a hierarchical way. Take the following array, that might be returned
    from, e.g., the ``find()`` function of a model::

        $players = [
            0 => [
                'name'     => 'John',
                'team_id'  => 2,
                'position' => 3,
                'team'     => [
                    'id'    => 1,
                    'order' => 2,
                ],
            ],
            1 => [
                'name'     => 'Maria',
                'team_id'  => 5,
                'position' => 4,
                'team'     => [
                    'id'    => 5,
                    'order' => 1,
                ],
            ],
            2 => [
                'name'     => 'Frank',
                'team_id'  => 5,
                'position' => 1,
                'team'     => [
                    'id'    => 5,
                    'order' => 1,
                ],
            ],
        ];

    Now sort this array by two keys. Note that the method supports the dot-notation
    to access values in deeper array levels, but does not support wildcards::

        array_sort_by_multiple_keys($players, [
            'team.order' => SORT_ASC,
            'position'   => SORT_ASC,
        ]);

    The ``$players`` array is now sorted by the 'order' value in each players'
    'team' subarray. If this value is equal for several players, these players
    will be ordered by their 'position'. The resulting array is::

        $players = [
            0 => [
                'name'     => 'Frank',
                'team_id'  => 5,
                'position' => 1,
                'team'     => [
                    'id' => 5,
                    'order' => 1,
                ],
            ],
            1 => [
                'name'     => 'Maria',
                'team_id'  => 5,
                'position' => 4,
                'team'     => [
                    'id' => 5,
                    'order' => 1,
                ],
            ],
            2 => [
                'name'     => 'John',
                'team_id'  => 2,
                'position' => 3,
                'team'     => [
                    'id' => 1,
                    'order' => 2,
                ],
            ],
        ];

    In the same way, the method can also handle an array of objects. In the example
    above it is further possible that each 'player' is represented by an array,
    while the 'teams' are objects. The method will detect the type of elements in
    each nesting level and handle it accordingly.

.. php:function:: array_flatten_with_dots(iterable $array[, string $id = '']): array

    :param iterable $array: The multidimensional array to flatten
    :param string $id: Optional ID to prepend to the outer keys. Used internally for flattening keys.
    :rtype: array
    :returns: The flattened array

    This function flattens a multidimensional array to a single key-value array by using dots
    as separators for the keys.

    ::

        $arrayToFlatten = [
            'personal' => [
                'first_name' => 'john',
                'last_name'  => 'smith',
                'age'        => '26',
                'address'    => 'US',
            ],
            'other_details' => 'marines officer',
        ];

        $flattened = array_flatten_with_dots($arrayToFlatten);

    On inspection, ``$flattened`` is equal to::

        [
            'personal.first_name' => 'john',
            'personal.last_name'  => 'smith',
            'personal.age'        => '26',
            'personal.address'    => 'US',
            'other_details'       => 'marines officer',
        ];

    Users may use the ``$id`` parameter on their own, but are not required to do so.
    The function uses this parameter internally to track the flattened keys. If users
    will be supplying an initial ``$id``, it will be prepended to all keys.

    ::

        // using the same data from above
        $flattened = array_flatten_with_dots($arrayToFlatten, 'foo_');

        // $flattened is now:
        [
            'foo_personal.first_name' => 'john',
            'foo_personal.last_name'  => 'smith',
            'foo_personal.age'        => '26',
            'foo_personal.address'    => 'US',
            'foo_other_details'       => 'marines officer',
        ];
