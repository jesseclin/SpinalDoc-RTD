
Uart decoder
============

.. code-block:: scala

   // uartPin を分析し、送信されたバイトをシミュレーションターミナルに出力するシミュレーションプロセスをフォークします。
   fork {
     // デザインが uartPin を true に設定するまで待機します（リセット効果を待ちます）。
     waitUntil(uartPin.toBoolean == true)

     while(true) {
       waitUntil(uartPin.toBoolean == false)
       sleep(baudPeriod/2)

       assert(uartPin.toBoolean == false)
       sleep(baudPeriod)

       var buffer = 0
       for(bitId <- 0 to 7) {
         if(uartPin.toBoolean)
           buffer |= 1 << bitId
         sleep(baudPeriod)
       }

       assert(uartPin.toBoolean == true)
       print(buffer.toChar)
     }
   }
