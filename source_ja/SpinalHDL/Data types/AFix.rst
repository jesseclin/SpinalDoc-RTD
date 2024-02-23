
.. _AFix:

AFix
====

説明
^^^^^^^^^^^

自動範囲固定小数点、　``AFix``、は、固定小数点演算を行う間に表現可能な値の範囲を追跡する固定小数点クラスです。

**警告: このコードの多くはまだ開発中です。API および関数呼び出しは変更される可能性があります。**

ユーザーのフィードバックを歓迎します！

宣言
^^^^^^^^^^^

AFix はビットサイズまたは指数を使用して作成することができます：

.. code-block:: scala

  AFix.U(12 bits)         // U12.0
  AFix.UQ(8 bits, 4 bits) // U8.4
  AFix.U(8 exp, 12 bits)  // U8.4
  AFix.U(8 exp, -4 exp)   // U8.4
  AFix.U(8 exp, 4 exp)    // U8.-4

  AFix.S(12 bits)         // S11 + sign
  AFix.SQ(8 bits, 4 bits) // S8.4 + sign
  AFix.S(8 exp, 12 bits)  // S8.3 + sign
  AFix.S(8 exp, -4 exp)   // S8.4 + sign

これらはすべてのビットに対して表現可能な範囲を持ちます。

例えば：

``AFix.U(12 bits)`` は、0 から 4095 の範囲を持ちます。

``AFix.SQ(8 bits, 4 bits)`` は、-4096 (-256) から 4095 (255.9375) の範囲を持ちます。

``AFix.U(8 exp, 4 exp)`` は、0 から 256 の範囲を持ちます。

カスタム範囲の ``AFix`` 値は、クラスを直接インスタンス化することで作成することができます。

.. code-block:: scala

  class AFix(val maxValue: BigInt, val minValue: BigInt, val exp: ExpNumber)

  new AFix(4096, 0, 0 exp)     // [0 to 4096, 2^0]
  new AFix(256, -256, -2 exp)  // [-256 to 256, 2^-2]
  new AFix(16, 8, 2 exp)       // [8 to 16, 2^2]

``maxValue`` と ``minValue``は、どのバッキング整数値が表現可能かを格納します。
これらの値は、 ``2^exp``を乗算した後の真の固定小数点値を表します。

``AFix.U(2 exp, -1 exp)`` は以下を表現できます：
``0, 0.5, 1.0, 1.5, 2, 2.5, 3, 3.5``

``AFix.S(2 exp, -2 exp)`` は以下を表現できます:
``-2.0, -1.75, -1.5, -1.25, -1, -0.75, -0.5, -0.25, 0, 0.25, 0.5, 0.75, 1, 1.25, 1.5, 1.75``

0 より大きい指数値は許可されており、1 より大きな値を表します。

``AFix.S(2 exp, 1 exp)`` は以下を表現できます：
``-4, 2, 0, 2``

``AFix(8, 16, 2 exp)`` は以下を表現できます：
``32, 36, 40, 44, 48, 52, 56, 60, 64``

注意： ``AFix`` は、これを保存するために 5 ビットを使用します。それは ``16``、その ``maxValue`` を格納できます。

数学的な操作
^^^^^^^^^^^^^^^^^^^^^^^

``AFix`` は、ハードウェアレベルでの加算 (``+``)、減算 (``-``)、および乗算 (``*``) をサポートしています。
除算 (``\``) および剰余 (``%``) 演算子が提供されていますが、ハードウェアの展開では推奨されていません。

演算は ``AFix`` 値が通常の ``Int`` 数値であるかのように実行されます。
符号付きと符号なしの数値は相互運用可能です。符号付きまたは符号なしの値の間には型の違いはありません。

.. code-block:: scala

  // 整数部と小数部の展開
  val a = AFix.U(4 bits)          // [   0 (  0.)     to  15 (15.  )]  4 bits, 2^0
  val b = AFix.UQ(2 bits, 2 bits) // [   0 (  0.)     to  15 ( 3.75)]  4 bits, 2^-2
  val c = a + b                   // [   0 (  0.)     to  77 (19.25)]  7 bits, 2^-2
  val d = new AFix(-4, 8, -2 exp) // [-  4 (- 1.25)   to   8 ( 2.00)]  5 bits, 2^-2
  val e = c * d                   // [-308 (-19.3125) to 616 (38.50)] 11 bits, 2^-4

  // 展開なしの整数
  val aa = new AFix(8, 16, -4 exp) // [8 to 16] 5 bits, 2^-4
  val bb = new AFix(1, 15, -4 exp) // [1 to 15] 4 bits, 2^-4
  val cc = aa + bb                 // [9 to 31] 5 bits, 2^-4


``AFix``は範囲の展開なしでの操作をサポートしています。
これは、各入力から整列した最大値と最小値の範囲を選択することで行います。

``+|`` 展開なしでの加算。
``-|`` 展開なしでの減算。


不等号演算
^^^^^^^^^^^^^^^^^^^^^

``AFix`` は標準の不等号演算をサポートしています。

.. code-block:: scala

  A === B
  A =\= B
  A < B
  A <= B
  A > B
  A >= B

注意: コンパイル時に範囲外の演算は最適化されます！

ビットシフト
^^^^^^^^^^^^^

``AFix`` は、10進数とビットのシフトをサポートしています。

``<<`` は小数点を左にシフトします。指数に加算されます。
``>>`` は小数点を右にシフトします。指数から減算されます。
``<<|`` はビットを左にシフトします。小数部にゼロを追加します。
``>>|`` はビットを右にシフトします。小数ビットを削除します。

飽和および丸め
^^^^^^^^^^^^^^^^^^^^^^^

``AFix`` は、飽和とすべての一般的な丸め方法を実装しています。

飽和は、 ``AFix`` 値のバッキング値の範囲を飽和させることで動作します。指数を考慮した複数のヘルパー関数があります。

.. code-block:: scala

  val a = new AFix(63, 0, -2 exp) // [0 to 63, 2^-2]
  a.sat(63, 0)                    // [0 to 63, 2^-2]
  a.sat(63, 0, -3 exp)            // [0 to 31, 2^-2]
  a.sat(new AFix(31, 0, -1 exp))  // [0 to 31, 2^-2]

``AFix`` の丸めモード:

.. code-block:: scala

  // 以下は exp < 0 が必要です
  .floor() or .truncate()
  .ceil()
  .floorToZero()
  .ceilToInf()
  // 以下は exp < -1 が必要です
  .roundHalfUp()
  .roundHalfDown()
  .roundHalfToZero()
  .roundHalfToInf()
  .roundHalfToEven()
  .roundHalfToOdd()

これらの丸めモードの数学的な例は、こちらの `Rounding - Wikipedia <https://en.wikipedia.org/wiki/Rounding>`_でよりよく説明されています。

これらのモードのいずれも、指数が 0 の ``AFix`` 値を結果とします。
異なる指数に丸める必要がある場合は、シフトを考慮するか、 ``truncated``タグを使用した割り当てを検討してください。

割り当て
^^^^^^^^^^

``AFix`` は、割り当て中に範囲と精度を自動的にチェックおよび拡張します。
デフォルトでは、範囲や精度が小さい ``AFix`` 値に別の ``AFix`` 値を割り当てることはエラーです。

``.truncated`` 関数は、小さな型への割り当てを制御するために使用されます。

.. code-block:: scala

  def truncated(saturation: Boolean = false,
                overflow  : Boolean = true,
                rounding  : RoundType = RoundType.FLOOR)

  def saturated(): AFix = this.truncated(saturation = true, overflow = false)

``RoundType``:

.. code-block:: scala

  RoundType.FLOOR
  RoundType.CEIL
  RoundType.FLOORTOZERO
  RoundType.CEILTOINF
  RoundType.ROUNDUP
  RoundType.ROUNDDOWN
  RoundType.ROUNDTOZERO
  RoundType.ROUNDTOINF
  RoundType.ROUNDTOEVEN
  RoundType.ROUNDTOODD

``saturation`` フラグは、割り当てられたデータ型の範囲に飽和ロジックを追加します。

``overflow``フラグは、範囲チェックなしに丸め後に直接割り当てを許可します。

精度が高い値を精度が低い値に割り当てる場合、常に丸めが必要です。