.. role:: raw-html-m2r(raw)
   :format: html

VHDL and Verilog generation
===========================

SpinalHDL コンポーネントから VHDL と Verilog を生成する
---------------------------------------------------------------

SpinalHDL コンポーネントから VHDL を生成するには、Scala の ``main`` 内で ``SpinalVhdl(new YourComponent)`` を呼び出すだけです。

Verilog を生成する方法も同様ですが、 ``SpinalVHDL`` の代わりに ``SpinalVerilog`` を使用します。


.. code-block:: scala

   import spinal.core._

   // 単純なコンポーネントの定義
   class MyTopLevel extends Component {
     // VHDL のレコードや Verilog の struct のように、入出力信号をバンドルします。
     val io = new Bundle {
       val a = in  Bool()
       val b = in  Bool()
       val c = out Bool()
     }

     // 非同期ロジックを定義します。
     io.c := io.a & io.b
   }

   // MyTopLevel に対応する VHDL と Verilog を生成するメイン関数です。
   object MyMain {
     def main(args: Array[String]) {
       SpinalVhdl(new MyTopLevel)
       SpinalVerilog(new MyTopLevel)
     }
   }

.. important::
   ``SpinalVhdl`` および ``SpinalVerilog`` では、コンポーネントクラスの複数のインスタンスを作成する必要がある場合があります。
   そのため、第1引数は ``Component`` の参照ではなく、新しいコンポーネントを返す関数です。

.. important::
   ``SpinalVerilog`` の実装は 2016 年 6 月 5 日に開始されました。
   このバックエンドは、VHDL と同じリグレッションテスト（RISCV CPU、マルチコアおよびパイプライン化されたマンデルブロ、UART RX/TX、シングルクロック FIFO、デュアルクロックFIFO、グレーカウンターなど）を正常にパスしています。

   この新しいバックエンドに問題がある場合は、 `GitHubの問題 <https://github.com/SpinalHDL/SpinalHDL/issues>`_  に問題を説明してください。
   
Scala からのパラメータ化
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 5

   * - 引数名
     - タイプ
     - デフォルト
     - 説明
   * - ``mode``
     - SpinalMode
     - null
     - | SpinalHDL の hdl 生成モードを設定します。
       | ``VHDL`` または ``Verilog`` に設定できます。
   * - ``defaultConfigForClockDomains``
     - ClockDomainConfig
     - | RisingEdgeClock
       | AsynchronousReset
       | ResetActiveHigh
       | ClockEnableActiveHigh
     - すべての新しい ``ClockDomain`` に対してデフォルト値として使用されるクロック構成を設定します。
   * - ``onlyStdLogicVectorAtTopLevelIo``
     - Boolean
     - false
     - すべての unsigned/signed をトップレベル io に変更して std_logic_vector にします。
   * - ``defaultClockDomainFrequency``
     - IClockDomainFrequency
     -  UnknownFrequency
     - デフォルトのクロック周波数。
   * - ``targetDirectory``
     - String
     - 現在のディレクトリ
     - ファイルが生成されるディレクトリ。


そして、それらを指定する構文は次のとおりです：

.. code-block:: scala

   SpinalConfig(mode=VHDL, targetDirectory="temp/myDesign").generate(new UartCtrl)

   // またはよりスケーラブルなフォーマットでVerilogの場合：
   SpinalConfig(
     mode=Verilog,
     targetDirectory="temp/myDesign"
   ).generate(new UartCtrl)

シェルからのパラメータ化
^^^^^^^^^^^^^^^^^^^^^^^^^^

コマンドライン引数を使用して生成パラメータを指定することもできます。

.. code-block:: scala

   def main(args: Array[String]): Unit = {
     SpinalConfig.shell(args)(new UartCtrl)
   }

コマンドライン引数の構文は次のとおりです：

.. code-block:: text

   Usage: SpinalCore [options]

     --vhdl
           Select the VHDL mode
     --verilog
           Select the Verilog mode
     -d | --debug
           Enter in debug mode directly
     -o <value> | --targetDirectory <value>
           Set the target directory


生成された VHDL および Verilog
---------------------------------

SpinalHDL の RTL 記述が VHDL および Verilog にどのように変換されるかは重要です：

* Scala の名前は VHDL および Verilog で保持されます。
* Scala の ``Component`` 階層は VHDL および Verilog で保持されます。
* Scala の ``when`` 文は、VHDL および Verilog においてはif文として出力されます。
* Scala の ``switch`` 文は、標準的なケースでは VHDL および Verilog において case 文として出力されます。
  
組織
^^^^^^^^^^^^

VHDL ジェネレーターを使用すると、すべてのモジュールが1つのファイルに生成されます。そのファイルには次の3つのセクションが含まれています：

#. すべての Enum の定義を含むパッケージ
#. アーキテクチャ要素で使用される関数を含むパッケージ
#. デザインに必要なすべてのコンポーネント

Verilog ジェネレーションを使用すると、すべてのモジュールが1つのファイルに生成されます。そのファイルには次の2つのセクションが含まれています：

#. 使用されるすべての Enum の定義
#. デザインに必要なすべてのモジュール
 

組み合わせロジック
^^^^^^^^^^^^^^^^^^^^^

Scala:

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val cond           = in  Bool()
       val value          = in  UInt(4 bits)
       val withoutProcess = out UInt(4 bits)
       val withProcess    = out UInt(4 bits)
     }
     io.withoutProcess := io.value
     io.withProcess := 0
     when(io.cond) {
       switch(io.value) {
         is(U"0000") {
           io.withProcess := 8
         }
         is(U"0001") {
           io.withProcess := 9
         }
         default {
           io.withProcess := io.value+1
         }
       }
     }
   }

VHDL:

.. code-block:: vhdl

   entity TopLevel is
     port(
       io_cond : in std_logic;
       io_value : in unsigned(3 downto 0);
       io_withoutProcess : out unsigned(3 downto 0);
       io_withProcess : out unsigned(3 downto 0)
     );
   end TopLevel;

   architecture arch of TopLevel is
   begin
     io_withoutProcess <= io_value;
     process(io_cond,io_value)
     begin
       io_withProcess <= pkg_unsigned("0000");
       if io_cond = '1' then
         case io_value is
           when pkg_unsigned("0000") =>
             io_withProcess <= pkg_unsigned("1000");
           when pkg_unsigned("0001") =>
             io_withProcess <= pkg_unsigned("1001");
           when others =>
             io_withProcess <= (io_value + pkg_unsigned("0001"));
         end case;
       end if;
     end process;
   end arch;

シーケンシャルロジック
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Scala:

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val cond   = in Bool()
       val value  = in UInt (4 bits)
       val resultA = out UInt(4 bits)
       val resultB = out UInt(4 bits)
     }

     val regWithReset = Reg(UInt(4 bits)) init(0)
     val regWithoutReset = Reg(UInt(4 bits))

     regWithReset := io.value
     regWithoutReset := 0
     when(io.cond) {
       regWithoutReset := io.value
     }

     io.resultA := regWithReset
     io.resultB := regWithoutReset
   }

VHDL:

.. code-block:: vhdl

   entity TopLevel is
     port(
       io_cond : in std_logic;
       io_value : in unsigned(3 downto 0);
       io_resultA : out unsigned(3 downto 0);
       io_resultB : out unsigned(3 downto 0);
       clk : in std_logic;
       reset : in std_logic
     );
   end TopLevel;

   architecture arch of TopLevel is

     signal regWithReset : unsigned(3 downto 0);
     signal regWithoutReset : unsigned(3 downto 0);
   begin
     io_resultA <= regWithReset;
     io_resultB <= regWithoutReset;
     process(clk,reset)
     begin
       if reset = '1' then
         regWithReset <= pkg_unsigned("0000");
       elsif rising_edge(clk) then
         regWithReset <= io_value;
       end if;
     end process;

     process(clk)
     begin
       if rising_edge(clk) then
         regWithoutReset <= pkg_unsigned("0000");
         if io_cond = '1' then
           regWithoutReset <= io_value;
         end if;
       end if;
     end process;
   end arch;

.. _vhdl-and-verilog-attributes:

VHDL および Verilog 属性
---------------------------

特定の状況では、設計内の一部の信号に属性を付与して、それらが合成される方法を変更することが役立ちます。

そのためには、設計内の任意の信号やメモリに対して、以下の関数を呼び出すことができます:

.. list-table::
   :header-rows: 1
   :widths: 1 2

   * - Syntax
     - Description
   * - ``addAttribute(name)``
     - 指定された ``name`` を持つブール属性をtrueに設定します
   * - ``addAttribute(name, value)``
     - 指定された ``name`` を持つ文字列属性を ``value`` に設定します


例:

.. code-block:: scala

   val pcPlus4 = pc + 4
   pcPlus4.addAttribute("keep")

VHDLで生成される宣言：

.. code-block:: vhdl

   attribute keep : boolean;
   signal pcPlus4 : unsigned(31 downto 0);
   attribute keep of pcPlus4: signal is true;

Verilogで生成される宣言：

.. code-block:: verilog

   (* keep *) wire [31:0] pcPlus4;
