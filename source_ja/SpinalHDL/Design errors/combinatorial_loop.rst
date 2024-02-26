
Combinatorial loop
==================

はじめに
------------

SpinalHDL は、設計内に組み合わせループがないかどうかを確認します。

例
-------

次のコード：

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits) // PlayDev.scala line 831
     val b = UInt(8 bits) // PlayDev.scala line 832
     val c = UInt(8 bits)
     val d = UInt(8 bits)

     a := b
     b := c | d
     d := a
     c := 0
   }

は次のエラーを発生します：

.. code-block:: text

   COMBINATORIAL LOOP :
     Partial chain :
       >>> (toplevel/a :  UInt[8 bits]) at ***(PlayDev.scala:831) >>>
       >>> (toplevel/d :  UInt[8 bits]) at ***(PlayDev.scala:834) >>>
       >>> (toplevel/b :  UInt[8 bits]) at ***(PlayDev.scala:832) >>>
       >>> (toplevel/a :  UInt[8 bits]) at ***(PlayDev.scala:831) >>>

     Full chain :
       (toplevel/a :  UInt[8 bits])
       (toplevel/d :  UInt[8 bits])
       (UInt | UInt)[8 bits]
       (toplevel/b :  UInt[8 bits])
       (toplevel/a :  UInt[8 bits])

修正方法の一例は次のとおりです：

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits) // PlayDev.scala line 831
     val b = UInt(8 bits) // PlayDev.scala line 832
     val c = UInt(8 bits)
     val d = UInt(8 bits)

     a := b
     b := c | d
     d := 42
     c := 0
   }

誤検知
---------------

SpinalHDL の組み合わせループを検出するアルゴリズムは悲観的であり、誤検知を与える場合があります。
誤検知が発生している場合は、ループの1つの信号でループのチェックを手動で無効にできます。次のように：

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     a := 0
     a(1) := a(0) // この行が原因で誤検知が発生している
   }

次のように修正できます：

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits).noCombLoopCheck
     a := 0
     a(1) := a(0)
   }

また、 ``(a(1) := a(0))`` のような代入は、 `Verilator <https://www.veripool.org/wiki/verilator>`_ などの一部のツールを不快にする可能性があります。
この場合は ``Vec(Bool(), 8)`` を使用した方が良いかもしれません。

