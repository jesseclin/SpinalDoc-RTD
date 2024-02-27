
Stub
======

以下のように、コンポーネント階層をスタブ化することができます：

.. code-block:: scala 

    class SubSysModule extends Component {
       val io = new Bundle {
         val dx = slave(Stream(Bits(32 bits)))
         val dy = master(Stream(Bits(32 bits)))
       }
       io.dy <-< io.dx
    }
    class TopLevel extends Component {
       val dut = new SubSysModule().stub   //サブシステムモジュールを空のスタブとしてインスタンス化
    }
   
これは、例えば次のような Verilog コードを生成します：

.. code-block:: verilog

    module SubSysModule (
      input               io_dx_valid,
      output              io_dx_ready,
      input      [31:0]   io_dx_payload,
      output              io_dy_valid,
      input               io_dy_ready,
      output     [31:0]   io_dy_payload,
      input               clk,
      input               reset
    );


      assign io_dx_ready = 1'b0;
      assign io_dy_valid = 1'b0;
      assign io_dy_payload = 32'h0;

    endmodule


また、トップコンポーネントも空にすることができます：

.. code-block:: scala

    SpinalVerilog(new Pinsec(500 MHz).stub)

`stub` は何をするのでしょうか？

* 最初にすべてのコンポーネントをウォークし、クロックを見つけて保持します
* 次に、すべての子コンポーネントを削除します
* その後、削除したいすべての代入およびロジックを削除します
* タイル 0 から出力ポートまで


