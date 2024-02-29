
Apb3
====

AMBA3-APB バスは、低帯域幅の周辺機器とのインタフェースに一般的に使用されます。

Configuration and instanciation
-------------------------------

APB3 バスを作成するたびに、まず構成オブジェクトが必要です。この構成オブジェクトは ``Apb3Config`` であり、以下の引数を持ちます：

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
     - PADDR の幅（バイト単位）
   * - dataWidth
     - Int
     - 
     - PWDATA と PRDATA の幅
   * - selWidth
     - Int
     - 1
     - PSEL の幅
   * - useSlaveError
     - Boolean
     - false
     - PSLVERROR の存在を指定します


SpinalHDL ライブラリで APB3 バスがどのように定義されているかを簡単に説明します：

.. code-block:: scala

   case class Apb3(config: Apb3Config) extends Bundle with IMasterSlave {
     val PADDR      = UInt(config.addressWidth bits)
     val PSEL       = Bits(config.selWidth bits)
     val PENABLE    = Bool()
     val PREADY     = Bool()
     val PWRITE     = Bool()
     val PWDATA     = Bits(config.dataWidth bits)
     val PRDATA     = Bits(config.dataWidth bits)
     val PSLVERROR  = if(config.useSlaveError) Bool() else null
     //...
   }

使用例を示します：

.. code-block:: scala

   val apbConfig = Apb3Config(
     addressWidth = 12,
     dataWidth    = 32
   )
   val apbX = Apb3(apbConfig)
   val apbY = Apb3(apbConfig)

   when(apbY.PENABLE){
     //...
   }

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
     - X を Y に接続します。 Y のアドレスは X よりも小さくすることができます
   * - X << Y
     - 
     - 演算子 ">>" の逆を行います


