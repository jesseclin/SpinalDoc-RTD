.. _section-tristate:

TriState
========

トライステート信号は多くの場合扱いにくいです：

* それらは本当の意味でデジタル的なものではありません
* IO を除いては、デジタル設計には使用されません
* トライステートの概念は、SpinalHDL の内部グラフに自然には適合しません。

SpinalHDL では、トライステート信号のための 2つの異なる抽象化が提供されています。 ``TriState`` バンドルと :ref:`section-analog_and_inout` 信号です。
それぞれ異なる目的に使用されます：

* デザイン内でのほとんどの目的、特にデザイン内での目的には ``TriState`` を使用する必要があります。
  バンドルには現在の方向を伝える追加の信号が含まれています。
* ``Analog`` と ``inout`` は、デバイス境界上のドライバやその他の特殊なケースで使用する必要があります。
  詳細については、参照されたドキュメントページを参照してください。

上記のように、推奨されるアプローチはデザイン内で ``TriState`` を使用することです。
トップレベルでは、 ``TriState`` バンドルがアナログ inout に割り当てられ、合成ツールが正しい I/O ドライバを推論できるようにします。
これは、必要に応じて :ref:`InOutWrapper <section-analog_and_inout>` を介して自動的に行うか、手動で行うことができます。

TriState
--------

TriState バンドルは以下のように定義されます：

.. code-block:: scala

   case class TriState[T <: Data](dataType : HardType[T]) extends Bundle with IMasterSlave {
     val read,write : T = dataType()
     val writeEnable = Bool()

     override def asMaster(): Unit = {
       out(write,writeEnable)
       in(read)
     }
   }

マスターは、外部の値を読み取るために ``read`` 信号、出力を有効にするために ``writeEnable`` 、
そして最後に出力される値を設定するために ``write`` を使用できます。

以下に使用例が示されています：

.. code-block:: scala

   val io = new Bundle {
     val dataBus = master(TriState(Bits(32 bits)))
   }

   io.dataBus.writeEnable := True
   io.dataBus.write := 0x12345678
   when(io.dataBus.read === 42) {

   }

TriStateArray
-------------

いくつかの場合、各個のピンの出力イネーブルを制御する必要があります（GPIOの場合など）。
この範囲のケースでは、TriStateArrayバンドルを使用できます。

次のように定義されます：

.. code-block:: scala

   case class TriStateArray(width : BitCount) extends Bundle with IMasterSlave {
     val read,write,writeEnable = Bits(width)

     override def asMaster(): Unit = {
       out(write,writeEnable)
       in(read)
     }
   }

これは TriState バンドルと同じですが、 ``writeEnable`` が各出力バッファを制御する Bits です。

使用例が以下に示されています：

.. code-block:: scala

   val io = new Bundle {
     val dataBus = master(TriStateArray(32 bits)
   }

   io.dataBus.writeEnable := 0x87654321
   io.dataBus.write := 0x12345678
   when(io.dataBus.read === 42) {

   }
