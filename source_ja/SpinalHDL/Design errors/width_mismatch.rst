
Width mismatch
===============

概要
------------

SpinalHDL は、代入式において、左右のオペランド (演算対象) とシグナルが同じビット幅であることを確認します。
ビット幅とは、信号が表現できる値の大きさを示す単位です。

代入式の例
------------------

以下のコード:

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     val b = UInt(4 bits)
     b := a
   }

は、以下のようなエラーを出力します:

.. code-block:: text

   WIDTH MISMATCH on (toplevel/b :  UInt[4 bits]) := (toplevel/a :  UInt[8 bits]) at
     ***
     Source file location of the OR operator via the stack trace
     ***

修正方法:

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     val b = UInt(4 bits)
     b := a.resized
   }

演算子の例
----------------

以下のコード:

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     val b = UInt(4 bits)
     val result = a | b
   }

は、以下のようなエラーを出力します:

.. code-block:: text

   WIDTH MISMATCH on (UInt | UInt)[8 bits]
   - Left  operand : (toplevel/a :  UInt[8 bits])
   - Right operand : (toplevel/b :  UInt[4 bits])
     at
     ***
     Source file location of the OR operator via the stack trace
     ***

修正方法:

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     val b = UInt(4 bits)
     val result = a | (b.resized)
   }
