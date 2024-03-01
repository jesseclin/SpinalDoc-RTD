
Colors
======

RGB
---

RGB バンドルを使用すると、ハードウェアで色をモデル化できます。この RGB バンドルは、
各チャンネルのビット数を指定する RgbConfig クラスをパラメーターとして取ります。

.. code-block:: scala

   case class RgbConfig(rWidth : Int,gWidth : Int,bWidth : Int) {
     def getWidth = rWidth + gWidth + bWidth
   }

   case class Rgb(c: RgbConfig) extends Bundle {
     val r = UInt(c.rWidth bits)
     val g = UInt(c.gWidth bits)
     val b = UInt(c.bWidth bits)
   }

これらのクラスは、次のように使用できます：

.. code-block:: scala

   val config = RgbConfig(5,6,5)
   val color = Rgb(config)
   color.r := 31
