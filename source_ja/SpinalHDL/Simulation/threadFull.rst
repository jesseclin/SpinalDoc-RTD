Thread-full API
==========================================

SpinalSim では、SystemVerilog のように複数のスレッドを使用してテストベンチを記述したり、
VHDL/Verilog のプロセス/alwaysブロックのように記述したりすることができます。
これにより、並行タスクを記述し、流暢な API を使用してシミュレーション時間を制御できます。


スレッドのフォークと結合
--------------------------------

.. code-block:: scala

   // 新しいスレッドを作成します
   val myNewThread = fork {
     // 新しいシミュレーションスレッドの本体
   }

   // `myNewThread`` の実行が完了するまで待ちます
   myNewThread.join()

Sleep と waitUntil
-----------------------

.. code-block:: scala

   //  1000 ユニットの時間をスリープします
   sleep(1000)

   // dut.io.a の値が 42 より大きくなるまで待機します
   waitUntil(dut.io.a > 42)

