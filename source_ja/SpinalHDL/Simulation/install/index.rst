SBT setup for simulation
========================

SpinalSim を有効にするには、build.sbt ファイルに次の行を追加する必要があります：

.. code-block:: scala

   fork := true

また、テストベンチのソースコードに次のインポートも追加する必要があります：

.. code-block:: scala

   import spinal.core._
   import spinal.core.sim._

.. _sim backend install:

また、make の代わりに gmake（例：OpenBSD）を使用する必要がある場合は、SPINAL_MAKE_CMD 環境変数を "gmake" に設定できます。

Backend-dependent installation instructions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. toctree::
   :glob:

   *
