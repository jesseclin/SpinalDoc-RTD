
.. _Bundle:

Bundle
======

説明
^^^^^^^^^^^

バンドル(``Bundle``)は、単一の名前の下に、名前付きのシグナル（SpinalHDLの基本型のいずれか）のグループを定義する複合型です。

バンドル(``Bundle``)は、データ構造、バス、およびインタフェースをモデル化するために使用できます。

宣言
^^^^^^^^^^^

バンドルを宣言する構文は次のとおりです：

.. code-block:: scala

   case class myBundle extends Bundle {
     val bundleItem0 = AnyType
     val bundleItem1 = AnyType
     val bundleItemN = AnyType
   }

例えば、色を保持するバンドルは次のように定義できます：

.. code-block:: scala

   case class Color(channelWidth: Int) extends Bundle {
     val r, g, b = UInt(channelWidth bits)
   }

:ref:`Spinal HDLの例 <example_list>`の中には、:ref:`APB3の定義 <example_apb3>`を見つけることができます。

条件付き信号
~~~~~~~~~~~~~~~~~~~
バンドル(``Bundle``)内の信号は条件付きで定義することができます。
以下の例で示すように、 ``dataWidth``が0より大きくない場合、展開された ``myBundle``に ``data``信号はありません。

.. code-block:: scala

   case class myBundle(dataWidth: Int) extends Bundle {
     val data = (dataWidth > 0) generate (UInt(dataWidth bits))
   }

.. note::

  このSpinalHDLメソッドに関する情報は、:ref:`generate <generate>`を参照してください。

演算子
^^^^^^^^^

バンドル(``Bundle``)型で利用可能な次の演算子があります：

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

   val color1 = Color(8)
   color1.r := 0 
   color1.g := 0 
   color1.b := 0

   val color2 = Color(8)
   color2.r := 0
   color2.g := 0 
   color2.b := 0

   myBool := color1 === color2  // バンドルのすべての要素を比較します
   // 以下と同等です:
   //myBool := color1.r === color2.r && color1.g === color2.g && color1.b === color2.b

型変換
~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 返り値
   * - x.asBits
     - ビットへのバイナリキャスト
     - Bits(w(x) bits)

.. code-block:: scala

   val color1 = Color(8)
   val myBits := color1.asBits

バンドルの要素は、定義された順序で配置され、LSBが最初に配置されます。
したがって、 ``color1``の ``r``は、 ``myBits``のビット0から8（LSB）を占有し、次に ``g``と ``b``がこの順序で続きます。
その際、 ``b.msb`` は結果のBits型のMSBとなります。


Bits を Bundle に戻す
~~~~~~~~~~~~~~~~~~~~~~~~~~~
``.assignFromBits``演算子は、 ``.asBits``の逆と見なすことができます。


.. list-table::
   :header-rows: 1

   * - 演算子
     - 説明
     - 返り値
   * - x.assignFromBits(y)
     - Bits (y) を Bundle (x) に変換します
     - Unit   
   * - x.assignFromBits(y, hi, lo)
     - 高/低境界を持つBits (y) をBundle (x) に変換します
     - Unit     

以下の例では、CommonDataBusというバンドルを循環バッファ（サードパーティのメモリ）に保存し、
後でBitsを読み出してそれらをCommonDataBus形式に変換します。

.. image:: /asset/image/bundle/CommonDataBus.png

.. code-block:: scala

   case class TestBundle () extends Component {
     val io = new Bundle {
       val we      = in     Bool()
       val addrWr  = in     UInt (7 bits)
       val dataIn  = slave  (CommonDataBus())

       val addrRd  = in     UInt (7 bits)
       val dataOut = master (CommonDataBus())
     }

     val mm = Ram3rdParty_1w_1rs (G_DATA_WIDTH = io.dataIn.getBitsWidth, 
                                  G_ADDR_WIDTH = io.addrWr.getBitsWidth, 
                                  G_VENDOR     = "Intel_Arria10_M20K")

     mm.io.clk_in    := clockDomain.readClockWire
     mm.io.clk_out   := clockDomain.readClockWire

     mm.io.we        := io.we
     mm.io.addr_wr   := io.addrWr.asBits
     mm.io.d         := io.dataIn.asBits

     mm.io.addr_rd   := io.addrRd.asBits
     io.dataOut.assignFromBits(mm.io.q)
   }

IO要素の方向
^^^^^^^^^^^^^^^^^^^^

コンポーネントのIO定義の中でバンドル(``Bundle``)を定義する場合、その方向を指定する必要があります。


in/out
~~~~~~

バンドルのすべての要素が同じ方向に向かう場合は、 ``in(MyBundle())``または ``out(MyBundle())``を使用できます。

例えば:

.. code-block:: scala

   val io = new Bundle {
     val input  = in (Color(8))
     val output = out(Color(8))
   }

master/slave
~~~~~~~~~~~~

もしインタフェースがマスター/スレーブトポロジーに従っている場合、 ``IMasterSlave``トレイトを使用できます。
その後、各要素の方向をマスターの視点から設定するために、関数 ``def asMaster(): Unit``を実装する必要があります。
その後、IO定義で ``master(MyBundle())``および ``slave(MyBundle())``構文を使用できます。

``Flow``クラスの ``toStream``メソッドなど、 ``toXXX``という形式の関数が定義されています。
これらの関数は通常、マスターサイドから呼び出すことができます。
さらに、 ``fromXXX``関数はスレーブサイド向けに設計されています。
マスターサイドよりもスレーブサイド向けに利用可能な関数が少ないことが一般的です。

例えば:

.. code-block:: scala

   case class HandShake(payloadWidth: Int) extends Bundle with IMasterSlave {
     val valid   = Bool()
     val ready   = Bool()
     val payload = Bits(payloadWidth bits)

     // この asMaster 関数を実装する必要があります。
     // この関数は、各信号の方向をマスターの観点から設定する必要があります。
     override def asMaster(): Unit = {
       out(valid, payload)
       in(ready)
     }
   }

   val io = new Bundle {
     val input  = slave(HandShake(8))
     val output = master(HandShake(8))
   }
