
No driver on
============

導入
------------

SpinalHDL は、デザインに影響を与えるすべての組み合わせ信号が何かによって割り当てられていることを確認します。

例
-------

次のコード：

.. code-block:: scala

   class TopLevel extends Component {
     val result = out(UInt(8 bits))
     val a = UInt(8 bits)
     result := a
   }

次のエラーを発生させます：

.. code-block:: text

   NO DRIVER ON (toplevel/a :  UInt[8 bits]), defined at
     ***
     Source file location of the toplevel/a definition via the stack trace
     ***

修正方法は次のようになります：

.. code-block:: scala

   class TopLevel extends Component {
     val result = out(UInt(8 bits))
     val a = UInt(8 bits)
     a := 42
     result := a
   }
