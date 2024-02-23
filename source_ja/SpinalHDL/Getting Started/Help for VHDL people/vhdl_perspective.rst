.. role:: raw-html-m2r(raw)
   :format: html

VHDL の同等物
=================

エンティティとアーキテクチャ
------------------------------

SpinalHDL では、VHDL エンティティとアーキテクチャは両方とも ``Component`` 内で定義されます。

以下は 3 つの入力 (``a``、``b``、``c``) と 1 つの出力 (``result``) を持つコンポーネントの例です。
このコンポーネントには (VHDL ジェネリックのような) ``offset`` 構築パラメータもあります。

.. code-block:: scala

   case class MyComponent(offset: Int) extends Component {
     val io = new Bundle{
       val a, b, c = in UInt(8 bits)
       val result  = out UInt(8 bits)
     }
     io.result := a + b + c + offset
   }

次に、そのコンポーネントをインスタンス化するために、それをバインドする必要はありません:

.. code-block:: scala

   case class TopLevel extends Component {
     ...
     val mySubComponent = MyComponent(offset = 5)

     ...

     mySubComponent.io.a := 1
     mySubComponent.io.b := 2
     mySubComponent.io.c := 3
     ??? := mySubComponent.io.result

     ...
   }

データ型
----------

SpinalHDL データ型は VHDL のデータ型と似ています:

.. list-table::
   :header-rows: 1

   * - VHDL
     - SpinalHDL
   * - std_logic
     - Bool
   * - std_logic_vector
     - Bits
   * - unsigned
     - UInt
   * - signed
     - SInt

| VHDL では、8 ビット ``unsigned`` を定義するには、ビット範囲 ``unsigned(7 downto 0)`` を指定する必要があります。
| 一方、SpinalHDL では、単純にビット数 ``UInt(8 bits)`` を指定します。


.. list-table::
   :header-rows: 1

   * - VHDL
     - SpinalHDL
   * - records
     - Bundle
   * - array
     - Vec
   * - enum
     - SpinalEnum

以下は SpinalHDL の ``Bundle`` 定義の例です。 
``channelWidth`` は VHDL ジェネリックスと同様に構築パラメータですが、データ構造用です。

.. code-block:: scala

   case class RGB(channelWidth: Int) extends Bundle {
     val r, g, b = UInt(channelWidth bits)
   }

次に、たとえば ``Bundle`` をインスタンス化するには、 
``val myColor = RGB(channelWidth=8)`` と記述する必要があります。


信号
------

以下は信号のインスタンス化に関する例です。

.. code-block:: scala

   case class MyComponent(offset: Int) extends Component {
     val io = new Bundle {
       val a, b, c = UInt(8 bits)
       val result  = UInt(8 bits)
     }
     val ab = UInt(8 bits)
     ab := a + b

     val abc = ab + c            // 信号をその値で直接定義できます
     io.result := abc + offset
   }

割り当て
-----------

SpinalHDL では、 ``:=`` 代入演算子は VHDL 信号割り当て (``<=``) と同等です。

.. code-block:: scala

   val myUInt = UInt(8 bits)
   myUInt := 6

条件付き代入は、VHDL と同様に ``if``/``case`` ステートメントを使用して行われます。

.. code-block:: scala

   val clear   = Bool()
   val counter = Reg(UInt(8 bits))

   when(clear) {
     counter := 0
   }.elsewhen(counter === 76) {
     counter := 79
   }.otherwise {
     counter(7) := ! counter(7)
   }

   switch(counter) {
     is(42) {
       counter := 65
     }
     default {
       counter := counter + 1
     }
   }

リテラル
-----------

リテラルは VHDL とは少し異なります:

.. code-block:: scala

   val myBool = Bool()
   myBool := False
   myBool := True
   myBool := Bool(4 > 7)

   val myUInt = UInt(8 bits)
   myUInt := "0001_1100"
   myUInt := "xEE"
   myUInt := 42
   myUInt := U(54,8 bits)
   myUInt := ((3 downto 0) -> myBool, default -> true)
   when(myUInt === U(myUInt.range -> true)) {
     myUInt(3) := False
   }

レジスター
---------

SpinalHDL ではレジスタは明示的に指定されますが、VHDL ではレジスタが推論されます。 
SpinalHDL レジスタの例を次に示します。

.. code-block:: scala

   val counter = Reg(UInt(8 bits))  init(0)  
   counter := counter + 1   // サイクルごとにカウントアップする

   // init(0) は、リセットが発生したときにレジスタをゼロに初期化する必要があることを意味します

プロセスブロック
-------------------

プロセス ブロックは、RTL の設計には不要なシミュレーション機能です。 
SpinalHDL にはプロセス ブロックに類似した機能が含まれておらず、
必要なものを必要な場所に割り当てることができるのはこのためです。

.. code-block:: scala

   val cond = Bool()
   val myCombinatorial = Bool()
   val myRegister = UInt(8 bits)

   myCombinatorial := False
   when(cond) {
     myCombinatorial := True
     myRegister = myRegister + 1
   }
