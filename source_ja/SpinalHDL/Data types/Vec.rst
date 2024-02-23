.. role:: raw-html-m2r(raw)
   :format: html

.. _Vec:

Vec
===

説明
^^^^^^^^^^^

ベクトル(``Vec``)は、単一の名前の下に、インデックス付きのシグナル（SpinalHDLの基本型のいずれか）のグループを定義する複合型です。


宣言
^^^^^^^^^^^

ベクトルを宣言する構文は以下の通りです：

.. list-table::
   :header-rows: 1
   :widths: 1 2

   * - 宣言
     - 説明
   * - Vec.fill(size: Int)(type: Data)
     - ``Data``型の ``size``要素のベクトルを作成します。
   * - Vec(x, y, ...)
     - | 提供された要素を指すインデックスを持つベクトルを作成します。 
       | 新しいハードウェア信号を作成しません。
       | このコンストラクタは、要素の幅が混在している場合に対応しています。


例
~~~~~~~~

.. code-block:: scala

   // 2つの符号付き整数のベクトルを作成します。
   val myVecOfSInt = Vec.fill(2)(SInt(8 bits))
   myVecOfSInt(0) := 2                   // インデックス0を埋めるための代入
   myVecOfSInt(1) := myVecOfSInt(0) + 3  // インデックス1を埋めるための代入

   // 異なる3つのタイプの要素のベクトルを作成します。
   val myVecOfMixedUInt = Vec(UInt(3 bits), UInt(5 bits), UInt(8 bits))

   val x, y, z = UInt(8 bits)
   val myVecOf_xyz_ref = Vec(x, y, z)

   // ベクトルを反復処理します。
   for(element <- myVecOf_xyz_ref) {
     element := 0   // x、y、zに値0を割り当てます。
   }

   // Map on vector
   myVecOfMixedUInt.map(_ := 0) // すべての要素に値0を割り当てます。

   // ベクトルの最初の要素に3を割り当てます。
   myVecOf_xyz_ref(1) := 3

演算子
^^^^^^^^^

ベクトル (``Vec``)型で利用可能な演算子は以下の通りです：

比較
~~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 返り値の型
   * - x === y
     - 等号
     - Bool
   * - x =/= y
     - 不等号
     - Bool


.. code-block:: scala

   // 2つの符号付き整数のベクトルを作成します。
   val vec2 = Vec.fill(2)(SInt(8 bits))
   val vec1 = Vec.fill(2)(SInt(8 bits))

   myBool := vec2 === vec1  // すべての要素を比較します。
   // 次のものと同等です::
   //myBool := vec2(0) === vec1(0) && vec2(1) === vec1(1)

型変換
~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 返り値
   * - x.asBits
     - Bitsへのバイナリキャスト
     - Bits(w(x) bits)


.. code-block:: scala

   // 2つの符号付き整数のベクトルを作成します。
   val vec1 = Vec.fill(2)(SInt(8 bits))

   myBits_16bits := vec1.asBits

その他
~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 2 5 1

   * - 演算子
     - 説明
     - 返り値
   * - x.getBitsWidth
     - Vecの完全なサイズを返します。
     - Int


.. code-block:: scala

   // 2つの符号付き整数のベクトルを作成します。
   val vec1 = Vec.fill(2)(SInt(8 bits))

   println(widthOf(vec1)) // 16


ライブラリの補助関数
~~~~~~~~~~~~~~~~~~~~

.. note::
    これらの関数をスコープに入れるには、 ``import spinal.lib._``をインポートする必要があります。

.. list-table::
   :header-rows: 1
   :widths: 3 4 1

   * - 演算子
     - 説明
     - 戻り値
   * - x.sCount(condition: T => Bool)
     - Vec内の指定された条件に一致する出現回数を数えます。
     - UInt
   * - x.sCount(value: T)
     - Vec内の値の出現回数を数えます。
     - UInt
   * - x.sExists(condition: T => Bool)
     - Vec内に一致する条件があるかどうかをチェックします。
     - Bool
   * - x.sContains(value: T)
     - Vec 内に特定の値を持つ要素が存在しているかどうかをチェックします。
     - Bool
   * - x.sFindFirst(condition: T => Bool)
     - 与えられた条件に合致する Vec 内の最初の要素を見つけて、その要素のインデックスを返します。
     - UInt
   * - x.reduceBalancedTree(op: (T, T) => T)
     - バランス化された集約関数。これは、結果回路の深さを最小化するために使用されます。op は交換法則と結合法則を満たす必要があります。
     - T
   * - x.shuffle(indexMapping: Int => Int)
     - Vec をシャッフルするには、古いインデックスを新しいインデックスにマッピングする関数を使ってください。
     - Vec[T]

.. code-block:: scala

    import spinal.lib._

    // 4 つの符号なし整数を含むベクトルを作成
    val vec1 = Vec.fill(4)(UInt(8 bits))

    // ...ベクトルは実際にはどこかに割り当てられています

    val c1: UInt = vec1.sCount(_ < 128) // vec で 128 より小さい値はいくつありますか
    val c2: UInt = vec1.sCount(0) // vec でゼロに等しい値はいくつありますか

    val b1: Bool = vec1.sExists(_ > 250) // 250より大きい要素はありますか
    val b2: Bool = vec1.sContains(0) // vec にゼロはありますか

    val u1: UInt = vec1.sFindFirst(_ < 10) // 10 より小さい最初の要素のインデックスを取得します
    val u2: UInt = vec1.reduceBalancedTree(_ + _) // すべての要素を合計する


.. note::
    sXXX プレフィックスは、引数としてラムダ関数を受け取る同じ名前の Scala 関数との混同を避けるために使用されます。
