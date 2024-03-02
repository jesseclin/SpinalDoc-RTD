
Introduction
============

spinal.lib.misc.pipeline は、パイプライン API を提供します。手動のパイプライニングと比較して、主な利点は次のとおりです：

- 全体のステージングされたシステムに必要なすべての信号要素を事前に定義する必要はありません。
  デザインが要求するように、ステージャブルな信号をよりアドホックな方法で作成および利用できます。
  中間段階をすべて再構成して信号について知る必要はありません
- パイプラインの信号は、SpinalHDL の強力なパラメータ化機能を利用し、
  特定の設計ビルドが特定のパラメータ化された機能を必要としない場合は、
  最適化/削除されることができます。ステージングシステムの設計やプロジェクトコードベースを大幅に変更する必要はありません。
- 手動のリタイミングがはるかに簡単になります。レジスタ/アービトレーションを手動で処理する必要がありません
- アービトレーションを自動的に管理します

この API は 4つの主要な要素で構成されています：

- Node：パイプライン内のレイヤーを表す
- Link：ノードを互いに接続する
- Builder：全体のパイプラインに必要なハードウェアを生成する
- Payload：パイプライン沿いのノードでハードウェア信号を取得するために使用されます

Payload は ハードウェアのデータ/信号インスタンスではなく、
パイプライン沿いのノードでデータ/信号を取得するためのキーであることを理解することが重要であり、
その後、パイプラインビルダーは、ノード間の特定の Payload のすべての出現を自動的に相互接続/パイプライン化します。

次に示す例で説明します：

.. image:: /asset/image/pipeline/intro_pip.png
   :scale: 70 %

このAPIに関するビデオはこちらです：

- https://www.youtube.com/watch?v=74h_-FMWWIM


シンプルな例
----------------

以下は、API の基本のみを使用するシンプルな例です：


.. code-block:: scala

    import spinal.core._
    import spinal.core.sim._
    import spinal.lib._
    import spinal.lib.misc.pipeline._

    class TopLevel extends Component {
      val io = new Bundle{
        val up = slave Stream (UInt(16 bits))
        val down = master Stream (UInt(16 bits))
      }

      //  パイプラインのための 3つのノードを定義しましょう
      val n0, n1, n2 = Node()

      // これらのノードをシンプルなレジスタを使用して接続しましょう
      val s01 = StageLink(n0, n1)
      val s12 = StageLink(n1, n2)

      // パイプラインを通過するいくつかの Payload を定義しましょう
      val VALUE = Payload(UInt(16 bits))
      val RESULT = Payload(UInt(16 bits))

      // io.up を n0 にバインドしましょう
      io.up.ready := n0.ready
      n0.valid := io.up.valid
      n0(VALUE) := io.up.payload

      // n1 でいくつかの処理を行いましょう
      n1(RESULT) := n1(VALUE) + 0x1200

      // n2 を io.down にバインドしましょう
      n2.ready := io.down.ready
      io.down.valid := n2.valid
      io.down.payload := n2(RESULT)

      // ビルダーに必要なハードウェアを生成するように要求しましょう
      Builder(s01, s12)
    }

これにより、次のハードウェアが生成されます：

.. image:: /asset/image/pipeline/simple_pip.png
   :scale: 70 %

以下は、シミュレーション波形です：

.. wavedrom::

   {signal: [
     {name: 'clk', wave: '0...p........'},
     {name: 'reset', wave: '1..0.........'},
     {name: 'io_up_valid', wave: '0.....10.....'},
     {},
     {name: 'n0_valid', wave: '0.....10.....'},
     {name: 'n0_VALUE', wave: 'x.....2......', data: ['0042']},
     {},
     {name: 'n1_valid', wave: '0......10....'},
     {name: 'n1_VALUE', wave: 'x......2.....', data: ['0042']},
     {name: 'n1_RESULT', wave: 'x......2.....', data: ['1242']},
     {},
     {name: 'n2_valid', wave: '0.......10...'},
     {name: 'n2_RESULT', wave: 'x.......2....', data: ['1242']},
     {},
     {name: 'io_down_valid', wave: '0.......10...'},
   ]}

より多くの API を使用した同じ例は次の通りです：

.. code-block:: scala

    import spinal.core._
    import spinal.core.sim._
    import spinal.lib._
    import spinal.lib.misc.pipeline._

    class TopLevel extends Component {
      val VALUE = Payload(UInt(16 bits))

      val io = new Bundle{
        val up = slave Stream(VALUE)  // VALUE は HardType としても使用できます
        val down = master Stream(VALUE)
      }
      
      // ノードが作成され、ステージを介して接続され、ハードウェアが生成されるための NodesBuilder が使用されます
      val builder = new NodesBuilder()

      // io.up から接続するノードを定義しましょう
      val n0 = new builder.Node{
        arbitrateFrom(io.up)
        VALUE := io.up.payload
      }

      // いくつかの処理を行うノードを定義しましょう
      val n1 = new builder.Node{
        val RESULT = insert(VALUE + 0x1200)
      }

      // io.down に接続するノードを定義しましょう
      val n2 = new builder.Node {
        arbitrateTo(io.down)
        io.down.payload := n1.RESULT
      }

      // レジスタステージを使用してこれらのノードを接続し、関連するハードウェアを生成しましょう
      builder.genStagedPipeline()
    }

Payload
============

Payload オブジェクトは、パイプラインを通過するデータを参照するために使用されます。
技術的には、Payload は名前を持ち、特定のパイプライン段階で信号を取得するための "キー" として使用される HardType です。

.. code-block:: scala
    
    val PC = Payload(UInt(32 bits))
    val PC_PLUS_4 = Payload(UInt(32 bits))

    val n0, n1 = Node()
    val s01 = StageLink(n0, n1)

    n0(PC) := 0x42
    n1(PC_PLUS_4) := n1(PC) + 4

Payload インスタンスの名前を大文字で指定するのに慣れています。
これは、そのものがハードウェア信号ではなく、ある種の "キー/タイプ" であることを非常に明示的にするためです。

Node
============

ノードは主に、valid/ready のアービトレーション信号と、それを介して通過するすべての Payload 値に必要なハードウェア信号を保持します。

そのアービトレーションには以下のようにアクセスできます：

.. list-table::
   :header-rows: 1
   :widths: 2 1 10

   * - API
     - アクセス
     - 説明
   * - node.valid
     - RW
     - そのノードにトランザクションが存在するかどうかを指定する信号です。
       上流から駆動されます。アサートされると、
       有効およびレディまたは node.cancel のいずれかが高い状態になった後のサイクルのみに解除される必要があります。
       valid は ready に依存してはいけません。
   * - node.ready
     - RW
     - ノードのトランザクションが下流に進むことができるかどうかを指定する信号です。
       バックプレッシャーを作成するために下流から駆動されます。
       トランザクションがない場合（ node.valid が解除されている場合）、この信号に意味がありません。
   * - node.cancel
     - RW
     - パイプラインからノードのトランザクションがキャンセルされているかどうかを指定する信号です。
       下流から駆動されます。トランザクションがない場合（ node.valid が解除されている場合）、この信号に意味がありません。
   * - node.isValid
     - RO
     - node.valid の読み取り専用アクセサです。
   * - node.isReady
     - RO
     - node.ready の読み取り専用アクセサです。
   * - node.isCancel
     - RO
     - node.cancel の読み取り専用アクセサです。
   * - node.isFiring
     - RO
     - ノードトランザクションが正常に進行中である場合は True（valid && ready && !cancel）。状態の変更を確定するのに便利です。
   * - node.isMoving
     - RO
     - ノードのトランザクションが次のサイクル以降にノード上に存在しなくなる場合に True です。
       これは、下流がトランザクションを受け取る準備ができているか、
       またはパイプラインからトランザクションがキャンセルされるためです（valid && (ready || cancel)）。
       状態を「リセット」するのに役立ちます。
   * - node.isCanceling
     - RO
     - ノードトランザクションがキャンセルされている場合はTrueです。
       これは、将来のサイクルでパイプラインのどこにも表示されないことを意味します。

ノードの valid/node.ready 信号は、 :doc:`../stream` での信号と同じ規則に従います。

ノードの制御（valid/ready/cancel）およびステータス（isValid、isReady、isCancel、isFiringなど）信号は、必要に応じて作成されます。
たとえば、ready 信号を参照しないことで、バックプレッシャーのないパイプラインを作成できます。
そのため、何かの状態を読み取る場合はステータス信号を使用し、何かを駆動する場合は制御信号のみを使用することが重要です。

以下は、ノードで発生するアービトレーションのケースのリストです。
valid/ready/cancel は私たちがどの状態にあるかを定義し、isFiring/isMoving はそれらの結果です：

+-------+-------+-----------+------------------------------+----------+----------+
| valid | ready | cancel    | Description                  | isFiring | isMoving |
+=======+=======+===========+==============================+==========+==========+
|   0   |   X   |     X     | No transaction               |    0     |    0     |
+-------+-------+-----------+------------------------------+----------+----------+
|   1   |   1   |     0     | Going through                |    1     |    1     |
+-------+-------+-----------+------------------------------+----------+----------+
|   1   |   0   |     0     | Blocked                      |    0     |    0     |
+-------+-------+-----------+------------------------------+----------+----------+
|   1   |   X   |     1     | Canceled                     |    0     |    1     |
+-------+-------+-----------+------------------------------+----------+----------+


CPU のステージのようにブロックしたりフラッシュしたりするようなものをモデル化したい場合は、
CtrlLink を見てください。それはそのようなことを行うためのAPIを提供します。

Payload で参照されるシグナルにアクセスするには、次のようにします：

.. list-table::
   :header-rows: 1
   :widths: 2 5

   * - API
     - 説明
   * - node(Payload)
     - 対応するハードウェア信号を返します
   * - node(Payload, Any)
     - 上記と同様ですが、2番目の引数を含めると、それが "セカンダリキー" として使用されます。
       これにより、マルチレーンハードウェアの構築が容易になります。
       たとえば、マルチイシュー CPU パイプラインの場合、レーン Int id をセカンダリキーとして使用できます
   * - node.insert(Data)
     - 指定した Data ハードウェア信号に接続された新しいペイロードインスタンスを返します


.. code-block:: scala
    
    val n0, n1 = Node()

    val PC = Payload(UInt(32 bits))
    n0(PC) := 0x42
    n0(PC, "true") := 0x42
    n0(PC, 0x666) := 0xEE
    val SOMETHING = n0.insert(myHardwareSignal) // これで新しいペイロードが作成されます
    when(n1(SOMETHING) === 0xFFAA){ ... }
    

パイプラインの最初や最後の段階のアービトレーション/データを手動で制御/読み取ることができますが、
その境界を接続するためのいくつかのユーティリティがあります。

.. list-table::
   :header-rows: 1
   :widths: 5 5

   * - API
     - 説明
   * - node.arbitrateFrom(Stream[T]])
     - ストリームからノードのアービトレーションを駆動します。
   * - node.arbitrateFrom(Flow[T]])
     - フローからノードのアービトレーションを駆動します。 
   * - node.arbitrateTo(Stream[T]])
     - ノードからストリームのアービトレーションを駆動します。 
   * - node.arbitrateTo(Flow[T]])
     - ノードからフローのアービトレーションを駆動します。
   * - node.driveFrom(Stream[T]])((Node, T) => Unit)
     - ストリームからノードを駆動します。提供されたラムダ関数はデータを接続するために使用できます。
   * - node.driveFrom(Flow[T]])((Node, T) => Unit)
     - 上記と同じですが、フローの場合
   * - node.driveTo(Stream[T]])((T, Node) => Unit)
     - ノードからストリームを駆動します。提供されたラムダ関数はデータを接続するために使用できます。
   * - node.driveTo(Flow[T]])((T, Node) => Unit)
     - 上記と同じですが、フローの場合


.. code-block:: scala
    
    val n0, n1, n2 = Node()

    val IN = Payload(UInt(16 bits))
    val OUT = Payload(UInt(16 bits))

    n1(OUT) := n1(IN) + 0x42

    // 以降のパイプラインに接続される入力/出力ストリームを定義します
    val up = slave Stream(UInt(16 bits))
    val down = master Stream(UInt(16 bits)) // 注意：master Stream(OUT)も使用できます

    n0.driveFrom(up)((self, payload) => self(IN) := payload)
    n2.driveTo(down)((payload, self) => payload := self(OUT))

冗長性を減らすために、ノードに関連する Payload 間の一連の暗黙の変換があります。これらは、ノードのコンテキストで使用することができます。

.. code-block:: scala

    val VALUE = Payload(UInt(16 bits))
    val n1 = new Node{
        val PLUS_ONE = insert(VALUE + 1) // VALUE が暗黙的にn1(VALUE)の表現に変換されます
    }

You can also use those implicit conversions by importing them : 

.. code-block:: scala

    val VALUE = Payload(UInt(16 bits))
    val n1 = Node()

    val n1Stuff = new Area {
        import n1._
        val PLUS_ONE = insert(VALUE) + 1 // Equivalent to n1.insert(n1(VALUE)) + 1
    }

また、指定されたノードインスタンスの全APIを提供する新しい Area を作成する API もあります（インポートなしで暗黙の変換を含む）。

.. code-block:: scala

    val n1 = Node()
    val VALUE = Payload(UInt(16 bits))

    val n1Stuff = new n1.Area{
        val PLUS_ONE = insert(VALUE) + 1 // n1.insert(n1(VALUE)) + 1と同等
    }

このような機能は、ハードウェアのパラメータ指定可能なパイプラインの場所を持つ場合に非常に便利です（リタイミングの例を参照）。

Links
============

すでにいくつかの異なるリンクが実装されています（ただし、独自のカスタムリンクを作成することもできます）。
リンクのアイデアは、さまざまな方法で2つのノードを接続することです。

一般的に、 `up` ノードと `down` ノードがあります。

DirectLink
------------------

非常にシンプルで、ノードをワイヤで直接接続します。以下は例です。

.. code-block:: scala
    
    val c01 = DirectLink(n0, n1)



StageLink
------------------

データ/有効信号にレジスタを使用し、ready にアービトレーションを行うことで、2つのノードを接続します。

.. code-block:: scala
    
    val c01 = StageLink(n0, n1)


S2mLink
------------------

準備信号にレジスタを使用して、2つのノードを接続します。これは、バックプレッシャーの組み合わせタイミングを改善するのに役立ちます。

.. code-block:: scala
    
    val c01 = S2mLink(n0, n1)

CtrlLink
------------------

これは、任意のフロー制御/バイパスロジックで 2つのノードを接続する特別なリンクの一種です。
その API は、CPU ステージを実装するのに十分な柔軟性があるはずです。

以下はそのフロー制御 API です（Bool引数で機能を有効にします）：

.. list-table::
   :header-rows: 1
   :widths: 2 5

   * - API
     - Description
   * - haltWhen(Bool)
     - 現在のトランザクションをブロックできます（up.ready をクリアし、down.valid をクリアします）
   * - throwWhen(Bool)
     - パイプラインから現在のトランザクションをキャンセルできます（ down.valid をクリアし、トランザクションドライバーが現在の状態を忘れます）
   * - forgetOneWhen(Bool)
     - 上流に現在のトランザクションを忘れるように要求できます（ただし、down.valid はクリアされません）
   * - ignoreReadyWhen(Bool)
     - 下流のreadyを無視することができます（up.ready を設定します）
   * - duplicateWhen(Bool)
     - 現在のトランザクションを複製できます（up.ready をクリアします）
   * - terminateWhen(Bool)
     - 現在のトランザクションを下流から非表示にできます（down.valid をクリアします）

また、条件スコープ内でフロー制御を行いたい場合（たとえば、when ステートメント内）、次の関数を呼び出すことができます：

- haltIt(), duplicateIt(), terminateIt(), forgetOneNow(), ignoreReadyNow(), throwIt()

.. code-block:: scala
    
    val c01 = CtrlLink(n0, n1)

    c01.haltWhen(something) // 明示的な停止リクエスト

    when(somethingElse){
        c01.haltIt() // 条件スコープに敏感な停止リクエスト、c01.haltWhen(somethingElse)と同じ
    }

リンクに接続されているノードは、node.up / node.down を使用して取得できます。

CtrlLink には、Payload にアクセスするAPIも提供されています：

.. list-table::
   :header-rows: 1
   :widths: 2 5

   * - API
     - 説明
   * - link(Payload)
     - Link.down(Payload) と同じ
   * - link(Payload, Any)
     - Link.down(Payload, Any) と同じ
   * - link.insert(Data)
     - Link.down.insert(Data) と同じ
   * - link.bypass(Payload)
     - link.up -> link.down 間の Payload 値を条件付きで上書きできます。これは、CPU パイプラインでのデータハザードを修正するために使用できます。


.. code-block:: scala
    
    val c01 = CtrlLink(n0, n1)

    val PC = Payload(UInt(32 bits))
    c01(PC) := 0x42
    c01(PC, 0x666) := 0xEE

    val DATA = Payload(UInt(32 bits))
    // データが c01 の前にパイプラインに挿入されているとします
    when(hazard){
        c01.bypass(DATA) := fixedValue
    }
    
    // c01(DATA) 以下には、ハザードパッチが適用されます

CtrlLink を引数なしで作成すると、内部で独自のノードが作成されます。

.. code-block:: scala

    val decode = CtrlLink()
    val execute = CtrlLink()

    val d2e = StageLink(decode.down, execute.up)


その他のリンク
------------------------------------

JoinLink / ForkLink も実装されています。

カスタムリンク
------------------------------------

リンクベースクラスを実装することで、カスタムリンクを実装できます。

.. code-block:: scala

    trait Link extends Area{
      def ups : Seq[Node]
      def downs : Seq[Node]

      def propagateDown(): Unit
      def propagateUp(): Unit
      def build() : Unit
    }

ただし、この API はまだ新しいため、変更される可能性があります。

Builder
============

パイプラインのハードウェアを生成するには、パイプラインで使用されるすべてのリンクのリストを提供する必要があります。

.. code-block:: scala

      // パイプライン用の3つのノードを定義しましょう
      val n0, n1, n2 = Node()

      // これらのノードをシンプルなレジスタで接続しましょう
      val s01 = StageLink(n0, n1)
      val s12 = StageLink(n1, n2)

      // ビルダーに必要なすべてのハードウェアを生成するように依頼しましょう
      Builder(s01, s12)


また、自分自身を助けるためにインスタンス化できる "all in one" ビルダーのセットもあります。

たとえば、NodesBuilder クラスがあり、順次段階的なパイプラインを作成するために使用できます：

.. code-block:: scala
  
      val builder = new NodesBuilder()

      // いくつかのノードを定義しましょう
      val n0, n1, n2 = new builder.Node

      // レジスタを使用してこれらのノードを接続し、関連するハードウェアを生成しましょう
      builder.genStagedPipeline()

組み合わせ可能性
========================

この API の良いところは、複数の並列処理を持つパイプラインを簡単に構築できることです。
ここで言う "組み合わせる" とは、時には設計する必要のあるパイプラインが並列処理を行う場合があるということです。

4 組の数値で浮動小数点乗算を行う必要があると想像してみてください（後でそれらを合計するため）。
これらの 4組の数値が単一のデータストリームで同時に提供される場合、
それらをすべて別々のパイプラインで乗算するのではなく、同じパイプラインで並列に処理したいです。

以下の例は、複数のレーンで並列に処理するパイプラインを構成するパターンを示しています。


.. code-block:: scala

    // この領域では、入力値を受け取り、3つのステージで+1 +1 +1を行います。
    // 無駄なことをしているとわかっていますが、代わりに3つのステージで2つの数値の乗算を行うとしましょう（FMaxの理由で）
    class Plus3(INPUT: Payload[UInt], stage1: Node, stage2: Node, stage3: Node) extends Area {
      val ONE = stage1.insert(stage1(INPUT) + 1)
      val TWO = stage2.insert(stage2(ONE) + 1)
      val THREE = stage3.insert(stage3(TWO) + 1)
    }

    // ストリームを入力として受け取り、
    // 'lanesCount' 個の値を並列に処理し、結果を出力ストリームに配置するコンポーネントを定義します
    class TopLevel(lanesCount : Int) extends Component {
      val io = new Bundle{
        val up = slave Stream(Vec.fill(lanesCount)(UInt(16 bits))) 
        val down = master Stream(Vec.fill(lanesCount)(UInt(16 bits)))
      }

      // パイプライン用の3つのノードを定義しましょう
      val n0, n1, n2 = Node()

      // これらのノードをシンプルなレジスタで接続しましょう
      val s01 = StageLink(n0, n1)
      val s12 = StageLink(n1, n2)

      // io.up を n0 にバインドしましょう
      n0.arbitrateFrom(io.up)
      val LANES_INPUT = io.up.payload.map(n0.insert(_))

      // 使用可能な "再利用可能な" Plus3 エリアを使用して、各処理レーンを生成します
      val lanes = for(i <- 0 until lanesCount) yield new Plus3(LANES_INPUT(i), n0, n1, n2)

      // n2 を io.down にバインドしましょう
      n2.arbitrateTo(io.down)
      for(i <- 0 until lanesCount) io.down.payload(i) := n2(lanes(i).THREE)

      // すべての必要なハードウェアを生成するようにビルダーに依頼しましょう
      Builder(s01, s12)
    }

これにより、次のようなデータパスが生成されます (lanesCount = 2 の場合)、アービトレーションは表示されません：

.. image:: /asset/image/pipeline/composable_lanes.png
   :scale: 70 %


リタイミング／可変長
================================================

時には、パイプラインを設計したいと思うけれども、クリティカル・パスがどこにあるかや、
ステージ間の適切なバランスが何かを正確に知らないことがあります。
そして、自動リタイミングで合成ツールが十分な仕事をすることに頼ることができないことがよくあります。

だから、パイプラインのロジックを簡単に移動する方法が必要です。

このパイプライン API でこれがどのように行われるかを以下に示します：

.. code-block:: scala
    
    // RGB値の入力ストリームを受け取り、
    // (~(R + G + B)) * 0xEE を処理し、
    // その結果を出力ストリームに提供するコンポーネントを定義します
    class RgbToSomething(addAt : Int,
                         invAt : Int,
                         mulAt : Int,
                         resultAt : Int) extends Component {

      val io = new Bundle {
        val up = slave Stream(spinal.lib.graphic.Rgb(8, 8, 8))
        val down = master Stream (UInt(16 bits))
      }

      // パイプライン用のノードを定義しましょう
      val nodes = Array.fill(resultAt+1)(Node())

      // パイプラインのどの部分にどのノードを使用するかを指定しましょう
      val insertNode = nodes(0)
      val addNode = nodes(addAt)
      val invNode = nodes(invAt)
      val mulNode = nodes(mulAt)
      val resultNode = nodes(resultAt)

      // io.up ストリームをパイプラインにフィードするハードウェアを定義します
      val inserter = new insertNode.Area {
        arbitrateFrom(io.up)
        val RGB = insert(io.up.payload)
      }

      // 色の r g b 値を合計します
      val adder = new addNode.Area {
        val SUM = insert(inserter.RGB.r + inserter.RGB.g + inserter.RGB.b)
      }

      // RGB の合計のビットを反転します
      val inverter = new invNode.Area {
        val INV = insert(~adder.SUM)
      }

      // 反転されたビットを 0xEE で乗算します
      val multiplier = new mulNode.Area {
        val MUL = insert(inverter.INV*0xEE)
      }

      // パイプラインの末尾を io.down ストリームに接続します
      val resulter = new resultNode.Area {
        arbitrateTo(io.down)
        io.down.payload := multiplier.MUL
      }

      // これらのノードをシンプルなレジスタで順次接続します
      val links = for (i <- 0 to resultAt - 1) yield StageLink(nodes(i), nodes(i + 1))

      // ビルダーに必要なすべてのハードウェアを生成するように依頼します
      Builder(links)
    }

その後、次のようにしてこのコンポーネントを生成すると：

.. code-block:: scala
    
      SpinalVerilog(
        new RgbToSomething(
          addAt    = 0,
          invAt    = 1,
          mulAt    = 2,
          resultAt = 3
        )
      )

あなたは、3層のフリップフロップによって分離された 4つのステージを持ち、処理を行うことになります：

.. image:: /asset/image/pipeline/rgbToSomething.png
   :scale: 70 %


生成されたハードウェア Verilog は、かなりクリーンです（少なくとも私の基準では:P）：

.. code-block:: verilog

    // Generator : SpinalHDL dev    git head : 1259510dd72697a4f2c388ad22b269d4d2600df7
    // Component : RgbToSomething
    // Git hash  : 63da021a1cd082d22124888dd6c1e5017d4a37b2

    `timescale 1ns/1ps

    module RgbToSomething (
      input  wire          io_up_valid,
      output wire          io_up_ready,
      input  wire [7:0]    io_up_payload_r,
      input  wire [7:0]    io_up_payload_g,
      input  wire [7:0]    io_up_payload_b,
      output wire          io_down_valid,
      input  wire          io_down_ready,
      output wire [15:0]   io_down_payload,
      input  wire          clk,
      input  wire          reset
    );

      wire       [7:0]    _zz_nodes_0_adder_SUM;
      reg        [15:0]   nodes_3_multiplier_MUL;
      wire       [15:0]   nodes_2_multiplier_MUL;
      reg        [7:0]    nodes_2_inverter_INV;
      wire       [7:0]    nodes_1_inverter_INV;
      reg        [7:0]    nodes_1_adder_SUM;
      wire       [7:0]    nodes_0_adder_SUM;
      wire       [7:0]    nodes_0_inserter_RGB_r;
      wire       [7:0]    nodes_0_inserter_RGB_g;
      wire       [7:0]    nodes_0_inserter_RGB_b;
      wire                nodes_0_valid;
      reg                 nodes_0_ready;
      reg                 nodes_1_valid;
      reg                 nodes_1_ready;
      reg                 nodes_2_valid;
      reg                 nodes_2_ready;
      reg                 nodes_3_valid;
      wire                nodes_3_ready;
      wire                when_StageLink_l56;
      wire                when_StageLink_l56_1;
      wire                when_StageLink_l56_2;

      assign _zz_nodes_0_adder_SUM = (nodes_0_inserter_RGB_r + nodes_0_inserter_RGB_g);
      assign nodes_0_valid = io_up_valid;
      assign io_up_ready = nodes_0_ready;
      assign nodes_0_inserter_RGB_r = io_up_payload_r;
      assign nodes_0_inserter_RGB_g = io_up_payload_g;
      assign nodes_0_inserter_RGB_b = io_up_payload_b;
      assign nodes_0_adder_SUM = (_zz_nodes_0_adder_SUM + nodes_0_inserter_RGB_b);
      assign nodes_1_inverter_INV = (~ nodes_1_adder_SUM);
      assign nodes_2_multiplier_MUL = (nodes_2_inverter_INV * 8'hee);
      assign io_down_valid = nodes_3_valid;
      assign nodes_3_ready = io_down_ready;
      assign io_down_payload = nodes_3_multiplier_MUL;
      always @(*) begin
        nodes_0_ready = nodes_1_ready;
        if(when_StageLink_l56) begin
          nodes_0_ready = 1'b1;
        end
      end

      assign when_StageLink_l56 = (! nodes_1_valid);
      always @(*) begin
        nodes_1_ready = nodes_2_ready;
        if(when_StageLink_l56_1) begin
          nodes_1_ready = 1'b1;
        end
      end

      assign when_StageLink_l56_1 = (! nodes_2_valid);
      always @(*) begin
        nodes_2_ready = nodes_3_ready;
        if(when_StageLink_l56_2) begin
          nodes_2_ready = 1'b1;
        end
      end

      assign when_StageLink_l56_2 = (! nodes_3_valid);
      always @(posedge clk or posedge reset) begin
        if(reset) begin
          nodes_1_valid <= 1'b0;
          nodes_2_valid <= 1'b0;
          nodes_3_valid <= 1'b0;
        end else begin
          if(nodes_0_ready) begin
            nodes_1_valid <= nodes_0_valid;
          end
          if(nodes_1_ready) begin
            nodes_2_valid <= nodes_1_valid;
          end
          if(nodes_2_ready) begin
            nodes_3_valid <= nodes_2_valid;
          end
        end
      end

      always @(posedge clk) begin
        if(nodes_0_ready) begin
          nodes_1_adder_SUM <= nodes_0_adder_SUM;
        end
        if(nodes_1_ready) begin
          nodes_2_inverter_INV <= nodes_1_inverter_INV;
        end
        if(nodes_2_ready) begin
          nodes_3_multiplier_MUL <= nodes_2_multiplier_MUL;
        end
      end


    endmodule

また、ステージの数や処理を行う場所を簡単に調整することができます。
例えば、反転ハードウェアをアダーと同じステージに移動したい場合、次のようにします：

.. code-block:: scala
    
      SpinalVerilog(
        new RgbToSomething(
          addAt    = 0,
          invAt    = 0,
          mulAt    = 1,
          resultAt = 2
        )
      )

その後、出力レジスタステージを削除したい場合は：

.. code-block:: scala
    
      SpinalVerilog(
        new RgbToSomething(
          addAt    = 0,
          invAt    = 0,
          mulAt    = 1,
          resultAt = 1
        )
      )



シンプルなCPUの例
================================================

下は、シンプルで愚直な8ビットCPUの例です：

- 3つのステージ（フェッチ、デコード、実行）
- 組み込みのフェッチメモリ
- 追加／ジャンプ／LED／遅延命令

.. code-block:: scala

  class Cpu extends Component {
    val fetch, decode, execute = CtrlLink()
    val f2d = StageLink(fetch.down, decode.up)
    val d2e = StageLink(decode.down, execute.up)

    val PC = Payload(UInt(8 bits))
    val INSTRUCTION = Payload(Bits(16 bits))

    val led = out(Reg(Bits(8 bits))) init(0)

    val fetcher = new fetch.Area{
      val pcReg = Reg(PC) init (0)
      up(PC) := pcReg
      up.valid := True
      when(up.isFiring) {
        pcReg := PC + 1
      }

      val mem = Mem.fill(256)(INSTRUCTION).simPublic
      INSTRUCTION := mem.readAsync(PC)
    }

    val decoder = new decode.Area{
      val opcode = INSTRUCTION(7 downto 0)
      val IS_ADD   = insert(opcode === 0x1)
      val IS_JUMP  = insert(opcode === 0x2)
      val IS_LED   = insert(opcode === 0x3)
      val IS_DELAY = insert(opcode === 0x4)
    }


    val alu = new execute.Area{
      val regfile = Reg(UInt(8 bits)) init(0)
      
      val flush = False
      for (stage <- List(fetch, decode)) {
        stage.throwWhen(flush, usingReady = true)
      }

      val delayCounter = Reg(UInt(8 bits)) init (0)

      when(isValid) {
        when(decoder.IS_ADD) {
          regfile := regfile + U(INSTRUCTION(15 downto 8))
        }
        when(decoder.IS_JUMP) {
          flush := True
          fetcher.pcReg := U(INSTRUCTION(15 downto 8))
        }
        when(decoder.IS_LED) {
          led := B(regfile)
        }
        when(decoder.IS_DELAY) {
          delayCounter := delayCounter + 1
          when(delayCounter === U(INSTRUCTION(15 downto 8))) {
            delayCounter := 0
          } otherwise {
            execute.haltIt()
          }
        }
      }
    }

    Builder(fetch, decode, execute, f2d, d2e)
  }

以下は、LED が数え上がるループを実装する単純なテストベンチです。

.. code-block:: scala

  SimConfig.withFstWave.compile(new Cpu).doSim(seed = 2){ dut =>
    def nop() = BigInt(0)
    def add(value: Int) = BigInt(1 | (value << 8))
    def jump(target: Int) = BigInt(2 | (target << 8))
    def led() = BigInt(3)
    def delay(cycles: Int) = BigInt(4 | (cycles << 8))
    val mem = dut.fetcher.mem
    mem.setBigInt(0, nop())
    mem.setBigInt(1, nop())
    mem.setBigInt(2, add(0x1))
    mem.setBigInt(3, led())
    mem.setBigInt(4, delay(16))
    mem.setBigInt(5, jump(0x2))

    dut.clockDomain.forkStimulus(10)
    dut.clockDomain.waitSampling(100)
  }



