
UART
====

UART プロトコルは、たとえば RS232 / RS485 フレームの送受信に使用できます。

以下は、パリティなしで 1ストップビットの 8ビットフレームの例です：

.. image:: /asset/picture/uart.png

Bus definition
--------------

.. code-block:: scala

   case class Uart() extends Bundle with IMasterSlave {
     val txd = Bool() // フレームを送信するために使用
     val rxd = Bool() // フレームを受信するために使用

     override def asMaster(): Unit = {
       out(txd)
       in(rxd)
     }
   }

UartCtrl
--------

ライブラリには、Uart コントローラーが実装されています。
このコントローラーは、 ``rxd`` ピンを読み取るためにサンプリングウィンドウを使用し、
その値をフィルタリングするために多数決を使用するという特異性があります。

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 10

   * - IO name
     - direction
     - type
     - Description
   * - config
     - in
     - UartCtrlConfig
     - コントローラーのクロック分周器/パリティ/ストップ/データ長を設定するために使用されま
   * - write
     - slave
     - Stream[Bits]
     - フレームの送信を要求するために使用されるストリームポート
   * - read
     - master
     - Flow[Bits]
     - デコードされたフレームを受信するために使用されるフローポート
   * - uart
     - master
     - Uart
     - 実際の世界とのインタフェース


このコントローラーは、 ``UartCtrlGenerics`` 設定オブジェクトを介してインスタンス化できます：

.. list-table::
   :header-rows: 1
   :widths: 1 1 10

   * - Attribute
     - type
     - Description
   * - dataWidthMax
     - Int
     - フレーム内の最大ビット数
   * - clockDividerWidth
     - Int
     - 内部クロック分周器の幅
   * - preSamplingSize
     - Int
     - UART ボーレートの最初のサンプリング時にドロップされる samplingTick の数を指定します
   * - samplingSize
     - Int
     - UARTボーレートの中間で ``rxd`` の値をサンプリングするために使用される samplingTick の数を指定します
   * - postSamplingSize
     - Int
     - UART ボーレートの最後にドロップされるサンプリングTickの数を指定します

