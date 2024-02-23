Area
====

時々、ロジックを定義するために ``Component`` を作成することは過剰であり、なぜなら：

- すべての構築パラメータと IO を定義する必要がある（冗長性、重複）
- コードを分割する必要がある（必要以上に）

このような場合には、シグナルやロジックのグループを定義するために ``Area`` を使用できます：

.. code-block:: scala

   class UartCtrl extends Component {
     ...
     val timer = new Area {
       val counter = Reg(UInt(8 bits))
       val tick = counter === 0
       counter := counter - 1
       when(tick) {
         counter := 100
       }
     }

     val tickCounter = new Area {
       val value = Reg(UInt(3 bits))
       val reset = False
       when(timer.tick) {          // タイマーエリアからのティックを参照してください。
         value := value + 1
       }
       when(reset) {
         value := 0
       }
     }

     val stateMachine = new Area {
       ...
     }
   }

.. tip::
   | VHDL や Verilog では、変数を論理的なセクションに分割するために接頭辞が使用されることがあります。SpinalHDL では、これに代わって ``Area`` を使用することが提案されています。
.. note::
   \ :ref:`ClockingArea <clock_domain>` は、特定の ``ClockDomain`` を使用するハードウェアのチャンクを定義するための特別な種類の ``Area`` です。\