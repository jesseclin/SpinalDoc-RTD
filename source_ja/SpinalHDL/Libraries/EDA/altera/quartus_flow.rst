
QuartusFlow
===========

コンパイルフローは、Altera が定義した一連のコマンドのシーケンスであり、
コマンドラインの実行可能ファイルの組み合わせを使用します。
完全なコンパイルフローは、すべてのコンパイラモジュールをシーケンスで起動して、
最終的なタイミングの合成、適合、解析、およびデバイスプログラミングファイルの生成を行います。

`このファイル <https://github.com/SpinalHDL/SpinalHDL/blob/dev/lib/src/main/scala/spinal/lib/eda/altera/QuartusFlow.scala>`_ 内のツールは、Quartus GUI の冗長性を排除するのに役立ちます。


単一のrtlファイルの場合
--------------------------

``spinal.lib.eda.altera.QuartusFlow`` オブジェクトは、単一の rtl ファイルの使用された領域と最大周波数を自動的にレポートします。


例
^^^^^^^

.. code-block:: scala

   val report = QuartusFlow(
      quartusPath="/eda/intelFPGA_lite/17.0/quartus/bin/",
      workspacePath="/home/spinalvm/tmp",
      toplevelPath="TopLevel.vhd",
      family="Cyclone V",
      device="5CSEMA5F31C6",
      frequencyTarget = 1 MHz
   )
   println(report)

上記のコードは、新しい Quartus プロジェクトを ``TopLevel.vhd`` で作成します。

.. warning::
   この操作は、フォルダー ``workspacePath`` を削除します！

.. note::
   ``family`` および ``device`` の値は、Quartus CLI にパラメーターとして直接渡されます。プロジェクトで使用する正しい値については、Quartus のドキュメントを確認してください。

ヒント
^^^^^^^^^

ピンが多すぎるコンポーネントをテストするには、それらを ``VIRTUAL_PIN`` として設定します。

.. code-block:: scala

   val miaou: Vec[Flow[Bool]] = Vec(master(Flow(Bool())), 666)
   miaou.addAttribute("altera_attribute", "-name VIRTUAL_PIN ON")

既存のプロジェクトの場合
--------------------------

クラス ``spinal.lib.eda.altera.QuartusProject`` は、既存のプロジェクト内の設定ファイルを自動的に見つけることができます。
これらは、デバイスのコンパイルおよびプログラミングに使用されます。

例
^^^^^^^

プロジェクトファイル ``.qpf`` および ``.cdf`` を含むパスを指定します。

.. code-block:: scala

   val prj = new QuartusProject(
      quartusPath = "F:/intelFPGA_lite/20.1/quartus/bin64/",
      workspacePath = "G:/"
   )
   prj.compile()
   prj.program()  //プロジェクトの Chain Description File を自動的に検出します

.. important::
   ``prj.program()`` を呼び出す前に、プロジェクトの ``.cdf`` を保存することを忘れないでください。