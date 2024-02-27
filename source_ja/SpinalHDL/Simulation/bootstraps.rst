
Boot a simulation
=================

Introduction
------------

以下は、ハードウェア定義とテストベンチの例です：

.. code-block:: scala

   import spinal.core._

   // アイデンティティは、a から n ビットを取り、それらを z に返します
   class Identity(n: Int) extends Component {
     val io = new Bundle {
       val a = in Bits(n bits)
       val z = out Bits(n bits)
     }
   
     io.z := io.a
   }

.. code-block:: scala

   import spinal.core.sim._

   object TestIdentity extends App {
     // n = 3 ビットのコンポーネントを "dut" (device under test) として使用します
     SimConfig.withWave.compile(new Identity(3)).doSim{ dut =>
       // 3'b000 から 3'b111 までの各数字に対して
       for (a <- 0 to 7) {
         // 入力を適用します
         dut.io.a #= a
         // シミュレーション時間単位を待ちます
         sleep(1)
         // 出力を読み取ります
         val z = dut.io.z.toInt
         // 結果を確認します
         assert(z == a, s"Got $z, expected $a")
       }
     }
   }

設定
-------------

``SimConfig`` は、シミュレーションを設定するために複数の関数を呼び出せる、デフォルトのシミュレーション設定インスタンスを返します：

.. list-table::
   :header-rows: 1
   :widths: 2 5

   * - 構文
     - 説明
   * - ``withWave``
     - シミュレーション波形キャプチャを有効にします（デフォルトの形式）
   * - ``withVcdWave``
     - シミュレーション波形キャプチャを有効にします（VCD テキスト形式）
   * - ``withFstWave``
     - シミュレーション波形キャプチャを有効にします（FST バイナリ形式）
   * - ``withConfig(SpinalConfig)``
     - ハードウェア生成に使用される ``SpinalConfig`` を指定します
   * - ``allOptimisation``
     - シミュレーション時間を短縮するためにすべての RTL コンパイル最適化を有効にします（コンパイル時間が増加します）
   * - ``workspacePath(path)``
     - シミュレーションファイルが生成されるフォルダを変更します
   * - ``withVerilator``
     - シミュレーションバックエンドとして Verilator を使用します（デフォルト）
   * - ``withGhdl``
     - シミュレーションバックエンドとして GHDL を使用します
   * - ``withIVerilog``
     - シミュレーションバックエンドとして Icarus Verilog を使用します
   * - ``withVCS``
     - シミュレーションバックエンドとして Synopsys VCS を使用します

その後、 ``compile(rtl)`` 関数を呼び出してハードウェアをコンパイルし、シミュレータを起動できます。
この関数は ``SimCompiled`` インスタンスを返します。

この ``SimCompiled`` インスタンスで、次の関数を使用してシミュレーションを実行できます：

``doSim[(simName[, seed])]{dut => /* main stimulus code */}``
  シミュレーションを実行し、メインスレッドが完了して終了／リターンするまで実行します。
  シミュレーションが完全に停止した場合、エラーを検出して報告します。
  たとえば、クロックが実行されている場合、シミュレーションは永遠に続行する可能性があるため、
  可能な実行時間を制限するために ``SimTimeout(cycles)`` を使用することが推奨されます。

``doSimUntilVoid[(simName[, seed])]{dut => ...}``
  シミュレーションを実行し、 ``simSuccess()`` または ``simFailure()`` の呼び出しによって終了するまで実行します。
  メインスティミュラススレッドは続行するか早期に終了できます。
  イベントを処理する必要がある限り、シミュレーションは続行されます。シミュレーションが完全に停止した場合、エラーを報告します。

以下のテストベンチのテンプレートは、次のトップレベルを使用します：

.. code-block:: scala
    
   class TopLevel extends Component {
      val counter = out(Reg(UInt(8 bits)) init (0))
      counter := counter + 1
   }

以下は、多くのシミュレーション設定を持つテンプレートです：

.. code-block:: scala
    
   val spinalConfig = SpinalConfig(defaultClockDomainFrequency = FixedFrequency(10 MHz))

   SimConfig
     .withConfig(spinalConfig)
     .withWave
     .allOptimisation
     .workspacePath("~/tmp")
     .compile(new TopLevel)
     .doSim { dut =>
       SimTimeout(1000)
       // シミュレーションコード
   }

以下は、シミュレーションがシミュレーションメインスレッドの実行を完了して終了するテンプレートです：

.. code-block:: scala

    SimConfig.compile(new TopLevel).doSim { dut =>
      SimTimeout(1000)
      dut.clockDomain.forkStimulus(10)
      dut.clockDomain.waitSamplingWhere(dut.counter.toInt == 20)
      println("done")
    }

以下は、シミュレーションが `simSuccess()` を明示的に呼び出して終了するテンプレートです：

.. code-block:: scala

    SimConfig.compile(new TopLevel).doSimUntilVoid{ dut =>
      SimTimeout(1000)
      dut.clockDomain.forkStimulus(10)
      fork {
        dut.clockDomain.waitSamplingWhere(dut.counter.toInt == 20)
        println("done")
        simSuccess()
      }
    }

これは次のコードと等価です：

.. code-block:: scala

    SimConfig.compile(new TopLevel).doSim{ dut =>
      SimTimeout(1000)
      dut.clockDomain.forkStimulus(10)
      fork {
        dut.clockDomain.waitSamplingWhere(dut.counter.toInt == 20)
        println("done")
        simSuccess()
      }
      simThread.suspend() // "doSim" 完了を避ける
    }

.. _env_SPINALSIM_WORKSPACE:

デフォルトでは、シミュレーションファイルは ``simWorkspace/xxx`` フォルダに配置されます。
``SPINALSIM_WORKSPACE`` 環境変数を設定することで、simWorkspace の場所を上書きできます。

同じハードウェアに複数のテストを実行する
-------------------------------------------

.. code-block:: scala

    val compiled = SimConfig.withWave.compile(new Dut)

    compiled.doSim("testA") { dut =>
       // シミュレーションコード
    }

    compiled.doSim("testB") { dut =>
       // シミュレーションコード
    }

シミュレーションの成功または失敗をスレッドから投げる
--------------------------------------------------------

シミュレーション中にいつでも、 ``simSuccess`` または ``simFailure`` を呼び出して終了できます。

シミュレーションが長すぎて失敗することがあります。たとえば、テストベンチが発生しない条件を待っている間などです。
そのような場合、 ``SimTimeout(maxDuration)`` を呼び出します。
ここで、 ``maxDuration`` はシミュレーションが失敗と見なされる時間（シミュレーション単位で表される時間）です。

たとえば、クロックサイクルの持続時間の 1000 倍後にシミュレーションを失敗させるには：

.. code-block:: scala

    val period = 10
    dut.clockDomain.forkStimulus(period)
    SimTimeout(1000 * period)

失敗前の特定ウィンドウでの波形キャプチャ
------------------------------------------------

非常に長いシミュレーションを行っており、すべての波形をキャプチャしたくない場合（大きすぎる、遅すぎるなど）、主に2つの方法があります。

まず、シミュレーションが失敗した ``simTime`` をすでに知っている場合は、テストベンチで次のようにします：

.. code-block:: scala
    
    disableSimWave()
    delayed(timeFromWhichIWantToCapture)(enableSimWave())

または、1つのシミュレーションがわずかに遅れて実行され、
先行シミュレーションが失敗した後に波形の記録を開始するデュアルロックステップシミュレーションを実行することもできます。

これを行うには、DualSimTracer ユーティリティを使用します。コンパイルされたハードウェア、
失敗前にキャプチャしたい時間ウィンドウ、およびシードのパラメータが必要です。

以下は例です：

.. literalinclude:: /../examples/src/main/scala/spinaldoc/libraries/sim/DualSimExample.scala
   :language: scala

これにより、次のファイル構造が生成されます：

- simWorkspace/Toplevel/explorer/stdout.log：先行するシミュレーションの標準出力
- simWorkspace/Toplevel/tracer/stdout.log：波形トレースを実行するシミュレーションの標準出力
- simWorkspace/Toplevel/tracer.fst：失敗時の波形

Scala ターミナルには、explorer シミュレーションの標準出力が表示されます。


