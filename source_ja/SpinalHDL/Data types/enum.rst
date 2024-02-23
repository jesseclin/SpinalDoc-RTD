
.. _Enum:

SpinalEnum
==========

説明
^^^^^^^^^^^

列挙型 ``Enumeration`` は、名前付き値のリストに対応しています。

宣言
^^^^^^^^^^^

列挙型データの宣言は次のようになります：

.. code-block:: scala

   object Enumeration extends SpinalEnum {
     val element0, element1, ..., elementN = newElement()
   }

上記の例では、デフォルトのエンコーディングが使用されます。
VHDLではネイティブの列挙型が使用され、Verilogではバイナリエンコーディングが使用されます。

列挙型のエンコーディングを強制するには、次のように列挙型を定義します：

.. code-block:: scala

   object Enumeration extends SpinalEnum(defaultEncoding=encodingOfYourChoice) {
     val element0, element1, ..., elementN = newElement()
   }
   
.. note::
  特定のコンポーネントの列挙型をin/outとして定義したい場合は、次のようにします： ``in(MyEnum())`` または ``out(MyEnum())``


エンコーディング
~~~~~~~~~~~~~~~~~~

以下の列挙型のエンコーディングがサポートされています：

.. list-table::
   :header-rows: 1
   :widths: 1 1 8

   * - エンコーディング
     - ビット幅
     - 説明
   * - ``native``
     - 
     - VHDLの列挙システムを使用します。これがデフォルトのエンコーディングです。
   * - ``binarySequential``
     - ``log2Up(stateCount)``
     - 宣言の順序で状態を保存するために Bits を使用します（値は0からn-1まで）。
   * - ``binaryOneHot``
     - stateCount
     - 状態を保存するために Bits を使用します。各ビットは1つの状態に対応し、
       ハードウェアでエンコードされた状態表現では一度に1つのビットのみが設定されます。
   * - ``graySequential``
     - ``log2Up(stateCount)``
     - インデックスを ``binarySequential`` を使用しているかのように、バイナリグレーコードとしてエンコードします。

カスタムエンコーディングは、静的または動的の2つの異なる方法で行うことができます。

.. code-block:: scala

   /* 
    * 静的エンコーディング 
    */
   object MyEnumStatic extends SpinalEnum {
     val e0, e1, e2, e3 = newElement()
     defaultEncoding = SpinalEnumEncoding("staticEncoding")(
       e0 -> 0,
       e1 -> 2,
       e2 -> 3,
       e3 -> 7)
   }

   /*
    * 次の関数を使った動的エンコーディング:  _ * 2 + 1
    *   e.g. : e0 => 0 * 2 + 1 = 1
    *          e1 => 1 * 2 + 1 = 3
    *          e2 => 2 * 2 + 1 = 5
    *          e3 => 3 * 2 + 1 = 7
    */
   val encoding = SpinalEnumEncoding("dynamicEncoding", _ * 2 + 1)

   object MyEnumDynamic extends SpinalEnum(encoding) {
     val e0, e1, e2, e3 = newElement()
   }

例
~~~~~~~

列挙信号をインスタンス化し、それに値を割り当てます：

.. code-block:: scala

   object UartCtrlTxState extends SpinalEnum {
     val sIdle, sStart, sData, sParity, sStop = newElement()
   }

   val stateNext = UartCtrlTxState()
   stateNext := UartCtrlTxState.sIdle

   // 列挙型をインポートして、その要素を参照できます。 
   import UartCtrlTxState._
   stateNext := sIdle

演算子
^^^^^^^^^

``列挙型``には、以下の演算子が利用可能です：

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

   import UartCtrlTxState._

   val stateNext = UartCtrlTxState()
   stateNext := sIdle

   when(stateNext === sStart) {
     ...
   }

   switch(stateNext) {
     is(sIdle) {
       ...
     }
     is(sStart) {
       ...
     }
     ...
   }

型
~~~~~

列挙型を、例えば関数内で使用する場合、その型が必要になることがあります。

値の型（例：sIdleの型）は、

.. code-block:: scala

    spinal.core.SpinalEnumElement[UartCtrlTxState.type]

あるいは同等に

.. code-block:: scala

    UartCtrlTxState.E

バンドルの型（例：stateNextの型）は、

.. code-block:: scala

    spinal.core.SpinalEnumCraft[UartCtrlTxState.type]

あるいは同等に

.. code-block:: scala

    UartCtrlTxState.C

型変換
~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 返り値
   * - x.asBits
     - Bits へのバイナリキャスト
     - Bits(w(x) bits)
   * - x.asBits.asUInt
     - UIntへのバイナリキャスト
     - UInt(w(x) bits)
   * - x.asBits.asSInt
     - SIntへのバイナリキャスト 
     - SInt(w(x) bits)
   * - e.assignFromBits(bits)
     - Bits を列挙型にキャスト
     - MyEnum()

.. code-block:: scala

   import UartCtrlTxState._

   val stateNext = UartCtrlTxState()
   myBits := sIdle.asBits

   stateNext.assignFromBits(myBits)

