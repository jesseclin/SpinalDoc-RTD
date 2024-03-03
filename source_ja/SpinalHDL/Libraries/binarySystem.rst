 
BinarySystem
============

Specification
-------------

| ここでは、HDL とは関係ありませんが、デジタルシステムで非常に一般的であり、特にアルゴリズムのリファレンスモデルが広く使用されています。また、テストベンチの構築にも使用されます。  


.. list-table::
   :header-rows: 1
   :widths: 4 8  1

   * - 構文
     - 説明
     - 戻り値

   * - **String**.asHex
     - HexString を BigIntに変換 == BigInt(string, 16)
     - BigInt
   * - **String**.asDec
     - Decimal String を BigInt に変換 == BigInt(string, 10)
     - BigInt
   * - **String**.asOct
     - Octal String を BigInt に変換 == BigInt(string, 8)
     - BigInt
   * - **String**.asBin
     - Binary String を BigInt に変換 == BigInt(string, 2)
     - BigInt
   * - 
     - 
     -
   * - **Byte|Int|Long|BigInt**.hexString()
     - HEX 文字列に変換
     - String
   * - **Byte|Int|Long|BigInt**.octString()
     - Oct 文字列に変換
     - String
   * - **Byte|Int|Long|BigInt**.binString()
     - Bin 文字列に変換
     - String
   * - **Byte|Int|Long|BigInt**.hexString(bitSize)
     - 最初にビットサイズに整列し、次に HEX 文字列に変換
     - String
   * - **Byte|Int|Long|BigInt**.octString(bitSize)
     - 最初にビットサイズに整列し、次に Oct 文字列に変換
     - String
   * - **Byte|Int|Long|BigInt**.binString(bitSize)
     - 最初にビットサイズに整列し、次に Bin 文字列に変換
     - String
   * - 
     - 
     -
   * - **Byte|Int|Long|BigInt**.toBinInts()
     - Binary リストに変換
     - List[Int]
   * - **Byte|Int|Long|BigInt**.toDecInts()
     - Decimal リストに変換
     - List[Int]
   * - **Byte|Int|Long|BigInt**.toOctInts()
     - Octal リストに変換
     - List[Int]
   * - **Byte|Int|Long|BigInt**.toBinInts(num)
     - Binary リストに変換、num のサイズに整列して 0で埋める
     - List[Int]
   * - **Byte|Int|Long|BigInt**.toDecInts(num)
     - Decimal リストに変換、num のサイズに整列して 0で埋める
     - List[Int]
   * - **Byte|Int|Long|BigInt**.toOctInts(num)
     - Octal リストに変換、num のサイズに整列して 0で埋める
     - List[Int]
   * - **"3F2A"**.hexToBinInts
     - Hex 文字列を Binary リストに変換
     - List[Int]
   * - **"3F2A"**.hexToBinIntsAlign
     - Hex 文字列を Binary リストに変換し、 4の倍数に整列
     - List[Int]
   * - 
     - 
     -
   * - **List(1,0,1,0,...)**.binIntsToHex 
     - Binary リストを HexString に変換
     - String
   * - **List(1,0,1,0,...)**.binIntsToOct 
     - Binary リストを OctString に変換
     - String  
   * - **List(1,0,1,0,...)**.binIntsToHexAlignHigh 
     - Binary リストのサイズを 4の倍数に整列（ 0で埋める）してから HexString に変換
     - String
   * - **List(1,0,1,0,...)**.binIntsToOctAlignHigh
     - Binary リストのサイズを 3の倍数に整列（ 0で埋める）してから HexString に変換
     - String
   * - **List(1,0,1,0,...)**.binIntsToInt
     - Binary リスト（最大サイズ 32）を Int に変換
     - Int
   * - **List(1,0,1,0,...)**.binIntsToLong
     - Binary リスト（最大サイズ 64）を Long に変換
     - Long
   * - **List(1,0,1,0,...)**.binIntsToBigInt
     - Binary リスト（サイズ制限なし）を BigInt に変換
     - BigInt
   * - 
     - 
     -
   * - **Int**.toBigInt
     - 32.toBigInt == BigInt(32)
     - BigInt
   * - **Long**.toBigInt
     - 3233113232L.toBigInt == BigInt(3233113232L)
     - BigInt
   * - **Byte**.toBigInt
     - 8.toByte.toBigInt == BigInt(8.toByte)
     - BigInt
    
String を Int/Long/BigInt に変換
------------------------------------

.. code-block:: scala 

   import spinal.core.lib._

   $: "32FF190".asHex

   $: "12384798999999".asDec

   $: "123456777777700".asOct

   $: "10100011100111111".asBin


Int/Long/BigInt を String に変換
---------------------------------

.. code-block:: scala 

   import spinal.core.lib._

   $: "32FF190".asHex.hexString()
   "32FF190"
   $: "123456777777700".asOct.octString() 
   "123456777777700"
   $: "10100011100111111".asBin.binString() 
   "10100011100111111"
   $: 32323239988L.hexString()
   7869d8034
   $: 3239988L.octString()
   14270064
   $: 34.binString()
   100010
 

Int/Long/BigIntをBinary-Listに変換
-----------------------------------------

.. code-block:: scala

   import spinal.core.lib._

   $: 32.toBinInts
   List(0, 0, 0, 0, 0, 1)
   $: 1302309988L.toBinInts
   List(0, 0, 1, 0, 0, 1, 1, 0, 0, 0, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 0, 1, 1, 0, 1, 1, 0, 0, 1)
   $: BigInt("100101110", 2).toBinInts
   List(0, 1, 1, 1, 0, 1, 0, 0, 1)
   $: BigInt("123456789abcdef0", 16).toBinInts
   List(0, 0, 0, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 0, 1, 1, 1, 1, 0, 1, 0, 1, 0, 1, 1, 0, 0, 1, 0, 0, 0, 1, 1, 1, 1, 0, 0, 1, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 1, 0, 0, 0, 1, 0, 0, 1)
   $: BigInt("1234567", 8).toBinInts
   List(1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 0, 1, 1, 1, 0, 0, 1, 0, 1)
   $: BigInt("123451118", 10).toBinInts
   List(0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 1, 1, 0, 1, 0, 1, 1, 1)
    
固定幅に整列する

.. code-block:: scala

   import spinal.core.lib._

   $: 39.toBinInts()
   List(1, 1, 1, 0, 0, 1)
   $: 39.toBinInts(8)    // align to 8 bit zero filled at MSB
   List(1, 1, 1, 0, 0, 1, 0, 0)


Binary-List を Int/Long/BigInt に変換
---------------------------------------

.. code-block:: scala

   import spinal.core.lib._

   $: List(1, 1, 1, 0, 0, 1).binIntsToInt
   39
   $: List(1, 1, 1, 0:, 0, 1).binIntsToLong
   39
   $: List(0, 0, 1, 0, 0, 1, 1, 0, 0, 0, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 0, 1, 1, 0, 1, 1, 0, 0, 1).binIntsToBigInt
   1302309988


    
.. code-block:: scala

   $: List(1, 1, 1, 0, 0, 1).binIntsToHex
   27
   $: List(1, 1, 1, 0, 0, 1).binIntsToHexAlignHigh
   9c
   $: List(1, 1, 1, 0, 0, 1).binIntsToOct
   47
   $: List(1, 1, 1, 0, 0, 1).binIntsToHexAlignHigh
   47


BigIntエンリッチャー
-------------------------

.. code-block:: scala

   $: 32.toBigInt
   32
   $: 3211323244L.toBigInt
   3211323244
   $: 8.toByte.toBigInt
   8
