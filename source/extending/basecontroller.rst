************************
擴充控制器
************************

CodeIgniter 的核心控制器不應該被改變，但是在 **app/Controllers/BaseController.php** 中，我們為你提供了一個預設的控制器擴充類別。你所製作的任何控制器都應該繼承 ``BaseController`` 來利用預載組件以及你提供的任何額外功能：

::

	<?php namespace App\Controllers;
	
	use CodeIgniter\Controller;
	
	class Home extends BaseController {
	
	}

預載組件
=====================

基本控制器是一個好地方，可以在每次專案運作時載入任何輔助函數、模型、程式庫，以及服務等。輔助函數應該被加入到預先定義好的 ``$helpers`` 陣列中。例如：你想讓 HTML 以及文字輔助函數普遍可用：

::

	protected $helpers = ['html', 'text'];

任何需要載入的組件或需要處理的資料都應該加入到建構函數 ``initController()`` 中。例如：你的專案大量使用到會談程式庫，你可以會需要在這裡啟動它：

::

	public function initController(...)
	{
		// 請不要編輯這行
		parent::initController($request, $response, $logger);
		
		$this->session = \Config\Services::session();
	}

新增方法
==================

基本控制器是不允許被路由的（系統設定將其路由導向至 404 找不到頁面）。作為撰寫程式的安全實現，你所創建的 **所有** 新方法都應該宣告為 ``保護`` 或 ``私有`` ，並且只能透過繼承 BaseController 類別的控制器才能存取。

其他選項
=============

你可能會發現你需要一個或以上的基本控制器，你可以為你的需求創建基本控制器，只要你創建的其他控制器繼承了正確的基本控制器，就可以創建新的基本控制器。例如：你的專案有一個涉及到公開的介面和一個簡單的管理入口，你可能會希望將公開的介面繼承 ``BaseController`` ，並且將任何與管理功能相關的控制器繼承 ``AdminController`` 類別。

如果你不想使用基本控制器，你可以透過讓你的控制器繼承系統控制器來繞過它：

::

	class Home extends \CodeIgniter\Controller
	{
	
	}
