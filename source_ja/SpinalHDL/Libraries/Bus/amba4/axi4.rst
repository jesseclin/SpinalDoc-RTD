Axi4
====

AXI4 は、ARM によって定義された高帯域幅のバスです。

Configuration and instanciation
-------------------------------

AXI4 バスを作成するたびに、まず構成オブジェクトが必要です。この構成オブジェクトは ``Axi4Config`` であり、以下の引数を持ちます：

注意：useXXX は、バスに XXX 信号が存在するかどうかを指定します。

.. list-table::
   :header-rows: 1

   * - パラメータ名
     - タイプ
     - デフォルト
   * - addressWidth
     - Int
     - 
   * - dataWidth
     - Int
     - 
   * - idWidth
     - Int
     - 
   * - userWidth
     - Int
     - 
   * - useId
     - Boolean
     - true
   * - useRegion
     - Boolean
     - true
   * - useBurst
     - Boolean
     - true
   * - useLock
     - Boolean
     - true
   * - useCache
     - Boolean
     - true
   * - useSize
     - Boolean
     - true
   * - useQos
     - Boolean
     - true
   * - useLen
     - Boolean
     - true
   * - useLast
     - Boolean
     - true
   * - useResp
     - Boolean
     - true
   * - useProt
     - Boolean
     - true
   * - useStrb
     - Boolean
     - true
   * - useUser
     - Boolean
     - false


SpinalHDL ライブラリで AXI4 バスがどのように定義されているかを簡単に説明します：

.. code-block:: scala

   case class Axi4(config: Axi4Config) extends Bundle with IMasterSlave{
     val aw = Stream(Axi4Aw(config))
     val w  = Stream(Axi4W(config))
     val b  = Stream(Axi4B(config))
     val ar = Stream(Axi4Ar(config))
     val r  = Stream(Axi4R(config))

     override def asMaster(): Unit = {
       master(ar,aw,w)
       slave(r,b)
     }
   }

使用例を示します：

.. code-block:: scala

   val axiConfig = Axi4Config(
     addressWidth = 32,
     dataWidth    = 32,
     idWidth      = 4
   )
   val axiX = Axi4(axiConfig)
   val axiY = Axi4(axiConfig)

   when(axiY.aw.valid){
     //...
   }

バリエーション
------------------

AXI4 バスの他に、3つのバリエーションがあります：

.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - タイプ
     - 説明
   * - Axi4ReadOnly
     - AR と R チャンネルのみが存在します
   * - Axi4WriteOnly
     - AW、W、および B チャンネルのみが存在します
   * - Axi4Shared
     - | このバリエーションは、ライブラリのイニシアティブです。
       | W、B、R の4つのチャンネルを使用し、新しいチャンネルである AWR も使用します。
       | AWR チャンネルは、AR と AW トランザクションを転送するために使用できます。それらを分離するために、 ``write`` 信号が存在します。
       | この Axi4Shared バリエーションの利点は、特にインターコネクトでの領域を少なく使用できることです。


関数と演算子
-----------------------

.. list-table::
   :header-rows: 1
   :widths: 1 1 5

   * - 名前
     - 戻り値
     - 説明
   * - X >> Y
     - 
     - X を Y に接続します。AXI4 仕様で指定されたデフォルト値を推測でき、安全な方法で幅を適応することもできます。
   * - X << Y
     - 
     - 演算子 ">>" の逆を行います
   * - X.toWriteOnly
     - Axi4WriteOnly
     - X によって駆動される Axi4WriteOnly バスを返します
   * - X.toReadOnly
     - Axi4ReadOnly
     - X によって駆動される Axi4ReadOnly バスを返します

