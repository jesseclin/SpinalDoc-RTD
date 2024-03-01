
ReadableOpenDrain
=================

ReadableOpenDrain バンドルは、次のように定義されます：

.. code-block:: scala

   case class ReadableOpenDrain[T<: Data](dataType : HardType[T]) extends Bundle with IMasterSlave {
     val write,read : T = dataType()

     override def asMaster(): Unit = {
       out(write)
       in(read)
     }
   }

次に、マスターとして、外部値を読み取るために ``read`` シグナルを使用し、出力にドライブする値を設定するために ``write`` を使用できます。

使用例があります：

.. code-block:: scala

   val io = new Bundle {
     val dataBus = master(ReadableOpenDrain(Bits(32 bits)))
   }

   io.dataBus.write := 0x12345678
   when(io.dataBus.read === 42) {

   }
