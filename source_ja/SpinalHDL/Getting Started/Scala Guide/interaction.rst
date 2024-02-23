
Interaction
===========

導入
------------

SpinalHDL は実際には言語ではなく、通常の Scala ライブラリです。
一見すると奇妙に思えるかもしれませんが、非常に強力な組み合わせです。

SpinalHDL ライブラリを介してハードウェアの記述に役立つように、
Scala の世界全体を使用できますが、それを適切に行うには、
SpinalHDL が Scala とどのように相互作用するかを理解することが重要です。


SpinalHDL が API の背後でどのように動作するか
------------------------------------------------

SpinalHDL ハードウェア記述を実行すると、SpinalHDL 関数、演算子、またはクラスを使用するたびに、
デザインのネットリストを表すメモリ内グラフが構築されます。

次に、エラボレーションが完了すると (最上位の ``Component`` クラスのインスタンス化)、
SpinalHDL は構築されたグラフに対していくつかのパスを実行し、
すべてが正常であれば、そのグラフを VHDL または Verilog ファイルにフラッシュします。


どれも参考になります
-------------------------

たとえば、 ``Bits`` 型のパラメータを取る Scala 関数を定義した場合、それを呼び出すと、
その関数は参照として渡されます。
その結果、関数内でその引数を代入すると、基礎となる ``Bits`` オブジェクトに対して、
関数の外側で引数を代入した場合と同じ効果が生じます。

.. _hardware_type:

ハードウェアの型
--------------------

SpinalHDL のハードウェア データ型は、次の 2 つの組み合わせです。

* 特定の Scala 型のインスタンス
* そのインスタンスの構成

たとえば、 ``Bits(8 bits)`` は、Scala 型の ``Bits`` とその ``8 bits`` 
設定 (構築パラメータとして) の組み合わせです。

.. note:

   ``8 bits`` 式は Scala によって ``BitCount(8)`` に変換され、
   BitCount オブジェクトがコンストラクター引数として渡されます。
   これは、Scala によって提供される一種の糖衣構文であり、
   表現力を高めることで型安全性を強化します。これは、
   SpinalHDL がまったく新しい言語のように見える理由の一例です。


RGBの例
^^^^^^^^^^^

Rgb バンドル クラスを例に挙げてみましょう。

.. code-block:: scala

   case class Rgb(rWidth: Int, gWidth: Int, bWidth: Int) extends Bundle {
     val r = UInt(rWidth bits)
     val g = UInt(gWidth bits)
     val b = UInt(bWidth bits)
   }

ここでのハードウェア データ型は、Scala の 
``Rgb`` クラスとその ``rWidth``、 ``gWidth``、および ``bWidth`` パラメータ化の組み合わせです。

使用例を次に示します。

.. code-block:: scala

   // Rgb 信号を定義する
   val myRgbSignal = Rgb(5, 6, 5)               

   // 前の信号と同じデータ型の別の Rgb 信号を定義します
   val myRgbCloned = cloneOf(myRgbSignal)

関数を使用して、さまざまな種類のタイプ ファクトリ (typedef) を定義することもできます: 

.. code-block:: scala

   // タイプ ファクトリ関数を定義する
   def myRgbTypeDef = Rgb(5, 6, 5)

   // そのタイプ ファクトリを使用して Rgb 信号を作成します
   val myRgbFromTypeDef = myRgbTypeDef

生成された RTL 内の信号の名前
-------------------------------------

生成された RTL 内の信号に名前を付けるために、
SpinalHDL は Java リフレクションを使用してコンポーネント階層全体を調べ、
クラス属性内に格納されているすべての参照を収集し、その属性名で名前を付けます。

これが、関数内で定義されたすべての信号の名前が失われる理由です。

.. code-block:: scala

   def myFunction(arg: UInt) {
     val temp = arg + 1  // 生成された RTL では `temp` 信号は取得されません。
     return temp
   }

   val value = myFunction(U"000001") + 42

生成された RTL に内部変数の名前を保存したい場合の解決策の 1 つは、 ``Area`` を使用することです。


.. code-block:: scala

   def myFunction(arg: UInt) new Area {
     val temp = arg + 1  // 生成された RTL では temp 信号は取得されません。
   }

   val myFunctionCall = myFunction(U"000001")  // `myFunctionCall_t mp` を名前として持つ `temp` を生成します
   val value = myFunctionCall.temp  + 42

Scala はエラボレーション用、SpinalHDL はハードウェア記述用
------------------------------------------------------------

たとえば、何らかのハードウェアを生成するために Scala の for ループを作成すると、
展開された結果が VHDL/Verilog で生成されます。

また、定数が必要な場合は、SpinalHDL ハードウェア リテラルではなく、
Scala ハードウェア リテラルを使用する必要があります。例えば：

.. code-block:: scala

   // ハードウェア ブール値を構築パラメータとして使用できないため、これは誤りです。 (階層違反の原因となります。)
   class SubComponent(activeHigh: Bool) extends Component { 
     // ...
   }

   // そうです。Scala の世界をすべて使用してハードウェアをパラメータ化できます。
   class SubComponent(activeHigh: Boolean) extends Component {
     // ...
   }

Scala エラボレーション機能 (関数型プログラミングの場合、関数型プログラミングの場合)
------------------------------------------------------------------------------------

Scala の構文はすべて、ハードウェア設計を精緻化するために使用できます。
たとえば、Scala の ``if`` ステートメントを使用して、ハードウェアの生成を有効または無効にすることができます。

.. code-block:: scala

   val counter = Reg(UInt(8 bits))
   counter := counter + 1
   if(generateAClearWhenHit42) {  // vhdl で生成される if のようなエレーション テスト
     when(counter === 42) {       // ハードウェアテスト
       counter := 0
     }
   }

Scala の ``for`` ループにも同じことが当てはまります:

.. code-block:: scala

   val value = Reg(Bits(8 bits))
   when(something) {
     // Scala for ループを使用して値のすべてのビットを設定します (ハードウェアのエラボレーション中に評価されます)。
     for(idx <- 0 to 7) {
       value(idx) := True
     }
   }


また、関数型プログラミング手法は、多くの SpinalHDL タイプで使用できます。

.. code-block:: scala

   val values = Vec(Bits(8 bits), 4)

   val valuesAre42    = values.map(_ === 42)
   val valuesAreAll42 = valuesAre42.reduce(_ && _)

   val valuesAreEqualToTheirIndex = values.zipWithIndex.map{ case (value, i) => value === i }
