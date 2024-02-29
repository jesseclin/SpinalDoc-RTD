
USB OHCI
========

SpinalHDL ライブラリには、USB OHCi コントローラー（ホスト）が存在します。

サポートの要点をまとめると次のとおりです：

- USB の OpenHCI Open Host Controller Interface Specification for USB（OHCI）仕様に従います。
- 既存の Linux / U-Boot OHCI ドライバーと互換性があります（tinyUSB にも OHCI ドライバーがあります）。
- USB ホストフルスピードおよびロースピード機能を提供します（12Mbps および 1.5Mbps）。
- Linux および U-Boot でテスト済み。
- 1つのコントローラーで複数のポートをホストできます（最大16個）。
- DMA アクセス用の Bmb メモリインターフェイス。
- 構成用の Bmb メモリインターフェイス。
- 内部 PHY 用のクロックが必要であり、12 MHz の倍数であり、少なくとも 48 MHz である必要があります。
- コントローラーの周波数に制限はありません。
- 外部 PHY は不要です。

検証済みかつ機能するデバイス：

- マスストレージ（ArtyA7 Linuxで約8 Mbps）
- キーボード/マウス
- オーディオ出力
- ハブ

制限事項：

- 一部の USB ハブ（これまでに 1つありました）は、ロースピードデバイスが接続されているフルスピードホストを受け付けません。
- 一部の最新のデバイスは、USB フルスピードで動作しません（例：Gbps イーサネットアダプター）。
- CPU とメモリの整合性が必要です（またはドライバーでデータキャッシュをフラッシュできる必要があります）。

デプロイメント：

- https://github.com/SpinalHDL/SaxonSoc/tree/dev-0.3/bsp/digilent/ArtyA7SmpLinux
- https://github.com/SpinalHDL/SaxonSoc/tree/dev-0.3/bsp/radiona/ulx3s/smp

Usage
--------------

.. code-block:: scala

    import spinal.core._
    import spinal.core.sim._
    import spinal.lib.bus.bmb._
    import spinal.lib.bus.bmb.sim._
    import spinal.lib.bus.misc.SizeMapping
    import spinal.lib.com.usb.ohci._
    import spinal.lib.com.usb.phy.UsbHubLsFs.CtrlCc
    import spinal.lib.com.usb.phy._

    class UsbOhciTop(val p : UsbOhciParameter) extends Component {
      val ohci = UsbOhci(p, BmbParameter(
        addressWidth = 12,
        dataWidth = 32,
        sourceWidth = 0,
        contextWidth = 0,
        lengthWidth = 2
      ))

      val phyCd = ClockDomain.external("phyCd", frequency = FixedFrequency(48 MHz))
      val phy = phyCd(UsbLsFsPhy(p.portCount, sim=true))

      val phyCc = CtrlCc(p.portCount, ClockDomain.current, phyCd)
      phyCc.input <> ohci.io.phy
      phyCc.output <> phy.io.ctrl

      // 信号を伝播させる
      val irq = ohci.io.interrupt.toIo
      val ctrl = ohci.io.ctrl.toIo
      val dma = ohci.io.dma.toIo
      val usb = phy.io.usb.toIo
      val management = phy.io.management.toIo
    }

    object UsbHostGen extends App {
      val p = UsbOhciParameter(
        noPowerSwitching = true,
        powerSwitchingMode = true,
        noOverCurrentProtection = true,
        powerOnToPowerGoodTime = 10,
        dataWidth = 64, //DMA data width, up to 128
        portsConfig = List.fill(4)(OhciPortParameter()) //4 Ports
      )

      SpinalVerilog(new UsbOhciTop(p))
    }
