.. _sim_example_synchronous_adder:

Synchronous adder
=================

この例では、3つのオペランドに対して単純な算術演算を行うシーケンシャルロジックから ``Component`` を作成します。

テストベンチは、以下の手順を 100 回繰り返します：

 * ``a`` 、 ``b`` 、 ``c`` を 0 から 255 の範囲のランダムな整数で初期化します。
 * :abbr:`DUT (Device Under Test)` に対応する ``a`` 、 ``b`` 、 ``c`` の入力を刺激します。
 * シミュレーションが DUT の信号を再度サンプリングするまで待機します。
 * 正しい出力を確認します。

この例と:ref:`Asynchronous adder <sim_example_asynchronous_adder>` の例の主な違いは、
この ``Component`` が内部でシーケンシャルロジックを使用しているため、 ``forkStimulus`` を使用してクロック信号を生成する必要があることです。

.. code-block:: scala

   import spinal.core._
   import spinal.core.sim._

   import scala.util.Random


   object SimSynchronousExample {
     class Dut extends Component {
       val io = new Bundle {
         val a, b, c = in UInt (8 bits)
         val result = out UInt (8 bits)
       }
       io.result := RegNext(io.a + io.b - io.c) init(0)
     }

     def main(args: Array[String]): Unit = {
       SimConfig.withWave.compile(new Dut).doSim{ dut =>
         dut.clockDomain.forkStimulus(period = 10)

         var resultModel = 0
         for(idx <- 0 until 100){
           dut.io.a #= Random.nextInt(256)
           dut.io.b #= Random.nextInt(256)
           dut.io.c #= Random.nextInt(256)
           dut.clockDomain.waitSampling()
           assert(dut.io.result.toInt == resultModel)
           resultModel = (dut.io.a.toInt + dut.io.b.toInt - dut.io.c.toInt) & 0xFF
         }
       }
     }
   }
