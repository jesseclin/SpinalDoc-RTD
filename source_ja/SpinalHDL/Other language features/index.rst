=======================
Other language features
=======================

SpinalHDL のコアとなる部分は、さまざまな機能の構文を定義しています:

* 型とリテラル
* レジスタとクロックドメイン
* コンポーネントとエリア
* RAM/ROM
* When / Switch / Mux
* BlackBox ( VHDL や Verilog で記述された既存の IP を利用するための機能)
* SpinalHDL から VHDL への変換: SpinalHDL で記述された回路を VHDL 形式に変換する機能

これらの機能を利用して、デジタル回路を定義したり、強力なライブラリや抽象化を作成することができます。

SpinalHDL が他の一般的な HDL と比べて大きな利点の一つは、コンパイラの詳細な知識がなくても言語を拡張できることです。

この利点を活用した良い例として、 :ref:`SpinalHDL lib <lib_introduction>` があります。このライブラリは、ユーティリティ、ツール、バス、設計手法など、さまざまな機能を提供します。

以降の章で紹介される機能を使用するには、ソースコードで ``import spinal.core._`` をインポートする必要があります


.. toctree::
   :hidden:

   utils
   stub
   assertion
   report
   scope_property
   analog_inout
   vhdl_generation
