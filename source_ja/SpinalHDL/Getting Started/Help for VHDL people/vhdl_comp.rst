.. role:: raw-html-m2r(raw)
   :format: html

VHDL comparison
===============

導入
------------

このページでは、VHDL と SpinalHDL の主な違いを示します。物事は深く説明されません。

Process
-------

RTL を記述するときにプロセスが必要になることがよくありますが、
プロセスのセマンティクスは扱いにくい場合があります。 
VHDL での動作方法により、コードを分割して内容を複製する必要が生じる可能性があります。

次の RTL を生成するには:

.. image:: /asset/picture/process_rtl.svg

次の VHDL を記述する必要があります。:

.. code-block:: ada

     signal mySignal : std_logic;
     signal myRegister : std_logic_vector(3 downto 0);
     signal myRegisterWithReset : std_logic_vector(3 downto 0);
   begin
     process(cond)
     begin
       mySignal <= '0';
       if cond = '1' then
         mySignal <= '1';
       end if;
     end process;

     process(clk)
     begin
       if rising_edge(clk) then
         if cond = '1' then
           myRegister <= myRegister + 1;
         end if;
       end if;
     end process;

     process(clk,reset)
     begin
       if reset = '1' then
         myRegisterWithReset <= (others => '0');
       elsif rising_edge(clk) then
         if cond = '1' then
           myRegisterWithReset <= myRegisterWithReset + 1;
         end if;
       end if;
     end process;

SpinalHDL では:

.. code-block:: scala

   val mySignal = Bool()
   val myRegister = Reg(UInt(4 bits))
   val myRegisterWithReset = Reg(UInt(4 bits)) init(0)

   mySignal := False
   when(cond) {
     mySignal := True
     myRegister := myRegister + 1
     myRegisterWithReset := myRegisterWithReset + 1
   }

暗黙的な定義と明示的な定義
--------------------------------

VHDL では、信号を宣言するときに、それが組み合わせ信号であるかレジスタであるかを指定しません。
どこにどのように割り当てるかによって、それが組み合わせであるか登録されているかが決まります。

SpinalHDL では、このようなことが明示的に行われます。レジスタは、宣言内で直接レジスタとして定義されます。


クロックドメイン
---------------------

VHDL では、多数のレジスタを定義するたびに、クロックとリセット ワイヤをそれらのレジスタに転送する必要があります。
さらに、これらのクロック信号とリセット信号の使用方法 (クロック エッジ、リセット極性、リセットの性質 (非同期、同期)) 
をあらゆる場所でハードコーディングする必要があります。

SpinalHDL では、 ``ClockDomain`` を定義し、それを使用するハードウェアの領域を定義できます。

例えば:

.. code-block:: scala

   val coreClockDomain = ClockDomain(
     clock = io.coreClk,
     reset = io.coreReset,
     config = ClockDomainConfig(
       clockEdge = RISING,
       resetKind = ASYNC,
       resetActiveLevel = HIGH
     )
   )
   val coreArea = new ClockingArea(coreClockDomain) {
     val myCoreClockedRegister = Reg(UInt(4 bits))
     // ...
     // coreClockDomain は、エリア内でインスタンス化されたすべてのサブコンポーネントにも適用されます。
     // ... 
   }

コンポーネントの内部組織
---------------------------------

VHDL には、コンポーネント内にロジックのサブエリアを定義できる ``block`` 機能があります。
ただし、ほとんどの人がこの機能について知らないことと、
これらの領域内で定義されたすべての信号が外部から読み取れないため、
この機能を使用する人はほとんどいません。

SpinalHDL には、この概念をより適切に実行する ``Area`` 機能があります。

.. code-block:: scala

   val timeout = new Area {
     val counter = Reg(UInt(8 bits)) init(0)
     val overflow = False
     when(counter =/= 100) {
       counter := counter + 1
     } otherwise {
       overflow := True
     }
   }

   val core = new Area {
     when(timeout.overflow) {
       timeout.counter := 0
     }
   }

``Area`` 内で定義された変数と信号は、他の ``Area`` 領域を含め、
コンポーネント内の他の場所からアクセスできます

安全性
------

SpinalHDL と同様、VHDL では、組み合わせループを作成したり、
プロセスのパスで信号を駆動するのを忘れてラッチを推論したりすることが簡単に起こります。

次に、それらの問題を検出するには、VHDL を分析する ``lint`` ツールを使用できますが、これらのツールは無料ではありません。 
SpinalHDL では、 ``lint`` プロセスがコンパイラ内に統合されており、すべてが正常になるまで RTL コードは生成されません。
また、クロック ドメインの交差もチェックします。

関数とプロシージャ
------------------------

関数とプロシージャは、おそらく非常に制限されているため、VHDL ではあまり使用されません。

* 組み合わせハードウェアのチャンクのみ、またはレジスタのチャンクのみを定義できます (クロックされたプロセス内で関数/プロシージャを呼び出す場合)。
* 内部にプロセスを定義することはできません。
* 内部でコンポーネントをインスタンス化することはできません。
* 内部で読み書きできる範囲は限られています。

SpinalHDL では、これらの制限がすべて削除されます。

組み合わせロジックとレジスタを 1 つの関数に混在させる例:

.. code-block:: scala

   def simpleAluPipeline(op: Bits, a: UInt, b: UInt): UInt = {
     val result = UInt(8 bits)

     switch(op) {
       is(0){ result := a + b }
       is(1){ result := a - b }
       is(2){ result := a * b }
     }

     return RegNext(result)
   }

ストリーム バンドル内のキュー関数 (ハンドシェイク) の例。この関数は FIFO コンポーネントをインスタンス化します。

.. code-block:: scala

   class Stream[T <: Data](dataType:  T) extends Bundle with IMasterSlave with DataCarrier[T] {
     val valid = Bool()
     val ready = Bool()
     val payload = cloneOf(dataType)

     def queue(size: Int): Stream[T] = {
       val fifo = new StreamFifo(dataType, size)
       fifo.io.push <> this
       fifo.io.pop
     }
   }

関数がそれ自体の外部で定義された信号を割り当てる例:

.. code-block:: scala

   val counter = Reg(UInt(8 bits)) init(0)
   counter := counter + 1

   def clear() : Unit = {
     counter := 0
   }

   when(counter > 42) {
     clear()
   }

バスとインターフェース
-------------------------

VHDL は、バスとインターフェイスに関しては非常に退屈です。次の 2 つのオプションがあります。

1) いつでもどこでも、ワイヤーバイワイヤーでバスとインターフェイスを定義:

.. code-block:: ada

   PADDR   : in unsigned(addressWidth-1 downto 0);
   PSEL    : in std_logic
   PENABLE : in std_logic;
   PREADY  : out std_logic;
   PWRITE  : in std_logic;
   PWDATA  : in std_logic_vector(dataWidth-1 downto 0);
   PRDATA  : out std_logic_vector(dataWidth-1 downto 0);

2) レコードを使用しますが、パラメータ化が失われ (パッケージ内で静的に固定されています)、方向ごとに 1 つ定義する必要があります。:

.. code-block:: ada

   P_m : in APB_M;
   P_s : out APB_S;

SpinalHDL は、無制限のパラメータ化を伴うバスおよびインターフェイス宣言を非常に強力にサポートしています。

.. code-block:: scala

   val P = slave(Apb3(addressWidth, dataWidth))

オブジェクト指向プログラミングを使用して構成オブジェクトを定義することもできます:

.. code-block:: scala

   val coreConfig = CoreConfig(
     pcWidth = 32,
     addrWidth = 32,
     startAddress = 0x00000000,
     regFileReadyKind = sync,
     branchPrediction = dynamic,
     bypassExecute0 = true,
     bypassExecute1 = true,
     bypassWriteBack = true,
     bypassWriteBackBuffer = true,
     collapseBubble = false,
     fastFetchCmdPcCalculation = true,
     dynamicBranchPredictorCacheSizeLog2 = 7
   )

   // CPU には、コアに新しい機能を追加できるプラグイン システムがあります。
   // これらの拡張機能はコアに直接実装されていませんが、別の領域で定義された一種の追加ロジック パッチです。
   coreConfig.add(new MulExtension)
   coreConfig.add(new DivExtension)
   coreConfig.add(new BarrelShifterFullExtension)

   val iCacheConfig = InstructionCacheConfig(
     cacheSize = 4096,
     bytePerLine = 32,
     wayCount = 1,  // 現時点では 1 つだけにすることができます
     wrappedMemAccess = true,
     addressWidth = 32,
     cpuDataWidth = 32,
     memDataWidth = 32
   )

   new RiscvCoreAxi4(
     coreConfig = coreConfig,
     iCacheConfig = iCacheConfig,
     dCacheConfig = null,
     debug = debug,
     interruptCount = interruptCount
   )

シグナル宣言
------------------

VHDL では、アーキテクチャ記述の先頭ですべての信号を定義する必要がありますが、これは煩わしいことです。

.. code-block:: VHDL

     ..
     .. (多くのシグナル宣言)
     ..
     signal a : std_logic;
     ..
     .. (多くのシグナル宣言)
     ..
   begin
     ..
     .. (多くのロジック定義)
     ..
     a <= x & y
     ..
     .. (多くのロジック定義)
     ..

SpinalHDL は信号宣言に関して柔軟です。

.. code-block:: scala

   val a = Bool()
   a := x & y

また、信号を 1 行で定義して割り当てることもできます。

.. code-block:: scala

   val a = x & y

コンポーネントのインスタンス化
--------------------------------

VHDL は、サブコンポーネント エンティティのすべての信号を再定義し、コンポーネントをインスタンス化するときに 1 つずつバインドする必要があるため、
これに関して非常に冗長です。

.. code-block:: VHDL

   divider_cmd_valid : in std_logic;
   divider_cmd_ready : out std_logic;
   divider_cmd_numerator : in unsigned(31 downto 0);
   divider_cmd_denominator : in unsigned(31 downto 0);
   divider_rsp_valid : out std_logic;
   divider_rsp_ready : in std_logic;
   divider_rsp_quotient : out unsigned(31 downto 0);
   divider_rsp_remainder : out unsigned(31 downto 0);

   divider : entity work.UnsignedDivider
     port map (
       clk             => clk,
       reset           => reset,
       cmd_valid       => divider_cmd_valid,
       cmd_ready       => divider_cmd_ready,
       cmd_numerator   => divider_cmd_numerator,
       cmd_denominator => divider_cmd_denominator,
       rsp_valid       => divider_rsp_valid,
       rsp_ready       => divider_rsp_ready,
       rsp_quotient    => divider_rsp_quotient,
       rsp_remainder   => divider_rsp_remainder
     );

SpinalHDL はそれを取り除き、オブジェクト指向の方法でサブコンポーネントの IO にアクセスできるようにします。

.. code-block:: scala

   val divider = new UnsignedDivider()

   // And then if you want to access IO signals of that divider:
   divider.io.cmd.valid := True
   divider.io.cmd.numerator := 42

キャスティング
---------------

VHDL には、迷惑なキャスト メソッドが 2 つあります:

* boolean <> std_logic (例: ``mySignal <= myValue < 10`` などの条件を使用してシグナルを割り当てることは正当ではありません)
* unsigned <> integer  (例: 配列にアクセスするには)

SpinalHDL は、統合することでこれらのキャストを削除します。

boolean/std_logic:

.. code-block:: scala

   val value = UInt(8 bits)
   val valueBiggerThanTwo = Bool()
   valueBiggerThanTwo := value > 2  // 値 > 2 は Bool を返します

unsigned/integer:

.. code-block:: scala

   val array = Vec(UInt(4 bits),8)
   val sel = UInt(3 bits)
   val arraySel = array(sel) // 配列は UInt を使用して直接インデックス付けされます

リサイズ
-----------

VHDL がビット サイズについて厳密であるという事実は、おそらく良いことです。

.. code-block:: ada

   my8BitsSignal <= resize(my4BitsSignal, 8);

SpinalHDL では、同じことを行う方法が 2 つあります:

.. code-block:: scala

   // 伝統的な方法
   my8BitsSignal := my4BitsSignal.resize(8)

   // 賢い方法
   my8BitsSignal := my4BitsSignal.resized

パラメータ化
----------------

| 2008 年リビジョンより前の VHDL には、ジェネリックに関して多くの問題があります。たとえば、レコードをパラメーター化することはできません。エンティティ内の配列をパラメーター化することも、型パラメーターを持つこともできません。
| その後、VHDL 2008 が登場し、これらの問題が修正されました。ただし、VHDL 2008 に対する RTL ツールのサポートは、ベンダーによっては非常に貧弱です。

SpinalHDL は、コンパイラーにネイティブに統合されたジェネリックスを完全にサポートしており、VHDL ジェネリックスに依存しません。

パラメータ化されたデータ構造の例を次に示します:

.. code-block:: scala

   val colorStream = Stream(Color(5, 6, 5)))
   val colorFifo   = StreamFifo(Color(5, 6, 5), depth = 128)
   colorFifo.io.push <> colorStream

パラメータ化されたコンポーネントの例を次に示します:

.. code-block:: scala

   class Arbiter[T <: Data](payloadType: T, portCount: Int) extends Component {
     val io = new Bundle {
       val sources = Vec(slave(Stream(payloadType)), portCount)
       val sink = master(Stream(payloadType))
     }
     // ...
   }

メタハードウェアの説明
-------------------------

VHDL には一種の閉じられた構文があります。その上に抽象化レイヤーを追加することはできません。

SpinalHDL は Scala 上に構築されているため、非常に柔軟であり、新しい抽象化レイヤーを非常に簡単に定義できます。

この柔軟性の例としては、:ref:`FSM <state_machine>` ライブラリ、
:ref:`BusSlaveFactory <bus_slave_factory>` ライブラリ、
および :ref:`JTAG <jtag>` ライブラリがあります。
