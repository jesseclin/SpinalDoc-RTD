
QSysify
=======

QSysify は、SpinalHDL コンポーネントの IO 定義を解析して QSys IP（tclスクリプト）を生成できるツールです。
現在、次のインターフェース機能が実装されています：

* マスター/スレーブ AvalonMM
* マスター/スレーブ APB3
* クロックドメイン入力
* リセット出力
* 割り込み入力
* コンデュイト（最後の手段として使用されます）

例
-------

UART コントローラの場合：

.. code-block:: scala

   case class AvalonMMUartCtrl(...) extends Component {
     val io = new Bundle {
       val bus =  slave(AvalonMM(AvalonMMUartCtrl.getAvalonMMConfig))
       val uart = master(Uart())
     }

     //...
   }

次の ``main`` では、io.bus が AvalonMM として、io.uart がコンデュイトとして、Verilog と QSys TCL スクリプトを生成します：

.. code-block:: scala

   object AvalonMMUartCtrl {
     def main(args: Array[String]) {
       // Verilogを生成する
       val toplevel = SpinalVerilog(AvalonMMUartCtrl(UartCtrlMemoryMappedConfig(...))).toplevel

       // Avalon バスにいくつかのタグを追加して、そのクロックドメインを指定する（QSysifyで使用される情報）
       toplevel.io.bus addTag(ClockDomainTag(toplevel.clockDomain))

       // QSys IP（tclスクリプト）を生成する
       QSysify(toplevel)
     }
   }

タグ
-------

QSys は、SpinalHDL ハードウェア仕様で指定されていない情報が必要ですので、いくつかのタグをインターフェースに追加する必要があります：

AvalonMM / APB3
^^^^^^^^^^^^^^^

.. code-block:: scala

   io.bus addTag(ClockDomainTag(busClockDomain))

Interrupt input
^^^^^^^^^^^^^^^

.. code-block:: scala

   io.interrupt addTag(InterruptReceiverTag(relatedMemoryInterfacei, interruptClockDomain))

Reset output
^^^^^^^^^^^^

.. code-block:: scala

   io.resetOutput addTag(ResetEmitterTag(resetOutputClockDomain))


新しいインターフェースのサポートの追加
-----------------------------------------

基本的に、QSysify ツールは、インターフェース ``emitter`` のリストでセットアップできます `(ここで見ることができます) <https://github.com/SpinalHDL/SpinalHDL/blob/764193013f84cfe4f82d7d1f1739c4561ef65860/lib/src/main/scala/spinal/lib/eda/altera/QSys.scala#L12>`_。

独自のエミッタを作成するには、 `QSysifyInterfaceEmiter <https://github.com/SpinalHDL/SpinalHDL/blob/764193013f84cfe4f82d7d1f1739c4561ef65860/lib/src/main/scala/spinal/lib/eda/altera/QSys.scala#L24>`_ を拡張する新しいクラスを作成します。
