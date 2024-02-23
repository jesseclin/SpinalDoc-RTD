.. _Simple example:

A simple example
================

以下は `getting started <https://github.com/SpinalHDL/SpinalTemplateSbt>`_リポジトリからの簡単なハードウェア説明です。


.. code-block:: scala

   case class MyTopLevel() extends Component {
     val io = new Bundle {
       val cond0 = in port Bool()
       val cond1 = in port Bool()
       val flag  = out port Bool()
       val state = out port UInt(8 bits)
     }

     val counter = Reg(UInt(8 bits)) init 0

     when(io.cond0) {
       counter := counter + 1
     }

     io.state := counter
     io.flag  := (counter === 0) | io.cond1
   }


このセクションでは、いくつかの部分に分けて説明します。


コンポーネント
-------------

ここでは、SpinalHDL の基本となる ``Component`` について説明します。

Component は、必要な回数だけインスタンス化 (貼り付け) できる論理ブロックです。
このブロックから外部にアクセスできるのは、入力信号と出力信号のみです。


.. code-block:: scala

   case class MyTopLevel() extends Component {
     val io = new Bundle {
       // ポート定義はここにあります
     }

     // コンポーネントロジックがここに入ります
   }


このコードでは、 ``MyTopLevel``  という名前のコンポーネントを定義しています。S

SpinalHDL では、コンポーネント名は ``UpperCamelCase`` (最初の文字と単語の最初の文字は大文字) で書くのが慣例です。

.. note::

   詳細については、:ref:`Component` も参照してください。
   

ポート
------

次に、ポートが定義されます

.. code-block:: scala

       val cond0 = in port Bool()
       val cond1 = in port Bool()
       val flag = out port Bool()
       val state = out port UInt(8 bits)

方向:

* ``cond0`` と ``cond1`` は入力ポートです
* ``flag`` と ``state`` は出力ポートです

タイプ:

* ``cond0``、 ``cond1``、および ``flag`` はそれぞれ 1 ビットです (3 つの個別のワイヤとして)
* ``state`` は 8 ビットの符号なし整数です (8 つのワイヤからなるバスは、符号なし整数)

.. note::

   この構文は SpinalHDL 1.8 以降でのみ利用可能です。レガシーについては :ref:`io` を参照してください。
   構文と詳細情報。
   

内部ロジック
--------------

最後に、コンポーネント ロジックがあります:

.. code-block:: scala

     val counter = Reg(UInt(8 bits)) init(0)

     when(io.cond0) {
       counter := counter + 1
     }

     io.state := counter
     io.flag := (counter === 0) | io.cond1


このコードでは、 ``counter``  という名前のレジスタを定義しています。このレジスタは以下のような特徴を持ちます。

* 8 ビットの符号なし整数を格納します。
* 初期値は 0 です。
* レジスタの状態を変更する代入は、次のクロックの立ち上がりの後でのみ読み取り可能になります。


.. note::

   このコードではレジスタを使用しているため、クロック信号とリセット信号という 2つの暗黙の信号がコンポーネントに追加されます。
   これらの信号の詳細については、:ref:`Reg`` と :ref:`clock_domain` を参照してください。


次に、when 文による条件付きルールが記述されています。このルールは次のように動作します。

入力信号 ``cond0`` (これは ``io`` バンドル内に定義されています) が 1のとき、レジスタ ``counter`` の値は 1ずつ加算されます。
そうでない場合、 ``counter`` は最後に代入された値を保持します。
ここで、"最後の代入" とは、直前のルールによる代入ではなく、レジスタが本来持つ「前の値を保持する」という性質を指します。
単なる信号 (レジスタではないもの) で同様の記述をすると、ラッチが発生しエラーとなりますが、 ``counter`` はレジスタであるため、
明示的な代入がない場合でも前の値を保持するという既定の動作があるのです。

この条件付きルールは、マルチプレクサを生成します: ここでは、 ``io.cond0`` の値に応じて、
レジスタ counter の入力を以下のように選択します。

* ``io.cond0`` が1のとき: counter の現在の出力に1を加算したものが入力になる。
* ``io.cond0`` が0のとき: counter の現在の出力がそのまま入力になる。


次に、無条件のルール (代入文) が記述されています。

* 出力信号 ``state`` は、レジスタ ``counter`` の出力に直接接続されます。
* 出力信号 ``flag`` は、 ``counter`` の出力が 0のときだけ 1になる信号と、
  入力信号 ``cond1`` を ``or`` ゲードで接続した結果になります。

.. note::

   詳細については、:ref:`semantics` も参照してください。
