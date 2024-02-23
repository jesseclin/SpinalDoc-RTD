.. _Component:

Components and hierarchy
========================

VHDLやVerilogと同様に、デザイン階層を構築するために使用できるコンポーネントを定義することができます。
ただし、SpinalHDLでは、インスタンス化時にポートをバインドする必要はありません:

.. code-block:: scala

   class AdderCell() extends Component {
     // `io` という名前の Bundle で外部ポートを宣言することが推奨されています。
     val io = new Bundle {
       val a, b, cin = in port Bool()
       val sum, cout = out port Bool()
     }
     // いくつかのロジックを行う
     io.sum := io.a ^ io.b ^ io.cin
     io.cout := (io.a & io.b) | (io.a & io.cin) | (io.b & io.cin)
   }

   class Adder(width: Int) extends Component {
     ...
     // AdderCell インスタンスを 2 つ作成します。
     val cell0 = new AdderCell()
     val cell1 = new AdderCell()
     cell1.io.cin := cell0.io.cout   // cell0 の cout を cell1 の cin に接続します。

     // ArrayCell インスタンスの配列を作成する別の例
     val cellArray = Array.fill(width)(new AdderCell())
     cellArray(1).io.cin := cellArray(0).io.cout   // cell(0) の cout を cell(1) の cin に接続します。
     ...
   }

.. tip::
   | ``val io = new Bundle { ... }``
   | 外部ポートを ``io`` という ``Bundle`` で宣言することが推奨されています。
     ``io`` という名前のバンドルを使用すると、SpinalHDL がそのすべての要素が入力または出力として定義されているかどうかをチェックします。
   
.. tip::
   好みに合わせて、 ``Component`` の代わりに ``Module`` 構文を使用することもできます（これらは同じものです）。

.. _io:

入力 / 出力の定義
-------------------------

入力と出力を定義する構文は次のとおりです：

.. list-table::
   :header-rows: 1
   :widths: 2 3 1

   * - 構文
     - 説明
     - 戻り値
   * - | ``in port Bool()``
       | ``out port Bool()``
     - 入力 Bool / 出力 Bool を作成します。
     - Bool
   * - | ``in Bits/UInt/SInt[(x bits)]``
       | ``out Bits/UInt/SInt[(x bits)]``
       | ``in Bits(3 bits)``
     - 対応する型の入力/出力を作成します。
     - Bits/UInt/SInt
   * - | ``in(T)``
       | ``out(T)``
       | ``out UInt(7 bits)``
     - 他のすべてのデータ型については、それを括弧で囲む必要があるかもしれません。申し訳ありませんが、これはScalaの制限です。
     - T
   * - | ``master(T)``
       | ``slave(T)``
       | ``master(Bool())``
     - この構文は ``spinal.lib`` ライブラリによって提供されています（オブジェクトに ``slave`` 構文の注釈を付ける場合は、代わりに ``spinal.lib.slave`` をインポートしてください）。T は ``IMasterSlave`` を拡張する必要があります。いくつかのドキュメントは `こちら <interface_example_apb>` で利用可能です。実際には括弧は必要ないかもしれませんので、 ``master T`` でも問題ありません。
     - T

コンポーネント間の相互接続に関する遵守すべきルールがいくつかあります：

- コンポーネントは、子コンポーネントの出力および入力信号を **読み取る** ことしかできません。
- コンポーネントは、自分自身の出力ポートの値を読み取ることができます（VHDLとは異なります）。

.. tip::
   何らかの理由で階層構造の遠くから信号を読み取る必要がある場合（デバッグや一時的なパッチなど）、
   ``some.where.else.theSignal.pull()`` で返される値を使用することができます。

削除された信号
--------------

SpinalHDL は、すべての名前付き信号とそれらの依存関係を生成しますが、無駄な匿名 / ゼロ幅の信号は RTL 生成から削除されます。

生成された ``SpinalReport`` オブジェクトの ``printPruned`` および ``printPrunedIo`` 関数を使用して、削除された無駄な信号のリストを収集できます。

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a,b = in port UInt(8 bits)
       val result = out port UInt(8 bits)
     }

     io.result := io.a + io.b

     val unusedSignal = UInt(8 bits)
     val unusedSignal2 = UInt(8 bits)

     unusedSignal2 := unusedSignal
   }

   object Main {
     def main(args: Array[String]) {
       SpinalVhdl(new TopLevel).printPruned()
       //これにより、次のようなレポートが生成されます：
       //  [Warning] Unused wire detected : toplevel/unusedSignal : UInt[8 bits]
       //  [Warning] Unused wire detected : toplevel/unusedSignal2 : UInt[8 bits]
     }
   }


パラメータ化されたハードウェア（VHDLでは "Generic"、Verilogでは "Parameter" と呼ばれるもの）
----------------------------------------------------------------------------------------------

コンポーネントをパラメータ化する場合は、以下のようにコンポーネントのコンストラクタにパラメータを指定することができます：

.. code-block:: scala

   class MyAdder(width: BitCount) extends Component {
     val io = new Bundle {
       val a, b   = in port UInt(width)
       val result = out port UInt(width)
     }
     io.result := io.a + io.b
   }

   object Main {
     def main(args: Array[String]) {
       SpinalVhdl(new MyAdder(32 bits))
     }
   }

複数のパラメータがある場合は、以下のように特定の設定クラスを指定するのが良い習慣です：

.. code-block:: scala

   case class MySocConfig(axiFrequency  : HertzNumber,
                          onChipRamSize : BigInt,
                          cpu           : RiscCoreConfig,
                          iCache        : InstructionCacheConfig)

   class MySoc(config: MySocConfig) extends Component {
     ...
   }

設定内に機能を追加することができます。設定属性に対する要件も設定できます。

.. code-block:: scala

   case class MyBusConfig(addressWidth: Int, dataWidth: Int) {
     def bytePerWord = dataWidth / 8
     def addressType = UInt(addressWidth bits)
     def dataType = Bits(dataWidth bits)

     require(dataWidth == 32 || dataWidth == 64, "Data width must be 32 or 64")
   }

.. note::

   このパラメータ化は完全に SpinalHDL のコード生成中に行われます。これにより、非ジェネリックな HDL コードが生成されます。
   ここで説明する方法は、VHDL のジェネリックや Verilog のパラメータを使用しません。

   また、そのメカニズムのサポートに関する詳細については、:ref:`Blackbox <blackbox>` を参照してください。

合成されたコンポーネント名
---------------------------

モジュール内では、各コンポーネントには「部分名」と呼ばれる名前があります。
「完全」な名前は、各コンポーネントの親名を "_" で結合して構築されます。例： ``io_clockDomain_reset``。
この規則をカスタム名で置き換えるには ``setName`` を使用できます。
これは特に外部コンポーネントとのインターフェースで便利です。
他のメソッドはそれぞれ ``getName``、 ``setPartialName``、``getPartialName`` と呼ばれます。

合成されると、各モジュールはそれを定義する Scala クラスの名前を取得します。
これも ``setDefinitionName`` でオーバーライドできます。

.. raw:: html

   <!--
   TODO
   ### Input or Output is a basic type

   ### Input or Output is a bundle type

   ## Master/Slave interface

   -->

