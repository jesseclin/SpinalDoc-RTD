.. _lib_introduction:

=========
Libraries
=========

spinal.lib パッケージの目標は次のとおりです：

* ハードウェア設計で一般的に使用されるものを提供する（FIFO、クロック交差ブリッジ、便利な機能）
* シンプルなペリフェラルを提供する（UART、JTAG、VGA など）
* いくつかのバス定義を提供する（Avalon、AMBA など）
* いくつかの方法論を提供する（Stream、Flow、Fragment）
* spinal のスピリットを理解するための例を提供する
* いくつかのツールと施設を提供する（レイテンシアナライザー、QSys コンバーターなど）

次の章で紹介される機能を使用するには、ほとんどの場合、ソースコードで ``import spinal.lib._`` を行う必要があります。

.. important::
   | このパッケージは現在開発中です。文書化された機能は安定しているとは限りません。
   | 提案/バグ修正/機能強化のために GitHub を利用してください。

.. toctree::
   :hidden:

   utils
   stream
   flow
   fragment
   fsm
   vexriscv
   bus_slave_factory
   fiber
   binarySystem
   regIf
   Bus/index
   Com/index
   IO/index
   Graphics/index
   EDA/index
   Pipeline/index
   Misc/index

