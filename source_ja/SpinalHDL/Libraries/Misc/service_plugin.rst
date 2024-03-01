.. role:: raw-html-m2r(raw)
   :format: html

Plugin
==========================

Introduction
--------------------

ある設計では、コンポーネントのハードウェアを直接実装する代わりに、
いくつかの種類のプラグインを使用してそのハードウェアを構成したい場合があります。
これにより、いくつかの重要な機能が提供されます：

- 新しいプラグインをパラメータに追加することで、コンポーネントの機能を拡張できます。たとえば、CPU に浮動小数点サポートを追加します。
- 同じ機能のさまざまな実装を、別のセットのプラグインを使用して簡単に交換できます。たとえば、ある FPGA に適した CPU 乗算器の1つの実装は、他の ASIC に適したものとは異なる場合があります。
- すべてを手動で接続する必要がある非常に非常に大きな手書きのトップレベルシンドロームを回避します。代わりに、プラグインは他のプラグインのソフトウェアインターフェースを見て/使用して、自分の近隣を発見できます。

VexRiscv と NaxRiscv プロジェクトがこれの例です。これらはほとんど空のトップレベルを持つ CPU で、そのハードウェア部分はプラグインを使用して注入されます。たとえば：

- PcPlugin
- FetchPlugin
- DecoderPlugin
- RegFilePlugin
- IntAluPlugin
- ...

そして、これらのプラグインは、それぞれのサービスプールを介して相互に交渉/伝達/相互接続します。

VexRiscv は厳格な同期 2フェーズシステム（セットアップ/ビルドコールバック）を使用しますが、NaxRiscv は、スパイナルコアファイバー API を使用してエラボレーションスレッドをフォークし、互いに相互ロックして動作可能なエラボレーション順序を確保するより柔軟なアプローチを使用します。

プラグイン API は、プラグインを使用して合成可能なコンポーネントを定義する NaxRiscv のようなシステムを提供します。

Execution order
--------------------

基本的な考え方は、複数の 2つの実行フェーズがあることです：

- セットアップフェーズ：このフェーズでは、プラグインがお互いをロック/保持できます。考え方は、まだ交渉/エラボレーションを開始しないことです。
- ビルドフェーズ：このフェーズでは、プラグインがハードウェアの交渉/エラボレーションを行うことができます。

ビルドフェーズは、すべての FiberPlugin がセットアップフェーズを終えるまで開始されません。

.. code-block:: scala

      class MyPlugin extends FiberPlugin {
        val logic = during setup new Area {
          // こでは、セットアップフェーズでコードを実行しています
          awaitBuild()
          // ここでは、ビルドフェーズでコードを実行しています
        }
      }

      class MyPlugin2 extends FiberPlugin {
        val logic = during build new Area {
          // ここでは、ビルドフェーズでコードを実行しています
        }
      }


Simple example
--------------------

以下は、2 つのプラグインを使用して構成される SubComponent を持つ簡単なダミー例です：

.. code-block:: scala

      import spinal.core._
      import spinal.lib.misc.plugin._

      // プラグインホストインスタンスを持つComponentを定義します
      class SubComponent extends Component {
        val host = new PluginHost()
      }

      // レジスタを作成するプラグインを定義します
      class StatePlugin extends FiberPlugin {
        // during build new Area { body } は、Plugin のコンテキスト Fiber ビルドフェーズでコードを実行します
        val logic = during build new Area {
          val signal = Reg(UInt(32 bits))
        }
      }

      // StatePlugin のレジスタをインクリメントするプラグインを定義します
      class DriverPlugin extends FiberPlugin {
        // PluginHost から StatePlugin.logic のインスタンスを取得する方法を定義します。これは遅延評価される必要があるため、lazy val としています。
        lazy val sp = host[StatePlugin].logic.get

        val logic = during build new Area {
          // インクリメントハードウェアを生成します
          sp.signal := sp.signal + 1
        }
      }

      class TopLevel extends Component {
        val sub = new SubComponent()

        // ここで、プラグインを作成し、sub.host に埋め込みます
        new DriverPlugin().setHost(sub.host)
        new StatePlugin().setHost(sub.host)
      }

このような TopLevel は、以下の Verilog コードを生成します：

.. code-block:: verilog

    module TopLevel (
      input  wire          clk,
      input  wire          reset
    );


      SubComponent sub (
        .clk   (clk  ), //i
        .reset (reset)  //i
      );

    endmodule

    module SubComponent (
      input  wire          clk,
      input  wire          reset
    );

      reg        [31:0]   StatePlugin_logic_signal; //Created by StatePlugin

      always @(posedge clk) begin
        StatePlugin_logic_signal <= (StatePlugin_logic_signal + 32'h00000001); //incremented by DriverPlugin
      end
    endmodule

注意： "during build" はそれぞれのエラボレーションスレッドをフォークします。
DriverPlugin.logic スレッドの実行は、"sp"の評価が完了するまで StatePlugin.logic の実行でブロックされます。

Interlocking / Ordering
----------------------------------------

プラグインは、Retainer インスタンスを使用して互いに相互作用できます。
各プラグインインスタンスには、retain/release 関数を使用して制御できる組み込みのロックがあります。

以下は、上記の `シンプルな例` に基づく例ですが、
この場合、DriverPlugin は StatePlugin.logic.signal を他のプラグイン（今回の場合は SetupPlugin ）によって設定された量だけ増分します。
そして、DriverPlugin がハードウェアを早すぎる段階で生成しないようにするために、SetupPlugin は DriverPlugin.retain/release 関数を使用します。

.. code-block:: scala

  import spinal.core._
  import spinal.lib.misc.plugin._
  import spinal.core.fiber._

  class SubComponent extends Component {
    val host = new PluginHost()
  }

  class StatePlugin extends FiberPlugin {
    val logic = during build new Area {
      val signal = Reg(UInt(32 bits))
    }
  }

  class DriverPlugin extends FiberPlugin {
    // incrementBy は、展開時に他のプラグインによって設定されます。
    var incrementBy = 0
    // retainer は、他のプラグインがロックを作成できるようにし、このプラグインが incrementBy を使用する前に待機します。
    val retainer = Retainer()

    val logic = during build new Area {
      val sp = host[StatePlugin].logic.get
      retainer.await()

      // インクリメンターのハードウェアを生成します
      sp.signal := sp.signal + incrementBy
    }
  }

  // ドライバープラグインの incrementBy 変数を変更するプラグインを定義します。なぜなら、それがハードウェアを詳細に記述することを許可するからです。
  class SetupPlugin extends FiberPlugin {
    // during setup { body } は、コードの本体を Fiber のセットアップフェーズに生成します（これは Fiber ビルドフェーズより前です）。
    val logic = during setup new Area {
      // *** セットアップフェーズのコード ***
      val dp = host[DriverPlugin]

      // DriverPlugin がビルドの本体を実行するのを防止します（release() が呼び出されるまで）。
      val lock = dp.retainer()
      // ファイバーフェーズがビルドフェーズに達するまで待機します。
      awaitBuild()

      // *** ビルドフェーズのコード ***
      // DriverPlugin.incrementBy を変更しましょう。
      dp.incrementBy += 1

      // DriverPlugin がビルドの本体を実行するのを許可します。
      lock.release()
    }
  }

  class TopLevel extends Component {
    val sub = new SubComponent()

    sub.host.asHostOf(
      new DriverPlugin(),
      new StatePlugin(),
      new SetupPlugin(),
      new SetupPlugin() // できるので、第二の SetupPlugin を追加しましょう。
    )
  }

生成されたVerilogは次のようになります。

.. code-block:: verilog

    module TopLevel (
      input  wire          clk,
      input  wire          reset
    );


      SubComponent sub (
        .clk   (clk  ), //i
        .reset (reset)  //i
      );

    endmodule

    module SubComponent (
      input  wire          clk,
      input  wire          reset
    );

      reg        [31:0]   StatePlugin_logic_signal;

      always @(posedge clk) begin
        StatePlugin_logic_signal <= (StatePlugin_logic_signal + 32'h00000002); // + 2 as we have two SetupPlugin
      end
    endmodule

これらの例は、行っていることに対してやや過剰ですが、一般的なアイデアは次のようなものです：

- プラグイン間のインターフェイスの交渉/作成（例：ジャンプ/フラッシュポート）
- エラボレーションのスケジューリング（例：デコード/ディスパッチ仕様）
- 最小限のハードコーディングでスケールアップできる分散フレームワークを提供
