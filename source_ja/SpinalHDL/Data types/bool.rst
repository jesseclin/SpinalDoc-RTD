.. _Bool:

Bool
====

説明
^^^^^^^^^^^

``Bool`` 型は、ハードウェア設計で使用されるブール値 (True または False) または単一のビット/ワイヤに対応します。
名前は似ていますが、ハードウェアを記述するのではなく、Scala ジェネレーター コードの真理値を記述する Scala の `Boolean` 型と混同しないでください。

理解すべき重要な概念と経験則は、Scala コード内でエラボレーション時の HDL コード生成の意思決定が行われる場所で Scala の `Boolean` 型が使用されるということです。
通常のプログラムと同様に、プログラムの実行時に SpinalHDL である Scala プログラムの実行に影響を与えます。
HDL コード生成を実行するために実行されます。

したがって、Scala の `Boolean` の値はハードウェアからは観察できません。
これは、HDL コード生成時に SpinalHDL プログラムに事前に存在するだけであるためです。

設計にこれが必要になる可能性があるシナリオでは、たとえば、
Scala からハードウェア設計に値 (パラメーター化された定数入力として機能する可能性があります) を入力する場合は、
コンストラクター `Bool(value:Boolean)` を使用してそれを Bool に型変換できます。

同様に、SpinalHDL の `Bool` の値はコード生成時には確認できません。
表示および操作できるのは、 `wire` に関する HDL 構造と、それがどのように (モジュール/コンポーネントを介して) ルーティングされ、
駆動 (ソース) され、接続 (サンク) されるかです。

代入演算子 `:=` の信号方向は SpinalHDL によって管理されます。
代入演算子 `:=` の左側または右側で Bool インスタンスを使用すると、
それが特定の代入のソース (状態を提供する) かシンク (状態をキャプチャする) かが決まります。

代入演算子は複数回使用できます。これは通常のことです。
信号線がソースとして機能し (HDL 状態を駆動する値を提供する)、他の HDL 構造の複数の入力を接続して駆動できるようになります。 
Bool インスタンスがソースとして使用される場合、シンク (状態をキャプチャ) として使用される場合とは異なり、
代入ステートメントが Scala で表示または実行される順序は重要ではありません。

複数の代入演算子が Bool を駆動すると、最後の代入ステートメントが優先されます。
これは、Scala コードの最後で実行されるからです。

この問題は、ハードウェア内の Bool に新しい状態を割り当てるときに影響する可能性があります。
正しい優先順位をハードウェア設計に反映させるには、
SpinalHDL Scala コードのレイアウトと順序に注意する必要があります。

Scala/SpinalHDL と関連付けて概念を理解するのに役立つかもしれません。
ネットリスト内の HDL `net` への参照としての `Bool` インスタンス。代入 `:=` 演算子が HDL 構造を同じネットに接続していること。

Scala/SpinalHDL の `Bool` は、回路網表 (ネットリスト) 内の HDL `net` を指す参照だと思ってみると、
より理解しやすいかもしれません。 `:=` (代入) 演算子は、複数の HDL 命令を同じ net に結びつけているイメージです。


宣言
^^^^^^^^^^^

ブール値を宣言する構文は次のとおりです ([] の間はすべてオプションです)。


.. list-table::
   :header-rows: 1
   :widths: 1 3 1

   * - 構文
     - 説明
     - リターン
   * - Bool()
     - Bool を作成する
     - Bool
   * - True
     - ``true`` が割り当てられた Bool を作成します
     - Bool
   * - False
     - ``false`` が割り当てられた Bool を作成します
     - Bool
   * - Bool(value: Boolean)
     - Scala Boolean 型 (true、false) の値を割り当てた Bool を作成します。
       これは明示的に ``True`` または ``False`` に変換されます。
     - Bool


.. code-block:: scala

   val myBool_1 = Bool()        // Bool を作成する
   myBool_1 := False            // := は代入演算子 (verilog <= など)

   val myBool_2 = False         // 上記のコードと同等

   val myBool_3 = Bool(5 > 12)  // Scala Boolean を使用して Bool を作成する

演算子
^^^^^^^^^

次の演算子は ``Bool`` 型で使用できます。

.. note:

   論理式 ``x`` と ``y`` の両辺は Bool 型である必要があります。

Logic
~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 戻り値の型
   * - !x
     - 論理 NOT
     - Bool
   * - | x && y
       | x & y
     - 論理 AND
     - Bool
   * - | x || y
       | x | y
     - 論理 OR
     - Bool
   * - x ^ y
     - 論理 XOR
     - Bool
   * - ~x
     - 論理 NOT
     - Bool
   * - x.set[()]
     - x を True に設定します
     - Unit (none)
   * - x.clear[()]
     - x を False に設定します
     - Unit (none)
   * - x.setWhen(cond)
     - cond が True の場合に x を設定します
     - Bool
   * - x.clearWhen(cond)
     - cond が True の場合、x をクリアします
     - Bool
   * - x.riseWhen(cond)
     - x が False で cond が True の場合に x を設定します
     - Bool
   * - x.fallWhen(cond)
     - x が True で cond が True の場合、x をクリアします
     - Bool


.. code-block:: scala

   val a, b, c = Bool()
   val res = (!a & b) ^ c   // ((NOT a) AND b) XOR c

   val d = False
   when(cond) {
     d.set()                // d := True と同等
   }

   val e = False
   e.setWhen(cond)          // when(cond) { d := True } と同等

   val f = RegInit(False) fallWhen(ack) setWhen(req)
   /** 以下と同等
    * when(f && ack) { f := False }
    * when(req) { f := True }
    * or
    * f := req || (f && !ack)
    */

  // 課題の順序に注意してください!最後の人が勝ちます
  val g = RegInit(False) setWhen(req) fallWhen(ack)
  // g := ((!g) && req) || (g && !ack) と同等

エッジ検出
~~~~~~~~~~~~~~

すべてのエッジ検出関数は、:ref:`RegNext <regnext>` を通して追加のレジスタを生成して、
対象の ``Bool`` の遅延値を取得します。

この機能は、Dフリップフロップを再構成して別の CLKソースを使用するものではなく、
2つの Dフリップフロップを直列に接続することでエッジ検出を実現します。
両方の CLKピンはデフォルトのClockDomainからクロック信号を受け取り、
出力される Qの状態に基づいて組み合わせ回路でエッジ検出を行います。

.. list-table::
   :header-rows: 1
   :widths: 2 5 1

   * - 演算子
     - 説明
     - 戻り値の型
   * - x.edge[()]
     - x の状態が変化すると True を返します
     - Bool
   * - x.edge(initAt: Bool)
     - x.edge と同じですが、値がリセットされます
     - Bool
   * - x.rise[()]
     - x が最後のサイクルで Low であり、現在 High になっている場合に True を返します。
     - Bool
   * - x.rise(initAt: Bool)
     - x.rise と同じですが、値がリセットされます
     - Bool
   * - x.fall[()]
     - Rx が最後のサイクルで High であり、現在 Low の場合に True を返します
     - Bool
   * - x.fall(initAt: Bool)
     - x.fall と同じですが、値がリセットされます
     - Bool
   * - x.edges[()]
     - バンドルを返す (rise, fall, toggle)
     - BoolEdges
   * - x.edges(initAt: Bool)
     - x.edges と同じですが、値がリセットされます
     - BoolEdges
   * - x.toggle[()]
     - すべてのエッジで True を返します
     - Bool


.. code-block:: scala

   when(myBool_1.rise(False)) {
       // 立ち上がりエッジが検出されたときに何かを行う
   } 


   val edgeBundle = myBool_2.edges(False)
   when(edgeBundle.rise) {
       // 立ち上がりエッジが検出されたときに何かを行う
   }
   when(edgeBundle.fall) {
       // 立ち下がりエッジが検出されたときに何かを行う
   }
   when(edgeBundle.toggle) {
       // 各エッジで何かをする
   }

比較
~~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 戻り値の型
   * - x === y
     - 同等
     - Bool
   * - x =/= y
     - 不等
     - Bool


.. code-block:: scala

   when(myBool) { // when(myBool === True) と同等
       // myBool が True のときに何かを行う
   }

   when(!myBool) { // when(myBool === False) と同等
       // myBool が False のときに何かを行う
   }

型キャスト
~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - リターン
   * - x.asBits
     - バイナリをビットにキャスト
     - Bits(1 bit)
   * - x.asUInt
     - UInt へのバイナリ キャスト
     - UInt(1 bit)
   * - x.asSInt
     - SInt へのバイナリ キャスト
     - SInt(1 bit)
   * - x.asUInt(bitCount)
     - バイナリを UInt にキャストしてサイズ変更し、Bool 値を LSB に入れてゼロを埋め込みます。
     - UInt(bitCount bits)
   * - x.asBits(bitCount)
     - バイナリをビットにキャストしてサイズ変更し、Bool 値を LSB に入れてゼロを埋め込みます。
     - Bits(bitCount bits)


.. code-block:: scala

   // SInt 値にキャリーを加算する
   val carry = Bool()
   val res = mySInt + carry.asSInt

その他
~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - リターン
   * - x ## y
     - 連結する, x->high, y->low
     - Bits(w(x) + w(y) bits)
   * - x #* n
     - x が n 回繰り返す
     - Bits(n bits)


.. code-block:: scala

   val a, b, c = Bool()

   // 3 つの Bool を 1 つの Bits (3 ビット) 型に連結します。
   val myBits = a ## b ## c


MaskedBoolean
~~~~~~~~~~~~~

マスクされたブール値ではドントケア値が許可されます。
これらは通常、単独で使用されるのではなく、:ref:`MaskedLiteral <maskedliteral>` を通じて使用されます。

.. code-block:: scala

  // 最初の引数: Scala のブール値
  // Scala のブール値として表現される `?` を気にしますか
  val masked = new MaskedBoolean(true, false)
