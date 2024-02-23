
Coding conventions
==================

導入
------------

SpinalHDL で使用されるコーディング規約は、 `Scala スタイル ガイド <https://docs.scala-lang.org/style/>`_ に記載されているものと同じです。

いくつかの追加の実際的な詳細とケースについては、次のページで説明します。


クラスと case クラス
-------------------

``Bundle`` または ``Component`` を定義するときは、それを case クラスとして宣言することをお勧めします。

理由は次のとおりです:

* ``new`` キーワードの使用を避けます。状況によっては、時々使用するよりも、まったく使用しない方が良いでしょう。
* ``case class`` は ``clone`` 関数を提供します。これは SpinalHDL で、たとえば、新しい ``Reg`` や何らかの種類の新しい ``Stream`` を定義するときなど、 ``Bundle`` のクローンを作成する必要がある場合に便利です。
* コンストラクタパラメータは外部から直接確認できます。

[case] クラス
^^^^^^^^^^^^^^^

すべてのクラス名は大文字で始める必要があります

.. code-block:: scala

   class Fifo extends Component {

   }

   class Counter extends Area {

   }

   case class Color extends Bundle {

   }

コンパニオンオブジェクト
^^^^^^^^^^^^^^^^^^^^^^^^^^

`コンパニオン オブジェクト <https://docs.scala-lang.org/overviews/scala-book/companion-objects.html>`_ は大文字で始める必要があります。

.. code-block:: scala

   object Fifo {
     def apply(that: Stream[Bits]): Stream[Bits] = {...}
   }

   object MajorityVote {
     def apply(that: Bits): UInt = {...}
   }

このルールの例外は、コンパニオン オブジェクトが関数として使用され (内部でのみ ``apply``)、
これらの ``apply`` 関数がハードウェアを生成しない場合です。

.. code-block:: scala

   object log2 {
     def apply(value: Int): Int = {...}
   }

関数
^^^^^^^^

関数は常に小文字で始める必要があります。

.. code-block:: scala

   def sinTable = (0 until sampleCount).map(sampleIndex => {
     val sinValue = Math.sin(2 * Math.PI * sampleIndex / sampleCount)
     S((sinValue * ((1 << resolutionWidth) / 2 - 1)).toInt, resolutionWidth bits)
   })

   val rom =  Mem(SInt(resolutionWidth bits), initialContent = sinTable)

インスタンス
^^^^^^^^^^^^^^^

クラスのインスタンスは常に小文字で始める必要があります。

.. code-block:: scala

   val fifo   = new Fifo()
   val buffer = Reg(Bits(8 bits))

if / when
^^^^^^^^^

Scala ``if`` と SpinalHDL ``when`` は通常、次のように記述する必要があります。

.. code-block:: scala

   if(cond) {
     ...
   } else if(cond) {
     ...
   } else {
     ...
   }

   when(cond) {
     ...
   } elsewhen(cond) {
     ...
   } otherwise {
     ...
   }

例外としては次のようなものがあります:

* メソッド ``.elsewhen`` や ``.otherwise`` のように、キーワードの前にドットを含めても問題ありません。
* コードが読みやすくなる場合は、 ``if``\ /\ ``when`` ステートメントを 1 行に圧縮しても問題ありません。

switch
^^^^^^

SpinalHDL の ``switch`` は通常、次のように記述する必要があります:

.. code-block:: scala

   switch(value) {
     is(key) {

     }
     is(key) {

     }
     default {

     }
   }

コードが読みやすくなる場合は、 ``is``\ /\ ``default`` ステートメントを 1 行に圧縮しても問題ありません。


パラメーター
^^^^^^^^^^^^^^

ケースクラス内の ``Component``/ ``Bundle`` のパラメータをグループ化することは、
一般に次の理由から歓迎されます。

* デザインを構成するための持ち運び/操作が簡単になりました
* メンテナンス性の向上

.. code-block:: scala

   case class RgbConfig(rWidth: Int, gWidth: Int, bWidth: Int) {
     def getWidth = rWidth + gWidth + bWidth
   }

   case class Rgb(c: RgbConfig) extends Bundle {
     val r = UInt(c.rWidth bits)
     val g = UInt(c.gWidth bits)
     val b = UInt(c.bWidth bits)
   }

ただし、これをすべての場合に適用する必要はありません。
例: FIFO では、一般に ``dataType`` はデザインに関連するものであるため、
``dataType`` パラメータを FIFO の ``Depth`` パラメータとグループ化することは意味がありません。 、
``Depth`` はデザインの構成に関連するものです。

But this should not be applied in all cases. For example: in a FIFO, it doesn't make sense to group the ``dataType`` parameter with the ``depth`` parameter of the fifo because, in general, the ``dataType`` is something related to the design, while the ``depth`` is something related to the configuration of the design.

.. code-block:: scala

   class Fifo[T <: Data](dataType: T, depth: Int) extends Component {

   }
