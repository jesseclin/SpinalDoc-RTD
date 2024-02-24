Parametrization
==================

パラメータ化には複数の側面があります：

- スパイナル HDL の設計エラボレーション中に提供される、
  および管理されるエラボレーション時間パラメータ
- パラメータデータを使用して、設計者が設計に必要な任意の種類のハードウェア構築、
  構成、および相互接続タスクを実行できるようにすること。ハードウェア設計内でのオプションのコンポーネント生成など。

Verilog モジュールパラメータや VHDL ジェネリックなどの HDL 機能の目標との類似点があります。
SpinalHDL は、Scala の型安全性と SpinalHDLの 組み込み HDL 設計ルールチェックの追加保護を備えた、
より豊富で強力な機能セットをこの領域にもたらします。

コンポーネントのパラメータ化のための SpinalHDL のメカニズムは、ネイティブ HDL メカニズムの上に構築されていないため、
手書きの HDL で実現できることに関する HDL 言語レベル/バージョンのサポートや制限の影響を受けません。

パラメータ化された Verilog やジェネリック化された VHDL との相互運用を望む読者は、
プロジェクトが必要とするシナリオに関する :ref:`BlackBox <BlackBox>` IP のセクションを参照してください。


エラボレーション時間パラメータ
------------------------------------------

エラボレーション時間パラメータを提供するために、Scala の完全な構文を使用できます。

「完全な構文」とは、Scala 言語のすべての機能と機能セットを利用して、
プロジェクトのパラメータ化要件を解決するための複雑さのレベルを選択できることを意味します。

SpinalHDL は、パラメータ化の目標を達成する方法について、特定の意見を持つ制限を課しません。
そのため、さまざまなパラメータ管理シナリオに適した多くの Scala デザインパターンと、
いくつかの SpinalHDL ヘルパーが使用できます。

以下は、可能性のいくつかの例とアイデアです：

- ハード配線されたコードと定数（厳密にはパラメータ管理ではなく、コードの変更であり、
  パラメータデータの変更ではありません）
- Scala の静的定数として提供されるコンパニオンオブジェクトからの定数値
- Scala クラスのコンストラクタに提供される値、
  通常は Scala にそれらのコンストラクタ引数値を定数としてキャプチャさせる ``case class``
- 普通の Scala フロー制御構文。条件付き、ループ、ラムダ/モナド、その他
- Config クラスパターン（UartCtrlConfig_、SpiMasterCtrlConfigなどのライブラリアイテムに例があります）
- プロジェクトで定義された 'プラグイン' パターン（ VexRiscV_ プロジェクトには、
  結果として得られるCPU IPコアの機能セットを構成するための例があります）
- ファイルまたはネットワークベースのソースからロードされた値と情報。
  標準の Scala/JVM ライブラリとAPIを使用します。
- `作成できる任意のメカニズム`

これらのすべてのメカニズムにより、結果として生成される HDL 出力が変更されます。

これは、単一の定数値の変更から始まり、Scala プログラミングパラダイムから離れることなく、
SoC 全体のバスや相互接続アーキテクチャを記述するまでさまざまです。

以下は、クラスパラメータの例です。

.. code-block:: scala

  case class MyBus(width : Int) extends Bundle {
    val mySignal = UInt(width bits)
  }  
  
.. code-block:: scala

  case class MyComponent(width : Int) extends Component {
    val bus = MyBus(width)
  }

また、Scala オブジェクトで定義されたグローバル変数（companion object pattern）も使用できます。

また、設定には :ref:`ScopeProperty <scopeproperty>` も使用できます。


オプションのハードウェア
------------------------------------------

ここでは、より多くの可能性があります。

.. _generate:

オプションのシグナルの場合：

.. code-block:: scala

  case class MyComponent(flag : Boolean) extends Component {
    val mySignal = flag generate (Bool())
    // "val mySignal = if (flag) Bool() else null" と同等
  }

``generate`` メソッドは、後に続く式をオプションの値として評価するメカニズムです。
述語が true の場合、generate は指定された式を評価して結果を返し、それ以外の場合は null を返します。

これは、SpinalHDL のハードウェア記述をエラボレーション時の条件式を使用してパラメータ化する場合に使用されます。
結果として生成される HDL に HDL 構造を出力するかどうかを決定します。 
generate メソッドは、SpinalHDL のシンタックスシュガーと見なすことができ、言語の混乱を減らします。

プロジェクト SpinalHDL コードで ``mySignal`` を参照する場合、null の可能性を適切に処理する必要があります。
これは通常、設計のその部分も ``flag`` の値に応じて省略されるため、問題ありません。
したがって、このコンポーネントのパラメータ化の機能が示されます。


同じことが Bundle でもできます。

また、scalaの Option も使用できます。

ハードウェアの生成を無効にする場合：

.. code-block:: scala

  case class MyComponent(flag : Boolean) extends Component {
    val myHardware = flag generate new Area {
      //ここにオプションのハードウェア
    }
  }

scala の for ループも使用できます：

.. code-block:: scala

  case class MyComponent(amount : Int) extends Component {
    val myHardware = for(i <- 0 until amount) yield new Area {
      // ハードウェア
    }
  }

したがって、エラボレーション時にこれらの scala の使用法を拡張することができ、
（List、Set、Mapなどの）scala コレクション全体を使用してデータモデルを構築し、
それらを手続き的な方法でハードウェアに変換することができます。

.. _UartCtrlConfig: https://spinalhdl.github.io/SpinalDoc-RTD/master/SpinalHDL/Examples/Intermediates%20ones/uart.html#controller-construction-parameters
.. _VexRiscV: https://github.com/SpinalHDL/VexRiscv#plugins
