.. _type_introduction:

==========
Data types
==========

この言語では、使用できる 5 つの基本タイプと 2 つの複合タイプが提供されます。

* Base types: :ref:`Bool <Bool>` 、 :ref:`Bits <Bits>` 、 :ref:`UInt <Int>` (符号なし整数の場合)、 :ref:`SInt <Int>` (符号付き整数の場合) と :ref:`Enum <Enum>`.
* Composite types: :ref:`Bundle <Bundle>` と :ref:`Vec <Vec>`.

.. image:: /asset/picture/types.svg
   :width: 400px

基本タイプに加えて、Spinal は次のサポートを開発中です:

* :ref:`Fixed-point <fixed>` 数値 (部分的にサポート)
* :ref:`Auto-range Fixed-point <afix>` 数値 (add、sub、mul のサポート)
* :ref:`Floating-point <Floating>` 数値 (実験的サポート)


さらに、デフォルト値を提供するなど、一部のハードウェアにドントケア値を割り当てたい場合は、
assignDontCare API を使用してこれを行うことができます。


.. code-block:: scala

   val myBits  = Bits(8 bits)
   myBits.assignDontCare() // すべてのビットを `x` に割り当てます


最後に、BitVector と、ビットマスクのように定義されたホール 
(等価式によって比較されないビット位置) を含むビット定数パターンとの間の等価性をチェックするための特別なタイプが使用可能です。

これを実現する方法を示す例を次に示します (接頭辞 `M` の使用に注意してください)。

.. code-block:: scala

   val myBits  = Bits(8 bits)
   val itMatch = myBits === M"00--10--" // - for don't care value



.. toctree::
   :hidden: 

   bool
   bits
   Int
   enum
   bundle
   Vec
   Fix
   Floating
   AFix
