.. _utils:

Utils
=====

General
-------

:ref:`spinal.lib <lib_introduction>` に多くのツールやユーティリティが含まれていますが、いくつかはすでに SpinalHDL コアに含まれています。

.. list-table::
   :header-rows: 1
   :widths: 3 1 4

   * - 構文
     - 戻り値
     - 説明
   * - ``widthOf(x : BitVector)``
     - Int
     - Bits/UInt/SInt 信号の幅を返します
   * - ``log2Up(x : BigInt)``
     - Int
     - ``x`` の状態を表現するのに必要なビット数を返します
   * - ``isPow2(x : BigInt)``
     - Boolean
     - ``x`` が 2 の累乗であれば true を返します
   * - ``roundUp(that : BigInt, by : BigInt)``
     - BigInt
     - 最初の ``by`` 個の ``that``の倍数を返します（含む）
   * - ``Cat(x: Data*)``
     - Bits
     - 全ての引数を MSB から LSB に連結します。 `Cat`_ を参照してください
   * - ``Cat(x: Iterable[Data])``
     - Bits
     - 引数を LSB から MSB に連結します。 `Cat`_ を参照してください

.. _Cat:

Cat
^^^

上記にリストされているように、 ``Cat`` には 2つのバージョンがあります。どちらも含まれる信号を連結しますが、微妙な違いがあります：

- ``Cat(x: Data*)`` は任意の数のハードウェア信号をパラメータとして受け取ります。
  これは他の HDL と同様で、最も左のパラメータが生成される Bits の MSB になり、最も右のパラメータが LSB 側になります。
  言い換えると、入力は書かれた順序で連結されます。
- ``Cat(x: Iterable[Data])`` は、ハードウェア信号を含む単一の Scala iterable コレクション（Seq / Set / List / ...）を取ります。
  このバージョンは、リストの最初の要素をLSBに配置し、最後の要素をMSBに配置します。
  
この見かけの違いは、ほとんどが、 ``Bits`` が最も高いインデックスから最も低いインデックスまで書かれるという慣習から来ていますが、
リストはインデックス 0 から最高のインデックスまで書かれます。 ``Cat`` は両方の慣習のインデックス 0 を LSB に配置します。

.. code-block:: scala

   val bit0, bit1, bit2 = Bool()

   val first = Cat(bit2, bit1, bit0)

   // は以下と同等です
   
   val signals = List(bit0, bit1, bit2)
   val second = Cat(signals)

Cloning hardware datatypes
--------------------------

与えられたハードウェアデータ型を、``cloneOf(x)`` 関数を使用してクローンすることができます。
これにより、同じ Scala 型とパラメータの新しいインスタンスが返されます。

例えば:

.. code-block:: scala

   def plusOne(value : UInt) : UInt = {
     // ``value`` と同じ幅の UInt の新しいインスタンスを提供します
     val temp = cloneOf(value)
     temp := value + 1
     return temp
   }

   //  treePlusOne は 8ビットの値になります
   val treePlusOne = plusOne(U(3, 8 bits))

ハードウェアデータ型の管理方法については、:ref:`Hardware types page <hardware_type>` を参照してください。

.. note::
   ``Bundle`` で ``cloneOf`` 関数を使用する場合、この ``Bundle`` は ``case class`` であるか、
   内部でクローン関数をオーバーライドする必要があります。
   
.. code-block:: scala

   // 'class' に 'override def clone()' 関数を持つ例
   class MyBundle(ppp : Int) extends Bundle {
      val a = UInt(ppp bits)
      override def clone = new MyBundle(ppp)
    }
    val x = new MyBundle(3)
    val typeDef = HardType(new MyBundle(3))
    val y = typeDef()

    cloneOf(x) // クローンメソッドが必要です、そうでないとエラーになります
    cloneOf(y) // 問題ありません


Passing a datatype as construction parameter
--------------------------------------------

多くの再利用可能なハードウェアの部品は、あるデータ型によってパラメータ化される必要があります。
例えば、FIFO やシフトレジスタを定義したい場合、コンポーネントでどの種類のペイロードを使用するかを指定するパラメータが必要です。

これを行うための2つの類似した方法があります。

The old way
^^^^^^^^^^^

これを行う古い方法の良い例は、 ``ShiftRegister`` コンポーネントの定義です：


.. code-block:: scala

   case class ShiftRegister[T <: Data](dataType: T, depth: Int) extends Component {
     val io = new Bundle {
       val input  = in (cloneOf(dataType))
       val output = out(cloneOf(dataType))
     }
     // ...
   }

そして、コンポーネントをインスタンス化する方法は以下の通りです：

.. code-block:: scala

   val shiftReg = ShiftRegister(Bits(32 bits), depth = 8)

ご覧の通り、生のハードウェアタイプが直接構築パラメータとして渡されます。
その後、その種類のハードウェアデータ型の新しいインスタンスを作成するたびに、 ``cloneOf(...)`` 関数を使用する必要があります。
この方法で作業を行うのは、 ``cloneOf`` を使用するのを忘れるのが簡単なため、非常に安全ではありません。

The safe way
^^^^^^^^^^^^

データ型パラメータを安全に渡す方法の例は次の通りです：

.. code-block:: scala

   case class ShiftRegister[T <: Data](dataType: HardType[T], depth: Int) extends Component {
     val io = new Bundle {
       val input  = in (dataType())
       val output = out(dataType())
     }
     // ...
   }

そして、コンポーネントをインスタンス化する方法は以下の通りです（前述のものとまったく同じです）：

.. code-block:: scala

   val shiftReg = ShiftRegister(Bits(32 bits), depth = 8)

上記の例では、生のデータ型 ``T`` の周りに ``HardType`` ラッパーを使用しており、これはハードウェアデータ型の「設計図」の定義です。
この方法は、「旧式の方法」よりも使いやすいです。なぜなら、ハードウェアデータ型の新しいインスタンスを作成するには、
その ``HardType`` の ``apply`` 関数を呼び出すだけで済むからです（言い換えると、パラメータの後ろに括弧を追加するだけです）。

さらに、このメカニズムはユーザーの観点から完全に透明であり、ハードウェアデータ型は暗黙的に ``HardType`` に変換されることがあります。

Frequency and time
------------------

SpinalHDLには、周波数や時間値を定義するための専用の構文があります：

.. code-block:: scala

   val frequency = 100 MHz	// 型 HertzNumber が推論されます
   val timeoutLimit = 3 ms	// 型 TimeNumber が推論されます
   val period = 100 us		// 型 TimeNumber が推論されます

   val periodCycles = frequency * period             // 型 BigDecimal が推論されます
   val timeoutCycles = frequency * timeoutLimit      // 型 BigDecimal が推論されます

| 時間の定義では、以下の接尾辞を使用して ``TimeNumber`` を取得できます：
| ``fs``, ``ps``, ``ns``, ``us``, ``ms``, ``sec``, ``mn``, ``hr``

| 周波数の定義では、以下の接尾辞を使用して ``HertzNumber`` を取得できます：
| ``Hz``, ``KHz``, ``MHz``, ``GHz``, ``THz``

`` TimeNumber`` と ``HertzNumber`` は、 ``PhysicalNumber`` クラスに基づいており、数値の保存には scala の ``BigDecimal`` が使用されています。

Binary prefix
-------------


SpinalHDL では、IEC に従ったバイナリ接頭辞表記を使用して整数を定義できます。

.. code-block:: scala

   val memSize = 512 MiB      // 型 BigInt が推論されます
   val dpRamSize = 4 KiB      // 型 BigInt が推論されます

以下のバイナリ接頭辞表記が利用可能です：

.. list-table::
   :header-rows: 1
   :widths: 1 2

   * - バイナリ接頭辞
     - 値
   * - Byte, Bytes
     - 1
   * - KiB
     - 1024 == 1 << 10
   * - MiB
     - 1024\ :sup:`2` == 1 << 20
   * - GiB
     - 1024\ :sup:`3` == 1 << 30
   * - TiB
     - 1024\ :sup:`4` == 1 << 40
   * - PiB
     - 1024\ :sup:`5` == 1 << 50
   * - EiB
     - 1024\ :sup:`6` == 1 << 60
   * - ZiB
     - 1024\ :sup:`7` == 1 << 70
   * - YiB
     - 1024\ :sup:`8` == 1 << 80


もちろん、BigInt はバイト単位の文字列として出力することもできます。 ``BigInt(1024).byteUnit`` 。

.. code-block:: scala

   val memSize = 512 MiB
    
   println(memSize)
   >> 536870912 

   println(memSize.byteUnit)
   >> 512MiB

   val dpRamSize = BigInt("123456789", 16)

   println(dpRamSize.byteUnit())
   >> 4GiB+564MiB+345KiB+905Byte

   println((32.MiB + 12.KiB + 223).byteUnit())
   >> 32MiB+12KiB+223Byte

   println((32.MiB + 12.KiB + 223).byteUnit(ceil = true))
   >> 33~MiB



