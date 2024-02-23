.. _Using VSCodium:

VSCodium の Spinal の使用
==========================

.. note::
    `VSCodium <https://vscodium.com/>`_ は、Visual Studio Code のオープン ソース ビルドですが、
    Microsoft のダウンロード可能なバージョンにはテレメトリが含まれていません

1 回限りのセットアップ タスクとして、``view`` > ``extensions search`` に移動して "Scala" を検索し、"Scala (Metals)" `拡張機能 <https://marketplace.visualstudio.com/items?itemName=scalameta.metals>`_ をインストールします。 _。

ワークスペースを開きます: ``File`` > ``Open Folder...`` を選択し、
先ほどダウンロードしたフォルダを :ref:`template` で開きます。

もう 1 つの起動方法は、 cd で適切なディレクトリに移動し、 ``codium``と入力することです。

少し待つと、右下に通知ポップアップが表示されます。
corner: "Multiple build definitions found. Which would you like to use?" 
``sbt``をクリックすると別のポップアップが表示されるので、``Import build`` をクリックします。


``sbt bloopInstall`` を実行しながら待ちます。
その後、警告ポップアップが表示されますが、無視してかまいません (今後表示されない)。

``hw/spinal/projectname/MyTopLevel.scala``を見つけて開きます。
少し待って、 Metals によって ``run | debug``行が 各 ``App`` の前に表示される。 
たとえば、 ``object MyTopLevelVerilog`` のすぐ上にある ``run``をクリックします。
あるいは、 ``Menu Bar`` > ``Run`` > ``Run Without Debugging`` を選択することもできます。
どちらのアプローチでもデザイン チェックが実行され、
チェックに合格すると Verilog ファイル ``./hw/gen/MyTopLevel.v`` が生成されます。

VSCodium の SpinalHDL を使用するために必要な作業はこれだけです。
これで、デザイン ルールがチェックされた Verilog および/または VHDL が作成され、
お気に入りの合成ツールへの入力として使用できます。

VSCode 開発環境の使用方法がわかったので、コードを見てみましょう: :ref:`Simple example`。
