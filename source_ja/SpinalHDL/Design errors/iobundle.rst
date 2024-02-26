
IO bundle
=========

導入
------------

SpinalHDL は、各 ``io`` バンドルが入出力/入力/出力信号のみを含んでいることを確認します。

その他の種類の信号は、無方向信号と呼ばれます。

例
-------

次のコード：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a = UInt(8 bits)
     }
   }

は次のエラーを表示します：

.. code-block:: text

   IO BUNDLE ERROR : A direction less (toplevel/io_a :  UInt[8 bits]) signal was defined into toplevel component's io bundle
     ***
     Source file location of the toplevel/io_a definition via the stack trace
     ***

修正方法は次のとおりです：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a = in UInt(8 bits)  // 'in' 方向の宣言を提供します
     }
   }

しかし、メタハードウェア記述の理由で ``io.a`` が方向を持たない場合は、次のようにします：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a = UInt(8 bits)
     }
     a.allowDirectionLessIo
   }
