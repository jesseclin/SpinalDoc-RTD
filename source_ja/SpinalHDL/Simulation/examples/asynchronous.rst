.. _sim_example_asynchronous_adder:

Asynchronous adder
==================

この例では、3つのオペランドに対して単純な算術演算を行う組み合わせ論理の ``Component`` を作成します。

テストベンチは、以下の手順を 100 回実行します：

 * a、b、および c を 0 から 255 の範囲のランダムな整数に初期化します。
 * :abbr:`DUT (Device Under Test)` に一致する a、b、c の入力を刺激します。
 * 1つのシミュレーションタイムステップ待ちます（入力が伝播するのを許可するため）。
 * 正しい出力を確認します。

.. code-block:: scala

   import spinal.core._
   import spinal.core.sim._

   import scala.util.Random


   object SimAsynchronousExample {
     class Dut extends Component {
       val io = new Bundle {
         val a, b, c = in UInt (8 bits)
         val result = out UInt (8 bits)
       }
       io.result := io.a + io.b - io.c
     }

     def main(args: Array[String]): Unit = {
       SimConfig.withWave.compile(new Dut).doSim{ dut =>
         var idx = 0
         while(idx < 100){
           val a, b, c = Random.nextInt(256)
           dut.io.a #= a
           dut.io.b #= b
           dut.io.c #= c
           sleep(1) // 1つのシミュレーションタイムステップ待ち
           assert(dut.io.result.toInt == ((a + b - c) & 0xFF))
           idx += 1
         }
       }
     }
   }
