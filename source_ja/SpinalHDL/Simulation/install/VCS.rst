
VCS Simulation Configuration
==============================

.. _vcs_env:

環境変数
----------------------
以下の環境変数が定義されている必要があります：

* ``VCS_HOME``: VCS のインストール先のホームパス
* ``VERDI_HOME``: Verdi のインストール先のホームパス
* ``$VCS_HOME/bin`` と ``$VERDI_HOME/bin`` をあなたの PATH に追加します。
  
PLI 機能を有効にするために、以下のパスをあなたの ``LD_LIBRARY_PATH`` に追加してください。

.. code-block:: bash

  export LD_LIBRARY_PATH=$VERDI_HOME/share/PLI/VCS/LINUX64:$LD_LIBRARY_PATH 
  export LD_LIBRARY_PATH=$VERDI_HOME/share/PLI/IUS/LINUX64:$LD_LIBRARY_PATH 
  export LD_LIBRARY_PATH=$VERDI_HOME/share/PLI/lib/LINUX64:$LD_LIBRARY_PATH 
  export LD_LIBRARY_PATH=$VERDI_HOME/share/PLI/Ius/LINUX64:$LD_LIBRARY_PATH 
  export LD_LIBRARY_PATH=$VERDI_HOME/share/PLI/MODELSIM/LINUX64:$LD_LIBRARY_PATH 

``Compilation of SharedMemIface.cpp failed`` エラーが発生した場合、
C++ boost ライブラリが正しくインストールされていることを確認してください。
ヘッダーファイルとライブラリファイルのパスは、それぞれ ``CPLUS_INCLUDE_PATH`` 、 
``LIBRARY_PATH``、 ``LD_LIBRARY_PATH`` に追加する必要があります。

ユーザー定義の環境設定
------------------------------

時には、VCS シミュレーションを実行するために ``synopsys_sim.setup`` という VCS 環境設定ファイルが必要になることがあります。
また、VCS のコンパイル開始前に環境を設定するためにスクリプトやコードを実行したい場合があります。
これは ``withVCSSimSetup`` を使用して行うことができます。

.. code-block:: scala
  
  val simConfig = SimConfig
    .withVCS
    .withVCSSimSetup(
      setupFile = "~/work/myproj/sim/synopsys_sim.setup",
      beforeAnalysis = () => { // VCS の解析ステップの前にこのコードブロックが実行されます
        "pwd".!
        println("Hello, VCS")
      }
    )

この方法により、あなた自身の `synopsys_sim.setup`` ファイルが VCS の作業ディレクトリの `workspacePath`（デフォルトは `simWorkspace`）ディレクトリにコピーされ、
スクリプトが実行されます。

VCS フラグ
--------------

VCS バックエンドは、次の三つの段階のコンパイルフローに従います：

1. 解析ステップ： ``vlogan`` および ``vhdlan`` を使用して HDL モデルを解析します。

2. エラボレートステップ： ``vcs`` を使用してモデルをエラボレートし、実行可能なハードウェアモデルを生成します。

3. シミュレーションステップ：シミュレーションを実行します。

各ステップで、ユーザーは SDF バックアノテーションやマルチスレッドなどの特定のフラグを ``VCSFlags`` を介して渡すことができます。

``VCSFlags`` は三つのパラメーターを取ります。


.. list-table::
   :header-rows: 1
   :widths: 2 5 5

   * - Name
     - Type
     - Description
   * - ``compileFlags``
     - ``List[String]``
     - ``vlogan`` または ``vhdlan`` に渡されるフラグ。
   * - ``elaborateFlags``
     - ``List[String]``
     - ``vcs`` に渡されるフラグ。
   * - ``runFlags``
     - ``List[String]``
     - 実行可能なハードウェアモデルに渡されるフラグ。 

たとえば、Verdi デバッグのために ``-kdb`` フラグを両方のコンパイルステップとエラボレーションステップに渡す場合は、次のようにします。

.. code-block:: scala

   val flags = VCSFlags(
       compileFlags = List("-kdb"),
       elaborateFlags = List("-kdb")
   )

   val config = 
     SimConfig
       .withVCS(flags)
       .withFSDBWave
       .workspacePath("tb")
       .compile(UIntAdder(8))

   ...

ウェーブ形式の生成
--------------------

VCS バックエンドは、 ``VCD``、 ``VPD``、 ``FSDB`` （Verdi が必要）の三つのウェーブ形式を生成することができます。

``SpinalSimConfig`` の以下のメソッドを使用してこれらを有効にできます。

.. list-table::
   :header-rows: 1
   :widths: 2 5

   * - Method
     - Description
   * - ``withWave``
     - ``VCD`` ウェーブ形式を有効にします。
   * - ``withVPDWave``
     - ``VPD`` ウェーブ形式を有効にします。
   * - ``withFSDBWave``
     - ``FSDB`` ウェーブ形式を有効にします。

また、 ``withWaveDepth(depth: Int)`` を使用してウェーブトレースの深度を制御することもできます。


``Blackbox`` を使用したシミュレーション
---------------------------------------------

時には、IP ベンダーが Verilog/VHDL フォーマットのデザインエンティティを提供し、それらを SpinalHDL デザインに統合したい場合があります。
統合は、以下の二つの方法で行うことができます：

1. ``Blackbox`` 定義内で、 ``addRTLPath(path: String)`` を使用して外部の Verilog/VHDL ファイルをこのブラックボックスに割り当てます。
2. ``SpinalReport`` の ``mergeRTLSource(fileName: String=null)`` メソッドを使用します。

