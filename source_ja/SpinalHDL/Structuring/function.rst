.. _function:

Function
========

Scala 関数を使用してハードウェアを生成する方法は、多くの理由から VHDL/Verilog とは根本的に異なります：

- レジスタ、組み合わせ論理、およびコンポーネントをインスタンス化することができます。
- 信号の代入の範囲を制限する ``process``\ /\ ``@always`` ブロックを扱う必要がありません。
- | すべてが参照渡しで渡されるため、簡単に操作できます。
  | たとえば、バスを関数の引数として渡すことができ、関数内部で読み取り/書き込みを行うことができます。また、Scala から Component、Bus、またはその他のオブジェクトを返すこともできます。


RGB to gray
-----------

例えば、係数を使用して赤/緑/青の色をグレースケールに変換したい場合、それらを適用するために関数を使用できます:

.. code-block:: scala

   // 入力RGBカラー
   val r, g, b = UInt(8 bits)

   // UIntをScalaのFloat値で乗算する関数を定義します。
   def coef(value: UInt, by: Float): UInt = (value * U((255 * by).toInt, 8 bits) >> 8)

   // グレーレベルを計算します。
   val gray = coef(r, 0.3f) + coef(g, 0.4f) + coef(b, 0.3f)

Valid Ready Payload bus
--------------------------------

たとえば、 ``valid``、 ``ready``、 ``payload``信号を持つ単純なバスを定義した場合、その内部にいくつかの有用な関数を定義できます。

.. code-block:: scala

   case class MyBus(payloadWidth: Int) extends Bundle with IMasterSlave {
     val valid   = Bool()
     val ready   = Bool()
     val payload = Bits(payloadWidth bits)

     // マスターモードでのデータの方向を定義します。
     override def asMaster(): Unit = {
       out(valid, payload)
       in(ready)
     }

     // それをこれに接続します。
     def <<(that: MyBus): Unit = {
       this.valid   := that.valid
       that.ready   := this.ready
       this.payload := that.payload
     }

     // FIFOの入力にこれを接続し、FIFOの出力を返します。
     def queue(size: Int): MyBus = {
       val fifo = new MyBusFifo(payloadWidth, size)
       fifo.io.push << this
       return fifo.io.pop
     }
   }

   class MyBusFifo(payloadWidth: Int, depth: Int) extends Component {

     val io = new Bundle {
       val push = slave(MyBus(payloadWidth))
       val pop  = master(MyBus(payloadWidth))
     }

     val mem = Mem(Bits(payloadWidth bits), depth)

     // ...

   }
