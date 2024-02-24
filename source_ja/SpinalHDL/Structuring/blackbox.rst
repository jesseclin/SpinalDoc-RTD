.. _blackbox:

Instantiate VHDL and Verilog IP
===============================

説明
-----------

ブラックボックスは、既存のVHDL/Verilogコンポーネントを設計に統合するために、
そのインターフェースを指定するだけでユーザーが利用できるようにします。
エラボレーションは、シミュレータまたは合成ツールによって正しく行われます。


ブラックボックスの定義
-------------------------

以下に、ブラックボックスを定義する方法の例を示します：

.. code-block:: scala

   // Ram をブラックボックスとして定義する
   class Ram_1w_1r(wordWidth: Int, wordCount: Int) extends BlackBox {
     // VHDL ジェネリック/Verilog パラメータをブラックボックスに追加
     // String、Int、Double、Boolean、およびすべての SpinalHDL 基本型をジェネリック値として使用できます。
     addGeneric("wordCount", wordCount)
     addGeneric("wordWidth", wordWidth)

     // VHDLエンティティ/VerilogモジュールのIOを定義
     val io = new Bundle {
       val clk = in Bool()
       val wr = new Bundle {
         val en   = in Bool()
         val addr = in UInt (log2Up(wordCount) bits)
         val data = in Bits (wordWidth bits)
       }
       val rd = new Bundle {
         val en   = in Bool()
         val addr = in UInt (log2Up(wordCount) bits)
         val data = out Bits (wordWidth bits)
       }
     }

     // 現在のクロックドメインを io.clk ピンにマップします。
     mapClockDomain(clock=io.clk)
   }

| VHDLでは、型が ``Bool`` のシグナルは ``std_logic`` に、そして ``Bits`` は ``std_logic_vector`` に変換されます。もし ``std_ulogic`` を得たいなら、 ``BlackBox`` の代わりに ``BlackBoxULogic`` を使用する必要があります。
| Verilogでは、 ``BlackBoxUlogic`` を使用しても生成されるVerilogは変わりません。

.. code-block:: scala

   class Ram_1w_1r(wordWidth: Int, wordCount: Int) extends BlackBoxULogic {
     ...
   }

ジェネリック
---------------

ジェネリックを宣言する方法は2つあります：

.. code-block:: scala

   class Ram(wordWidth: Int, wordCount: Int) extends BlackBox {
       addGeneric("wordCount", wordCount)
       addGeneric("wordWidth", wordWidth)

       // または

       val generic = new Generic {
         val wordCount = Ram.this.wordCount
         val wordWidth = Ram.this.wordWidth
       }
   }

ブラックボックスのインスタンス化
-----------------------------------

``ブラックボックス`` のインスタンス化は、 ``コンポーネント`` のインスタンス化と同じです：

.. code-block:: scala

   // トップレベルを作成し、RAMをインスタンス化します。
   class TopLevel extends Component {
     val io = new Bundle {    
       val wr = new Bundle {
         val en   = in Bool()
         val addr = in UInt (log2Up(16) bits)
         val data = in Bits (8 bits)
       }
       val rd = new Bundle {
         val en   = in Bool()
         val addr = in UInt (log2Up(16) bits)
         val data = out Bits (8 bits)
       }
     }

     // ブラックボックスをインスタンス化します。
     val ram = new Ram_1w_1r(8,16)

     // すべてのシグナルを接続します。
     io.wr.en   <> ram.io.wr.en
     io.wr.addr <> ram.io.wr.addr
     io.wr.data <> ram.io.wr.data
     io.rd.en   <> ram.io.rd.en
     io.rd.addr <> ram.io.rd.addr
     io.rd.data <> ram.io.rd.data
   }

   object Main {
     def main(args: Array[String]): Unit = {
       SpinalVhdl(new TopLevel)
     }
   }

クロックとリセットのマッピング
------------------------------

ブラックボックスの定義では、クロックとリセットのワイヤを明示的に定義する必要があります。 
``ClockDomain`` のシグナルをブラックボックスの対応する入力にマッピングするには、 
``mapClockDomain`` または ``mapCurrentClockDomain`` 関数を使用できます。 
``mapClockDomain`` のパラメータは以下のとおりです：

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 5

   * - 名前
     - タイプ
     - デフォルト
     - 説明
   * - clockDomain
     - ClockDomain
     - ClockDomain.current
     - シグナルを提供するクロックドメインを指定する
   * - clock
     - Bool
     - なし
     - クロックドメインのクロックに接続されるべきブラックボックスの入力
   * - reset
     - Bool
     - なし
     - クロックドメインのリセットに接続されるべきブラックボックスの入力
   * - enable
     - Bool
     - なし
     - クロックドメインの有効信号に接続されるべきブラックボックスの入力

``mapCurrentClockDomain`` は、 ``mapClockDomain`` とほぼ同じパラメータを持っていますが、クロックドメインは含まれません。

例えば：

.. code-block:: scala

   class MyRam(clkDomain: ClockDomain) extends BlackBox {

     val io = new Bundle {
       val clkA = in Bool()
       ...
       val clkB = in Bool()
       ...
     }

     // クロック A は特定のクロックドメインにマップされます。 
     mapClockDomain(clkDomain, io.clkA)
     // クロック B は現在のクロックドメインにマップされます。
     mapCurrentClockDomain(io.clkB)
   }

ioプレフィックス
-----------------

ブラックボックスの各IOに"io\_" のプレフィックスを付けないようにするために、以下に示すように ``noIoPrefix()`` 関数を使用できます。

.. code-block:: scala

   // Ramをブラックボックスとして定義します。
   class Ram_1w_1r(wordWidth: Int, wordCount: Int) extends BlackBox {

     val generic = new Generic {
       val wordCount = Ram_1w_1r.this.wordCount
       val wordWidth = Ram_1w_1r.this.wordWidth
     }

     val io = new Bundle {
       val clk = in Bool()

       val wr = new Bundle {
         val en   = in Bool()
         val addr = in UInt (log2Up(_wordCount) bits)
         val data = in Bits (_wordWidth bits)
       }
       val rd = new Bundle {
         val en   = in Bool()
         val addr = in UInt (log2Up(_wordCount) bits)
         val data = out Bits (_wordWidth bits)
       }
     }

     noIoPrefix()

     mapCurrentClockDomain(clock=io.clk)
   }

ブラックボックスのすべての IO を名前変更する
---------------------------------------------

``BlackBox`` や ``Component`` の IO は、 ``addPrePopTask`` 関数を使用してコンパイル時に名前変更することができます。
この関数は、コンパイル中に適用される引数なしの関数を取り、名前変更のパスを追加するのに便利です。以下の例に示すように:

.. code-block:: scala

   class MyRam() extends Blackbox {

     val io = new Bundle {
       val clk = in Bool()
       val portA = new Bundle{
         val cs   = in Bool()
         val rwn  = in Bool()
         val dIn  = in Bits(32 bits)
         val dOut = out Bits(32 bits)
       }
       val portB = new Bundle{
         val cs   = in Bool()
         val rwn  = in Bool()
         val dIn  = in Bits(32 bits)
         val dOut = out Bits(32 bits)
       }
     }

     // クロックをマッピングします。 
     mapCurrentClockDomain(io.clk)

     // io_ プレフィックスを削除します。 
     noIoPrefix() 

     // ブラックボックスのすべてのシグナルの名前を変更するために使用される関数 
     private def renameIO(): Unit = {
       io.flatten.foreach(bt => {
         if(bt.getName().contains("portA")) bt.setName(bt.getName().replace("portA_", "") + "_A") 
         if(bt.getName().contains("portB")) bt.setName(bt.getName().replace("portB_", "") + "_B") 
       })
     }

     // コンポーネントの作成後に、renameIO 関数を実行します。 
     addPrePopTask(() => renameIO())
   }

   // このコードは、これらの名前を生成します：
   //    clk 
   //    cs_A, rwn_A, dIn_A, dOut_A
   //    cs_B, rwn_B, dIn_B, dOut_B


RTL ソースを追加する
----------------------

``addRTLPath()`` 関数を使用すると、RTL ソースをブラックボックスに関連付けることができます。
SpinalHDL コードを生成した後、 ``mergeRTLSource`` 関数を呼び出してすべてのソースを結合することができます。

.. code-block:: scala

   class MyBlackBox() extends Blackbox {

     val io = new Bundle {
       val clk   = in  Bool()
       val start = in Bool()
       val dIn   = in  Bits(32 bits)
       val dOut  = out Bits(32 bits)    
       val ready = out Bool()
     }

     // クロックをマッピングします。
     mapCurrentClockDomain(io.clk)

     // io_ プレフィックスを削除します。 
     noIoPrefix() 

     // すべての RTL 依存関係を追加します。
     addRTLPath("./rtl/RegisterBank.v")                         // Verilog ファイルを追加します。 
     addRTLPath(s"./rtl/myDesign.vhd")                          // VHDL ファイルを追加します。
     addRTLPath(s"${sys.env("MY_PROJECT")}/myTopLevel.vhd")     // 環境変数 MY_PROJECT を使用します（System.getenv("MY_PROJECT")）。
   }

   ...

   class TopLevel() extends Component{
     //...
     val bb = new MyBlackBox()
     //...
   }

   val report = SpinalVhdl(new TopLevel)
   report.mergeRTLSource("mergeRTL") // すべての RTL ソースを mergeRTL.vhd と mergeRTL.v ファイルに結合します。

VHDL - No numeric 型
---------------------------

ブラックボックスコンポーネントで ``std_logic_vector`` のみを使用したい場合は、ブラックボックスにタグ ``noNumericType`` を追加できます。

.. code-block:: scala

   class MyBlackBox() extends BlackBox{
     val io = new Bundle {
       val clk       = in  Bool()
       val increment = in  Bool()
       val initValue = in  UInt(8 bits)
       val counter   = out UInt(8 bits)
     }

     mapCurrentClockDomain(io.clk)

     noIoPrefix()

     addTag(noNumericType)  // std_logic_vector のみ
   }

上記のコードは、次のVHDLを生成します：

.. code-block:: vhdl

   component MyBlackBox is
     port( 
       clk       : in  std_logic;
       increment : in  std_logic;
       initValue : in  std_logic_vector(7 downto 0);
       counter   : out std_logic_vector(7 downto 0)    
     );
   end component;
