==========
Simulation
==========

いつも通り、SpinalHDL が生成した VHDL/Verilog をシミュレートするために標準のシミュレーションツールを使用できます。
ただし、SpinalHDL 1.0.0 以降、この言語は Scala でテストベンチを記述し、ハードウェアを直接テストするための API を統合しています。
この API には、DUT 信号の読み書き、シミュレーションプロセスの分岐と結合、待機、特定の条件が満たされるまで待機する機能が備わっています。
そのため、SpinalHDL のシミュレーション API を使用すると、最も一般的な Scala ユニットテストフレームワークと簡単にテストベンチを統合できます。

ユーザー定義のコンポーネントをシミュレートするために、SpinalHDL は外部 HDL シミュレーターをバックエンドとして使用しています。
現在、次の4つのシミュレーターがサポートされています：

- `Verilator <https://www.veripool.org/verilator/>`_
- `GHDL <http://ghdl.free.fr/>`_ **(SpinalHDL 1.4.1から実験的にサポート)**
- `Icarus Verilog <https://steveicarus.github.io/iverilog/>`_ **(SpinalHDL 1.4.1から実験的にサポート)**
- `VCS <https://www.synopsys.com/verification/simulation/vcs.html>`_ **(SpinalHDL 1.7.0から実験的にサポート)**
- `XSim <https://www.google.com/search?q=site%3Axilinx.com+xsim>`_ **(SpinalHDL 1.7.0から実験的にサポート)**


外部 HDL シミュレーターを使用すると、SpinalHDL のコードベースの複雑さを増やさずに生成された HDL ソースを直接テストできます。

.. toctree::
   :hidden:
   
   install/index
   bootstraps
   signal
   clock
   threadFull
   threadLess
   sensitive
   simulator_specifics
   engine
   examples/index
