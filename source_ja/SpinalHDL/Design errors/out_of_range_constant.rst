
Out of Range Constant
=====================

導入
------------

SpinalHDL は、リテラルとの比較において、比較される値よりもリテラルが広くないことを確認します。

例
-------

たとえば、次のコード：

.. code-block:: scala

    val value = in UInt(2 bits)
    val result = out(value < 42)

次のエラーが発生します：

.. code-block:: text

   OUT OF RANGE CONSTANT. Operator UInt < UInt
   - Left  operand : (toplevel/value : in UInt[2 bits])
   - Right operand : (U"101010" 6 bits)
    is checking a value against a out of range constant

例外の指定
---------------------

設計のパラメータ化により、値をより大きな定数と比較し、静的に既知の ``True/False`` 結果を得ることが合理的な場合があります。

範囲外の定数との比較の特定のインスタンスをホワイトリストに指定するオプションがあります。

.. code-block:: scala

    val value = in UInt(2 bits)
    val result = out((value < 42).allowOutOfRangeLiterals)


また、設計全体で範囲外の定数との比較を許可することもできます。

.. code-block:: scala

   SpinalConfig(allowOutOfRangeLiterals = true)
