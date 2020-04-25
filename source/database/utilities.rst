########################
資料庫工具
########################

資料庫工具類別包含一系列幫助你管理資料庫的方法。

.. contents::
    :local:
    :depth: 2

*******************
從結果中得到XML格式
*******************

**getXMLFromResult()**

這個方法會將查詢結果以xml格式回傳，你可以參考以下：

::

    $model = new class extends \CodeIgniter\Model {
        protected $table      = 'foo';
        protected $primaryKey = 'id';
    };
    $db = \Closure::bind(function ($model) {
        return $model->db;
    }, null, $model)($model);

    $util = (new \CodeIgniter\Database\Database())->loadUtils($db);
    echo $util->getXMLFromResult($model->get());

它將會得到以下的xml結果：


::

    <root>
        <element>
            <id>1</id>
            <name>bar</name>
        </element>
    </root>
