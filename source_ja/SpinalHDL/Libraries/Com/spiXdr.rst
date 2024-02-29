
SPI XDR
========

SPI コントローラーは、以下をサポートしています：

- ハーフ/フルデュプレックス
- シングル/デュアル/クアッドSPI
- SDR/DDR/...データレート

その APB3 の実装はこちらにあります：

You can find its APB3 implementation here : 

https://github.com/SpinalHDL/SpinalHDL/blob/68b6158700fc2440ea7980406f927262c004faca/lib/src/main/scala/spinal/lib/com/spi/xdr/Apb3SpiXdrMasterCtrl.scala#L43

Configuration
----------------------------

以下は例です。

.. code-block:: scala

      Apb3SpiXdrMasterCtrl(
        SpiXdrMasterCtrl.MemoryMappingParameters(
          SpiXdrMasterCtrl.Parameters(
            dataWidth = 8, // 各転送は8ビットになります
            timerWidth = 12, // タイマーは送信を遅くします
            spi = SpiXdrParameter( // 物理的なSPIインターフェースを指定します
              dataWidth = 4, // 物理SPIデータピンの数
              ioRate = 1, // 1クロックあたりの各SPIピンの転送回数を指定します。1 => SDR、2 => DDR
              ssWidth = 1 // チップセレクトの数
            )
          )
          .addFullDuplex(id = 0) // モード ID 0 を使用して通常のSPI（MISO / MOSI）をサポートする
          .addHalfDuplex( // 別のモードを追加
            id = 1,  // モード ID 1 にマップされます
            rate = 1, // rate が 1の場合、クロックごとに 1つのトグルが行われ、（タイマー+1）で割られます
                      // rate が大きい場合（例：2）、コントローラーはタイマーを無視し、SpiXdrParameter.ioRateを使用して
                      // クロックサイクルあたりの最大 "rate" のトランジションを送信します。
            ddr = false, // sdr => SPI クロックごとに 1ビット、DDR => SPI クロックごとに2ビット
            spiWidth = 4 // シリアライゼーションに使用される物理 SPI データピンの数
          ),
          cmdFifoDepth = 32,
          rspFifoDepth = 32,
          xip = null
        )
      )

Software Driver
----------------------------

以下を参照してください：

https://github.com/SpinalHDL/SaxonSoc/blob/dev-0.3/software/standalone/driver/spi.h
https://github.com/SpinalHDL/SaxonSoc/blob/dev-0.3/software/standalone/spiDemo/src/main.c

