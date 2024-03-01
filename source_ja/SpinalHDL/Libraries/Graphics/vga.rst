.. role:: raw-html-m2r(raw)
   :format: html

VGA
===

VGA バス
------------

VGA バスの定義は、Vga バンドルを使用して利用できます。

.. code-block:: scala

   case class Vga (rgbConfig: RgbConfig) extends Bundle with IMasterSlave {
     val vSync = Bool()
     val hSync = Bool()

     val colorEn = Bool()  // フレームがカラーエリア内にあるときにHighになります
     val color = Rgb(rgbConfig)

     override def asMaster() = this.asOutput()
   }

VGA タイミング
----------------

VGA タイミングは、VgaTimings バンドルを使用してハードウェア内でモデル化できます：

.. code-block:: scala

   case class VgaTimingsHV(timingsWidth: Int) extends Bundle {
     val colorStart = UInt(timingsWidth bits)
     val colorEnd = UInt(timingsWidth bits)
     val syncStart = UInt(timingsWidth bits)
     val syncEnd = UInt(timingsWidth bits)
   }

   case class VgaTimings(timingsWidth: Int) extends Bundle {
     val h = VgaTimingsHV(timingsWidth)
     val v = VgaTimingsHV(timingsWidth)

      def setAs_h640_v480_r60 = ...
      def driveFrom(busCtrl : BusSlaveFactory,baseAddress : Int) = ...
   }

VGAコントローラ
-------------------

VGA コントローラが利用可能です。その定義は以下の通りです：

.. code-block:: scala

   case class VgaCtrl(rgbConfig: RgbConfig, timingsWidth: Int = 12) extends Component {
     val io = new Bundle {
       val softReset = in Bool()
       val timings   = in(VgaTimings(timingsWidth))

       val frameStart = out Bool()
       val pixels     = slave Stream (Rgb(rgbConfig))
       val vga        = master(Vga(rgbConfig))

       val error      = out Bool()
     }
     // ...
   }

| ``frameStart`` は、各新しいフレームの開始時に 1サイクルのパルスとなります。
| ``pixels`` は、必要なときに VGA インタフェースにデータを供給する色のストリームです。
| ``error`` は、pixels のトランザクションが必要ですが、何も存在しないときに High になります。
