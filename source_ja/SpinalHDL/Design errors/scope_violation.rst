
Scope violation
===============

概要
------------

SpinalHDL は、定義されたスコープの外に信号が割り当てられていないことを確認します。
このエラーは、特定のメタハードウェア記述トリックが必要なため、簡単にトリガーされるものではありません。

例
-------

次のコード：

.. code-block:: scala

   class TopLevel extends Component {
     val cond = Bool()

     var tmp : UInt = null
     when(cond) {
       tmp = UInt(8 bits)
     }
     tmp := U"x42"
   }

次のエラーが発生します：

.. code-block:: text

   SCOPE VIOLATION : (toplevel/tmp :  UInt[8 bits]) is assigned outside its declaration scope at
     ***
     Source file location of the tmp := U"x42" via the stack trace
     ***

修正方法は以下の通りです：

.. code-block:: scala

   class TopLevel extends Component {
     val cond = Bool()

     var tmp : UInt = UInt(8 bits)
     when(cond) {

     }
     tmp := U"x42"
   }
