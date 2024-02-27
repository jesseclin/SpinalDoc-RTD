
Setup and installation of Icarus Verilog
========================================

.. note::
   SpinalHDL の :ref:`セットアップ <Install>` 中に推奨される oss-cad-suite をインストールした場合、
   以下の手順をスキップできますが、oss-cad-suite 環境をアクティブにする必要があります。

ほとんどの最新の Linux ディストリビューションでは、一般に Icarus Verilog の最新バージョンがパッケージシステムを介して利用できます。
Debian ベースのディストリビューションでは、libboost-dev パッケージに含まれる C++ ライブラリ boost-interprocess もインストールする必要があります。
boost-interprocess は、共有メモリ通信インターフェースの生成に必要です。

Linux
^^^^^

.. code-block:: sh

   sudo apt-get install build-essential libboost-dev iverilog


また、Java バージョンに対応する openjdk パッケージもインストールする必要があります。
Windows やソースからのインストールについては、詳細は `<https://iverilog.fandom.com/wiki/Installation_Guide>`_ を参照してください。

