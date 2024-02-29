
AHB-Lite3
=========

Configuration and instanciation
-------------------------------

まず、AHB-Lite3 バスを作成するたびに、構成オブジェクトが必要です。この構成オブジェクトは AhbLite3Config であり、以下の引数を持ちます：

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 2

   * - パラメータ名
     - タイプ
     - デフォルト
     - 説明
   * - addressWidth
     - Int
     - 
     - HADDRの幅（バイト単位）
   * - dataWidth
     - Int
     - 
     - HWDATAとHRDATAの幅


SpinalHDL ライブラリで AHB-Lite3 バスがどのように定義されているかを簡単に説明します：

.. code-block:: scala

   case class AhbLite3(config: AhbLite3Config) extends Bundle with IMasterSlave{
     //  アドレスと制御
     val HADDR = UInt(config.addressWidth bits)
     val HSEL = Bool()
     val HREADY = Bool()
     val HWRITE = Bool()
     val HSIZE = Bits(3 bits)
     val HBURST = Bits(3 bits)
     val HPROT = Bits(4 bits)
     val HTRANS = Bits(2 bits)
     val HMASTLOCK = Bool()

     //  データ
     val HWDATA = Bits(config.dataWidth bits)
     val HRDATA = Bits(config.dataWidth bits)

     //  転送応答
     val HREADYOUT = Bool()
     val HRESP = Bool()

     override def asMaster(): Unit = {
       out(HADDR,HWRITE,HSIZE,HBURST,HPROT,HTRANS,HMASTLOCK,HWDATA,HREADY,HSEL)
       in(HREADYOUT,HRESP,HRDATA)
     }
   }

使用例を示します：

.. code-block:: scala

   val ahbConfig = AhbLite3Config(
     addressWidth = 12,
     dataWidth    = 32
   )
   val ahbX = AhbLite3(ahbConfig)
   val ahbY = AhbLite3(ahbConfig)

   when(ahbY.HSEL){
     //...
   }

バリエーション
------------------

AhbLite3Master バリエーションがあります。唯一の違いは ``HREADYOUT`` シグナルの欠如です。
このバリエーションは、インターコネクトとスレーブが ``AhbLite3`` を使用する間、マスターだけが使用すべきです。