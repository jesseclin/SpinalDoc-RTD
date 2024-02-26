.. role:: raw-html-m2r(raw)
   :format: html

NullPointerException
====================

導入
------------

``NullPointerException`` は、変数が初期化される前にアクセスされたときに発生する Scala ランタイムのエラーです。

例
-------

次のコード：

.. code-block:: scala

   class TopLevel extends Component {
     a := 42
     val a = UInt(8 bits)
   }

次のエラーが発生します：

.. code-block:: text

   Exception in thread "main" java.lang.NullPointerException
     ***
     Source file location of the a := 42 assignment via the stack trace
     ***

修正方法は次のようになります：

.. code-block:: scala

   class TopLevel extends Component {
     val a = UInt(8 bits)
     a := 42
   }

問題の説明
^^^^^^^^^^^^^^^^^

SpinalHDL は言語ではなく、Scala ライブラリです。これは、一般的な目的の Scala プログラミング言語と同じルールに従います。

上記の SpinalHDL ハードウェア記述を実行して対応する VHDL/Verilog RTLを生成する際に、
SpinalHDL ハードウェア記述が Scala プログラムとして実行され、 ``a`` は ``val a = UInt(8 bits)`` が実行されるまで null 参照になります。
そのため、それ以前に代入しようとすると、 ``NullPointerException`` が発生します。
