##############
表單輔助函數
##############

表單輔助涵式檔案包含與型式互動的輔助功能

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

載入此輔助函數
===================

此輔助函數可以利用以下的程式碼讀取：

::

	helper('form');

脫離領域的值
=====================

你或許需要使用 HTML 與像是引用的字元在你的型式元素中。 
為了安全的實作，你會需要使用
:doc:`common function <../general/common_functions>`
:func:`esc()`.

以底下的範例為考量::

	$string = 'Here is a string containing "quoted" text.';

	<input type="text" name="myfield" value="<?= $string; ?>" />

由於上述的字串包含一組引用，他將會造成型式破裂
:php:func:`esc()` 功能會轉換特殊的 HTML 字元以便它可以安全地被利用::

	<input type="text" name="myfield" value="<?= esc($string); ?>" />

.. note:: 如果你使用此頁面所列出的任一 form 輔助函數，並且你傳遞值作為關聯陣列，
	form 的值會被自動脫離，因此你沒有必要呼叫此函式。只有你在建立你自己的 form 元素的時候使用，在你會以字串型式傳遞的時候使用

可用的功能
===================

以下敘述的功能皆可用：

.. php:function:: form_open([$action = ''[, $attributes = ''[, $hidden = []]]])

	:param	string	$action: Form 動作／目標 URI 字串
    	:param	mixed	$attributes: HTML 屬性，是一個陣列或跳脫的字串
    	:param	array	$hidden: 一個隱藏域的定義陣列
    	:returns:	HTML form 開啟標籤
    	:rtype:	string

    	建立一個啟動標籤內含一個 **從你的配置偏好建立的** 基礎 URL。
		它將會讓你選擇增加 form 屬性與隱藏輸入區，同時也會根據你的 config 檔案中的 charset 值永遠增加 `accept-charset` 屬性。

	使用這個標籤而不是強硬的編寫你自己的 HTML 程式的主要好處是它將會讓你的網站在變更 URL 時更輕便

	這是一個簡單的範例::

		echo form_open('email/send');

	上述的範例會建立一個 form 指向你的基礎 URL 加上 "email/send" URI 段落，像是這個::

		<form method="post" accept-charset="utf-8" action="http://example.com/index.php/email/send">

	**增加屬性**

		屬性可以透過傳遞一個關聯的陣列到第二個參數來增加，像是這樣::

			$attributes = ['class' => 'email', 'id' => 'myform'];
			echo form_open('email/send', $attributes);

		或者，你可以指定第二個參數為一個字串::

			echo form_open('email/send', 'class="email" id="myform"');

		上述的範例會產生一個類似於這個的 form::

			<form method="post" accept-charset="utf-8" action="http://example.com/index.php/email/send" class="email" id="myform">
			
		如果 CSRF 過濾器是開啟的，`form_open()` 將會在 form 的最一開始產生 CSRF 域。你可以透過將 csrf_id 作為其中一個 $attribute 陣列傳送進 form 以指定這個域的ID:
		
			form_open('/u/sign-up', ['csrf_id' => 'my-id']);
			
		它將會回傳:
		
			<form action="/u/sign-up" method="post" accept-charset="utf-8">
			<input type="hidden" id="my-id" name="csrf_field" value="964ede6e0ae8a680f7b8eab69136717d" />

	**增加隱藏的輸入域**

		隱藏的輸入域可以藉由傳送一個關聯的陣列到第三個參數以增加，項是這樣::

			$hidden = ['username' => 'Joe', 'member_id' => '234'];
			echo form_open('email/send', '', $hidden);

		你可以透過傳遞任何 false 值到第二參數以跳過第二個參數。

		上述的範例會產生一個類似於這個的 form::

			<form method="post" accept-charset="utf-8" action="http://example.com/index.php/email/send">
				<input type="hidden" name="username" value="Joe" />
				<input type="hidden" name="member_id" value="234" />

.. php:function:: form_open_multipart([$action = ''[, $attributes = ''[, $hidden = []]]])

	:param	string	$action: Form 動作／目標 URI 字串
    	:param	mixed	$attributes: HTML 屬性，是一個陣列或跳脫的字串
    	:param	array	$hidden: 一個隱藏域的定義陣列
    	:returns:	HTML multipart 多部份的 form 開啟標籤
    	:rtype:	string

    	This function is identical to :php:func:`form_open()` above,
	except that it adds a *multipart* attribute, which is necessary if you
	would like to use the form to upload files with.

.. php:function:: form_hidden($name[, $value = ''])

	:param	string	$name: Field name
    	:param	string	$value: Field value
    	:returns:	An HTML hidden input field tag
    	:rtype:	string

    	Lets you generate hidden input fields. You can either submit a
    	name/value string to create one field::

		form_hidden('username', 'johndoe');
		// Would produce: <input type="hidden" name="username" value="johndoe" />

	... or you can submit an associative array to create multiple fields::

		$data = [
			'name'	=> 'John Doe',
			'email'	=> 'john@example.com',
			'url'	=> 'http://example.com'
		];

		echo form_hidden($data);

		/*
			Would produce:
			<input type="hidden" name="name" value="John Doe" />
			<input type="hidden" name="email" value="john@example.com" />
			<input type="hidden" name="url" value="http://example.com" />
		*/

	You can also pass an associative array to the value field::

		$data = [
			'name'	=> 'John Doe',
			'email'	=> 'john@example.com',
			'url'	=> 'http://example.com'
		];

		echo form_hidden('my_array', $data);

		/*
			Would produce:

			<input type="hidden" name="my_array[name]" value="John Doe" />
			<input type="hidden" name="my_array[email]" value="john@example.com" />
			<input type="hidden" name="my_array[url]" value="http://example.com" />
		*/

	If you want to create hidden input fields with extra attributes::

		$data = [
			'type'	=> 'hidden',
			'name'	=> 'email',
			'id'	=> 'hiddenemail',
			'value'	=> 'john@example.com',
			'class'	=> 'hiddenemail'
		];

		echo form_input($data);

		/*
			Would produce:

			<input type="hidden" name="email" value="john@example.com" id="hiddenemail" class="hiddenemail" />
		*/

.. php:function:: form_input([$data = ''[, $value = ''[, $extra = ''[, $type = 'text']]]])

	:param	array	$data: Field attributes data
	:param	string	$value: Field value
	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
	:param  string  $type: The type of input field. i.e. 'text', 'email', 'number', etc.
	:returns:	An HTML text input field tag
	:rtype:	string

	Lets you generate a standard text input field. You can minimally pass
	the field name and value in the first and second parameter::

		echo form_input('username', 'johndoe');

	Or you can pass an associative array containing any data you wish your
	form to contain::

		$data = [
			'name'      => 'username',
			'id'        => 'username',
			'value'     => 'johndoe',
			'maxlength' => '100',
			'size'      => '50',
			'style'     => 'width:50%'
		];

		echo form_input($data);

		/*
			Would produce:

			<input type="text" name="username" value="johndoe" id="username" maxlength="100" size="50" style="width:50%"  />
		*/

	If you would like your form to contain some additional data, like
	JavaScript, you can pass it as a string in the third parameter::

		$js = 'onClick="some_function()"';
		echo form_input('username', 'johndoe', $js);

	Or you can pass it as an array::

		$js = ['onClick' => 'some_function();'];
		echo form_input('username', 'johndoe', $js);

	To support the expanded range of HTML5 input fields, you can pass an input type in as the fourth parameter::

		echo form_input('email', 'joe@example.com', ['placeholder' => 'Email Address...'], 'email');

		/*
			 Would produce:

			<input type="email" name="email" value="joe@example.com" placeholder="Email Address..." />
		*/

.. php:function:: form_password([$data = ''[, $value = ''[, $extra = '']]])

	:param	array	$data: Field attributes data
    	:param	string	$value: Field value
    	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML password input field tag
    	:rtype:	string

    	This function is identical in all respects to the :php:func:`form_input()`
	function above except that it uses the "password" input type.

.. php:function:: form_upload([$data = ''[, $value = ''[, $extra = '']]])

	:param	array	$data: Field attributes data
    	:param	string	$value: Field value
    	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML file upload input field tag
    	:rtype:	string

    	This function is identical in all respects to the :php:func:`form_input()`
	function above except that it uses the "file" input type, allowing it to
	be used to upload files.

.. php:function:: form_textarea([$data = ''[, $value = ''[, $extra = '']]])

	:param	array	$data: Field attributes data
    	:param	string	$value: Field value
    	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML textarea tag
    	:rtype:	string

    	This function is identical in all respects to the :php:func:`form_input()`
	function above except that it generates a "textarea" type.

	.. note:: Instead of the *maxlength* and *size* attributes in the above example,
		you will instead specify *rows* and *cols*.

.. php:function:: form_dropdown([$name = ''[, $options = [][, $selected = [][, $extra = '']]]])

	:param	string	$name: Field name
	:param	array	$options: An associative array of options to be listed
    	:param	array	$selected: List of fields to mark with the *selected* attribute
	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML dropdown select field tag
    	:rtype:	string

    	Lets you create a standard drop-down field. The first parameter will
    	contain the name of the field, the second parameter will contain an
    	associative array of options, and the third parameter will contain the
    	value you wish to be selected. You can also pass an array of multiple
    	items through the third parameter, and the helper will create a
    	multiple select for you.

    	Example::

		$options = [
			'small'  => 'Small Shirt',
			'med'    => 'Medium Shirt',
			'large'  => 'Large Shirt',
			'xlarge' => 'Extra Large Shirt',
		];

		$shirts_on_sale = ['small', 'large'];
		echo form_dropdown('shirts', $options, 'large');

		/*
			Would produce:

			<select name="shirts">
				<option value="small">Small Shirt</option>
				<option value="med">Medium Shirt</option>
				<option value="large" selected="selected">Large Shirt</option>
				<option value="xlarge">Extra Large Shirt</option>
			</select>
		*/

		echo form_dropdown('shirts', $options, $shirts_on_sale);

		/*
			Would produce:

			<select name="shirts" multiple="multiple">
				<option value="small" selected="selected">Small Shirt</option>
				<option value="med">Medium Shirt</option>
				<option value="large" selected="selected">Large Shirt</option>
				<option value="xlarge">Extra Large Shirt</option>
			</select>
		*/

	If you would like the opening <select> to contain additional data, like
	an id attribute or JavaScript, you can pass it as a string in the fourth
	parameter::

		$js = 'id="shirts" onChange="some_function();"';
		echo form_dropdown('shirts', $options, 'large', $js);

	Or you can pass it as an array::

		$js = [
			'id'       => 'shirts',
			'onChange' => 'some_function();'
		];
		echo form_dropdown('shirts', $options, 'large', $js);

	If the array passed as ``$options`` is a multidimensional array, then
	``form_dropdown()`` will produce an <optgroup> with the array key as the
	label.

.. php:function:: form_multiselect([$name = ''[, $options = [][, $selected = [][, $extra = '']]]])

	:param	string	$name: Field name
    	:param	array	$options: An associative array of options to be listed
    	:param	array	$selected: List of fields to mark with the *selected* attribute
	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML dropdown multiselect field tag
    	:rtype:	string

    	Lets you create a standard multiselect field. The first parameter will
    	contain the name of the field, the second parameter will contain an
    	associative array of options, and the third parameter will contain the
    	value or values you wish to be selected.

    	The parameter usage is identical to using :php:func:`form_dropdown()` above,
	except of course that the name of the field will need to use POST array
	syntax, e.g. foo[].

.. php:function:: form_fieldset([$legend_text = ''[, $attributes = []]])

	:param	string	$legend_text: Text to put in the <legend> tag
    	:param	array	$attributes: Attributes to be set on the <fieldset> tag
    	:returns:	An HTML fieldset opening tag
    	:rtype:	string

    	Lets you generate fieldset/legend fields.

    	Example::

		echo form_fieldset('Address Information');
		echo "<p>fieldset content here</p>\n";
		echo form_fieldset_close();

		/*
			Produces:

				<fieldset>
					<legend>Address Information</legend>
						<p>form content here</p>
				</fieldset>
		*/

	Similar to other functions, you can submit an associative array in the
	second parameter if you prefer to set additional attributes::

		$attributes = [
			'id'	=> 'address_info',
			'class'	=> 'address_info'
		];

		echo form_fieldset('Address Information', $attributes);
		echo "<p>fieldset content here</p>\n";
		echo form_fieldset_close();

		/*
			Produces:

			<fieldset id="address_info" class="address_info">
				<legend>Address Information</legend>
				<p>form content here</p>
			</fieldset>
		*/

.. php:function:: form_fieldset_close([$extra = ''])

	:param	string	$extra: Anything to append after the closing tag, *as is*
	:returns:	An HTML fieldset closing tag
	:rtype:	string

	Produces a closing </fieldset> tag. The only advantage to using this
	function is it permits you to pass data to it which will be added below
	the tag. For example

	::

		$string = '</div></div>';
		echo form_fieldset_close($string);
		// Would produce: </fieldset></div></div>

.. php:function:: form_checkbox([$data = ''[, $value = ''[, $checked = FALSE[, $extra = '']]]])

	:param	array	$data: Field attributes data
    	:param	string	$value: Field value
    	:param	bool	$checked: Whether to mark the checkbox as being *checked*
	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML checkbox input tag
    	:rtype:	string

    	Lets you generate a checkbox field. Simple example::

		echo form_checkbox('newsletter', 'accept', TRUE);
		// Would produce:  <input type="checkbox" name="newsletter" value="accept" checked="checked" />

	The third parameter contains a boolean TRUE/FALSE to determine whether
	the box should be checked or not.

	Similar to the other form functions in this helper, you can also pass an
	array of attributes to the function::

		$data = [
			'name'    => 'newsletter',
			'id'      => 'newsletter',
			'value'   => 'accept',
			'checked' => TRUE,
			'style'   => 'margin:10px'
		];

		echo form_checkbox($data);
		// Would produce: <input type="checkbox" name="newsletter" id="newsletter" value="accept" checked="checked" style="margin:10px" />

	Also as with other functions, if you would like the tag to contain
	additional data like JavaScript, you can pass it as a string in the
	fourth parameter::

		$js = 'onClick="some_function()"';
		echo form_checkbox('newsletter', 'accept', TRUE, $js);

	Or you can pass it as an array::

		$js = ['onClick' => 'some_function();'];
		echo form_checkbox('newsletter', 'accept', TRUE, $js);

.. php:function:: form_radio([$data = ''[, $value = ''[, $checked = FALSE[, $extra = '']]]])

	:param	array	$data: Field attributes data
    	:param	string	$value: Field value
    	:param	bool	$checked: Whether to mark the radio button as being *checked*
	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML radio input tag
    	:rtype:	string

    	This function is identical in all respects to the :php:func:`form_checkbox()`
	function above except that it uses the "radio" input type.

.. php:function:: form_label([$label_text = ''[, $id = ''[, $attributes = []]]])

	:param	string	$label_text: Text to put in the <label> tag
    	:param	string	$id: ID of the form element that we're making a label for
    	:param	string	$attributes: HTML attributes
    	:returns:	An HTML field label tag
    	:rtype:	string

    	Lets you generate a <label>. Simple example::

		echo form_label('What is your Name', 'username');
		// Would produce:  <label for="username">What is your Name</label>

	Similar to other functions, you can submit an associative array in the
	third parameter if you prefer to set additional attributes.

	Example::

		$attributes = [
			'class' => 'mycustomclass',
			'style' => 'color: #000;'
		];

		echo form_label('What is your Name', 'username', $attributes);
		// Would produce:  <label for="username" class="mycustomclass" style="color: #000;">What is your Name</label>

.. php:function:: form_submit([$data = ''[, $value = ''[, $extra = '']]])

	:param	string	$data: Button name
    	:param	string	$value: Button value
    	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML input submit tag
    	:rtype:	string

    	Lets you generate a standard submit button. Simple example::

		echo form_submit('mysubmit', 'Submit Post!');
		// Would produce:  <input type="submit" name="mysubmit" value="Submit Post!" />

	Similar to other functions, you can submit an associative array in the
	first parameter if you prefer to set your own attributes. The third
	parameter lets you add extra data to your form, like JavaScript.

.. php:function:: form_reset([$data = ''[, $value = ''[, $extra = '']]])

	:param	string	$data: Button name
    	:param	string	$value: Button value
    	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML input reset button tag
    	:rtype:	string

    	Lets you generate a standard reset button. Use is identical to
    	:func:`form_submit()`.

.. php:function:: form_button([$data = ''[, $content = ''[, $extra = '']]])

	:param	string	$data: Button name
    	:param	string	$content: Button label
    	:param	mixed	$extra: Extra attributes to be added to the tag either as an array or a literal string
    	:returns:	An HTML button tag
    	:rtype:	string

    	Lets you generate a standard button element. You can minimally pass the
    	button name and content in the first and second parameter::

		echo form_button('name','content');
		// Would produce: <button name="name" type="button">Content</button>

	Or you can pass an associative array containing any data you wish your
	form to contain::

		$data = [
			'name'    => 'button',
			'id'      => 'button',
			'value'   => 'true',
			'type'    => 'reset',
			'content' => 'Reset'
		];

		echo form_button($data);
		// Would produce: <button name="button" id="button" value="true" type="reset">Reset</button>

	If you would like your form to contain some additional data, like
	JavaScript, you can pass it as a string in the third parameter::

		$js = 'onClick="some_function()"';
		echo form_button('mybutton', 'Click Me', $js);

.. php:function:: form_close([$extra = ''])

	:param	string	$extra: Anything to append after the closing tag, *as is*
	:returns:	An HTML form closing tag
	:rtype:	string

	Produces a closing </form> tag. The only advantage to using this
	function is it permits you to pass data to it which will be added below
	the tag. For example::

		$string = '</div></div>';
		echo form_close($string);
		// Would produce:  </form> </div></div>

.. php:function:: set_value($field[, $default = ''[, $html_escape = TRUE]])

	:param	string	$field: Field name
    	:param	string	$default: Default value
    	:param  bool	$html_escape: Whether to turn off HTML escaping of the value
    	:returns:	Field value
    	:rtype:	string

    	Permits you to set the value of an input form or textarea. You must
    	supply the field name via the first parameter of the function. The
    	second (optional) parameter allows you to set a default value for the
    	form. The third (optional) parameter allows you to turn off HTML escaping
    	of the value, in case you need to use this function in combination with
    	i.e. :php:func:`form_input()` and avoid double-escaping.

	Example::

		<input type="text" name="quantity" value="<?php echo set_value('quantity', '0'); ?>" size="50" />

	The above form will show "0" when loaded for the first time.

.. php:function:: set_select($field[, $value = ''[, $default = FALSE]])

	:param	string	$field: Field name
    	:param	string	$value: Value to check for
    	:param	string	$default: Whether the value is also a default one
    	:returns:	'selected' attribute or an empty string
    	:rtype:	string

    	If you use a <select> menu, this function permits you to display the
    	menu item that was selected.

    	The first parameter must contain the name of the select menu, the second
    	parameter must contain the value of each item, and the third (optional)
    	parameter lets you set an item as the default (use boolean TRUE/FALSE).

    	Example::

		<select name="myselect">
			<option value="one" <?php echo  set_select('myselect', 'one', TRUE); ?> >One</option>
			<option value="two" <?php echo  set_select('myselect', 'two'); ?> >Two</option>
			<option value="three" <?php echo  set_select('myselect', 'three'); ?> >Three</option>
		</select>

.. php:function:: set_checkbox($field[, $value = ''[, $default = FALSE]])

	:param	string	$field: Field name
    	:param	string	$value: Value to check for
    	:param	string	$default: Whether the value is also a default one
    	:returns:	'checked' attribute or an empty string
    	:rtype:	string

    	Permits you to display a checkbox in the state it was submitted.

    	The first parameter must contain the name of the checkbox, the second
    	parameter must contain its value, and the third (optional) parameter
    	lets you set an item as the default (use boolean TRUE/FALSE).

    	Example::

		<input type="checkbox" name="mycheck" value="1" <?php echo set_checkbox('mycheck', '1'); ?> />
		<input type="checkbox" name="mycheck" value="2" <?php echo set_checkbox('mycheck', '2'); ?> />

.. php:function:: set_radio($field[, $value = ''[, $default = FALSE]])

	:param	string	$field: Field name
    	:param	string	$value: Value to check for
    	:param	string	$default: Whether the value is also a default one
    	:returns:	'checked' attribute or an empty string
    	:rtype:	string

    	Permits you to display radio buttons in the state they were submitted.
    	This function is identical to the :php:func:`set_checkbox()` function above.

	Example::

		<input type="radio" name="myradio" value="1" <?php echo  set_radio('myradio', '1', TRUE); ?> />
		<input type="radio" name="myradio" value="2" <?php echo  set_radio('myradio', '2'); ?> />

	.. note:: If you are using the Form Validation class, you must always specify
		a rule for your field, even if empty, in order for the ``set_*()``
		functions to work. This is because if a Form Validation object is
		defined, the control for ``set_*()`` is handed over to a method of the
		class instead of the generic helper function.
