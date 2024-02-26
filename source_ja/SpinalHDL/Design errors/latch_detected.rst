
Latch detected
==============

導入
------------

SpinalHDL は、合成中に組み合わせ信号がラッチを推測しないようにチェックします。
つまり、これは組み合わせ信号が部分的に割り当てられないことを確認するチェックです。

例
-------

次のコード：

.. code-block:: scala

   class TopLevel extends Component {
     val cond = in(Bool())
     val a = UInt(8 bits)

     when(cond) {
       a := 42
     }
   }

次のエラーが発生します：

.. code-block:: text

   LATCH DETECTED from the combinatorial signal (toplevel/a :  UInt[8 bits]), defined at
     ***
     Source file location of the toplevel/io_a definition via the stack trace
     ***

修正方法：

.. code-block:: scala

   class TopLevel extends Component {
     val cond = in(Bool())
     val a = UInt(8 bits)

     a := 0
     when(cond) {
       a := 42
     }
   }

mux による原因
-------------------

ラッチが検出される別の理由は、しばしばデフォルトが欠落している非完全な ``mux``/``muxList`` ステートメントです：

.. code-block:: scala

  val u1 = UInt(1 bit)
  u1.mux(
    0 -> False,
    // 1のケースが不足しています
  )

これは、不足しているケース（またはデフォルトケース）を追加することで修正できます：

.. code-block:: scala

  val u1 = UInt(1 bit)
  u1.mux(
    0 -> False,
    default -> True
  )

たとえば、幅のジェネリックコードでは、デフォルトが必要ない場合でも、 ``muxListDc`` を使用する方が良い解決策です：

.. code-block:: scala

  val u1 = UInt(1 bit)
  // 必要に応じてデフォルトを自動的に追加します
  u1.muxListDc(Seq(0 -> True))
