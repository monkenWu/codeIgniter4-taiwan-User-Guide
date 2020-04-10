###################
Inflector 輔助函式
###################

Inflector 輔助函式檔案包含允許你更改 **英語** 為複數、單數、駝峰式諸類型式的功能

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

讀取此輔助函式
===================

此輔助函式可以利用以下的程式碼讀取::

	helper('inflector');

可用的功能
===================

以下敘述的功能皆可用：

.. php:function:: singular($string)

	:param	string	$string: 輸入字串
	:returns:	一個單數詞
	:rtype:	string

	將複數詞變更為單數詞。例如::

		echo singular('dogs'); // 印出 'dog'

.. php:function:: plural($string)

	:param	string	$string: 輸入字串
	:returns:	一個複數字
	:rtype:	string

	將一個單數詞變更為複數詞。例如::

		echo plural('dog'); // 印出 'dogs'

.. php:function:: counted($count, $string)

	:param	int 	$count:  物品的數字
	:param	string	$string: 輸入數字
	:returns:	一個單數或複數詞語
	:rtype:	string

	將一個字詞與它的數量變更為一個詞語。例如::

		echo counted(3, 'dog'); // 印出 '3 dogs'

.. php:function:: camelize($string)

	:param	string	$string: 輸入字串Input string
	:returns:	駝峰式字串
	:rtype:	string

	將一個字串中以空格隔開或是畫底線的詞語合併成為駝峰式。例如::

		echo camelize('my_dog_spot'); // 印出 'myDogSpot'

.. php:function:: pascalize($string)

	:param	string	$string: 輸入字串
	:returns:	Pascal 字串
	:rtype:	string

	將一個字串中以空格隔開或是畫底線的詞語合併成 Pascal 型式，意即首字為大寫的駝峰式命名。例如::

		echo pascalize('my_dog_spot'); // 印出 'MyDogSpot'

.. php:function:: underscore($string)

	:param	string	$string: 輸入字串
	:returns:	字串包含底線而非空格
	:rtype:	string

	選取多個空格隔開的字並改為底線。
	例如::

		echo underscore('my dog spot'); // 印出 'my_dog_spot'

.. php:function:: humanize($string[, $separator = '_'])

	:param	string	$string: 輸入字串
	:param	string	$separator: 輸入分隔符號
	:returns:	人性化的字串
	:rtype:	string

	選取多個由底線隔開的字並在其中增加空格。每一個詞的第一個字都是大寫的。

	例如::

		echo humanize('my_dog_spot'); // 印出 'My Dog Spot'

	使用破折號而非底線::

		echo humanize('my-dog-spot', '-'); // 印出 'My Dog Spot'

.. php:function:: is_pluralizable($word)

	:param	string	$word: 輸入字串
	:returns:	TRUE：當字詞是可數的 或 FALSE：當字詞不可數
	:rtype:	bool

	確認是否給定的字詞有複數形式。例如::

		is_pluralizable('equipment'); // 回傳 FALSE

.. php:function:: dasherize($string)

	:param	string	$string: 輸入字串
	:returns:	使用破折號的字串
	:rtype:	string

	在字串中以破折號取代底線。例如::

		dasherize('hello_world'); // 回傳 'hello-world'

.. php:function:: ordinal($integer)

	:param	int	$integer: 定義後綴的數字
	:returns:	序號的後綴
	:rtype:	string

	回傳應該加在數字後面以表示序位的後綴，像是
	1st, 2nd, 3rd, 4th。例如::

		ordinal(1); // 回傳 'st'

.. php:function:: ordinalize($integer)

	:param	int	$integer: 要排序的整數
	:returns:	排序過的整數
	:rtype:	string

	將一個數字字串列轉換為表示序位的序數字串，像是 1st, 2nd, 3rd, 4th。
	例如::

		ordinalize(1); // 回傳 '1st'
