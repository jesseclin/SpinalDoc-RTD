.. _clock_domain:

Clock domains
=============

紹介
------------

SpinalHDLでは、クロックとリセット信号を組み合わせて **クロックドメイン** を作成できます。クロックドメインは設計の特定の領域に適用でき、その領域にインスタンス化されたすべての同期要素は、その後 **暗黙的に** このクロックドメインを使用します。

クロックドメインの適用はスタックのように機能し、つまり、特定のクロックドメインにいる場合でも、別のクロックドメインをローカルに適用できます。

クロックドメインが割り当てられるタイミングは、レジスタが作成されるときであり、レジスタが割り当てられるときではありません。したがって、必ず所望の ``ClockingArea`` 内でそれらを作成するようにしてください。

.. _clock_domain_instantiation:

インスタンス化
----------------

クロックドメインを定義する構文は以下の通りです（EBNF構文を使用）：

.. code-block:: scala

   ClockDomain(
     clock: Bool 
     [,reset: Bool]
     [,softReset: Bool]
     [,clockEnable: Bool]
     [,frequency: IClockDomainFrequency]
     [,config: ClockDomainConfig]
   )

この定義には5つのパラメータが必要です：

.. list-table::
   :header-rows: 1
   :widths: 1 10 1

   * - 引数
     - 説明
     - デフォルト
   * - ``clock``
     - ドメインを定義するクロック信号
     - 
   * - ``reset``
     - リセット信号。リセットが必要なレジスタが存在し、クロックドメインがリセットを提供しない場合、エラーメッセージが表示されます。
     - null
   * - ``softReset``
     - 追加の同期リセットを推論するリセット
     - null
   * - ``clockEnable``
     - この信号の目的は、各同期要素に手動で実装することなく、クロックドメイン全体のクロックを無効にすることです。
     - null
   * - ``frequency``
     - 指定したクロックドメインの周波数を指定し、後で設計内でそれを読み取ることができます。
       このパラメータは、周波数を制御するための PLL やその他のハードウェアを生成しません。
     - UnknownFrequency
   * - ``config``
     - 信号の極性とリセットの性質を指定します。
     - Current config


設計内の特定のクロックドメインを定義するための適用例は次の通りです：

.. code-block:: scala

   val coreClock = Bool()
   val coreReset = Bool()

   // 新しいクロックドメインを定義します。
   val coreClockDomain = ClockDomain(coreClock, coreReset)

   // デザインの特定の領域でこのドメインを使用します。
   val coreArea = new ClockingArea(coreClockDomain) {
     val coreClockedRegister = Reg(UInt(4 bits))
   }

`Area`が不要な場合、クロックドメインを直接適用することもできます。2つの構文があります：

.. code-block:: scala

   class Counters extends Component {
     val io = new Bundle {
       val enable = in Bool ()
       val freeCount, gatedCount, gatedCount2 = out UInt (4 bits)
     }
     val freeCounter = CounterFreeRun(16)
     io.freeCount := freeCounter.value
   
     // 実際の設計では、グリッチのない単一用途の CLKGATE プリミティブを使用することを検討してください。
     val gatedClk = ClockDomain.current.readClockWire && io.enable
     val gated = ClockDomain(gatedClk, ClockDomain.current.readResetWire)
   
     // ここでは、"gatedCounter" と "gatedCounter2" に "gated" クロックドメインが適用されています。
     val gatedCounter = gated(CounterFreeRun(16))
     io.gatedCount := gatedCounter.value
     val gatedCounter2 = gated on CounterFreeRun(16)
     io.gatedCount2 := gatedCounter2.value
   
     assert(gatedCounter.value === gatedCounter2.value, "gated count mismatch")
   }


配置
^^^^^^^^^^^^^
 
:ref:`コンストラクタパラメーター <clock_domain_instantiation>`に加えて、各クロックドメインの以下の要素は、
``ClockDomainConfig``クラスを介して設定可能です：

.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - プロパティ
     - 有効な値
   * - ``clockEdge``
     - ``RISING``\ , ``FALLING``
   * - ``resetKind``
     - ``ASYNC``、 ``SYNC``、そして一部のFPGAでサポートされている ``BOOT``（FFの値がビットストリームでロードされる場所）
   * - ``resetActiveLevel``
     - ``HIGH``\ , ``LOW``
   * - ``softResetActiveLevel``
     - ``HIGH``\ , ``LOW``
   * - ``clockEnableActiveLevel``
     - ``HIGH``\ , ``LOW``


.. code-block:: scala

   class CustomClockExample extends Component {
     val io = new Bundle {
       val clk    = in Bool()
       val resetn = in Bool()
       val result = out UInt (4 bits)
     }

     // Configure the clock domain
     val myClockDomain = ClockDomain(
       clock  = io.clk,
       reset  = io.resetn,
       config = ClockDomainConfig(
         clockEdge        = RISING,
         resetKind        = ASYNC,
         resetActiveLevel = LOW
       )
     )

     // ``myClockDomain`` を使用するエリアを定義します。
     val myArea = new ClockingArea(myClockDomain) {
       val myReg = Reg(UInt(4 bits)) init(7)

       myReg := myReg + 1

       io.result := myReg
     }
   }

デフォルトでは、 ``ClockDomain`` は設計全体に適用されます。このデフォルト・ドメインの構成は以下の通りです:

- クロック: 立ち上がりエッジ
- リセット: 非同期、アクティブハイ
- クロックイネーブルなし

これに対応する ``ClockDomainConfig`` は以下のようになります:

.. code-block:: scala

   val defaultCC = ClockDomainConfig(
     clockEdge        = RISING,
     resetKind        = ASYNC,
     resetActiveLevel = HIGH
   )

内部クロック
^^^^^^^^^^^^^^

クロックドメインを作成する別の構文は次のようです:

.. code-block:: scala

   ClockDomain.internal(
     name: String,
     [config: ClockDomainConfig,] 
     [withReset: Boolean,] 
     [withSoftReset: Boolean,]
     [withClockEnable: Boolean,]
     [frequency: IClockDomainFrequency]
   )

この定義には6つのパラメータが必要です:

.. list-table::
   :header-rows: 1
   :widths: 1 5 1

   * - 引数
     - 説明
     - デフォルト
   * - ``name``
     - `clk` と `reset` シグナルの名前
     - 
   * - ``config``
     - シグナルの極性とリセットの性質を指定します。
     - Current config
   * - ``withReset``
     - リセット信号を追加します。
     - true
   * - ``withSoftReset``
     - ソフトリセット信号を追加します。
     - false
   * - ``withClockEnable``
     - クロックイネーブルを追加します。
     - false
   * - ``frequency``
     - クロックドメインの周波数
     - UnknownFrequency

このアプローチの利点は、継承された名前ではなく、既知または指定された名前でクロックとリセット信号を作成することです。

作成したら、以下の例に示すように、 ``ClockDomain``の信号を割り当てる必要があります：

.. code-block:: scala

   class InternalClockWithPllExample extends Component {
     val io = new Bundle {
       val clk100M = in Bool()
       val aReset  = in Bool()
       val result  = out UInt (4 bits)
     }
     // myClockDomain.clock は myClockName_clk と名付けられます。
     // myClockDomain.reset は myClockName_reset と名付けられます。
     val myClockDomain = ClockDomain.internal("myClockName")

     // Instantiate a PLL (probably a BlackBox)
     val pll = new Pll()
     pll.io.clkIn := io.clk100M

     // myClockDomain の信号に何かを割り当てます。
     myClockDomain.clock := pll.io.clockOut
     myClockDomain.reset := io.aReset || !pll.io.

     // myClockDomain で好きなことをしてください。
     val myArea = new ClockingArea(myClockDomain) {
       val myReg = Reg(UInt(4 bits)) init(7)
       myReg := myReg + 1

       io.result := myReg
     }
   }

.. warning::
   自分が ClockDomain を作成したコンポーネント以外では、 ``.clock``と ``.reset``ではなく、
   以下にリストされている ``.readClockWire`` と ``.readResetWire``を使用してはいけません。
   グローバルの ClockDomain の場合も、常にこれらの ``.readXXX`` 関数を使用する必要があります。

外部クロック
^^^^^^^^^^^^^^

ソースコードのどこにでも外部から駆動されるクロックドメインを定義できます。
その後、トップレベルの入力からすべての同期要素へクロックとリセットのワイヤが自動的に追加されます。

.. code-block:: scala

   ClockDomain.external(
     name: String,
     [config: ClockDomainConfig,] 
     [withReset: Boolean,] 
     [withSoftReset: Boolean,]
     [withClockEnable: Boolean,]
     [frequency: IClockDomainFrequency]
   )

``ClockDomain.external`` 関数への引数は、 ``ClockDomain.internal`` 関数とまったく同じです。以下は、 ``ClockDomain.external`` を使用した設計の例です：

.. code-block:: scala

   class ExternalClockExample extends Component {
     val io = new Bundle {
       val result = out UInt (4 bits)
     }

     // トップレベルには2つの信号があります：
     //     myClockName_clk と myClockName_reset
     val myClockDomain = ClockDomain.external("myClockName")

     val myArea = new ClockingArea(myClockDomain) {
       val myReg = Reg(UInt(4 bits)) init(7)
       myReg := myReg + 1

       io.result := myReg
     }
   }

HDL生成における信号の優先順位
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

現在のバージョンでは、リセットとクロックイネーブル信号には異なる優先順位があります。
その順序は、　``asyncReset``、　``clockEnable``、　``syncReset``、および 　``softReset``です。

クロックイネーブルが同期リセットよりも優先されることに注意してください。
クロックイネーブルが無効の場合に同期リセットを実行すると（特にシミュレーションの開始時に）、ゲーテッドレジスタがリセットされません。

以下に例を示します：


.. code-block:: scala

  val clockedArea = new ClockEnableArea(clockEnable) {
    val reg = RegNext(io.input) init(False)
  }

次のようなVerilogHDLコードが生成されます：

.. code-block:: verilog

  always @(posedge clk) begin
    if(clockedArea_newClockEnable) begin
      if(!resetn) begin
        clockedArea_reg <= 1'b0;
      end else begin
        clockedArea_reg <= io_input;
      end
    end
  end

その動作が問題である場合、ClockDomain.enable機能の代わりにwhen文をクロックイネーブルとして使用することができます。これは将来の改善のためにオープンです。

文脈
^^^^^^^

``ClockDomain.current``をどこでも呼び出すことで、現在のクロックドメインを取得できます。

返された　``ClockDomain``インスタンスには、以下の関数が呼び出せます：

.. list-table::
   :header-rows: 1
   :widths: 1 5 1

   * - 名前
     - 説明
     - 戻り値
   * - frequency.getValue
     - | クロックドメインの周波数を返します。
       | これは、ドメインを構成した任意の値です。
     - Double
   * - hasReset
     - クロックドメインにリセット信号がある場合は True を返します。
     - Boolean
   * - hasSoftReset
     - クロックドメインにソフトリセット信号がある場合は True を返します。
     - Boolean
   * - hasClockEnable
     - クロックドメインにクロックイネーブル信号がある場合は True を返します。
     - Boolean
   * - readClockWire
     - クロック信号から派生した信号を返します。
     - Bool
   * - readResetWire
     - リセット信号から派生した信号を返します。
     - Bool
   * - readSoftResetWire
     - ソフトリセット信号から派生した信号を返します。
     - Bool
   * - readClockEnableWire
     - クロックイネーブル信号から派生した信号を返します。
     - Bool
   * - isResetActive
     - リセットがアクティブな場合は True を返します。
     - Bool
   * - isSoftResetActive
     - ソフトリセットがアクティブな場合は True を返します。
     - Bool
   * - isClockEnableActive
     - クロックイネーブルがアクティブな場合は True を返します。
     - Bool

以下に、UARTコントローラがクロック分周器を設定するために周波数仕様を使用する例を示します：

.. code-block:: scala

   val coreClockDomain = ClockDomain(coreClock, coreReset, frequency=FixedFrequency(100e6))

   val coreArea = new ClockingArea(coreClockDomain) {
     val ctrl = new UartCtrl()
     ctrl.io.config.clockDivider := (coreClk.frequency.getValue / 57.6e3 / 8).toInt
   }


クロックドメインクロッシング
----------------------------

SpinalHDLは、コンパイル時に、望ましくない/未指定のクロックドメイン間の信号読み取りがないことを確認します。
別の ``ClockDomain``領域から発信された信号を読み取りたい場合は、
次の例に示すように、宛先信号に ``crossClockDomain`` タグを追加する必要があります：

.. code-block:: scala

   //             _____                        _____             _____
   //            |     |  (crossClockDomain)  |     |           |     |
   //  dataIn -->|     |--------------------->|     |---------->|     |--> dataOut
   //            | FF  |                      | FF  |           | FF  |
   //  clkA   -->|     |              clkB -->|     |   clkB -->|     |
   //  rstA   -->|_____|              rstB -->|_____|   rstB -->|_____|



   // コンポーネントのIOで与えられるクロックとリセットピンがある実装
   class CrossingExample extends Component {
     val io = new Bundle {
       val clkA = in Bool()
       val rstA = in Bool()

       val clkB = in Bool()
       val rstB = in Bool()

       val dataIn  = in Bool()
       val dataOut = out Bool()
     }

     // clkAで dataIn をサンプリングします。
     val area_clkA = new ClockingArea(ClockDomain(io.clkA,io.rstA)) {
       val reg = RegNext(io.dataIn) init(False)
     }

     // メタスタビリティの問題を回避するための2つのレジスタステージ
     val area_clkB = new ClockingArea(ClockDomain(io.clkB,io.rstB)) {
       val buf0   = RegNext(area_clkA.reg) init(False) addTag(crossClockDomain)
       val buf1   = RegNext(buf0)          init(False)
     }

     io.dataOut := area_clkB.buf1
   }


   // クロックドメインがパラメータとして与えられる代替実装
   class CrossingExample(clkA : ClockDomain,clkB : ClockDomain) extends Component {
     val io = new Bundle {
       val dataIn  = in Bool()
       val dataOut = out Bool()
     }

     // clkAで dataIn をサンプリングします。
     val area_clkA = new ClockingArea(clkA) {
       val reg = RegNext(io.dataIn) init(False)
     }

     // メタスタビリティの問題を回避するための2つのレジスタステージ
     val area_clkB = new ClockingArea(clkB) {
       val buf0   = RegNext(area_clkA.reg) init(False) addTag(crossClockDomain)
       val buf1   = RegNext(buf0)          init(False)
     }

     io.dataOut := area_clkB.buf1
   }

一般的に、メタスタビリティを防ぐために、宛先クロックドメインによって駆動される2つ以上のフリップフロップを使用できます。
``spinal.lib._``で提供される ``BufferCC(input: T, init: T = null, bufferDepth: Int = 2)``関数は、
必要なフリップフロップをインスタンス化します（フリップフロップの数は ``bufferDepth``パラメータによって異なります）。

.. code-block:: scala

   class CrossingExample(clkA : ClockDomain,clkB : ClockDomain) extends Component {
     val io = new Bundle {
       val dataIn  = in Bool()
       val dataOut = out Bool()
     }

     // clkAで dataIn をサンプリングします。
     val area_clkA = new ClockingArea(clkA) {
       val reg = RegNext(io.dataIn) init(False)
     }

     // メタスタビリティの問題を回避するためのBufferCC
     val area_clkB = new ClockingArea(clkB) {
       val buf1   = BufferCC(area_clkA.reg, False)
     }

     io.dataOut := area_clkB.buf1
   }

.. warning::
   ``BufferCC`` 関数は、型が ``Bit`` またはGrayコード化されたカウンターとして機能する ``Bits``のシグナルにのみ適用されます
   （1クロックサイクルごとに1ビットのフリップのみ）。
   複数ビットのクロスドメインプロセスの場合、 ``StreamFifoCC``を高帯域幅要件に使用するか、
   帯域幅が重要でない場合はリソース使用量を削減するために ``StreamCCByToggle`` を使用することをお勧めします。
   
特別なクロッキングエリア
-------------------------

低速エリア
^^^^^^^^^^^

``SlowArea``は、現在のクロック・ドメインよりも遅い新しいクロック・ドメイン・エリアを作成するために使用されます:

.. code-block:: scala

   class TopLevel extends Component {

     // 現在のクロック・ドメインを使用します。: 100MHz
     val areaStd = new Area {    
       val counter = out(CounterFreeRun(16).value)
     }

     // 現在のクロック・ドメインを4倍遅くします。: 25 MHz
     val areaDiv4 = new SlowArea(4) {
       val counter = out(CounterFreeRun(16).value)
     }

     // 現在のクロック・ドメインを50MHzに遅くします。
     val area50Mhz = new SlowArea(50 MHz) {
       val counter = out(CounterFreeRun(16).value)
     }
   }

   def main(args: Array[String]) {
     new SpinalConfig(
       defaultClockDomainFrequency = FixedFrequency(100 MHz)
     ).generateVhdl(new TopLevel)
   }

BootReset
^^^^^^^^^

`clockDomain.withBootReset()` は、レジスタの resetKind を BOOT として指定できます。
`clockDomain.withSyncReset()` は、レジスタの resetKind を SYNC（同期リセット）として指定できます。


.. code-block:: scala 

    class  Top extends Component {
        val io = new Bundle {
          val data = in Bits(8 bit)
          val a, b, c, d = out Bits(8 bit)
        }
        io.a  :=  RegNext(io.data) init 0
        io.b  :=  clockDomain.withBootReset()  on RegNext(io.data) init 0
        io.c  :=  clockDomain.withSyncReset()  on RegNext(io.data) init 0
        io.d  :=  clockDomain.withAsyncReset() on RegNext(io.data) init 0
    }
    SpinalVerilog(new Top)

ResetArea
^^^^^^^^^

``ResetArea``は、特別なリセット信号が現在のクロックドメインのリセットと組み合わされる新しいクロックドメインエリアを作成するために使用されます。

.. code-block:: scala

   class TopLevel extends Component {

     val specialReset = Bool()

     // このエリアのリセットは、specialReset信号で行われます。 
     val areaRst_1 = new ResetArea(specialReset, false) {
       val counter = out(CounterFreeRun(16).value)
     }

     // このエリアのリセットは、現在のリセットとspecialResetとの組み合わせです。
     val areaRst_2 = new ResetArea(specialReset, true) {
       val counter = out(CounterFreeRun(16).value)
     }
   }

ClockEnableArea
^^^^^^^^^^^^^^^

``ClockEnableArea`` は、現在のクロックドメインに追加のクロックイネーブルを追加するために使用されます。

.. code-block:: scala

   class TopLevel extends Component {

     val clockEnable = Bool()

     // この領域にクロックイネーブルを追加します。
     val area_1 = new ClockEnableArea(clockEnable) {
       val counter = out(CounterFreeRun(16).value)
     }
   }
