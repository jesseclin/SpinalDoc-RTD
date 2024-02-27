
Setup and installation of Verilator
===================================

.. note::
   SpinalHDL の :ref:`セットアップ <Install>` 中に推奨される oss-cad-suite をインストールした場合、
   以下の手順はスキップできますが、oss-cad-suite 環境をアクティブにする必要があります。

SpinalSim + Verilator は、Linux および Windows の両方でサポートされています。

v4.218 が最も古い Verilator のバージョンとして推奨されています。古い Verilator バージョンを使用することは可能ですが、
SpinalHDL が使用するいくつかのオプションと Scala ソース依存の機能（Verilogの ``$urandom`` サポートなど）は、
古い Verilator バージョンではサポートされない場合があり、シミュレーション時にエラーが発生する可能性があります。

理想的には、最新の v4.xxx および v5.xxx が十分にサポートされ、問題がある場合はバグレポートを開いてください。

Scala
^^^^^

``build.sbt`` ファイルに次の行を追加することを忘れないでください：

.. code-block:: scala

   fork := true

また、Scala のテストベンチには常に次のインポートが必要です：

.. code-block:: scala

   import spinal.core._
   import spinal.core.sim._

Linux
^^^^^

Verilator の最新バージョンが必要です:

.. code-block:: sh

   sudo apt-get install git make autoconf g++ flex bison  # 最初に必要なパッケージ
   git clone http://git.veripool.org/git/verilator   # 初回のみ
   unsetenv VERILATOR_ROOT  # csh の場合 (エラーが出ても無視)
   unset VERILATOR_ROOT  # bash の場合
   cd verilator
   git pull             # 最新版に更新します
   git checkout v4.218  # より新しいバージョン v4.228 または v5.xxx を使用することもできます
   autoconf             # ./configure スクリプトを作成します
   ./configure
   make -j$(nproc)
   sudo make install
   echo "DONE"

Windows
^^^^^^^

SpinalSim + Verilator を Windows で動作させるためには、以下の手順を実行する必要があります：

* `MSYS2 <https://www.msys2.org/>`_ をインストールします
* MSYS2 を使用して gcc/g++/verilator を取得します（Verilator の場合、ソースからコンパイルすることもできます）
* MSYS2 の ``bin`` と ``usr\bin`` を Windows の ``PATH`` に追加します（例: ``C:\msys64\usr\bin;C:\msys64\mingw64\bin``）
* JAVA_HOME 環境変数が JDK のインストールフォルダを指していることを確認します（例: ``C:\Program Files\Java\jdk-13.0.2``）

これで、MSYS2 を使用せずに Scala プロジェクトから SpinalSim + Verilator を実行できるはずです。

MSYS2 MinGW 64-bit を新規にインストールした場合、MSYS2 MinGW 64-bit シェル内で以下のコマンドを実行する必要があります（コマンドを1つずつ入力してください）：

From the MinGW package manager
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: sh

   pacman -Syuu
   # MSYS2 シェルを閉じるように求められたら、閉じてください
   pacman -Syuu
   pacman -S --needed base-devel mingw-w64-x86_64-toolchain \
                      git flex\
                      mingw-w64-x86_64-cmake

   pacman -U http://repo.msys2.org/mingw/x86_64/mingw-w64-x86_64-verilator-4.032-1-any.pkg.tar.xz
   
   # Windows の PATH に C:\msys64\usr\bin;C:\msys64\mingw64\bin を追加します
   
From source
~~~~~~~~~~~

.. code-block:: sh

   pacman -Syuu
   # MSYS2 シェルを閉じるように求められたら、閉じてください
   pacman -Syuu
   pacman -S --needed base-devel mingw-w64-x86_64-toolchain \
                      git flex\
                      mingw-w64-x86_64-cmake

   git clone http://git.veripool.org/git/verilator  
   unset VERILATOR_ROOT
   cd verilator
   git pull        
   git checkout v4.218   # より新しいバージョン v4.228 または v5.xxx を使用することもできます
   autoconf      
   ./configure
   export CPLUS_INCLUDE_PATH=/usr/include:$CPLUS_INCLUDE_PATH
   export PATH=/usr/bin/core_perl:$PATH
   cp /usr/include/FlexLexer.h ./src

   make -j$(nproc)
   make install
   echo "DONE"
   # Windows の PATH に C:\msys64\usr\bin;C:\msys64\mingw64\bin を追加します

.. important::
   ``PATH`` 環境変数が JDK 1.8 を指しており、JRE のインストールが含まれていないことを確認してください。

.. important::
   MSYS2 の ``bin`` フォルダを Windows の ``PATH`` に追加すると、副作用が生じる可能性があります。
   これは、 ``PATH`` の優先順位を下げるため、安全性のために最後の要素として追加することが推奨されます。
