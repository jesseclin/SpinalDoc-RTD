.. _Bits:

Bits
====

``Bits`` 型は、算術的な意味をまったく伝えないビットのベクトルです。

宣言
-----------

ビット ベクトルを宣言する構文は次のとおりです ([] の間はすべてオプションです)。

.. list-table::
   :header-rows: 1
   :widths: 5 10

   * - 構文
     - 説明
   * - Bits [()]
     - ビットを作成します。ビット数は構築後の最も幅の広い代入ステートメントから推測されます。
   * - Bits(x bits)
     - x ビットでビットを作成
   * - | B(value: Int[, x bits])
       | B(value: BigInt[, x bits])
     - x ビットに 'value' を割り当ててビットを作成する
   * - B"[[size']base]value"
     - 'value'を割り当てたビットを作成します (base: 'h'、'd'、'o'、'b')
   * - B([x bits,] elements: Element*)
     - :ref:`elements <element>` で指定された値が割り当てられたビットを作成します


.. code-block:: scala

   val myBits1 = Bits(32 bits)   
   val myBits2 = B(25, 8 bits)
   val myBits3 = B"8'xFF"   // ベースは次のとおりです:
                            //               x,h (base 16)                         
                            //               d   (base 10)
                            //               o   (base 8)
                            //               b   (base 2)    
   val myBits4 = B"1001_0011"  // _ 読みやすさのために使用できます

   // すべて 1 のビット ("11111111")
   val myBits5 = B(8 bits, default -> True)

   // いくつかの要素を介して "10111000" で初期化します
   val myBits6 = B(8 bits, (7 downto 5) -> B"101", 4 -> true, 3 -> True, default -> false)

   // "10000000" (割り当ての目的で、B を省略できます)
   val myBits7 = Bits(8 bits)
   myBits7 := (7 -> true, default -> false) 

``Bits`` の幅を推論するときは、割り当てられた値のサイズが信号の最終的なサイズと一致する必要があります:

.. code-block:: scala

   // 宣言
   val myBits = Bits()     // サイズは最も広い割り当てから推測されます
   // ....
   // WIDTH MISMATCH エラーを防ぐために定数として .resize が必要
   // 幅が以下の割り当てから推測されるサイズと一致しません
   myBits := B("1010").resized  // ビット (4 ビット) をビット (6 ビット) に自動拡張します
   when(condxMaybe) {
     // myBits に対して Bits(6 ビット) が推論されます。これは最も広い割り当てです。
     myBits := B("110000")
   }

演算子
---------

``Bits`` タイプでは次の演算子が使用できます

ロジック
^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 2 3 2

   * - 演算子
     - 説明
     - 戻り値の型
   * - ~x
     - ビットごとの NOT
     - Bits(w(x) bits)
   * - x & y
     - ビットごとの AND
     - Bits(w(xy) bits)
   * - x | y
     - ビットごとの OR
     - Bits(w(xy) bits)
   * - x ^ y
     - ビットごとの XOR
     - Bits(w(xy) bits)
   * - x.xorR
     - x のすべてのビットの XOR
     - Bool
   * - x.orR
     - x のすべてのビットの OR
     - Bool
   * - x.andR
     - x のすべてのビットの AND
     - Bool
   * - | y = 1 // Int
       | x \>\> y
     - | 論理右シフト, y: Int
       | 結果として幅が狭くなる可能性があります
     - Bits(w(x) - y bits)
   * - | y = U(1) // UInt
       | x \>\> y
     - | 論理右シフト, y: UInt
       | 結果は同じ幅になります
     - Bits(w(x) bits)
   * - | y = 1 // Int
       | x \<\< y
     - | 論理左シフト, y: Int
       | 結果として幅が広がる可能性があります
     - Bits(w(x) + y bits)
   * - | y = U(1) // UInt
       | x \<\< y
     - | 論理左シフト, y: UInt
       | 結果として幅が広がる可能性があります
     - Bits(w(x) + max(y) bits)
   * - x \|\>\> y
     - | 論理右シフト, y: Int/UInt
       | 結果は同じ幅になります
     - Bits(w(x) bits)
   * - x \|\<\< y
     - | 論理左シフト, y: Int/UInt
       | 結果は同じ幅になります
     - Bits(w(x) bits)
   * - x.rotateLeft(y)
     - | 論理的な左回転, y: UInt/Int
       | 結果は同じ幅になります
     - Bits(w(x) bits)
   * - x.rotateRight(y)
     - | 論理右回転, y: UInt/Int
       | 結果は同じ幅になります
     - Bits(w(x) bits)
   * - x.clearAll[()]
     - すべてのビットをクリア
     - *modifies x*
   * - x.setAll[()]
     - すべてのビットを設定します
     - *modifies x*
   * - x.setAllTo(value: Boolean)
     - すべてのビットを指定された Boolean 値に設定します
     - *modifies x*
   * - x.setAllTo(value: Bool)
     - すべてのビットを指定された Bool 値に設定します
     - *modifies x*

.. code-block:: scala

   // ビット演算子
   val a, b, c = Bits(32 bits)
   c := ~(a & b) // Inverse(a AND b)

   val all_1 = a.andR // すべてのビットが 1 に等しいことを確認します

   // 論理シフト
   val bits_10bits = bits_8bits << 2  // 左にシフト (結果は 10 ビット)
   val shift_8bits = bits_8bits |<< 2 // 左にシフト (結果は 8 ビット)

   // 論理回転
   val myBits = bits_8bits.rotateLeft(3) // 左ビット回転

   // 設定/クリア
   val a = B"8'x42"
   when(cond) {
     a.setAll() // cond が True の場合、すべてのビットを True に設定します
   }

Comparison
^^^^^^^^^^

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

   when(myBits === 3) {
     // ...
   }

   val notMySpecialValue = myBits_32 =/= B"32'x44332211"

型キャスト
^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 戻り値の型
   * - x.asBits
     - バイナリをビットにキャスト
     - Bits(w(x) bits)
   * - x.asUInt
     - UInt へのバイナリ キャスト
     - UInt(w(x) bits)
   * - x.asSInt
     - SInt へのバイナリ キャスト
     - SInt(w(x) bits)
   * - x.asBools
     - ブール値の配列にキャストします
     - Vec(Bool(), w(x))
   * - x.asBool
     - :code:`x` の LSB を抽出します
     - Bool(x.lsb)
   * - B(x: T)
     - データをビットにキャストする
     - Bits(w(x) bits)



``Bool``、 ``UInt``、または ``SInt`` を ``Bits`` にキャストするには、 ``B(something)`` または ``B(something[, x bits])`` を使用できます:

To cast a ``Bool``, ``UInt`` or an ``SInt`` into a ``Bits``, you can use ``B(something)`` or ``B(something[, x bits])``:

.. code-block:: scala

   // SInt にビットをキャストする
   val mySInt = myBits.asSInt

   // ブール値のベクトルを作成する
   val myVec = myBits.asBools

   // SInt をビットにキャストする
   val myBits = B(mySInt)

   // 同じ SInt を Bits にキャストしますが、サイズを 3 ビットに変更します
   //  (必要に応じて拡張/切り捨てられ、LSB が保持されます)
   val myBits = B(mySInt, 3 bits)

ビット抽出
^^^^^^^^^^^^^^

すべてのビット抽出操作を使用して、ビットまたはビットのグループを読み取ることができます。
他の HDL と同様に、抽出演算子を使用して ``Bits`` の一部を割り当てることもできます。

すべてのビット抽出操作を使用して、ビットまたはビットのグループを読み取ることができます。
他の HDL と同様に、書き込むビット範囲を選択するためにも使用できます。


.. list-table::
   :header-rows: 1
   :widths: 2 4 2

   * - 演算子
     - 説明
     - 戻り値の型
   * - x(y: Int)
     - y ビット目の静的ビットアクセス
     - Bool
   * - x(x: UInt)
     - y ビット目の可変ビットアクセス
     - Bool
   * - x(offset: Int, width bits)
     - 固定幅の固定部分選択、オフセットはLSBインデックス
     - Bits(width bits)
   * - x(offset: UInt, width bits)
     - 固定幅の可変部分選択、オフセットはLSBインデックス
     - Bits(width bits)
   * - x(range: Range)
     - ビットの :ref:`range <range>` にアクセスします。例: myBits(4 downto 2)
     - Bits(range.size bits)
   * - x.subdivideIn(y slices, [strict: Boolean])
     - x を y スライスに再分割します、y: Int
     - Vec(Bits(...), y)
   * - x.subdivideIn(y bits, [strict: Boolean])
     - x を y ビットの複数のスライスに再分割します、y: Int
     - Vec(Bits(y bit), ...)
   * - x.msb
     - x の最上位ビット (最高位のインデックス) にアクセスする
     - Bool
   * - x.lsb
     - x の最下位ビット (インデックス 0) にアクセスします
     - Bool


いくつかの基本的な例:

.. code-block:: scala

   // インデックス 4 の要素を取得します
   val myBool = myBits(4)
   // 要素 1 を割り当てる
   myBits(1) := True

   // 動的にインデックスを付ける
   val index = UInt(2 bit)
   val indexed = myBits(index, 2 bit)

   // 範囲インデックス
   val myBits_8bit = myBits_16bit(7 downto 0)
   val myBits_7bit = myBits_16bit(0 to 6)
   val myBits_6bit = myBits_16bit(0 until 6)
   // myBits_16bit(3 downto 0) に割り当てます
   myBits_8bit(3 downto 0) := myBits_4bit

   // 同等のスライス、逆転は発生しません
   val a = myBits_16bit(8 downto 4)
   val b = myBits_16bit(4 to 8)

   // msb / 左端ビット / x.high ビットの読み取り / 割り当て
   val isNegative = myBits_16bit.msb
   myBits_16bit.msb := False

Subdivide の詳細
""""""""""""""""""""

``subdivideIn`` の両方のオーバーロードには、オプションのパラメータ ``strict`` 
(つまり ``subdivideIn(slices: SlicesCount, strict: Boolean = true)``) があります。 
``strict`` が ``true`` の場合、入力が等しい部分に分割できなかった場合にエラーが発生します。 
``false`` に設定すると、最後の要素は他の (同じサイズの) 要素よりも小さくなる可能性があります。

.. code-block:: scala

   // Subdivide
   val sel = UInt(2 bits)
   val myBitsWord = myBits_128bits.subdivideIn(32 bits)(sel)
       // sel = 3 => myBitsWord = myBits_128bits(127 downto 96)
       // sel = 2 => myBitsWord = myBits_128bits( 95 downto 64)
       // sel = 1 => myBitsWord = myBits_128bits( 63 downto 32)
       // sel = 0 => myBitsWord = myBits_128bits( 31 downto  0)

    // 逆の順序でアクセスしたい場合は、次のようにします"
    val myVector   = myBits_128bits.subdivideIn(32 bits).reverse
    val myRevBitsWord = myVector(sel)

    // 細分化して割り当てることもできます
    val output8 = Bits(8 bit)
    val pieces = output8.subdivideIn(2 slices)
    // output8 に割り当て
    pieces(0) := 0xf
    pieces(1) := 0x5

その他
^^^^^^^^^

上記のビット抽出操作とは対照的に、戻り値を使用して元の信号に割り当てることはできません。


.. list-table::
   :header-rows: 1
   :widths: 2 4 2

   * - 演算子
     - 説明
     - 戻り値の型
   * - x.getWidth
     - ビット数を返す
     - Int
   * - x.bitsRange
     - 範囲 (0 ～ x.high) を返します。
     - Range
   * - x.valueRange
     - x 値の最小値から最大値までの範囲を符号なし整数 (0 to 2 \*\* width - 1) として解釈して返します
     - Range
   * - x.high
     - MSB のインデックス (x に対して許可される最大の 0 から始まるインデックス) を返します。
     - Int
   * - x.reversed
     - 逆ビット順序 (MSB<>LSB がミラーリング) で x のコピーを返します。
     - Bits(w(x) bits)
   * - x ## y
     - 連結する, x->high, y->low
     - Bits(w(x) + w(y) bits)
   * - x #* n
     - x が n 回繰り返す
     - Bits(w(x) * n bits)
   * - x.resize(y)
     - サイズ変更された x の表現を返します。拡大した場合は、必要に応じて MSB にゼロ パディングで拡張されます。y: Int
     - Bits(y bits)
   * - x.resized
     - 自動的にサイズ変更できる x のバージョンを返す必要がありました。
       サイズ変更操作は、後の割り当て時点まで延期されます。
       サイズ変更により、LSB が保持されたまま拡大または切り詰められる場合があります。
     - Bits(w(x) bits)
   * - x.resizeLeft(x)
     - MSB を同じ場所に保持してサイズ変更します (x:Int) サイズ変更により、
       MSB を保持したまま拡大または切り詰められる場合があります。
     - Bits(x bits)
   * - x.getZero
     - Return a new instance of Bits that is assigned a constant value of zeros the same width as x.
     - Bits(0, w(x) bits)
   * - x.getAllTrue
     - x と同じ幅の定数値が割り当てられた Bits の新しいインスタンスを返します。
     - Bits(w(x) bits).setAll()

.. note::
  `validRange` は、最小値と最大値が符号付き 32 ビット整数に収まる型にのみ使用できます。 
  (これは、 `Int` を使用する Scala `scala.collection.immutable.Range` 型によって与えられる制限です)

.. code-block:: scala
   
   println(myBits_32bits.getWidth) // 32

   // 連結
   myBits_24bits := bits_8bits_1 ## bits_8bits_2 ## bits_8bits_3
   // or
   myBits_24bits := Cat(bits_8bits_1, bits_8bits_2, bits_8bits_3)

   // サイズ変更
   myBits_32bits := B"32'x112233344"
   myBits_8bits  := myBits_32bits.resized       // 自動サイズ変更 (myBits_8bits = 0x44)
   myBits_8bits  := myBits_32bits.resize(8)     // 8 ビットにサイズ変更する (myBits_8bits = 0x44)
   myBits_8bits  := myBits_32bits.resizeLeft(8) // 8 ビットにサイズ変更する (myBits_8bits = 0x11)

.. _maskedliteral:

MaskedLiteral
-------------

MaskedLiteral 値は、 ``-`` で示されるドントケア値を持つビット ベクトルです。
これらは、直接比較したり、 ``switch`` ステートメントや ``mux`` に使用したりできます。

.. code-block:: scala

     val myBits = B"1101"

     val test1 = myBits === M"1-01" // True
     val test2 = myBits === M"0---" // False
     val test3 = myBits === M"1--1" // True
