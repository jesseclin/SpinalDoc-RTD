.. _foreword:

序文
========

予備的なメモ:

* 以下の記述はすべて、デジタル ハードウェアの説明に関するものです。検証も興味深いトピックです。
* 簡潔にするために、SystemVerilog が最新のリビジョンであると仮定します。
  Verilog。
* これを読むとき、私たちのお気に入りの HDL に対する愛着が私たちの判断をどれほど偏らせるかを過小評価してはなりません


従来の HDL から離れる理由
------------------------------------

VHDL/Verilog はハードウェア記述言語ではありません
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

こちらのような言語は、もともとシミュレーションやドキュメント作成を目的としたイベント駆動型言語です。
シンセサイザーの入力言語として使われるようになったのは、後になってからです。
これが、以下のような多くの点の根源を説明しています。


イベント駆動型パラダイムは RTL にとって意味がありません
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

よく考えてみると、デジタルハードウェア (RTL) を process/always ブロックを使って記述するのは、実用的な意味が全くありません。
なぜ感度リストを気にする必要があるのか、
なぜ設計を異なる性質の process/always ブロック (コンビネーション論理 / リセットなしレジスタ / 非同期リセットレジスタ) に分割する必要があるのか、不思議に思います。

たとえば、これを実装するには:

.. image:: example_design.png
   :align: center

VHDL process を使用してこれを作成します:

.. code-block:: vhdl

   signal mySignal : std_logic;
   signal myRegister : unsigned(3 downto 0);
   signal myRegisterWithReset : unsigned(3 downto 0);

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
           myRegisterWithReset <= 0;
       elsif rising_edge(clk) then
           if cond = '1' then
               myRegisterWithReset <= myRegisterWithReset + 1;
           end if;
       end if;
   end process;

SpinalHDL を使用してこれを記述します:

.. code-block:: scala

   val mySignal             = Bool()
   val myRegister           = Reg(UInt(4 bits))
   val myRegisterWithReset  = Reg(UInt(4 bits)) init(0)

   mySignal := False
   when(cond) {
       mySignal            := True
       myRegister          := myRegister + 1
       myRegisterWithReset := myRegisterWithReset + 1
   }


なんでも慣れればイベント駆動型セマンティクスにも慣れてしまいますよ。でも、もっと良いものに出会えば話は別です。


最近改訂された VHDL と Verilog はまだ十分に利用可能とは言えない。
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

EDA 業界は VHDL 2008 や SystemVerilog の合成機能の実装にとても遅れています。
さらに、実装されたとしても、通常は限定されたサブセットのみで、興味深い新機能を利用するのは安全ではないようです 
(シミュレーション機能は除きます)。

* おそらく、あなたのコードは多くの EDA ツールと互換性がなくなってしまうでしょう。
* 他社は、おそらくあなたの IP を受け入れてくれないでしょう。なぜなら、彼らのフローはまだそれに対応できていないからです。

根本問題は依然として残っています。
VHDL や SystemVerilog の最近の改訂は、イベント駆動型という根本的なパラダイムを変えていないからです。
このパラダイムは、本来デジタルハードウェアを記述するのに適していないのです。


VHDL レコードと Verilog 構造体は不十分です (使える環境なら、SystemVerilog が良さそうです)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

VHDL レコード/Verilog 構造体をインターフェース定義に使うのは厳しいですね。
内部の信号の方向を定義できないからです。さらに悪いことに、コンストラクタパラメータも使えません。
つまり、RGB レコード/構造体を一度定義したら、色チャネルの大きさを変更する必要が出たときに困ることになります…気をつけましょう！

VHDL のもう一つの "お決まり" なところは、コンポーネントエンティティに配列を使いたい場合には、その配列の型をパッケージで定義しなければならない点です。
このパッケージは、残念ながらパラメータ化することができません...

たとえば、以下は SpinalHDL APB3 バス定義です:

.. code-block:: scala

   // 特定の APB3 構成を表すためにインスタンス化できるクラス
   case class Apb3Config(
     addressWidth  : Int,
     dataWidth     : Int,
     selWidth      : Int     = 1,
     useSlaveError : Boolean = true
   )

   // 特定のハードウェア APB3 バスを表すためにインスタンス化できるクラス
   case class Apb3(config: Apb3Config) extends Bundle with IMasterSlave {
     val PADDR      = UInt(config.addressWidth bits)
     val PSEL       = Bits(config.selWidth bits)
     val PENABLE    = Bool()
     val PREADY     = Bool()
     val PWRITE     = Bool()
     val PWDATA     = Bits(config.dataWidth bits)
     val PRDATA     = Bits(config.dataWidth bits)
     val PSLVERROR  = if(config.useSlaveError) Bool() else null  // Optional signal

     // 特定の APB3 バスをホスト コンポーネントのマスター インターフェイスにセットアップするために使用できます
     // `asSlave` は対称性によって自動的に実装されます
     override def asMaster(): Unit = {
       out(PADDR, PSEL, PENABLE, PWRITE, PWDATA)
       in(PREADY, PRDATA)
       if(config.useSlaveError) in(PSLVERROR)
     }
   }

VHDL 2008 の部分解決策や SystemVerilog の interface/modport は、
使えるなら使えるけど、使える環境が限られてるのがネックだよね。
EDA ツール、開発フロー、会社の方針次第で、なかなか使えないことも多いからね…


VHDL と Verilog は非常に冗長です
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

VHDL と Verilog は、特にコンポーネントのインスタンス化と接続に関する作業になると、
神頼み(コードのコピーペースト)に陥りがちですよね。

その実態をより深く理解するために、SpinalHDL で周辺機器のインスタンス化を行い、
アクセスするための APB3 デコーダーを追加する例を見てみましょう。

.. code-block:: scala

   // AXI4 から APB3 へのブリッジをインスタンス化する
   val apbBridge = Axi4ToApb3Bridge(
     addressWidth = 20,
     dataWidth    = 32,
     idWidth      = 4
   )

   // Instantiate some APB3 peripherals
   val gpioACtrl = Apb3Gpio(gpioWidth = 32)
   val gpioBCtrl = Apb3Gpio(gpioWidth = 32)
   val timerCtrl = PinsecTimerCtrl()
   val uartCtrl = Apb3UartCtrl(uartCtrlConfig)
   val vgaCtrl = Axi4VgaCtrl(vgaCtrlConfig)

   // 一部の APB3 ペリフェラルをインスタンス化する
   // - apbBridge によって駆動される
   // - メモリ領域内の各ペリフェラルをマッピングする
   val apbDecoder = Apb3Decoder(
     master = apbBridge.io.apb,
     slaves = List(
       gpioACtrl.io.apb -> (0x00000, 4 KiB),
       gpioBCtrl.io.apb -> (0x01000, 4 KiB),
       uartCtrl.io.apb  -> (0x10000, 4 KiB),
       timerCtrl.io.apb -> (0x20000, 4 KiB),
       vgaCtrl.io.apb   -> (0x30000, 4 KiB)
     )
   )


モジュール/コンポーネントをインスタンス化するときに、オブジェクト指向ライクにインターフェースにアクセスできるので、
個々の信号をバインドする必要がなくなります。これは大きな利点ですね。

さらに、VHDL/Verilog の構造体/レコードについては、真のパラメータ化や再利用機能を持たない、
いわば "姑息な手段" だと考えることができます。これらの言語はそもそも設計時に大きな欠陥があったことを隠ぺいしようとしているのかもしれませんね。


メタハードウェア記述言語の機能拡張可能について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

VHDL や Verilog は、ループ、generate 文、マクロ、関数、プロシージャ、タスクなどの構造を直接ハードウェアにマッピングしない抽象化ツールを提供していますが、それ以上の機能は限定的です。

たとえば、process/always/コンポーネント/モジュール ブロックをタスク/プロシージャ内に定義することはできません。
これは、より洗練された機能の実現を妨げる大きなボトルネックとなっています。

SpinalHDL を使用すると、そのようなバス上でユーザー定義のタスク/プロシージャを呼び出すことができます:
``myHandshakeBus.queue(depth=64)``. 以下は定義を含むコードです.

.. code-block:: scala

   // 握手バスの概念を定義する
   class Stream[T <: Data](dataType:  T) extends Bundle {
     val valid   = Bool()
     val ready   = Bool()
     val payload = cloneOf(dataType)

     // 左のオペランド (this) を右のオペランド (that) に接続する演算子を定義します
     def >>(that: Stream[T]): Unit = {
       that.valid := this.valid
       this.ready := that.ready
       that.payload := this.payload
     }

     // 深度要素の FIFO を介してこれに接続されたストリームを返します
     def queue(depth: Int): Stream[T] = {
       val fifo = new StreamFifo(dataType, depth)
       this >> fifo.io.push
       return fifo.io.pop
     }
   }

VHDL/Verilog でステートマシンを定義する場合、スイッチ文を使った膨大なローコードを書く必要があります。
各ステートを定義するための適切な構文を提供する "StateMachine" という概念も存在しません。
代わりに、サードパーティのツールを使ってステートマシンをグラフィカルに定義し、VHDL/Verilog コードを生成する方法もありますが...

SpinalHDL のメタハードウェア記述機能を使えば、独自のツールを定義し、ステートマシンなどの要素を抽象的な方法で定義することができます。

以下は、SpinalHDL上に定義されたステートマシン抽象化の簡単な使用例です:

.. code-block:: scala

   // 新しいステートマシンを定義する
   val fsm = new StateMachine{
     // Define all states
     val stateA, stateB, stateC = new State

     // エントリーポイントを設定する
     setEntry(stateA)

     // Define a register used into the state machine
     val counter = Reg(UInt(8 bits)) init (0)

     // ステートマシンで使用されるレジスタを定義する
     stateA.whenIsActive (goto(stateB))

     stateB.onEntry(counter := 0)
     stateB.onExit(io.result := True)
     stateB.whenIsActive {
       counter := counter + 1
       when(counter === 4){
         goto(stateC)
       }
     }

     stateC.whenIsActive(goto(stateA))
   }

CPU の命令デコードを生成したい場合を考えてみましょう。
できるだけ少ないロジックで生成するために、高度な合成時のアルゴリズムが必要になるかもしれません。
しかし、VHDL/Verilog では、このような作業を行う唯一の方法は、希望する .vhd ファイルと .v ファイルを生成するスクリプトを書くことです。

メタハードウェア記述については他にも多くのポイントがありますが、
その本当の意味を理解し、実際にその利点を味わうための最良の方法は、実際に試してみる以外にはありません。
メタハードウェア記述の目的は、配線やゲートレベルでの作業から脱却し、
低レベルの詳細から一歩離れて、再利用性を重視した設計を行うことです。
