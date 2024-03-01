.. role:: raw-html-m2r(raw)
   :format: html

.. _plic_mapper:

Plic Mapper
=================

PLIC Mapper は、PLIC（Platform Level Interrupt Controller）のレジスタ生成とアクセスを定義します。

``PlicMapper.apply``
--------------------

``(bus: BusSlaveFactory, mapping: PlicMapping)(gateways : Seq[PlicGateway], targets : Seq[PlicTarget])``

PlicMapperの

* **bus**: このコントローラがアタッチされているバス
* **mapping**: マッピングの構成（上記参照）
* **gateways**: バスアクセス制御を生成する PlicGateway（割り込みソース）のシーケンス
* **targets**: バスアクセス制御を生成する PlicTarget（たとえば、複数のコア）のシーケンス

これは riscv によって与えられたインターフェースに従います： https://github.com/riscv/riscv-plic-spec/blob/master/riscv-plic.adoc

現時点では、2つのメモリマッピングが利用可能です：

``PlicMapping.sifive``
-----------------------------
SiFive PLIC マッピングに従います（たとえば、 `E31 core complex Manual <https://sifive.cdn.prismic.io/sifive/9169d157-0d50-4005-a289-36c684de671b_e31_core_complex_manual_21G1.pdf>`_）。基本的に、完全な PLIC です。

``PlicMapping.light``
----------------------------
このマッピングでは、いくつかのオプション機能が欠落している軽量な PLIC が生成されます：

* 割り込みの優先度を読み取る機能がない
* 割り込みのペンディングビットを読み取る機能がない（claim/complete メカニズムを使用する必要があります）
* ターゲットのスレッショルドを読み取る機能がない

残りのレジスタとロジックは生成されます。
