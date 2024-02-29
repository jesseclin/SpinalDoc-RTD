
AvalonMM
========

AvalonMM バスは FPGA に非常に適しています。非常に柔軟性があります：

* APB と同じくらいシンプルな操作が可能です
* 帯域幅が必要な多くのアプリケーションにとって、AHB よりも優れています。なぜなら、AvalonMM には読み取り応答をコマンドから分離するモードがあるためです（読み取り遅延の影響を減らす）。
* AXI よりも性能は低いですが、領域をはるかに少なく使用します（読み取りおよび書き込みコマンドは同じハンドシェイクチャネルを使用します。マスターは読み取り/書き込みハザードを避けるために保留中のリクエストのアドレスを保存する必要はありません）

Configuration and instanciation
-------------------------------

``AvalonMM`` バンドルには、構成引数 ``AvalonMMConfig`` があります。 
``Avalon`` バスの柔軟な性質から、 ``AvalonMMConfig`` には多くの構成要素があります。
詳細については、Avalon 仕様書を Intel のウェブサイトで見つけることができます。

.. code-block:: scala

   case class AvalonMMConfig( addressWidth : Int,
                              dataWidth : Int,
                              burstCountWidth : Int,
                              useByteEnable : Boolean,
                              useDebugAccess : Boolean,
                              useRead : Boolean,
                              useWrite : Boolean,
                              useResponse : Boolean,
                              useLock : Boolean,
                              useWaitRequestn : Boolean,
                              useReadDataValid : Boolean,
                              useBurstCount : Boolean,
                              //useEndOfPacket : Boolean,

                              addressUnits : AddressUnits = symbols,
                              burstCountUnits : AddressUnits = words,
                              burstOnBurstBoundariesOnly : Boolean = false,
                              constantBurstBehavior : Boolean = false,
                              holdTime : Int = 0,
                              linewrapBursts : Boolean = false,
                              maximumPendingReadTransactions : Int = 1,
                              maximumPendingWriteTransactions : Int = 0, // unlimited
                              readLatency : Int = 0,
                              readWaitTime : Int = 0,
                              setupTime : Int = 0,
                              writeWaitTime : Int = 0
                              )

この構成クラスにはいくつかの関数もあります：

.. list-table::
   :header-rows: 1
   :widths: 1 1 5

   * - 名前
     - 戻り値
     - 説明
   * - getReadOnlyConfig
     - AvalonMMConfig
     - すべての書き込み機能を無効にした類似の構成を返します
   * - getWriteOnlyConfig
     - AvalonMMConfig
     - すべての読み取り機能を無効にした類似の構成を返します


この構成コンパニオンオブジェクトには、いくつかの ``AvalonMMConfig``  テンプレートを提供するための関数もあります：

.. list-table::
   :header-rows: 1
   :widths: 1 1 4

   * - 名前
     - 戻り値
     - 説明
   * - fixed(addressWidth,dataWidth,readLatency)
     - AvalonMMConfig
     - 固定された読み取りタイミングを持つ単純な構成を返します
   * - pipelined(addressWidth,dataWidth)
     - AvalonMMConfig
     - 変動レイテンシの読み取り（readDataValid）を持つ構成を返します
   * - bursted(addressWidth,dataWidth,burstCountWidth)
     - AvalonMMConfig
     - 変動レイテンシの読み取りおよびバースト機能を持つ構成を返します


.. code-block:: scala

   // バースト機能とバイトイネーブルを備えた書き込み専用 AvalonMM 構成を作成します
   val myAvalonConfig =  AvalonMMConfig.bursted(
                           addressWidth = addressWidth,
                           dataWidth = memDataWidth,
                           burstCountWidth = log2Up(burstSize + 1)
                         ).copy(
                           useByteEnable = true,
                           constantBurstBehavior = true,
                           burstOnBurstBoundariesOnly = true
                         ).getWriteOnlyConfig

   // この構成を使用して AvalonMM バスのインスタンスを作成します
   val bus = AvalonMM(myAvalonConfig)
