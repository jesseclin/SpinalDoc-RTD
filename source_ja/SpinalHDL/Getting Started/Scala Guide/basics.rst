
.. important::
   Variables and functions should be defined into ``object``\ , ``class``\ , ``function``. You can't define them on the root of a Scala file.

Basics
======

型
-------

In Scala, there are 5 major types:

.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - 型
     - リテラル
     - 説明
   * - Boolean
     - true, false
     - 
   * - Int
     - 3, 0x32
     - 32 ビット整数
   * - Float
     - 3.14f
     - 32 ビット浮動小数点
   * - Double
     - 3.14
     - 64 ビット浮動小数点
   * - String
     - "Hello world"
     - UTF-16文字列


変数
---------

Scala では、 ``var`` キーワードを使用して変数を定義できます:

.. code-block:: scala

   var number : Int = 0
   number = 6
   number += 4
   println(number) // 10

Scala は型を自動的に推論できます。宣言時に変数が代入されている場合は指定する必要はありません。

.. code-block:: scala

   var number = 0   // `number` の型は、コンパイル中に Int として推論されます。

ただし、Scala で var を使用することはあまり一般的ではありません。代わりに、 ``val`` で定義された定数値がよく使用されます。

.. code-block:: scala

   val two   = 2
   val three = 3
   val six   = two * three

関数
---------

たとえば、2 つの引数の合計がゼロより大きい場合に ``true`` を返す関数を定義したい場合は、
次のように実行できます:

.. code-block:: scala

   def sumBiggerThanZero(a: Float, b: Float): Boolean = {
     return (a + b) > 0
   }


次に、この関数を呼び出すには、次のように記述します:

.. code-block:: scala

   sumBiggerThanZero(2.3f, 5.4f)

引数を名前で指定することもできます。これは、引数が多い場合に便利です。

.. code-block:: scala

   sumBiggerThanZero(
     a = 2.3f,
     b = 5.4f
   )

戻り値
^^^^^^^^

``return`` キーワードは必要ありません。これがない場合、Scala は関数の最後のステートメントを戻り値として受け取ります。

.. code-block:: scala

   def sumBiggerThanZero(a: Float, b: Float): Boolean = {
     (a + b) > 0
   }

戻り値の型推論
^^^^^^^^^^^^^^^^^^^^^^

Scala は戻り値の型を自動的に推論できます。指定する必要はありません。

.. code-block:: scala

   def sumBiggerThanZero(a: Float, b: Float) = {
     (a + b) > 0
   }

中括弧
^^^^^^^^^^^^

Scala 関数には、関数にステートメントが 1 つしか含まれていない場合、中括弧は必要ありません。

.. code-block:: scala

   def sumBiggerThanZero(a: Float, b: Float) = (a + b) > 0

何も返さない関数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

関数が何も返さないようにしたい場合は、戻り値の型を ``Unit`` に設定する必要があります。
これは、C/C++ の ``void`` 型と同等です。

.. code-block:: scala

   def printer(): Unit = {
     println("1234")
     println("5678")
   }

引数のデフォルト値
^^^^^^^^^^^^^^^^^^^^^^^

関数の各引数にデフォルト値を指定できます。

.. code-block:: scala

   def sumBiggerThanZero(a: Float, b: Float = 0.0f) = {
     (a + b) > 0
   }

アプライ
^^^^^

``apply`` という名前の関数は、名前を入力せずに呼び出すことができるため、特別です。

.. code-block:: scala

   class Array() {
     def apply(index: Int): Int = index + 3
   }

   val array = new Array()
   val value = array(4)   //array(4) is interpreted as array.apply(4) and will return 7


この概念は、Scala ``object`` (静的) にも適用できます。

.. code-block:: scala

   object MajorityVote {
     def apply(value: Int): Int = ...
   }

   val value = MajorityVote(4) // Will call MajorityVote.apply(4)

オブジェクト
------------

Scala には、 ``static`` キーワードはありません。その代わりにあるのが  ``object`` です。 
``object`` 定義内で定義されたものはすべて静的です。

次の例では、浮動小数点値をパラメータとして受け取り、同様に浮動小数点値を返す 
``pow2`` という名前の静的関数を定義します。

.. code-block:: scala

   object MathUtils {
     def pow2(value: Float): Float = value * value
   }

次に、次のように記述して呼び出すことができます。

.. code-block:: scala

   MathUtils.pow2(42.0f)

エントリーポイント (main)
---------------------------

Scala プログラムのエントリ ポイント (main 関数) は、オブジェクト内で ``main`` という名前の関数として定義する必要があります。

.. code-block:: scala

   object MyTopLevelMain {
     def main(args: Array[String]) {
       println("Hello world")
     }
   }

クラス
--------

クラスの構文は Java に非常に似ています。
構築パラメータとして 3 つの Float 値 (r,g,b) を取る ``Color`` クラスを定義するとします。

.. code-block:: scala

   class Color(r: Float, g: Float, b: Float) {
     def getGrayLevel(): Float = r * 0.3f + g * 0.4f + b * 0.4f
   }


次に、前の例のクラスをインスタンス化し、その ``getGrayLevel`` 関数を使用します。

.. code-block:: scala

   val blue = new Color(0, 0, 1)
   val grayLevelOfBlue = blue.getGrayLevel()


クラスの構築パラメータに外部からアクセスしたい場合は、
この構築パラメータを ``val`` として定義する必要があることに注意してください。

.. code-block:: scala

   class Color(val r: Float, val g: Float, val b: Float) { ... }
   ...
   val blue = new Color(0, 0, 1)
   val redLevelOfBlue = blue.r

継承
^^^^^^^^^^^

例として、クラス ``Shape`` を拡張する 2 つのクラス ``Rectangle`` と ``Square`` を定義するとします。

.. code-block:: scala

   class Shape {
     def getArea(): Float
   }

   class Square(sideLength: Float) extends Shape {
     override def getArea() = sideLength * sideLength
   }

   class Rectangle(width: Float, height: Float) extends Shape {
     override def getArea() = width * height
   }

ケースクラス
^^^^^^^^^^^^^

Case クラスは、クラスを宣言する別の方法です。

.. code-block:: scala

   case class Rectangle(width: Float, height: Float) extends Shape {
     override def getArea() = width * height
   }

次に、 ``case class`` と ``class`` の間にはいくつかの違いがあります:

* case クラスはインスタンス化するために ``new`` キーワードを必要としません。
* 構築パラメータは外部からアクセス可能です。それらを ``val`` として定義する必要はありません。

SpinalHDL では、これがコーディング規約の背後にある理由を説明しています。
一般に、型付けを減らして一貫性を高めるために、 ``class`` の代わりに ``case class`` を使用することが推奨されています。


テンプレート/型パラメータ化
---------------------------------

特定のデータ型のキューであるクラスを設計したいとします。
その場合、クラスに型パラメーターを指定する必要があります

.. code-block:: scala

   class  Queue[T]() {
     def push(that: T) : Unit = ...
     def pop(): T = ...
   }

``T`` 型を特定の型 (たとえば ``Shape``) のサブクラスに制限したい場合は、 ``<: Shape`` 構文を使用できます。

.. code-block:: scala

   class Shape() {   
       def getArea(): Float
   }
   class Rectangle() extends Shape { ... }

   class  Queue[T <: Shape]() {
     def push(that: T): Unit = ...
     def pop(): T = ...
   }

関数についても同じことが可能です:

.. code-block:: scala

   def doSomething[T <: Shape](shape: T): Something = { shape.getArea() }
