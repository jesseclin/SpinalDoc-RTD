.. warning::
  SpinalHDLの固定小数点サポートは部分的にしか使用/テストされていません。
  もし問題が見つかった場合や機能が不足していると思われる場合は、 
  `Github issue <https://github.com/SpinalHDL/SpinalHDL/issues>`_を作成してください。
  また、コードで未記載の機能を使用しないでください。

.. _fixed:

UFix/SFix
=========

説明
^^^^^^^^^^^

``UFix``と　``SFix``の型は、固定小数点演算に使用できるビットのベクトルに対応しています。

宣言
^^^^^^^^^^^

固定小数点数を宣言する構文は次のようになります：

Unsigned Fixed-Point
~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 3 1 2 3 1

   * - 構文
     - ビット幅
     - 解像度
     - 最大
     - 最小
   * - UFix(peak: ExpNumber, resolution: ExpNumber)
     - peak-resolution
     - 2^resolution
     - 2^peak-2^resolution
     - 0
   * - UFix(peak: ExpNumber, width: BitCount)
     - width
     - 2^(peak-width)
     - 2^peak-2^(peak-width)
     - 0

Signed Fixed-Point
~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 3 1 2 3 1

   * - 構文
     - ビット幅
     - 解像度
     - 最大
     - 最小
   * - SFix(peak: ExpNumber, resolution: ExpNumber)
     - peak-resolution+1
     - 2^resolution
     - 2^peak-2^resolution
     - -(2^peak)
   * - SFix(peak: ExpNumber, width: BitCount)
     - width
     - 2^(peak-width-1)
     - 2^peak-2^(peak-width-1)
     - -(2^peak)

フォーマット
~~~~~~~~~~~~~~

このフォーマットは、通常、Q表記を使用した固定小数点数のフォーマット定義方法に従っています。
詳細は、 `Q数フォーマットに関するWikipediaページ <https://en.wikipedia.org/wiki/Q_(number_format)>`_ を参照してください。

たとえば、Q8.2 は 8 + 2 ビットの固定小数点数を表し、8 ビットが整数部分、2 ビットが小数部分に使用されます。
符号付き固定小数点数の場合は、さらに 1 ビットが符号に使用されます。

解像度は、この数で表現できる最小の 2 のべき乗として定義されます。

.. note::
   2 の累乗の数値を表現する際に誤りを少なくするために、 ``spinal.core`` には ``ExpNumber`` という数値型があります。
   これは固定小数点型のコンストラクタで使用されます。
   この型には便利なラッパーも存在し、このページのコードサンプルで使用される ``exp`` 関数の形で提供されています。

例
~~~~~~~~

.. code-block:: scala

   // 符号なし固定小数点
   val UQ_8_2 = UFix(peak = 8 exp, resolution = -2 exp) // ビット幅 = 8 - (-2) = 10 bits
   val UQ_8_2 = UFix(8 exp, -2 exp)

   val UQ_8_2 = UFix(peak = 8 exp, width = 10 bits)
   val UQ_8_2 = UFix(8 exp, 10 bits)

   // 符号付き固定小数点
   val Q_8_2 = SFix(peak = 8 exp, resolution = -2 exp) // ビット幅 = 8 - (-2) + 1 = 11 bits
   val Q_8_2 = SFix(8 exp, -2 exp)

   val Q_8_2 = SFix(peak = 8 exp, width = 11 bits)
   val Q_8_2 = SFix(8 exp, 11 bits)

割り当て
^^^^^^^^^^^

有効な割り当て
~~~~~~~~~~~~~~~~~

固定小数点値への割り当ては、ビットの損失がない場合に有効です。ビットの損失があるとエラーが発生します。

もしソースの固定小数点値が大きすぎる場合は、``truncated`` 関数を使用してソースの数値を目的のサイズにリサイズすることができます。

例
"""""""

.. code-block:: scala

   val i16_m2 = SFix(16 exp, -2 exp)
   val i16_0  = SFix(16 exp,  0 exp)
   val i8_m2  = SFix( 8 exp, -2 exp)
   val o16_m2 = SFix(16 exp, -2 exp)
   val o16_m0 = SFix(16 exp,  0 exp)
   val o14_m2 = SFix(14 exp, -2 exp)

   o16_m2 := i16_m2            // OK
   o16_m0 := i16_m2            // OK ではありません、ビット損失
   o14_m2 := i16_m2            // OK ではありません、ビット損失
   o16_m0 := i16_m2.truncated  // OK、割り当てターゲットに合わせてサイズ変更されるので、
   o14_m2 := i16_m2.truncated  // OK、割り当てターゲットに合わせてサイズ変更されるので、
   val o18_m2 = i16_m2.truncated(18 exp, -2 exp)
   val o18_22b = i16_m2.truncated(18 exp, 22 bit)

Scala の定数から
~~~~~~~~~~~~~~~~~~~~~

Scala の ``BigInt`` または ``Double`` 型は、 ``UFix`` または ``SFix`` シグナルに割り当てる際に定数として使用できます。

例
"""""""

.. code-block:: scala

   val i4_m2 = SFix(4 exp, -2 exp)
   i4_m2 := 1.25    // i4_m2.raw に 5 をロードします。
   i4_m2 := 4       // i4_m2.raw に 16 をロードします。

生の値
^^^^^^^^^

例
~~~~~~~

.. code-block:: scala

   val UQ_8_2 = UFix(8 exp, 10 bits)
   UQ_8_2.raw := 4        // 値を 1.0 に対応する値に割り当てます。
   UQ_8_2.raw := U(17)    // 値を 4.25 に対応する値に割り当てます。

演算子
^^^^^^^^^

以下の演算子は ``UFix`` 型で利用可能です：

算術
~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 1 7 4 7

   * - 演算子
     - 説明
     - 戻り解像度
     - 戻り振幅
   * - x + y
     - 加算
     - Min(x.resolution, y.resolution)
     - Max(x.amplitude, y.amplitude)
   * - x - y
     - 減算
     - Min(x.resolution, y.resolution)
     - Max(x.amplitude, y.amplitude)
   * - x * y
     - 乗算
     - x.resolution * y.resolution
     - x.amplitude * y.amplitude
   * - x >> y
     - 算術右シフト, y : Int
     - x.amplitude >> y
     - x.resolution >> y
   * - x << y
     - 算術左シフト, y : Int
     - x.amplitude << y
     - x.resolution << y
   * - x >>| y
     - 算術右シフト, y : Int
     - x.amplitude >> y
     - x.resolution
   * - x <<| y
     - 算術左シフト, y : Int
     - x.amplitude << y
     - x.resolution

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
   * - x > y
     - より大きい
     - Bool
   * - x >= y
     - 以上
     - Bool
   * - x < y
     - 未満
     - Bool
   * - x >= y
     - 以下
     - Bool

型キャスト
~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 1 3 2

   * - 演算子
     - 説明
     - 戻り値
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
     - CBool の配列にキャストします
     - Vec(Bool(),width(x))
   * - x.toUInt
     - 対応する UInt を返します (切り捨てあり)
     - UInt
   * - x.toSInt
     - 対応する SInt を返します (切り捨てあり)
     - SInt
   * - x.toUFix
     - 対応する UFix を返します
     - UFix
   * - x.toSFix
     - 対応する SFix を返します
     - SFix

その他
~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 2 5 2

   * - 名
     - 戻り値
     - 説明
   * - x.maxValue
     - 保存可能な最大値を返します
     - Double
   * - x.minValue
     - 保存可能な最小値を返します
     - Double
   * - x.resolution
     - x.amplitude * y.amplitude
     - Double

