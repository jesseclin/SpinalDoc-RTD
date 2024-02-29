
Tilelink
=========

Configuration and instanciation
-------------------------------

以下は、2つの非整合性 TileLink バスインスタンスを定義し、それらを接続する簡単な例です：

.. code-block:: scala

    import spinal.lib.bus.tilelink
    val param = tilelink.BusParameter.simple(
      addressWidth = 32,
      dataWidth    = 64,
      sizeBytes    = 64,
      sourceWidth  = 4
    )
    val busA, busB = tilelink.Bus(param)
    busA << busB

以下は、同じものですが、整合性チャネルを使用しています：

.. code-block:: scala

    import spinal.lib.bus.tilelink
    val param = tilelink.BusParameter(
      addressWidth = 32,
      dataWidth = 64,
      sizeBytes = 64,
      sourceWidth = 4,
      sinkWidth = 0,
      withBCE = false,
      withDataA = true,
      withDataB = false,
      withDataC = false,
      withDataD = true,
      node = null
    )
    val busA, busB = tilelink.Bus(param)
    busA << busB

上記はハードウェアのインスタンス化のためのもので、これは単純で簡単な部分です。
しかし、SoC やメモリの整合性に関わるときは、パラメータを交渉/伝播させるための追加のレイヤーが必要です。
それが tilelink.fabric.Node の役割です。


