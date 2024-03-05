.. role:: raw-html-m2r(raw)
   :format: html

Utils
=====

いくつかのユーティリティは、:ref:`spinal.core <utils>` にも存在します。

無状態のユーティリティ
----------------------------

.. list-table::
   :header-rows: 1
   :widths: 2 1 5

   * - 構文
     - 戻り値
     - 説明
   * - toGray(x : UInt)
     - Bits
     - ``x`` (UInt) からグレイ値に変換された値を返します
   * - fromGray(x : Bits)
     - UInt
     - ``x`` (gray) から UInt 値に変換された値を返します
   * - Reverse(x : T)
     - T
     - すべてのビットを反転します (lsb + n -> msb - n)
   * - | OHToUInt(x : Seq[Bool])
       | OHToUInt(x : BitVector)
     - UInt
     - ``x`` の単一ビットがセットされたインデックス（ワンホット）を返します
   * - | CountOne(x : Seq[Bool])
       | CountOne(x : BitVector)
     - UInt
     - ``x`` でセットされたビット数を返します
   * - | MajorityVote(x : Seq[Bool])
       | MajorityVote(x : BitVector)
     - Bool
     - セットされたビット数が x.size / 2 よりも大きい場合、True を返します
   * - EndiannessSwap(that: T[, base:BitCount])
     - T
     - ビッグエンディアン <-> リトルエンディアン
   * - OHMasking.first(x : Bits)
     - Bits
     - x にマスクを適用して最初のセットビットのみを保持します
   * - OHMasking.last(x : Bits)
     - Bits
     - x にマスクを適用して最後のセットビットのみを保持します
   * - | OHMasking.roundRobin(
       |  requests : Bits,
       |  ohPriority : Bits
       | )
     - Bits
     - | x にマスクを適用して、 ``requests`` からビットがセットされているものだけを保持します。
       | ``ohPriority`` 位置から ``requests`` を検索し始めます。
       | たとえば、 ``requests`` が "1001" で ``ohPriority`` が "0010" の場合、 ``roundRobin`` 関数は `requests` の2番目のビットから検索を開始し、"1000" を返します。
   * - | MuxOH (
       |   oneHot : IndexedSeq[Bool],
       |   inputs : Iterable[T]
       | )
     - T
     - ``oneHot`` ベクトルに基づいて ``inputs`` から Mux された ``T`` を返します
   * - | PriorityMux (
       |    sel: Seq[Bool],
       |    in:  Seq[T]
       | )
     - T
     - ``sel`` が ``True`` である最初の ``in`` 要素を返します。
   * - | PriorityMux (
       |    in:  Seq[(Bool, T)]
       | )
     - T
     - ``sel`` が ``True`` である最初の ``in`` 要素を返します。


状態を持つユーティリティ
---------------------------

.. list-table::
   :header-rows: 1
   :widths: 3 1 5

   * - 構文
     - 戻り値
     - 説明
   * - Delay(that: T, cycleCount: Int)
     - T
     - ``that`` を ``cycleCount`` サイクル遅延させたものを返します
   * - History(that: T, length: Int[,when : Bool])
     - List[T]
     - | ``length`` 要素の Vec を返します 
       | 最初の要素は ``that``\ であり、最後の要素は ``that`` を ``length``\ -1\ サイクル遅延させたものです
       | ``when`` がアサートされているときに内部シフトレジスタのサンプルが取得されます
   * - BufferCC(input : T)
     - T
     - 2つのフリップフロップを使用して、入力信号を現在のクロックドメインに同期させたものを返します

Counter
^^^^^^^

Counter ツールは、ハードウェアカウンターを簡単にインスタンス化するために使用できます。

.. list-table::
   :header-rows: 1
   :widths: 1 1

   * - インスタンス化構文
     - ノート
   * - ``Counter(start: BigInt, end: BigInt[, inc : Bool])``
     - 
   * - ``Counter(range : Range[, inc : Bool])``
     - ``x to y`` や ``x until y`` の構文と互換性があります
   * - ``Counter(stateCount: BigInt[, inc : Bool])``
     - ゼロから開始し、 ``stateCount - 1`` で終了します
   * - ``Counter(bitCount: BitCount[, inc : Bool])``
     - ゼロから開始し、 ``(1 << bitCount) - 1`` で終了します

カウンターは、メソッドによって制御され、ワイヤで読み取ることができます:

.. code-block:: scala

   val counter = Counter(2 to 9) // Creates a counter of 8 states (2 to 9)
   // メソッド
   counter.clear()               // カウンターをリセットします
   counter.increment()           // カウンターをインクリメントします
   // ワイヤ
   counter.value                 // 現在の値
   counter.valueNext             // 次の値
   counter.willOverflow          // カウンターが今サイクルでオーバーフローする場合は True
   counter.willOverflowIfInc     // インクリメントを行うと、カウンターが今サイクルでオーバーフローする場合は True
   // Cast
   when(counter === 5){ ... }    // カウンターは、暗黙的に現在の値にキャストされます

``Counter`` がオーバーフロー（終了値に達した）した場合、次のサイクルで開始値に戻ります。

.. note::
   現在、上向きのカウンターのみがサポートされています。

``CounterFreeRun`` は常に実行されるカウンターを構築します: ``CounterFreeRun(stateCount: BigInt)``。


Timeout
^^^^^^^

Timeout ツールは、ハードウェアのタイムアウトを簡単にインスタンス化するために使用できます。

.. list-table::
   :header-rows: 1
   :widths: 1 1

   * - インスタンス化構文
     - ノート
   * - Timeout(cycles : BigInt)
     - ``cycles`` クロック後にチックします
   * - Timeout(time : TimeNumber)
     - ``time`` の期間後にチックします
   * - Timeout(frequency : HertzNumber)
     - ``frequency`` のレートでチックします

Counter ツールと一緒に使用できる異なる構文の例があります

.. code-block:: scala

   val timeout = Timeout(10 ms)  // 10 ms後にチックするタイムアウト
   when(timeout) {               // タイムアウトがチックしたかどうかをチェック
       timeout.clear()           // タイムアウトにクリアフラグを要求します
   }

.. note::
   時間または周波数の設定で ``Timeout`` をインスタンス化する場合、暗黙の ``ClockDomain`` には周波数設定が必要です。


ResetCtrl
^^^^^^^^^

ResetCtrl はリセットを管理するためのいくつかのユーティリティを提供します。

asyncAssertSyncDeassert
~~~~~~~~~~~~~~~~~~~~~~~

非同期リセットをフィルタリングするには、非同期にアサートされた同期的に非アサートされるロジックを使用します。
これを行うには、 ``ResetCtrl.asyncAssertSyncDeassert`` 関数を使用します。この関数はフィルタリングされた値を返します。

.. list-table::
   :header-rows: 1
   :widths: 1 1 4

   * - 引数名
     - タイプ
     - 説明
   * - input
     - Bool
     - フィルタリングする必要があるシグナル
   * - clockDomain
     - ClockDomain
     - フィルタリングされた値を使用する ClockDomain 
   * - inputPolarity
     - Polarity
     - HIGH/LOW（デフォルト=HIGH）
   * - outputPolarity
     - Polarity
     - HIGH/LOW（デフォルト=clockDomain.config.resetActiveLevel）
   * - bufferDepth
     - Int
     - メタスタビリティを回避するために使用されるレジスタ段の数（デフォルト=2）

ツールの ``ResetCtrl.asyncAssertSyncDeassertDrive`` バージョンもあり、
これは直接 ``clockDomain`` リセットにフィルタリングされた値を割り当てます。

特別なユーティリティ
-----------------------

.. list-table::
   :header-rows: 1
   :widths: 3 1 5

   * - 構文
     - 戻り値
     - 説明
   * - LatencyAnalysis(paths : Node*)
     - Int
     - | 最初のノードから最後のノードまでのすべてのパスを通る最短の経路を、
       | サイクル単位で返します。


