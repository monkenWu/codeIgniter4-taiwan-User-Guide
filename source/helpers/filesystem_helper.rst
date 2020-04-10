####################
Filesystem 輔助函式
####################

Filesystem 輔助函式檔案包含可以協助關於目錄工作的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

Loading this Helper
===================

此輔助函式可以利用以下的程式碼讀取：

::

	helper('filesystem');

可用的功能：
===================

以下敘述的功能皆可用：

.. php:function:: directory_map($source_dir[, $directory_depth = 0[, $hidden = FALSE]])

	:param	string  $source_dir: 到來源目錄的路徑
	:param	int	    $directory_depth: 需要遍歷的目錄深度（0 = 完全遞迴，1 = 當前路徑，諸如此類）
	:param	bool	$hidden: 是否包含隱藏目錄
	:returns:	一個陣列的資料
	:rtype:	陣列

	範例::

		$map = directory_map('./mydirectory/');

	.. note:: 路徑幾乎永遠都與你主要的 index.php 檔案相關。

	子資料夾也包含會被映射的檔案所包含的路徑。
	如果你希望能夠控制遞迴的深度，你可以利用第二個參數（integer）來達成。
	深度為一將只會映射出最頂端的路徑::

		$map = directory_map('./mydirectory/', 1);

	在預設情況下，隱藏的檔案將不會被包含在回傳的陣列中。
	如果要複寫此行為，你可以設定第三個參數為 true (boolean)::

		$map = directory_map('./mydirectory/', FALSE, TRUE);

	每一個資料夾名稱會因為它所包含的資料被數值化而成為一個陣列索引。
	這裡是一個典型陣列的範例::

		Array (
			[libraries] => Array
				(
					[0] => benchmark.html
					[1] => config.html
					["database/"] => Array
						(
							[0] => query_builder.html
							[1] => binds.html
							[2] => configuration.html
							[3] => connecting.html
							[4] => examples.html
							[5] => fields.html
							[6] => index.html
							[7] => queries.html
						)
					[2] => email.html
					[3] => file_uploading.html
					[4] => image_lib.html
					[5] => input.html
					[6] => language.html
					[7] => loader.html
					[8] => pagination.html
					[9] => uri.html
				)

	如果沒有找到結果，他將會回傳一個空陣列。

.. php:function:: write_file($path, $data[, $mode = 'wb'])

	:param	string	$path: 檔案路徑
	:param	string	$data: 欲寫入檔案的資料
	:param	string	$mode: ``fopen()`` 模式
	:returns:	TRUE ：若寫入成功， FALSE ：避免出現錯誤
	:rtype: bool

	會將資料寫入路徑中指定的檔案。若是檔案不存在則該功能會新建該檔案。

	範例::

		$data = 'Some file data';
		if ( ! write_file('./path/to/file.php', $data))
		{     
			echo 'Unable to write the file';
		}
		else
		{     
			echo 'File written!';
		}

	你可以依照需求透過第三個參數以設定寫檔模式::

		write_file('./path/to/file.php', $data, 'r+');

	預設的模式為「 wb 」。更多選項請見 `PHP 使用者導引 <https://www.php.net/manual/en/function.fopen.php>`_ 。

	.. note:: 為了讓這個功能對一個檔案寫入資料，他的授權必須被設定為可寫入。
		若是檔案不是已存在的檔案，那麼他所包含的路徑必須是可寫入的。

	.. note:: 路徑與你的主要網址的 index.php 相關， **並不是** 你的 controller 或 view 檔案。CodeIgniter 使用的是前端的 controller 所以路徑永遠會與主要的網站 index 相關。

	.. note:: 此函數在寫入檔案的時候會獲得一個獨特的檔案鎖。

.. php:function:: delete_files($path[, $del_dir = FALSE[, $htdocs = FALSE]])

	:param	string	$path: 目標路徑
	:param	bool	$del_dir: 是否同時刪除目錄
	:param	bool	$htdocs: 是否跳過刪除 .htaccess 以及 index 頁面檔案。
	:returns:	TRUE ：為成功，FALSE ：避免出現錯誤
	:rtype:	bool

	刪除提供路徑中包含的全部檔案。

	範例::

		delete_files('./path/to/directory/');

	如果第二的參數被設定為 TRUE，任何包含在提供根路徑中的任何目錄也會同時被刪除。

	範例::

		delete_files('./path/to/directory/', TRUE);

	.. note:: 為了能夠被刪除，該檔案必須為可寫入或是被系統擁有

.. php:function:: get_filenames($source_dir[, $include_path = FALSE])

	:param	string	$source_dir: 目標路徑
	:param	bool	$include_path: 是否引入路徑為部分的檔案名稱
	:returns:	一個陣列的檔案名稱
	:rtype:	array

	以伺服器路徑作為輸入並回傳一個包含所有檔案名稱在內的陣列。透過設定第二個參數為 TRUE，這些檔案路徑可以依照需要加進檔案名稱

	範例::

		$controllers = get_filenames(APPPATH.'controllers/');

.. php:function:: get_dir_file_info($source_dir, $top_level_only)

	:param	string	$source_dir: 目標路徑
	:param	bool	$top_level_only: 是否僅鎖定特定路徑（不包含子目錄）
	:returns:	一個包含提供路徑中內容資訊的陣列
	:rtype:	array

	讀取特定的路徑並建立一個包含檔案名、檔案大小、日期、以及授權的陣列。
	因為這可以是一個刻意的操作，如果您強迫將第二個參數設定為 FALSE，被包在特定路徑的子資料夾都僅為唯讀。

	範例::

		$models_info = get_dir_file_info(APPPATH.'models/');

.. php:function:: get_file_info($file[, $returned_values = ['name', 'server_path', 'size', 'date']])

	:param	string	        $file: 檔案路徑
	:param	array|string    $returned_values: 由陣列或是由逗號隔開的字串來決定回傳何種種類的資訊
	:returns:	一個包含特定檔案資訊的陣列或是當失敗的時候回傳 FALSE。
	:rtype:	array

	給定一個檔案與路徑，回傳（由你決定）該資料的 *名稱* ， *路徑* ， *檔案大小* 與 *修改日期* 等資訊
	第二個參數允許你明確的宣告你想要回傳甚麼資訊。

	有效的 ``$returned_values`` 選項為： `名稱`, `檔案大小`, `日期`, `可讀取`, `可寫入`,
	`可執行` 以及 `檔案權限`.

.. php:function:: symbolic_permissions($perms)

	:param	int	$perms: 權限
	:returns:	符號表式的授權字串
	:rtype:	string

	拿取數值化的授權（例如被　``fileperms()`` 回傳）並且回傳標準的標點符號的檔案權限

	::

		echo symbolic_permissions(fileperms('./index.php'));  // -rw-r--r--

.. php:function:: octal_permissions($perms)

	:param	int	$perms: 權限
	:returns:	八進位的授權字串
	:rtype:	string

	獲取數值化的授權（例如被　``fileperms()`` 回傳）並且回傳三位數的檔案授權八進位符號

	::

		echo octal_permissions(fileperms('./index.php')); // 644

.. php:function:: set_realpath($path[, $check_existence = FALSE])

	:param	string	$path: 路徑
	:param	bool	$check_existence: 是否檢查路徑是否真的存在
	:returns:	一個絕對路徑
	:rtype:	string

	這個功能會回傳一個沒有符號的連結或相關的路徑結構的伺服器路徑
	如果路徑不能被分析，自選的第二個參數將會引發一個錯誤。

	範例::

		$file = '/etc/php5/apache2/php.ini';
		echo set_realpath($file); // 印出 '/etc/php5/apache2/php.ini'

		$non_existent_file = '/path/to/non-exist-file.txt';
		echo set_realpath($non_existent_file, TRUE);	// 因為路徑不能被解析，顯示一個錯誤
		echo set_realpath($non_existent_file, FALSE);	// 印出 '/path/to/non-exist-file.txt'

		$directory = '/etc/php5';
		echo set_realpath($directory);	// 印出 '/etc/php5/'

		$non_existent_directory = '/path/to/nowhere';
		echo set_realpath($non_existent_directory, TRUE);	//  因為路徑不能被解析，顯示一個錯誤
		echo set_realpath($non_existent_directory, FALSE);	// 印出 '/path/to/nowhere'
