.. _Using SBT:

SBT で CLI から Spinal を使用する
==============================

まず、前に :ref:`template` でダウンロードしたテンプレートのルートでターミナルを開きます。

コマンドはターミナルから直接実行できます:

.. code-block:: sh

   sbt "firstCommand with arguments" "secondCommand with more arguments"


ただし、 ``sbt`` は起動時間が非常に長いため、対話モードを使用することをお勧めします。

.. code-block:: sh

   sbt

``sbt`` はプロンプトを表示します。 Scala のコンパイルから始めましょう。それは
依存関係を取得するため、初回は時間がかかる場合があります:

.. code-block::

   compile

実際には、単に ``コンパイル`` する必要はなく、必要に応じて自動的に実行されます。 

``sbt`` ツールはコールド スタートからプロジェクト全体をビルドし、その後インクリメンタル ビルドを使用するため、
最初のビルド時間は今後のビルドに比べて少し長くかかります。その時点から可能な限り構築します。 

``sbt`` は、対話型シェル内でのオートコンプリートをサポートし、利用可能なコマンドの検出と使用を支援します。

対話型シェルは ``sbt shell`` で起動することも、コマンドラインから引数なしで ``sbt`` を実行することもできます。


特定の HDL コード生成またはシミュレーションを実行するには、
コマンドは ``runMain`` です。それで ``runMain``、スペース、タブを入力すると、次の結果が得られます:

.. code-block::

   sbt:SpinalTemplateSbt> runMain 
      ;                               projectname.MyTopLevelVerilog
      projectname.MyTopLevelFormal    projectname.MyTopLevelVhdl
      projectname.MyTopLevelSim

オートコンプリートは、実行できるすべてのものを提案します。 Verilog を実行してみましょう。 
たとえば:


.. code-block::

   runMain projectname.MyTopLevelVerilog

ディレクトリ ./hw/gen/ を見てください。新しい ``MyTopLevel.v`` ファイルがあります!

次に、コマンドの先頭に ``~`` を追加します:

.. code-block::

   ~ runMain projectname.MyTopLevelVerilog

これをプリントします:

.. code-block::

   sbt:SpinalTemplateSbt> ~ runMain mylib.MyTopLevelVerilog
   [info] running (fork) mylib.MyTopLevelVerilog
   [info] [Runtime] SpinalHDL v1.7.3    git head : aeaeece704fe43c766e0d36a93f2ecbb8a9f2003
   [info] [Runtime] JVM max memory : 3968,0MiB
   [info] [Runtime] Current date : 2022.11.17 21:35:10
   [info] running (fork) projectname.MyTopLevelVerilog 
   [info] [Runtime] SpinalHDL v1.9.3    git head : 029104c77a54c53f1edda327a3bea333f7d65fd9
   [info] [Runtime] JVM max memory : 4096.0MiB
   [info] [Runtime] Current date : 2023.10.05 19:30:19
   [info] [Progress] at 0.000 : Elaborate components
   [info] [Progress] at 0.508 : Checks and transforms
   [info] [Progress] at 0.560 : Generate Verilog
  [info] [Done] at 0.603
  [success] Total time: 1 s, completed Oct 5, 2023, 7:30:19 PM
  [info] 1. Monitoring source files for projectname/runMain projectname.MyTopLevelVerilog...
  [info]    Press <enter> to interrupt or '?' for more options.

これで、ソースファイルを保存するたびに、 ``MyTopLevel.v`` が再生成されます。
これを行うために、ソース ファイルが自動的にコンパイルされ、lint チェックが実行されます。
こうすることで、ソース ファイルの編集中にほぼリアルタイムでターミナルにエラーを出力できます。

Enter キーを押して自動生成を停止し、Ctrl-D キーを押して ``sbt`` を終了します。

``sbt`` の対話型プロンプトを使用せずに、ターミナルから直接開始することもできます:

.. code-block:: sh

   sbt "~ runMain mylib.MyTopLevelVerilog"


これで環境を使用できるようになりました。コードを調べてみましょう: :ref:`Simple example`。
