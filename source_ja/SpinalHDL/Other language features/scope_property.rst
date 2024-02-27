.. _scopeproperty:

ScopeProperty
==================

スコーププロパティは、現在のスレッドに値をローカルに保存できるものです。
その API は、その値を設定/取得するだけでなく、実行の一部に対して値を変更するためのもので、スタックのような方法で適用することもできます。

言い換えると、それはグローバル変数、Scala の暗黙的な変数、ThreadLocal の代替手段です。

* グローバル変数と比較すると、同じコードを独立して複数のスレッドで実行できます
* Scala の暗黙的な変数と比較すると、コードベースへの影響が少なくなります
* ThreadLocal と比較すると、すべてのスコーププロパティを収集し、後で同じ状態に復元するためのAPIがいくつかあります

.. code-block:: scala

  object Xlen extends ScopeProperty[Int]

  object ScopePropertyMiaou extends App {
    Xlen.set(1)
    println(Xlen.get) //1
    Xlen(2) {
      println(Xlen.get) //2
      Xlen(3) {
        println(Xlen.get) //3
        Xlen.set(4)
        println(Xlen.get) //4
      }
      println(Xlen.get) //2
    }
  }



