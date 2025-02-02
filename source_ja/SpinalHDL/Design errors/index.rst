=============
Design errors
=============

SpinalHDL コンパイラは、生成される VHDL/Verilog がシミュレーションと合成に安全であることを確認するために、
デザインに多くのチェックを実行します。
基本的に、壊れた VHDL/Verilog 設計を生成することはできません。
以下は、SpinalHDLのチェックの非網羅的なリストです：

* 代入の重複
* クロックのクロッシング
* 階層違反
* 組合せループ
* ラッチ
* 駆動されていない信号
* 幅の不一致
* アンリーチャブルなスイッチ文

SpinalHDL のエラーレポートには、デザインエラーの正確な場所を特定するのに役立つスタックトレースが含まれています。
これらのデザインチェックは初めて見たときには過剰なように思えるかもしれませんが、
従来のハードウェア記述方法から離れ始めるとますます貴重になります。

.. toctree::
   :glob:

   *

