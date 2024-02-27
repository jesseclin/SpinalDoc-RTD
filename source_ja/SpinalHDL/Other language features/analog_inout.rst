
.. _section-analog_and_inout:

Analog and inout
================

概要
------------

SpinalHDL では、 ``Analog`` キーワードと ``inout`` キーワードを使用して、
トライステート信号をネイティブに定義することができます。

この機能が追加された理由は次のとおりです。

* トライステート信号をトップレベルに追加できるようにするため (手動で VHDL/Verilog でラッピングする必要がなくなります)。
* ``inout`` ピンを含むブラックボックスの定義を許可するため。
* ブラックボックスの ``inout`` ピンを階層構造を通してトップレベルの ``inout`` ピンに接続できるようにするため。
* 
これらの機能は利便性向上のためだけに追加されたものであるため、トライステートロジックを使った複雑な機能の実装はまだ試さないでください。

メモリマップド GPIO ペリフェラルのようなコンポーネントをモデリングしたい場合は、
Spinal 標準ライブラリの  :ref:`TriState/TriStateArray <section-tristate>` バンドルを使用してください。
このバンドルは、トライステートドライバの真の性質を抽象化しています。

Analog
------

``Analog`` キーワードは、信号をアナログ信号として定義することを許可します。
デジタル世界では、アナログ信号は ``0``、``1``、または ``Z`` (非接続、ハイインピーダンス状態) を表すことができます。

例えば:

.. code-block:: scala

   case class SdramInterface(g : SdramLayout) extends Bundle {
     val DQ    = Analog(Bits(g.dataWidth bits)) // 双方向データバス
     val DQM   = Bits(g.bytePerWord bits)
     val ADDR  = Bits(g.chipAddressWidth bits)
     val BA    = Bits(g.bankWidth bits)
     val CKE, CSn, CASn, RASn, WEn  = Bool()
   }

inout
-----

``inout`` キーワードは、 ``Analog`` 信号を双方向信号 (入力と出力の両方) として設定することを許可します。

例えば:

.. code-block:: scala

   case class SdramInterface(g : SdramLayout) extends Bundle with IMasterSlave {
     val DQ    = Analog(Bits(g.dataWidth bits)) // 双方向データバス
     val DQM   = Bits(g.bytePerWord bits)
     val ADDR  = Bits(g.chipAddressWidth bits)
     val BA    = Bits(g.bankWidth bits)
     val CKE, CSn, CASn, RASn, WEn  = Bool()

     override def asMaster() : Unit = {
       out(ADDR, BA, CASn, CKE, CSn, DQM, RASn, WEn)
       inout(DQ) // Analog DQ をコンポーネントの inout 信号として設定
     }
   }

InOutWrapper
------------

``InOutWrapper`` は、コンポーネントのすべての ``master`` ``TriState`` / ``TriStateArray`` / ``ReadableOpenDrain`` 
バンドルをネイティブな ``inout(Analog(...))`` 信号に変換するツールです。
これにより、ハードウェア記述を ``Analog`` / ``inout`` 関係なく記述し、その後トップレベルを変換して合成可能なようにすることができます。

例えば:

.. code-block:: scala

   case class Apb3Gpio(gpioWidth : Int) extends Component {
     val io = new Bundle{
       val gpio = master(TriStateArray(gpioWidth bits))
       val apb  = slave(Apb3(Apb3Gpio.getApb3Config()))
     }
     ...
   }

   SpinalVhdl(InOutWrapper(Apb3Gpio(32)))

は、以下の VHDL コードを生成します。

.. code-block:: vhdl

   entity Apb3Gpio is
     port(
       io_gpio : inout std_logic_vector(31 downto 0); -- この io_gpio は元々 TriStateArray バンドルでした
       io_apb_PADDR : in unsigned(3 downto 0);
       io_apb_PSEL : in std_logic_vector(0 downto 0);
       io_apb_PENABLE : in std_logic;
       io_apb_PREADY : out std_logic;
       io_apb_PWRITE : in std_logic;
       io_apb_PWDATA : in std_logic_vector(31 downto 0);
       io_apb_PRDATA : out std_logic_vector(31 downto 0);
       io_apb_PSLVERROR : out std_logic;
       clk : in std_logic;
       reset : in std_logic
     );
   end Apb3Gpio;

以下のようになる代わりに:

.. code-block:: vhdl

   entity Apb3Gpio is
     port(
       io_gpio_read : in std_logic_vector(31 downto 0);
       io_gpio_write : out std_logic_vector(31 downto 0);
       io_gpio_writeEnable : out std_logic_vector(31 downto 0);
       io_apb_PADDR : in unsigned(3 downto 0);
       io_apb_PSEL : in std_logic_vector(0 downto 0);
       io_apb_PENABLE : in std_logic;
       io_apb_PREADY : out std_logic;
       io_apb_PWRITE : in std_logic;
       io_apb_PWDATA : in std_logic_vector(31 downto 0);
       io_apb_PRDATA : out std_logic_vector(31 downto 0);
       io_apb_PSLVERROR : out std_logic;
       clk : in std_logic;
       reset : in std_logic
     );
   end Apb3Gpio;

Analog バンドルの手動駆動
-------------------------------

``Analog``バンドルが駆動されない場合、デフォルトで高インピーダンスになります。
したがって、トライステート・ドライバーを手動で実装するには（ ``InOutWrapper`` タイプが使用できない場合）、
信号を条件付きで駆動する必要があります。

``TriState``信号を ``Analog`` バンドルに手動で接続するには：

.. code-block:: scala

    case class Example extends Component {
      val io = new Bundle {
        val tri = slave(TriState(Bits(16 bits)))
        val analog = inout(Analog(Bits(16 bits)))
      }
      io.tri.read := io.analog
      when(io.tri.writeEnable) { io.analog := io.tri.write }
    }
