
Assignment overlap
==================

はじめに
------------

SpinalHDL は、前の割り当てを完全に消去する信号割り当てがないことを確認します。

例
-------

以下のコード

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     a := 42
     a := 66 // a := 42の割り当てを完全に上書き
   }

は、次のエラーを発生させます:

.. code-block:: text

   ASSIGNMENT OVERLAP completely the previous one of (toplevel/a :  UInt[8 bits])
     ***
     Source file location of the a := 66 assignment via the stack trace
     ***

修正方法は次の通りです:

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     a := 42
     when(something) {
       a := 66
     }
   }

ただし、前の割り当てを実際に上書きしたい場合（上書きが意味をなす場合があるため）、次のようにすることができます:

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     a := 42
     a.allowOverride
     a := 66
   }
