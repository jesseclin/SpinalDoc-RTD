.. _sim_example_dual_clock_fifo:

Dual clock fifo
===============

この例では、クロックドメインをクロスするために設計された ``StreamFifoCC`` を作成し、3つのシミュレーションスレッドを含めています。

スレッドは次のような処理を行います：

 - 2つのクロックの管理
 - FIFO へのデータのプッシュ
 - FIFO からのデータのポップ

FIFO へのプッシュスレッドは、入力をランダム化します。

FIFO からのポップスレッドは、 :abbr:`DUT (Device Under Test)` の出力を通常の ``scala.collection.mutable.Queue`` インスタンスの参照モデルと比較します。

.. code-block:: scala

   import spinal.core._
   import spinal.core.sim._

   import scala.collection.mutable.Queue


   object SimStreamFifoCCExample {
     def main(args: Array[String]): Unit = {
       // シミュレータのためにコンポーネントをコンパイルします。
       val compiled = SimConfig.withWave.allOptimisation.compile(
         rtl = new StreamFifoCC(
           dataType = Bits(32 bits),
           depth = 32,
           pushClock = ClockDomain.external("clkA"),
           popClock = ClockDomain.external("clkB",withReset = false)
         )
       )

       // シミュレーションを実行します。
       compiled.doSimUntilVoid{dut =>
         val queueModel = mutable.Queue[Long]()

         // クロックドメイン信号を管理するスレッドをフォークします
         val clocksThread = fork {
           // クロックドメインの信号をクリアして、シミュレーションが最初のエッジを捕捉することを確認します。
           dut.pushClock.fallingEdge()
           dut.popClock.fallingEdge()
           dut.pushClock.deassertReset()
           sleep(0)

           // リセットを実行します。
           dut.pushClock.assertReset()
           sleep(10)
           dut.pushClock.deassertReset()
           sleep(1)

           // ランダムにクロックをトグルします。
           // これにより、固定されていない周波数の非同期クロックが作成されます。
           while(true) {
             if(Random.nextBoolean()) {
               dut.pushClock.clockToggle()
             } else {
               dut.popClock.clockToggle()
             }
             sleep(1)
           }
         }

         // データをランダムにプッシュし、プッシュされたトランザクションで queueModel を埋めます。
         val pushThread = fork {
           while(true) {
             dut.io.push.valid.randomize()
             dut.io.push.payload.randomize()
             dut.pushClock.waitSampling()
             if(dut.io.push.valid.toBoolean && dut.io.push.ready.toBoolean) {
               queueModel.enqueue(dut.io.push.payload.toLong)
             }
           }
         }

         // データをランダムにポップし、それが queueModel と一致するかどうかを確認します。
         val popThread = fork {
           for(i <- 0 until 100000) {
             dut.io.pop.ready.randomize()
             dut.popClock.waitSampling()
             if(dut.io.pop.valid.toBoolean && dut.io.pop.ready.toBoolean) {
               assert(dut.io.pop.payload.toLong == queueModel.dequeue())
             }
           }
           simSuccess()
         }
       }
     }
   }
