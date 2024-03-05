.. role:: raw-html-m2r(raw)
   :format: html

.. _stream:

Stream
======

仕様
-------------

| ストリームインターフェースは、ペイロードを運ぶためのシンプルなハンドシェイクプロトコルです。
| これは、例えば FIFO に要素をプッシュしたりポップしたり、UART コントローラーにリクエストを送信するために使用できます。

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 10 1

   * - シグナル
     - タイプ
     - ドライバー
     - 説明
     - いつも無視
   * - valid
     - Bool
     - マスター
     - ハイの場合 => インターフェースにペイロードが存在します
     - 
   * - ready
     - Bool
     - スレーブ
     - ローの場合 => トランザクションはスレーブによって消費されていません
     - valid がローの場合
   * - payload
     - T
     - マスター
     - トランザクションの内容
     - valid がローの場合

.. wavedrom::

   { "signal": [
     {"name": "clk",     "wave": "p........."},
     {"name": "valid"  , "wave": "0101..01.0"},
     {"name": "ready"  , "wave": "x1x0.1x1.x"},
     {"name": "payload", "wave": "x=x=..x==x","data":["D0","D1","D2","D3"]}
   ]}

SpinalHDL での使用例がいくつかあります：

.. code-block:: scala

   class StreamFifo[T <: Data](dataType: T, depth: Int) extends Component {
     val io = new Bundle {
       val push = slave Stream (dataType)
       val pop = master Stream (dataType)
     }
     ...
   }

   class StreamArbiter[T <: Data](dataType: T,portCount: Int) extends Component {
     val io = new Bundle {
       val inputs = Vec(slave Stream (dataType),portCount)
       val output = master Stream (dataType)
     }
     ...
   }

.. note::
   各スレーブは、valid がハイで ready がローの場合にペイロードを変更することができます。例：

* ロックロジックなしの優先度アービターは、1つの入力から他の入力に切り替えることができます（これによりペイロードが変更されます）。
* UART コントローラーは、直接書き込みポートを使用して UART ピンを駆動し、トランスミッションの終了時にトランザクションをのみ消費できます。それには注意してください。


Semantics
---------

ストリームの信号を手動で読み取ったり駆動したりする場合は、次のことを考慮してください：

* アサートされた後、 ``valid`` は、現在のペイロードが認識された後にのみ非アサートされる場合があります。
  つまり、 ``valid``は、スレーブが ``ready`` をアサートして読み取りを行った後のサイクルにのみ 0 にトグルできます。
* これに対して、 ``ready`` はいつでも変更できます。
* 転送は、 ``valid`` と ``ready`` がアサートされているサイクルでのみ行われます。
* ストリームの ``valid`` は、組み合わせ的な方法で ``ready`` に依存してはならず、
  両者の間の任意のパスはレジスタで登録されている必要があります。
* ``valid`` が ``ready`` にまったく依存しないようにすることが推奨されます。

機能
---------

.. list-table::
   :header-rows: 1
   :widths: 5 5 1 1

   * - 構文
     - 説明
     - 戻り値
     - レイテンシー
   * - Stream(type : Data)
     - 指定された型のストリームを作成します
     - Stream[T]
     - 
   * - master/slave Stream(type : Data)
     - | 指定された型のストリームを作成します
       | 対応する in/out のセットアップで初期化されます 
     - Stream[T]
     - 
   * - x.fire
     - バスでトランザクションが消費されると True を返します（valid && ready）
     - Bool
     - 
   * - x.isStall
     - バスでトランザクションが停止すると True を返します（valid && ! ready）
     - Bool
     - 
   * - x.queue(size:Int)
     - FIFO を介して x に接続されたストリームを返します
     - Stream[T]
     - 2
   * - | x.m2sPipe()
       | x.stage()
     - | x によって駆動されるストリームを返します
       | valid/payload パスを切断するレジスタステージを介して
       | コスト = (ペイロード幅 + 1) flop flop  
     - Stream[T]
     - 1
   * - x.s2mPipe()
     - | x によって駆動されるストリームを返します
       | ready パスはレジスタステージで切断されます
       | コスト = ペイロード幅 * (mux2 + 1 flip flop)
     - Stream[T]
     - 0
   * - x.halfPipe()
     - | x によって駆動されるストリームを返します
       | valid/ready/payload パスはいくつかのレジスタで切断されます
       | コスト = （ペイロード幅 + 2）flip flop、帯域幅は半分になります
     - Stream[T]
     - 1
   * - | x << y
       | y >> x
     - y を x に接続します
     - 
     - 0
   * - | x <-< y
       | y >-> x
     - y を x に m2sPipe を介して接続します
     - 
     - 1
   * - | x </< y
       | y >/> x
     - y を x に s2mPipe を介して接続します
     - 
     - 0
   * - | x <-/< y
       | y >/-> x
     - | y を x に s2mPipe().m2sPipe() を介して接続します
       | これにより、x と y の間に組み合わせパスがないことが前提とされます
     - 
     - 1
   * - x.haltWhen(cond : Bool)
     - | x に接続されたストリームを返します
       | cond が true の場合、ストリームは停止します
     - Stream[T]
     - 0
   * - x.throwWhen(cond : Bool)
     - | x に接続されたストリームを返します
       | cond が true の場合、トランザクションは破棄されます
     - Stream[T]
     - 0

次のコードはこのロジックを作成します：

.. image:: /asset/picture/stream_throw_m2spipe.svg
   :align: center

.. code-block:: scala

   case class RGB(channelWidth : Int) extends Bundle {
     val red   = UInt(channelWidth bits)
     val green = UInt(channelWidth bits)
     val blue  = UInt(channelWidth bits)

     def isBlack : Bool = red === 0 && green === 0 && blue === 0
   }

   val source = Stream(RGB(8))
   val sink   = Stream(RGB(8))
   sink <-< source.throwWhen(source.payload.isBlack)

ユーティリティ
-------------------

ストリームバスと組み合わせて設計できる多くのユーティリティがあります。この章ではそれらを文書化します。

StreamFifo
^^^^^^^^^^

各ストリームには、.queue(size) を呼び出してバッファリングされたストリームを取得できます。
ただし、FIFO コンポーネント自体をインスタンス化することもできます：

.. code-block:: scala

   val streamA,streamB = Stream(Bits(8 bits))
   //...
   val myFifo = StreamFifo(
     dataType = Bits(8 bits),
     depth    = 128
   )
   myFifo.io.push << streamA
   myFifo.io.pop  >> streamB

.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - パラメータ名
     - タイプ
     - 説明
   * - dataType
     - T
     - ペイロードデータ型
   * - depth
     - Int
     - 要素を格納するために使用されるメモリのサイズ

.. list-table::
   :header-rows: 1
   :widths: 1 4 5

   * - io 名
     - タイプ
     - 説明
   * - push
     - Stream[T]
     - 要素をプッシュするために使用
   * - pop
     - Stream[T]
     - 要素をポップするために使用
   * - flush
     - Bool
     - FIFO 内のすべての要素を削除するために使用
   * - occupancy
     - UInt of log2Up(depth + 1) bits
     - 内部メモリの占有率を示す


StreamFifoCC
^^^^^^^^^^^^

次の方法で、FIFO のデュアルクロックドメインバージョンをインスタンス化できます：

.. code-block:: scala

   val clockA = ClockDomain(???)
   val clockB = ClockDomain(???)
   val streamA,streamB = Stream(Bits(8 bits))
   //...
   val myFifo = StreamFifoCC(
     dataType  = Bits(8 bits),
     depth     = 128,
     pushClock = clockA,
     popClock  = clockB
   )
   myFifo.io.push << streamA
   myFifo.io.pop  >> streamB

.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - パラメータ名
     - タイプ
     - 説明
   * - dataType
     - T
     - ペイロードデータ型
   * - depth
     - Int
     - 要素を格納するために使用されるメモリのサイズ
   * - pushClock
     - ClockDomain
     - プッシュ側で使用されるクロックドメイン
   * - popClock
     - ClockDomain
     - ポップ側で使用されるクロックドメイン


.. list-table::
   :header-rows: 1
   :widths: 1 4 5

   * - io 名
     - タイプ
     - 説明
   * - push
     - Stream[T]
     - 要素をプッシュするために使用
   * - pop
     - Stream[T]
     - 要素をポップするために使用
   * - pushOccupancy
     - UInt of log2Up(depth + 1) bits
     - 内部メモリの占有率を示す（プッシュ側の視点から）
   * - popOccupancy
     - UInt of log2Up(depth + 1) bits
     - 内部メモリの占有率を示す（ポップ側の視点から）


StreamCCByToggle
^^^^^^^^^^^^^^^^

| トグル信号に基づいてクロックドメイン間を接続するコンポーネントです。
| このクロックドメイン間ブリッジの実装方法は、エリア使用量が少なく、帯域幅も低いという特徴があります。

.. code-block:: scala

   val clockA = ClockDomain(???)
   val clockB = ClockDomain(???)
   val streamA,streamB = Stream(Bits(8 bits))
   //...
   val bridge = StreamCCByToggle(
     dataType    = Bits(8 bits),
     inputClock  = clockA,
     outputClock = clockB
   )
   bridge.io.input  << streamA
   bridge.io.output >> streamB

.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - パラメータ名
     - タイプ
     - 説明
   * - dataType
     - T
     - ペイロードデータ型
   * - inputClock
     - ClockDomain
     - プッシュ側で使用されるクロックドメイン
   * - outputClock
     - ClockDomain
     - ポップ側で使用されるクロックドメイン


.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - io 名
     - タイプ
     - 説明
   * - input
     - Stream[T]
     - 要素をプッシュするために使用
   * - output
     - Stream[T]
     - 要素をポップするために使用

代替として、直接クロック間ストリームを返すこのより短い構文も使用できます：

.. code-block:: scala

   val clockA = ClockDomain(???)
   val clockB = ClockDomain(???)
   val streamA = Stream(Bits(8 bits))
   val streamB = StreamCCByToggle(
     input       = streamA,
     inputClock  = clockA,
     outputClock = clockB
   )

StreamWidthAdapter
^^^^^^^^^^^^^^^^^^

このコンポーネントは、入力ストリームの幅を出力ストリームに適応させます。

``outStream`` ペイロードの幅が ``inStream`` よりも大きい場合、複数の入力トランザクションのペイロードを1つに組み合わせます。
逆に、 ``outStream`` のペイロード幅が  ``inStream`` よりも小さい場合、1つの入力トランザクションが複数の出力トランザクションに分割されます。

最良の場合、 ``inStream`` のペイロードの幅は、次に示すように ``outStream`` の整数倍である必要があります。

.. code-block:: scala

   val inStream = Stream(Bits(8 bits))
   val outStream = Stream(Bits(16 bits))
   val adapter = StreamWidthAdapter(inStream, outStream)

上記の例のように、2つの ``inStream`` トランザクションは 1つの ``outStream`` トランザクションにマージされ、
最初の入力トランザクションのペイロードはデフォルト設定で出力ペイロードの下位ビットに配置されます。

入力トランザクションのペイロード配置の予想される順序がデフォルトの設定と異なる場合は、次の例を参照してください。

.. code-block:: scala

   val inStream = Stream(Bits(8 bits))
   val outStream = Stream(Bits(16 bits))
   val adapter = StreamWidthAdapter(inStream, outStream, order = SlicesOrder.HIGHER_FIRST)

また、 ``ORDER`` と同じ効果を持つ ``endianness`` という従来のパラメータもあります。
``endianness`` の値は、 ``LITTLE`` の場合は  ``order`` の ``LOWER_FIRST`` と同じであり、 ``BIG`` の場合は ``HIGHER_FIRST`` と同じです。
``padding`` パラメータは、アダプターが入力と出力のペイロード幅の整数倍でない場合を受け入れるかどうかを決定するオプションのブール値です。


StreamArbiter
^^^^^^^^^^^^^

複数のストリームがあり、それらを 1つのストリームに駆動させるためにアービトレーションを行いたい場合は、StreamArbiterFactory を使用できます。

.. code-block:: scala

   val streamA, streamB, streamC = Stream(Bits(8 bits))
   val arbitredABC = StreamArbiterFactory.roundRobin.onArgs(streamA, streamB, streamC)

   val streamD, streamE, streamF = Stream(Bits(8 bits))
   val arbitredDEF = StreamArbiterFactory.lowerFirst.noLock.onArgs(streamD, streamE, streamF)

.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - アービトレーション関数	
     - 説明
   * - lowerFirst
     - 下位ポートが上位ポートより優先されます
   * - roundRobin
     - フェアなラウンドロビンアービトレーション
   * - sequentialOrder
     - | トランザクションを連続した順序で取得するために使用できます
       | 最初のトランザクションはポート0から取得され、次にポート1から取得されます...

.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - ロック関数	
     - 説明
   * - noLock
     - 選択されたポートは、選択されたポート上のトランザクションが消費されなくても、毎サイクル変更される可能性があります
   * - transactionLock
     - 選択されたポートは、選択されたポート上のトランザクションが消費されるまでロックされます
   * - fragmentLock
     - | Stream[Flow[T]] を仲介するために使用できます。
       | このモードでは、ポートの選択は選択されたポートのバーストが終了するまでロックされます（last=True）。


.. list-table::
   :header-rows: 1
   :widths: 2 1

   * - 生成関数	
     - 戻り値
   * - on(inputs : Seq[Stream[T]])
     - Stream[T]
   * - onArgs(inputs : Stream[T]*)
     - Stream[T]

StreamJoin
^^^^^^^^^^

このユーティリティは、複数の入力ストリームを取り、すべてのストリームが `valid`` を発火するまで待機し、 
`ready`` を提供してすべてのストリームを通過させます。

.. code-block:: scala

   val cmdJoin = Stream(Cmd())
   cmdJoin.arbitrationFrom(StreamJoin.arg(cmdABuffer, cmdBBuffer))


StreamFork
^^^^^^^^^^

StreamFork は、各入力データをすべての出力ストリームに複製します。 synchronous が true の場合、
すべての出力ストリームは常に一緒に発火します。
これは、すべての出力ストリームが準備できるまで、ストリームが停止することを意味します。 
synchronous が false の場合、出力ストリームは一度に1つずつ準備されることがありますが、
それには追加のフリップフロップ（ 1ビットあたりの出力）がかかります。
入力ストリームは、すべての出力ストリームが各アイテムを処理するまでブロックされます。


.. code-block:: scala

   val inputStream = Stream(Bits(8 bits))
   val (outputstream1, outputstream2) = StreamFork2(inputStream, synchronous=false)

または

.. code-block:: scala

   val inputStream = Stream(Bits(8 bits))
   val outputStreams = StreamFork(inputStream,portCount=2, synchronous=true)

StreamMux
^^^^^^^^^

``Stream`` 向けの mux 実装です。
``select`` シグナルと ``inputs`` のストリームを取り、 ``select`` で指定された入力ストリームに接続された ``Stream`` を返します。
``StreamArbiter`` はこれと類似した機能を持っていますが、より強力です。

.. code-block:: scala

   val inputStreams = Vec(Stream(Bits(8 bits)), portCount)
   val select = UInt(log2Up(inputStreams.length) bits)
   val outputStream = StreamMux(select, inputStreams)

.. note::
   ``select`` シグナルの ``UInt`` 型は、出力ストリームが停止している間に変更されると、トランザクションが途中で中断される可能性があります。
   安全な操作を行うためには、 ``Stream`` 型の ``select`` を使用して、安全な場合にのみ発火してルーティングを変更するストリームインターフェースを生成できます。

StreamDemux
^^^^^^^^^^^

``Stream`` 向けの demux 実装です。
``input``、 ``select``、 ``portCount`` を取り、``select`` で指定された出力ストリームが ``input`` に接続され、他の出力ストリームは非アクティブになります。
安全なトランザクションについては、上記の注意事項を参照してください。

.. code-block:: scala

   val inputStream = Stream(Bits(8 bits))
   val select = UInt(log2Up(portCount) bits)
   val outputStreams = StreamDemux(inputStream, select, portCount)

StreamDispatcherSequencial
^^^^^^^^^^^^^^^^^^^^^^^^^^

このユーティリティは、入力ストリームを取り、それを ``outputCount`` のストリームに順次ルーティングします。

.. code-block:: scala

   val inputStream = Stream(Bits(8 bits))
   val dispatchedStreams = StreamDispatcherSequencial(
     input = inputStream,
     outputCount = 3
   )

StreamTransactionExtender
^^^^^^^^^^^^^^^^^^^^^^^^^

このユーティリティは、1つの入力転送を取り、複数の出力転送を生成します。これは、出力転送にペイロードの値を count+1 回繰り返す機能を提供します。
``count`` は、個々のペイロードのたびに inputStream がファイアされるたびにキャプチャされ、登録されます。

.. code-block:: scala

   val inputStream = Stream(Bits(8 bits))
   val outputStream = Stream(Bits(8 bits))
   val count = UInt(3 bits)
   val extender = StreamTransactionExtender(inputStream, outputStream, count) {
      // id は、現在の入力トランザクション内でこれまでの全出力トランザクションの 0ベースのインデックスです。
      // last は、最後の転送インジケーションで、extender の last シグナルと同じです。
      // 返されたペイロードは、id と last シグナルに基づいてのみ変更が許可され、その他の変換はこの外部で行う必要があります。
      (id, payload, last) => payload
   }

この ``extender`` は、 ``working``、 ``last``、 ``done`` などのいくつかのステータスシグナルを提供します。
``working`` は、1つの入力転送が受け入れられて進行中であることを意味し、
``last`` は、最後の出力転送が準備されて完了を待っていることを示し、 ``done`` が有効になると、
最後の出力転送がファイアされ、現在の入力トランザクション処理が完了し、別のトランザクションを開始できる状態になります。

.. wavedrom::

  { "signal": [
    { "name": "clk",         "wave": "p........." },
    { "name": "inputStream",        "wave": "x3x.....4x", "data": ["T1", "T2"] },
    { "name": "count",        "wave": "x3x.....4x", "data": ["2", "4"] },
    { "name": "outputStream",       "wave": "x..2x2x.2x", "data": ["D1", "D2", "D3"] },
    { "name": "working",      "wave": "0.1......."},
    { "name": "done",      "wave": "0.......10"},
    { "name": "first",      "wave": "0.1.0....."},
    { "name": "last",      "wave": "0.....1..0"},
  ]}

.. note::

   出力ストリームのカウントのみが必要な場合は、代わりに ``StreamTransactionCounter`` を使用してください。

Simulation support
------------------

シミュレーションのマスターとスレーブの実装には、以下のものがあります：

.. list-table::
  :header-rows: 1
  :widths: 1 5
  
  * - クラス
    - 用途
  * - StreamMonitor
    - マスターとスレーブの両方に使用され、ストリームが発火した場合にペイロードとともに関数を呼び出します。
  * - StreamDriver
    - テストベンチのマスターサイドで使用され、関数を呼び出して値を適用します（利用可能な場合）。関数は値が利用可能かどうかを返さなければなりません。ランダムな遅延をサポートしています。
  * - StreamReadyRandmizer
    - データの受信のための ``ready`` をランダム化し、テストベンチはスレーブサイドです。
  * - ScoreboardInOrder
    - リファレンス/dut データを比較するためによく使用されます。

.. code-block:: scala

  import spinal.core._
  import spinal.core.sim._
  import spinal.lib._
  import spinal.lib.sim.{StreamMonitor, StreamDriver, StreamReadyRandomizer, ScoreboardInOrder}

  object Example extends App {
    val dut = SimConfig.withWave.compile(StreamFifo(Bits(8 bits), 2))

    dut.doSim("simple test") { dut =>
      SimTimeout(10000)
      
      val scoreboard = ScoreboardInOrder[Int]()
      
      dut.io.flush #= false
      
      // ランダムなデータをドライブし、プッシュされたデータをスコアボードに追加する
      StreamDriver(dut.io.push, dut.clockDomain) { payload =>
        payload.randomize()
        true
      }
      StreamMonitor(dut.io.push, dut.clockDomain) { payload =>
        scoreboard.pushRef(payload.toInt)
      }

      // 出力の ready をランダム化し、ポップされたデータをスコアボードに追加する
      StreamReadyRandomizer(dut.io.pop, dut.clockDomain)
      StreamMonitor(dut.io.pop, dut.clockDomain) { payload =>
        scoreboard.pushDut(payload.toInt)
      }

      dut.clockDomain.forkStimulus(10)

      dut.clockDomain.waitActiveEdgeWhere(scoreboard.matches == 100)
    }
  }
