
Clock crossing violation
========================

はじめに
------------

SpinalHDL は、設計のすべてのレジスタが、同じまたは同期クロックドメインを使用するレジスタによってのみ
（組合せ論理パスを介して）依存していることを確認します。

例
-------

次のコード:


.. code-block:: scala

   class TopLevel extends Component {
     val clkA = ClockDomain.external("clkA")
     val clkB = ClockDomain.external("clkB")

     val regA = clkA(Reg(UInt(8 bits)))   // PlayDev.scala:834
     val regB = clkB(Reg(UInt(8 bits)))   // PlayDev.scala:835

     val tmp = regA + regA                // PlayDev.scala:838
     regB := tmp
   }

は、次のエラーを発生させます:

.. code-block:: text

   CLOCK CROSSING VIOLATION from (toplevel/regA :  UInt[8 bits]) to (toplevel/regB :  UInt[8 bits]).
   - Register declaration at
     ***
     Source file location of the toplevel/regA definition via the stack trace
     ***
   - through
         >>> (toplevel/regA :  UInt[8 bits]) at ***(PlayDev.scala:834) >>>
         >>> (toplevel/tmp :  UInt[8 bits]) at ***(PlayDev.scala:838) >>>
         >>> (toplevel/regB :  UInt[8 bits]) at ***(PlayDev.scala:835) >>>

複数の修正方法があります。以下にリストされています:

 - :ref:`crossClockDomain tags <crossclockdomain-tag>`
 - :ref:`setSynchronousWith method <setsynchronouswith>`
 - :ref:`BufferCC type <buffercc>`

.. _crossclockdomain-tag:

crossClockDomain tag
^^^^^^^^^^^^^^^^^^^^

``crossClockDomain`` タグを使用すると、SpinalHDL コンパイラに「これに関しては心配しないでください」と伝えることができます。

.. code-block:: scala

   class TopLevel extends Component {
     val clkA = ClockDomain.external("clkA")
     val clkB = ClockDomain.external("clkB")

     val regA = clkA(Reg(UInt(8 bits)))
     val regB = clkB(Reg(UInt(8 bits))).addTag(crossClockDomain)


     val tmp = regA + regA
     regB := tmp
   }

.. _setsynchronouswith:

setSynchronousWith
^^^^^^^^^^^^^^^^^^

ある ``ClockDomain`` オブジェクトの ``setSynchronousWith`` メソッドを使用して、
2つのクロックドメインが同期していることを指定することもできます。

.. code-block:: scala

   class TopLevel extends Component {
     val clkA = ClockDomain.external("clkA")
     val clkB = ClockDomain.external("clkB")
     clkB.setSynchronousWith(clkA)

     val regA = clkA(Reg(UInt(8 bits)))
     val regB = clkB(Reg(UInt(8 bits)))


     val tmp = regA + regA
     regB := tmp
   }

.. _buffercc:

BufferCC
^^^^^^^^

単一ビット信号（たとえば、 ``Bool`` 型）やグレーコード値をやり取りする場合は、
異なる ``ClockDomain`` 領域を安全にクロスするために ``BufferCC`` を使用できます。

.. warning::
   クロックが非同期の場合、受信側で読み取りが破損する可能性があるため、マルチビット信号に ``BufferCC`` を使用しないでください。
   詳細については、:ref:`Clock Domains <clock_domain>`  ページを参照してください。

.. code-block:: scala

   class AsyncFifo extends Component {
      val popToPushGray = Bits(ptrWidth bits)
      val pushToPopGray = Bits(ptrWidth bits)
     
      val pushCC = new ClockingArea(pushClock) {
        val pushPtr     = Counter(depth << 1)
        val pushPtrGray = RegNext(toGray(pushPtr.valueNext)) init(0)
        val popPtrGray  = BufferCC(popToPushGray, B(0, ptrWidth bits))
        val full        = isFull(pushPtrGray, popPtrGray)
        ...
      }
     
      val popCC = new ClockingArea(popClock) {
        val popPtr      = Counter(depth << 1)
        val popPtrGray  = RegNext(toGray(popPtr.valueNext)) init(0)
        val pushPtrGray = BufferCC(pushToPopGray, B(0, ptrWidth bits))
        val empty       = isEmpty(popPtrGray, pushPtrGray)   
        ...
      }
   }
