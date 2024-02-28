.. _sim_example_single_clock_fifo:

Single clock fifo
=================

この例では、 ``StreamFifo`` を作成し、3つのシミュレーションスレッドを生成します。
:ref:`Dual clock fifo <sim_example_dual_clock_fifo>` の例とは異なり、この FIFO は複雑なクロック管理が必要ありません。

3つのシミュレーションスレッドは次の処理を行います：

 - クロック/リセットの管理
 - FIFO へのデータのプッシュ
 - FIFO からのデータのポップ

FIFO へのプッシュスレッドは、入力をランダム化します。

FIFO からのポップスレッドは、:abbr:`DUT (Device Under Test)` の出力を通常の ``scala.collection.mutable.Queue`` インスタンスの参照モデルと比較します。

.. code-block:: scala

   import spinal.core._
   import spinal.core.sim._

   import scala.collection.mutable.Queue


   object SimStreamFifoExample {
     def main(args: Array[String]): Unit = {
       // シミュレータのためにコンポーネントをコンパイルします。
       val compiled = SimConfig.withWave.allOptimisation.compile(
         rtl = new StreamFifo(
           dataType = Bits(32 bits),
           depth = 32
         )
       )

       // シミュレーションを実行します。
       compiled.doSimUntilVoid{dut =>
         val queueModel = mutable.Queue[Long]()

         dut.clockDomain.forkStimulus(period = 10)
         SimTimeout(1000000*10)

         // データをランダムにプッシュし、プッシュされたトランザクションで queueModel を埋めます。
         val pushThread = fork {
           dut.io.push.valid #= false
           while(true) {
             dut.io.push.valid.randomize()
             dut.io.push.payload.randomize()
             dut.clockDomain.waitSampling()
             if(dut.io.push.valid.toBoolean && dut.io.push.ready.toBoolean) {
               queueModel.enqueue(dut.io.push.payload.toLong)
             }
           }
         }

         // データをランダムにポップし、それが queueModel と一致するかどうかを確認します。
         val popThread = fork {
           dut.io.pop.ready #= true
           for(i <- 0 until 100000) {
             dut.io.pop.ready.randomize()
             dut.clockDomain.waitSampling()
             if(dut.io.pop.valid.toBoolean && dut.io.pop.ready.toBoolean) {
               assert(dut.io.pop.payload.toLong == queueModel.dequeue())
             }
           }
           simSuccess()
         }
       }
     }
   }
