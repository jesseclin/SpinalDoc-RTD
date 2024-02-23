.. _Using IntelliJ:


IntelliJ IDEA から Spinal を使用する
===================================

前述の要件に加えて、IntelliJ IDEA もダウンロードする必要があります 
(無料の *コミュニティ エディション* で十分です)。IntelliJ をインストールしたら、その Scala プラグインが有効になっていることも確認してください 
(\ `ここ <https://www.jetbrains.com/help/idea/2016.1/enabling-and-disabling-plugins.html?origin=old_help>`_ で見つけることができます)。

In addition to the aforementioned requirements, you also need to download the IntelliJ IDEA (the free *Community edition* is enough). When you have installed IntelliJ, also check that you have enabled its Scala plugin 
(\ `install information <https://www.jetbrains.com/help/idea/2016.1/enabling-and-disabling-plugins.html?origin=old_help>`_ can be found here).

そして、次のことを実行します:

* *Intellij IDEA*\ で、このリポジトリのルートを使用して「プロジェクトをインポート」し、*Import project from external model SBT* を選択し、すべてのボックスを必ずオンにしてください。
* さらに、JDK を *IntelliJ* にインストールした場所などのパスを指定する必要がある場合があります。
* プロジェクト (Intellij プロジェクト GUI) で、 ``src/main/scala/mylib/MyTopLevel.scala`` を右クリックし、 "Run MyTopLevel" を選択します

これにより、単純な 8 ビット カウンタを実装する出力ファイル ``MyTopLevel.vhd`` がプロジェクト ディレクトリに生成されます。

これで環境を使用できるようになりました。コードを調べてみましょう: :ref:`Simple example`。
