.. _Reg:

レジスタ
=========

SpinalHDL でのレジスタの作成は、VHDL や Verilogとは非常に異なります。

Spinal では、プロセス/alwaysブロックはありません。レジスタは宣言時に明示的に定義されます。
これは、従来のイベント駆動型HDLとの違いが大きな影響を与えます：

* レジスタとワイヤを同じスコープで割り当てることができるため、コードをプロセス/alwaysブロックに分割する必要がありません
* これにより、柔軟性が大幅に向上します（:ref:`関数 <function>` を参照）

クロックとリセットは別々に処理されます。詳細については、:ref:`クロックドメイン <clock_domain>` の章を参照してください。

インスタンス化
----------------

レジスタをインスタンス化する方法は4つあります：


.. list-table::
   :header-rows: 1
   :widths: 50 55

   * - 構文
     - 説明
   * - ``Reg(type : Data)``
     - 指定された型のレジスタ
   * - ``RegInit(resetValue : Data)``
     - リセットが発生したときに指定された ``resetValue`` で読み込まれるレジスタ
   * - ``RegNext(nextValue : Data)``
     - 指定された ``nextValue`` を各サイクルでサンプリングするレジスタ
   * - ``RegNextWhen(nextValue : Data, cond : Bool)``
     - 条件が発生したときに指定された ``nextValue`` をサンプリングするレジスタ

以下は、いくつかのレジスタを宣言する例です：

.. code-block:: scala

   // 4 ビットの UInt レジスタ
   val reg1 = Reg(UInt(4 bits))

   // reg1 を 1 つ増やしたものをサンプリングして、毎サイクル更新されるレジスタ
   val reg2 = RegNext(reg1 + 1)

   // リセットが発生したときに 0 で初期化される 4 ビットの UInt レジスタ
   val reg3 = RegInit(U"0000")
   reg3 := reg2
   when(reg2 === 5) {
     reg3 := 0xF
   }

   // cond が True のときに reg3 をサンプリングするレジスタ
   val reg4 = RegNextWhen(reg3, cond)

上記のコードは、次のロジックを推論します：

.. image:: /asset/picture/register.svg
   :align: center

.. note::
   上記の ``reg3`` の例は、 ``RegInit`` レジスタの値を割り当てる方法を示しています。
   他のレジスタタイプ（ ``Reg``、 ``RegNext``、 ``RegNextWhen``）にも同じ構文を使用することができます。
   組み合わせ的な割り当てと同様に、「最後の割り当てが優先されます」が、割り当てが行われない場合、レジスタはその値を保持します。
   Reg が設計で宣言されており、適切な割り当てと消費が行われていない場合、EDA フローによって不要と見なされ、いつか設計から削除される可能性があります。


.. _RegNext:

また、 ``RegNext`` は、 ``Reg`` 構文の上に構築された抽象化です。次の2つのコードシーケンスは厳密に等価です:

.. code-block:: scala

   // 標準的な方法
   val something = Bool()
   val value = Reg(Bool())
   value := something

   // 短縮形
   val something = Bool()
   val value = RegNext(something)


次のような他の方法でも、同時に複数のオプションを持つことができます。
したがって、上記の基本的な理解をベースにしたやや高度な構成が構築されます：

.. code-block:: scala

   // リセット時に 42 で初期化された 6 ビットの UInt レジスタ
   val reg1 = Reg(UInt(6 bits)) init(42)

   // 各サイクルで reg1 をサンプリングするレジスタ（リセット時に 0 で初期化されます）
   // Scala の名前付きパラメータ引数形式を使用
   val reg2 = RegNext(reg1, init=0)

   // 複数の機能を組み合わせたレジスタ

   // 自身のレジスタ有効信号
   val reg3Enable = Bool()
   // 6 ビットの UInt レジスタ（reg1 の型から推論されます）
   //   reg1 からの更新を事前に設定された割り当てで
   //   reg3Enable が設定されている場合のみ更新されます
   //   リセット時に 99 で初期化されます
   val reg3 = RegNextWhen(reg1, reg3Enable, U(99))
   // when(reg3Enable) {
   //   reg3 := reg1; // この式はコンストラクタの使用ケースで暗黙的に意味されています
   // }

   when(cond2) {      // これは有効な割り当てで、実行時に優先されます
      reg3 := U(0)    //  (最後の割り当てが勝つルールによる)、割り当てには
   }                  //  reg3Enable 条件が必要ありません、それには `when(cond2 & reg3Enable)` を使用します

   // リセット時に 99 で初期化された 8 ビットの UInt レジスタ
   val reg4 = Reg(UInt(8 bits), U(99))
   // 自身のレジスタ有効信号
   val reg4Enable = Bool()
   // 暗黙の割り当ては存在せず、必要に応じて明示的に有効にする必要があります
   when(reg4Enable) {
      reg4 := newValue
   }

リセット値
----------------

``RegInit(value: Data)`` 構文に加えて、リセット値を直接指定してレジスタを作成する方法として、
レジスタに ``init(value: Data)`` 関数を呼び出してリセット値を設定することもできます。

.. code-block:: scala

   // リセット時に 0 で初期化された 4 ビットの UInt レジスタ
   val reg1 = Reg(UInt(4 bits)) init(0)

Bundle を含むレジスタがある場合、Bundle の各要素に ``init`` 関数を使用できます。

.. code-block:: scala

   case class ValidRGB() extends Bundle{
     val valid   = Bool()
     val r, g, b = UInt(8 bits)
   }

   val reg = Reg(ValidRGB())
   reg.valid init(False)  // そのレジスタのバンドルがリセット値を持つ場合は、valid のみがリセットされます。

シミュレーション目的の初期化値
---------------------------------

RTL でリセット値が必要ないが、シミュレーションでの初期化値が必要なレジスタ（X-伝播を避けるため）については、 
``randBoot()`` 関数を呼び出してランダムな初期化値を要求できます。

.. code-block:: scala

   // ランダムな値で初期化された 4 ビットの UInt レジスタ
   val reg1 = Reg(UInt(4 bits)) randBoot()


レジスタベクトル
-----------------

ワイヤーと同様に、 ``Vec`` を使用してレジスタのベクトルを定義することができます。

.. code-block:: scala
   
   val vecReg1 = Vec(Reg(UInt(8 bits)), 4)
   val vecReg2 = Vec.fill(8)(Reg(Bool()))

通常通り、初期化は ``init`` メソッドで行うことができます。これは、レジスタの ``foreach`` イテレーションと組み合わせることができます。

.. code-block:: scala

   val vecReg1 = Vec(Reg(UInt(8 bits)) init(0), 4)
   val vecReg2 = Vec.fill(8)(Reg(Bool()))
   vecReg2.foreach(_ init(False))

初期化の値がわからない場合に初期化を遅延させる必要がある場合は、以下の例のように関数を使用します。

.. code-block:: scala

   case class ShiftRegister[T <: Data](dataType: HardType[T], depth: Int, initFunc: T => Unit) extends Component {
      val io = new Bundle {
         val input  = in (dataType())
         val output = out(dataType())
      }

      val regs = Vec.fill(depth)(Reg(dataType()))
      regs.foreach(initFunc)

      for (i <- 1 to (depth-1)) {
            regs(i) := regs(i-1)
      }

      regs(0) := io.input
      io.output := regs(depth-1)
   }

   object SRConsumer {
      def initIdleFlow[T <: Data](flow: Flow[T]): Unit = {
         flow.valid init(False)
      }
   }

   class SRConsumer() extends Component {
      //...
      val sr = ShiftRegister(Flow(UInt(8 bits)), 4, SRConsumer.initIdleFlow[UInt])
   }

ワイヤをレジスタに変換する
-----------------------------------

既存のワイヤをレジスタに変換することが便利な場合があります。
例えば、Bundleを使用していて、Bundleの出力の一部をレジスタにしたければ、
``val PORT = Reg(...)`` でレジスタを宣言し、
その出力をポートに ``io.myBundle.PORT := PORT``で接続するのではなく、
``io.myBundle.PORT := newValue`` と書くほうが簡潔です。
そのためには、レジスタとして制御したいポートに ``.setAsReg()`` を使用するだけです:

.. code-block:: scala

   val io = new Bundle {
      val apb = master(Apb3(apb3Config))
   }

   io.apb.PADDR.setAsReg()
   io.apb.PWRITE.setAsReg() init(False)

   when(someCondition) {
      io.apb.PWRITE := True
   }

上記のコードで初期化値を指定することもできることに注意してください。

.. note::

   レジスタはワイヤのクロックドメインで作成され、 ``.setAsReg()`` が使用される場所に依存しません。

   上記の例では、ワイヤがコンポーネントと同じクロックドメインの ``io`` バンドルで定義されています。
   たとえ ``io.apb.PADDR.setAsReg()`` が異なるクロックドメインを持つ ``ClockingArea`` 内に書かれていても、
   レジスタはコンポーネントのクロックドメインを使用し、 ``ClockingArea`` のものではありません。

