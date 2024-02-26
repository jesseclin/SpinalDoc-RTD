
Unassigned register
===================

概要
------------

SpinalHDL は、設計に影響を与えるすべてのレジスタがどこかで割り当てられていることをチェックします。

例
-------
以下のコード:


.. code-block:: scala

   class TopLevel extends Component {
     val result = out(UInt(8 bits))
     val a = Reg(UInt(8 bits))
     result := a
   }

は、以下のようなエラーを出力します:

.. code-block:: text

   UNASSIGNED REGISTER (toplevel/a :  UInt[8 bits]), defined at
     ***
     Source file location of the toplevel/a definition via the stack trace
     ***

修正方法:

.. code-block:: scala

   class TopLevel extends Component {
     val result = out(UInt(8 bits))
     val a = Reg(UInt(8 bits))
     a := 42
     result := a
   }

初期値のみを持つレジスタ
--------------------------

設計のパラメータ化によっては、代入ではなく初期値 ``init`` ステートメントのみを持つレジスタを生成することが理にかなう場合もあります。

以下のコード:

.. code-block:: scala

   class TopLevel extends Component {
     val result = out(UInt(8 bits))
     val a = Reg(UInt(8 bits)) init(42)

     if(something)
       a := somethingElse
     result := a
   }

は、以下のようなエラーを出力します:

.. code-block:: text

   UNASSIGNED REGISTER (toplevel/a :  UInt[8 bits]), defined at
     ***
     Source file location of the toplevel/a definition via the stack trace
     ***

修正方法:

SpinalHDL に対して、代入がなく ``init`` ステートメントのみのレジスタを組み合わせ回路に変換するように指示できます。

.. code-block:: scala

   class TopLevel extends Component {
     val result = out(UInt(8 bits))
     val a = Reg(UInt(8 bits)).init(42).allowUnsetRegToAvoidLatch

     if(something)
       a := somethingElse
     result := a
   }
