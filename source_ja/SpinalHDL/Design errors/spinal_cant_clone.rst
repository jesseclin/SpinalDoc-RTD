
Spinal can't clone class
========================

概要
------------

このエラーは、SpinalHDL が ``cloneOf`` 関数を使用して新しいデータ型インスタンスを作成しようとした際に発生しますが、
それができない場合に発生します。
これはほぼ常に、 ``Bundle`` の構築パラメーターを取得できないためです。

例 1
---------

次のコード：


.. code-block:: scala

    // ``cloneOf(this)`` は自身の構築に使用された幅の値を取得できません
    class RGB(width : Int) extends Bundle {
      val r, g, b = UInt(width bits)
    }

    class TopLevel extends Component {
      val tmp = Stream(new RGB(8)) // Streamは、cloneOf(new RGB(8))の機能を必要とします
    }

以下のエラーが発生します：

.. code-block:: text

   *** Spinal can't clone class spinal.tester.PlayDevMessages$RGB datatype
   *** You have two way to solve that :
   *** In place to declare a "class Bundle(args){}", create a "case class Bundle(args){}"
   *** Or override by your self the bundle clone function
     ***
     Source file location of the RGB class definition via the stack trace
     ***

修正方法は以下の通りです：

.. code-block:: scala

   case class RGB(width : Int) extends Bundle {
     val r, g, b = UInt(width bits)
   }

   class TopLevel extends Component {
     val tmp = Stream(RGB(8))
   }

Example 2
---------
以下のコード：

.. code-block:: scala

  case class Xlen(val xlen: Int) {}

  case class MemoryAddress()(implicit xlenConfig: Xlen) extends Bundle {
      val address = UInt(xlenConfig.xlen bits)
  }

  class DebugMemory(implicit config: Xlen) extends Component {
      val io = new Bundle {
          val inputAddress = in(MemoryAddress())
      }   

      val someAddress = RegNext(io.inputAddress) // -> ERROR *****************************
  }

例外が発生します：

.. code-block:: text

  [error] *** Spinal can't clone class debug.MemoryAddress datatype

この場合、暗黙のパラメータを伝播させるためにクローン関数をオーバーライドする解決策があります。

.. code-block:: scala

  case class MemoryAddress()(implicit xlenConfig: Xlen) extends Bundle {
    val address = UInt(xlenConfig.xlen bits)

    override def clone = MemoryAddress()
  }

.. note::

  我々はハードウェア要素をクローンする必要があります。それが含まれる値ではありません。

.. note::

  別の方法として、:ref:`ScopeProperty <scopeproperty>` を使用することもできます。