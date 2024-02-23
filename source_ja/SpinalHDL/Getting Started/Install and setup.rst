.. _Install:

Install and setup
=================

SpinalHDL は Scala (Java 仮想マシンを使用するプログラミング言語) のライブラリなので、
Scala 環境の準備が必要です。方法は色々ありますが、
ここでは *SpinalHDL による記述からシミュレーションまで* をスムーズに進めるための公式推奨環境をご紹介します。
もちろん、他にも様々な変形・拡張可能性がありますので、後々自由にいじり倒してもらえればと思います！


必須/推奨ツール
--------------------------

SpinalHDL ツールをダウンロードする前に、Scala 環境をインストールする必要があります:

* `Java JDK <https://www.oracle.com/java/technologies/downloads/>`_, Java 環境
* `Scala 2 <https://www.scala-lang.org/>`_, コンパイラとライブラリ
* `SBT <https://www.scala-sbt.org/download.html>`_, Scala ビルド ツール

SpinalHDL はこれらのツールと連携することで有効化されるが、単独では HDL コード生成機能に留まる。

機能を拡張し、開発を快適にするための推奨ツールを紹介します:

* IDE:

  * `IntelliJ <https://www.jetbrains.com/idea/>`_ (Scalaプラグイン付き)
  * `VSCodium <https://vscodium.com/>`_ (Metals拡張機能付き)
  
  これら IDE は、次のような機能を提供します:

  * コード補完・提案 
  * 構文エラーのリアルタイム表示と自動ビルド
  * ワンクリックでのコード生成
  * テスト/シミュレーションのワンクリック実行 (対応シミュレータ必要)

* シミュレータ:

  `Verilator <https://www.veripool.org/verilator/>`_: SpinalHDL から直接デザインのテストが可能

* 波形ビューア:

  `Gtkwave <https://gtkwave.sourceforge.net/>`_: Verilator のシミュレーション結果を波形で表示

* バージョン管理システム:

  `Git <https://git-scm.com/>`_

* C++ツールチェーン (Verilatorシミュレーションに必要)

* Linuxシェル (Verilatorシミュレーションに必要)

これらのツールを使うことで、SpinalHDL開発をより効率的かつ快適に進められます!


Linuxのインストール
------------------


最新の Scala 環境構築 (執筆時点)

Scala と SBT の最新かつおすすめなインストール方法は、 
`Coursier <https://get-coursier.io/docs/cli-installation>`_.を使うことです。
Coursier は scala ツールに加えて、使用する Java JDK もインストール可能です。

今回、パッケージマネージャーを使用して Java のインストールに進みます。
使用される Scala バージョンとの互換性を考慮し、JDK 17 (LTS) のインストールをおすすめします。


Debian または Ubuntu の場合は:

.. code-block:: sh

   sudo apt-get update
   sudo apt-get install openjdk-17-jdk-headless curl git
   curl -fL "https://github.com/coursier/launchers/raw/master/cs-x86_64-pc-linux.gz" | gzip -d > cs
   chmod +x cs
   # should find the just installed jdk, agree to cs' questions for adding to your PATH
   ./cs setup
   source ~/.profile

シミュレーションや形式的証明を行う場合は、
`oss-cad-suite <https://github.com/YosysHQ/oss-cad-suite-build>`_ を使うことをおすすめします。
これは、波形ビューアー (gtkWave)、Verilog シミュレータ (verilator や iverilog)、
VHDL シミュレータ (GHDL) などを含むツールセットです。

自分でツールをビルドしたい場合は、従来のシミュレーションツールに関する :ref:`installation instructions <sim backend install>` を参照してください。

インストール手順としては、まず必要な C++ ツールチェーンをインストールし、oss-cad-suite をダウンロードします。
使用するシェルごとに、oss-cad-suite 環境を読み込む必要があります。
ただし、oss-cad-suite には Python 3 インタープリタが含まれており、
恒久的にロードするとシステムの Python インストールと競合する可能性がある点に注意してください。

最新のバージョンを入手するには、oss-cad-suite の リリースページ 
`release page <https://github.com/YosysHQ/oss-cad-suite-build/releases/latest>`_ にアクセスし、ダウンロードリンクを取得してください。
ダウンロードした oss-cad-suite は、任意のフォルダに展開できます。
(最後にテストした oss-cad-suite のバージョンは 2023-10-22 ですが、より新しいバージョンも正常に動作する可能性が高いです)


.. code-block:: sh

   sudo apt-get install make gcc g++ zlib1g-dev
   curl -fLO <download link>
   tar xzf <file that you downloaded>

oss-cad-suite を使うには、各シェルで環境を読み込む必要があります。例えば ``source <path to oss-cad-suite>/environment``。


Mac OS Xのインストール
----------------------
Homebrew を使用して Mac OS X にインストールできます。

デフォルトでは、Homebrew は Java 21 をインストールしますが、
SpinalHDL チュートリアルは SpinalTemplateSbt は Scala バージョン 2.12.16 を使用しますが、
これは Java 21 ではサポートされていません (推奨の LTS バージョンは Java 17 です, https://thatjdk.com/ )。 

Java バージョン 1.7 をインストールするには、次のようにします。

.. code-block:: sh

    brew install openjdk@17

そして、これをパスに追加します。

.. code-block:: sh

   export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"

Java の複数のバージョンを管理するには、jenv をインストールすることも不可欠です。

.. code-block:: sh

   brew install jenv

Jenv はこれらの行を .bash_profile に追加しました

.. code-block:: sh

   export PATH="$HOME/.jenv/bin:$PATH"
   eval "$(jenv init -)"

次に、scala の対話型ビルド ツール sbt をインストールする必要があります。

.. code-block:: sh

    brew install sbt

これでうまくいく場合は、お知らせください。
これがうまくいかない場合は、Mac o SX のインストールに関する github の問題をここで読むことができます。
https://github.com/SpinalHDL/SpinalHDL/issues/1216

シミュレーションや正式な証明用のツールをインストールする場合は、 `oss-cad-suite <https://github.com/YosysHQ/oss-cad-suite-build>`_ をお勧めします。


Windowsのインストール
--------------------

.. note::
   ネイティブ インストールも可能ですが、より簡単で現在推奨されている方法は、Windows で WSL を使用することです。
   WSL を使用する場合は、選択したディストリビューションである `it <https://learn.microsoft.com/en-us/windows/wsl/install>`_ 
   をインストールします。
   Linux のインストール手順に従ってください。 WSL インスタンス内のデータには、 ``\\wsl$`` の下のウィンドウからアクセスできます。
   IntelliJ を使用したい場合は、Linux バージョンを WSL にダウンロードする必要があります。
   VSCode を使用したい場合は、Windows をダウンロードする必要があります。
   このバージョンを使用して、WSL でリモート編集できます。


この記事の執筆時点では、Scala と SBT をインストールする推奨方法は、 `Coursier <https://get-coursier.io/docs/cli-installation>`_ を使用することです。
Coursier は、scala ツールに加えて、使用する Java JDK をインストールできます。
以下の例では、Java を手動でインストールします。Scalaとの互換性により、JDK 17 (LTS) をインストールすることをお勧めします。


まず、 `Adoptium JDK 17 <https://adoptium.net/temurin/releases/?os=windows&version=17>`_ をダウンロードしてインストールします。
`Coursier インストーラー <https://github.com/coursier/launchers/raw/master/cs-x86_64-pc-win32.zip>`_ をダウンロードして解凍し、実行します。
``PATH`` 変数の更新に同意するよう求められたら、再起動して ``PATH`` を強制的に更新します。

ハードウェアを生成するにはこれで十分です。シミュレーションの場合は、以下のいずれかを選択して続行します。
ツールを自分で構築したい場合は、従来のシミュレーション ツール :ref:`installation instructions <sim backend install>` を参照してください。

.. note::
   SpinalHDL のメンテナである `Readon <https://github.com/Readon>` が提供するオールインワン ソリューションは、
   SymbiYosys を介した Verilator シミュレーションと正式な検証を備えた SpinalHDL のインストールと実行に利用できます。
   `it <https://github.com/Readon/msys2-installer/releases>`_ をダウンロードし、ディスク上の任意の場所に環境をインストールします。
   **スタート** メニューの MSYS2-MINGW64 アイコンをクリックしてビルド環境を開始し、MSYS2 のデフォルト コンソールを使用します。
   別の方法は、Windows ターミナルまたは Tabby のようなアプリケーションを使用し、スタートアップ コマンド ``%MSYS2_ROOT%\msys2_shell.cmd -defterm -here -no-start -mingw64`` を使用することです。 msys2 がインストールされている場所です。
   オフラインで使用する場合は、プロジェクトが依存するライブラリを慎重に選択する必要があることに注意してください。
   選択しない場合は、パッケージを手動でダウンロードする必要があります。
   詳細については、リポジトリの README を参照してください。
   

シミュレーション用の MSYS2 verilator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`MSYS2 <https://www.msys2.org>` を通じてコン​​パイラ/ベリレータをインストールすることをお勧めします。
gcc/make/shell をインストールする他の方法 (chocolatey、scoop など) も機能する可能性がありますが、テストされていません。

SpinalHDL メンテナ `Readon <https://github.com/Readon>` は、デフォルトの MSYS2 フォークをメンテナンスしています。
必要なすべての正式に利用可能なパッケージとカスタム ビルドのパッケージをインストールします (Readon の `こちら <https://github.com/Readon/MINGW-SpinalHDL>`」 によっても保守されています)
シミュレーションと正式な検証用。これは「ここ <https://github.com/Readon/msys2-installer>」 にあります。
使用した場合、``pacman`` 経由で以下にインストールされるパッケージはすでにインストールされており、
インストール手順は省略できます。

現在、verilator 4.228 が動作することがわかっている最新の利用可能なバージョンです。

最新のインストーラーをダウンロードし、デフォルト設定で MSYS2 をインストールします。 MSYS2 端末は次の場所で入手できます。
インストールの最後で、以下を実行します:

.. code-block:: sh

   pacman -Syuu
   # will (request) close down terminal
   # open 'MSYS2 MINGW64' from start menu
   pacman -Syuu
   pacman -S --needed base-devel mingw-w64-x86_64-toolchain mingw-w64-x86_64-iverilog mingw-w64-x86_64-ghdl-llvm git 
   curl -O https://repo.msys2.org/mingw/mingw64/mingw-w64-x86_64-verilator-4.228-1-any.pkg.tar.zst
   pacman -U mingw-w64-x86_64-verilator-4.228-1-any.pkg.tar.zst   

MSYS2 MINGW64 ターミナルでは、Java/sbt を使用できるようにするためにいくつかの環境変数を設定する必要があります (これらの変数は設定できます)
(設定を永続化するには、MSYS2 の ``~/.bashrc`` に追加します):


.. code-block:: sh

   export VERILATOR_ROOT=/mingw64/share/verilator/
   export PATH=/c/Program\ Files/Eclipse\ Adoptium/jdk-17.0.8.101-hotspot/bin:$PATH
   export PATH=/c/Users/User/AppData/Local/Coursier/data/bin:$PATH

これにより、MSYS2 ターミナルから sbt/verilator シミュレーションを実行できるようになります 
(sbt は ``sbt.bat`` の呼び出しを介して実行されます)。

正式な検証用の MSYS2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

上記の手順に加えて、yosys、sby、z3、yices もインストールする必要があります。 yosys(yosys-smtbmc動作可能)とsbyの両方が可能です。
公式の MSYS2 パッケージとしては利用できませんが、パッケージは `Readon <https://github.com/Readon>` によって提供されています。
インストーラーを使用した場合、これらの手順は必要ありません (利用可能な新しいパッケージがあるかどうかを確認する必要があります)。

.. code-block:: sh

   pacman -S mingw-w64-x86_64-z3 mingw-w64-x86_64-yices mingw-w64-x86_64-autotools mingw-w64-x86_64-python3-pip
   python3 -m pip install click
   curl -OL https://github.com/Readon/MINGW-SpinalHDL/releases/download/v0.4.9/mingw-w64-x86_64-yosys-0.31-1-any.pkg.tar.zst
   curl -OL https://github.com/Readon/MINGW-SpinalHDL/releases/download/v0.4.9/mingw-w64-x86_64-python-sby-0.31-1-any.pkg.tar.zst
   pacman -U *-yosys-*.pkg.tar.*
   pacman -U *-python-sby-*.pkg.tar*

OCIコンテナ
-------------

SpinalHDL 開発用のコンテナも利用できます。
コンテナは ``ghcr.io/spinalhdl/docker:master`` でホストされており、Docker/Podman/Github コードスペースで使用できます。
これは SpinalHDL CI 回帰に使用されるため、CI コマンドをローカルで簡単に実行できます。

コンテナを実行するには、次のように実行します。 SpinalHDL プロジェクトの ``podman run -v .:/workspace -it ghcr.io/spinalhdl/docker:master``
ルートディレクトリを作成し、プロジェクトディレクトリを ``/workspace`` で利用できるようにします。

コンテナーのインストール方法については、ディストリビューション (Linux、WSL) または Docker (Windows) のドキュメントを参照してください。
使用するランタイム。複数のエディター/IDE (VSCode、IntelliJ、Neovide など) により、コンテナーでのリモート開発が可能になります。
リモート開発の方法については、エディタのドキュメントを参照してください。


インターネットのない Linux 環境に SBT をインストールする
----------------------------------------------------

.. note::
   エアギャップ環境を使用していない場合は、次のことをお勧めします。
   通常の Linux インストールを実行します。 (これは、
   エアギャップ環境用の設置)
   
通常、SBT はオンライン リポジトリを使用して、プロジェクトの依存関係をダウンロードしてキャッシュします。
このキャッシュはいくつかのフォルダーにあります:

* ``~/.sbt``
* ``~/.cache/JNA``
* ``~/.cache/coursier``

インターネットのない環境をセットアップするには、次のことができます:

#. インターネット環境をセットアップします (上記を参照)
#. Spinal コマンドを起動して (:ref:`SBT の使用` を参照) 依存関係をフェッチします 
   (たとえば、 `getting started <https://github.com/SpinalHDL/SpinalTemplateSbt>`_ リポジトリを使用します。)。
#. キャッシュをインターネットのない環境にコピーする


.. _template:

最初の SpinalHDL プロジェクトを作成する
================================

私たちは、すぐに使えるプロジェクトを `getting started <https://github.com/SpinalHDL/SpinalTemplateSbt>`_ リポジトリ」に用意しました。

`これ <https://codeload.github.com/SpinalHDL/SpinalTemplateSbt/zip/master>`_ をダウンロードするか、複製することができます。

次のコマンドは、プロジェクトを ``MySpinalProject`` という名前の新しいディレクトリにクローンし、新しい ``git`` 履歴を初期化します:

.. code-block:: sh

   git clone --depth 1 https://github.com/SpinalHDL/SpinalTemplateSbt.git MySpinalProject
   cd MySpinalProject
   rm -rf .git
   git init
   git add .
   git commit -m "Initial commit from template"


プロジェクトのディレクトリ構造
------------------------------------

.. note::

   ここで説明する構造はデフォルトの構造ですが、簡単に変更できます。


プロジェクトのルートには次のファイルがあります:

================== ===========================================================
File               Description
================== ===========================================================
``build.sbt``      ``sbt`` の Scala 設定
``build.sc``       ``sbt`` の代替となる ``mill`` の Scala 設定
``hw/``            ハードウェアの説明が含まれるフォルダー
``project/``       Scala 構成の詳細
``README.md``      プロジェクトを説明する ``text/markdown`` ファイル
``.gitignore``     バージョン管理で無視するファイルのリスト
``.mill-version``  ``mill`` の詳細な設定
``.scalafmt.conf`` コードを自動フォーマットするためのルールの構成
================== ===========================================================

ご想像のとおり、ここで興味深いのは ``hw/``です。
これには、IP 用の ``spinal/``, ``verilog/`` 、 ``vhdl/`` と、Spinal で生成された IP 用の ``gen/`` の 4 つのフォルダーが含まれています。

``hw/spinal/`` には、プロジェクト名にちなんで名付けられたフォルダが含まれています。
この名前は ``build.sbt`` (会社名とともに) および ``build.sc`` に設定する必要があります。そして
それは ``.scala`` ファイルの先頭にある ``package yourprojectname`` 内にあるものでなければなりません。

``hw/spinal/yourprojectname/`` には、IP の説明、シミュレーションテストと正式なテストが含まれます。
そして ``Config.scala`` があり、これには ``Spinal`` の設定があります。

.. note::

   ``sbt`` は、プロジェクトのルート、つまり ``build.sbt`` を含むフォルダ内で **のみ** 使用する必要があります。


SpinalHDL コードで Spinal を使用する
---------------------------------------

このチュートリアルでは、開発環境に応じて SpinalHDL コードで Spinal を使用する方法を示します。

* :ref:`Using SBT`
* :ref:`Using VSCodium`
* :ref:`Using IntelliJ`
