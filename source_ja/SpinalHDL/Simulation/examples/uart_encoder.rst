
Uart encoder
============

.. code-block:: scala

     // シミュレーションターミナルに入力された文字を取得し、シミュレーションの uartPin に送信するシミュレーションプロセスをフォークします。
     fork{
       uartPin #= true
       while(true) {
         // System.in は C の stdin に相当するJavaの機能です。 
         if(System.in.available() != 0) {
           val buffer = System.in.read()
           uartPin #= false
           sleep(baudPeriod)

           for(bitId <- 0 to 7) {
             uartPin #= ((buffer >> bitId) & 1) != 0
             sleep(baudPeriod)
           }

           uartPin #= true
           sleep(baudPeriod)
         } else {
           sleep(baudPeriod * 10) // System.inを頻繁にポーリングしないように、少し待機します。
         }
       }
     }
