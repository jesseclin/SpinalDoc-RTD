When/Switch/Mux
===============

When
----

VHDL や Verilog と同様に、指定された条件が満たされた場合に信号に条件付きで代入できます：

.. code-block:: scala

   when(cond1) {
     // cond1 が true の場合に実行
   } elsewhen(cond2) {
     // (cond1 が false)かつ cond2 の場合に実行
   } otherwise {
     // (cond1 が false)かつ(cond2 が false)の場合に実行
   }

.. warning::

     キーワード ``otherwise`` が ``when`` 条件の閉じ括弧 ``}`` と同じ行にある場合、ドットは不要です。
     
     .. code-block:: scala

            when(cond1) {
                // cond1 が true の場合に実行
            } otherwise {
                // (cond1 が false)かつ(cond2 が false)の場合に実行
            }

     しかし、 ``.otherwise`` が別の行にある場合、ドットは **必須** です：

     .. code-block:: scala

            when(cond1) {
                // cond1 が true の場合に実行
            }
            .otherwise {
                // (cond1 が false )かつ(cond2 が false )の場合に実行
            }

Switch
------

VHDL や Verilog と同様に、信号に定義された値に応じて条件付きで割り当てることができます：

.. code-block:: scala

   switch(x) {
     is(value1) {
       // x === value1 のときに実行
     }
     is(value2) {
       // x === value2 のときに実行
     }
     default {
       // 前述の条件に一致しない場合に実行
     }
   }

``is`` 節は、コンマで区切って ``is(value1, value2)`` のようにファクター化（論理OR）できます。

例
^^^^^^^

.. code-block:: scala

  switch(aluop) {
    is(ALUOp.add) {
      immediate := instruction.immI.signExtend
    }
    is(ALUOp.slt) {
      immediate := instruction.immI.signExtend
    }
    is(ALUOp.sltu) {
      immediate := instruction.immI.signExtend
    }
    is(ALUOp.sll) {
      immediate := instruction.shamt
    }
    is(ALUOp.sra) {
      immediate := instruction.shamt
    }
  }

以下と同等です:

.. code-block:: scala

  switch(aluop) {
    is(ALUOp.add, ALUOp.slt, ALUOp.sltu) {
        immediate := instruction.immI.signExtend
    }
    is(ALUOp.sll, ALUOp.sra) {
        immediate := instruction.shamt
    }
  }

追加のオプション
^^^^^^^^^^^^^^^^^^

デフォルトでは、SpinalHDL は、 ``switch`` に ``default`` ステートメントが含まれており、すべての可能な論理値が ``is`` ステートメントによって既にカバーされている場合、
"UNREACHABLE DEFAULT STATEMENT"エラーを生成します。これをエラー報告から除外するには、 ``switch(myValue, coverUnreachable = true) { ... }`` と指定します。

.. code-block:: scala
  
  switch(my2Bits, coverUnreachable = true) {
      is(0) { ... }
      is(1) { ... } 
      is(2) { ... }
      is(3) { ... }
      default { ... } // これでエラーなしで解析および検証されます
  }
  
.. note::

   このチェックは、物理的な値ではなく論理値で行われます。たとえば、1ホット方式でエンコードされた SpinalEnum(A、B、C) がある場合、
   SpinalHDL はA、B、Cの値（"001" "010" "100"）のみを考慮します。物理的な値は"000" "011" "101" "110" "111"のようになるため、考慮されません。

デフォルトでは、SpinalHDL は、指定された ``is`` ステートメントが同じ値を複数回提供する場合に、
"DUPLICATED ELEMENTS IN SWITCH IS(...) STATEMENT" エラーを生成します。例えば ``is(42,42) { ... }`` です。
このエラー報告を削除するには、 ``switch(myValue, strict = true){ ... }`` を指定します。SpinalHDL は、重複した値を削除するようにします。

.. code-block:: scala
  
  switch(value, strict = false) {
      is(0) { ... }
      is(1,1,1,1,1) { ... } // これは問題ありません
      is(2) { ... }
  }

ローカル宣言
-----------------

when/switch 文の内部で新しいシグナルを定義することができます：

.. code-block:: scala

   val x, y = UInt(4 bits)
   val a, b = UInt(4 bits)

   when(cond) {
     val tmp = a + b
     x := tmp
     y := tmp + 1
   } otherwise {
     x := 0
     y := 0
   }

.. note::
   SpinalHDL は、スコープ内で定義されたシグナルがそのスコープ内でのみ割り当てられていることをチェックします。

Mux
---

``Bool`` 選択信号を持つ単純な ``Mux`` が必要な場合、2つの同等の構文があります：

.. list-table::
   :header-rows: 1
   :widths: 4 1 4

   * - 構文
     - 戻り値
     - 説明
   * - Mux(cond, whenTrue, whenFalse)
     - T
     - ``cond`` が True の場合は ``whenTrue`` を返し、それ以外の場合は ``whenFalse`` を返します。 
   * - cond ? whenTrue | whenFalse
     - T
     - ``cond`` が True の場合は ``whenTrue`` を返し、それ以外の場合は ``whenFalse`` を返します。 

.. code-block:: scala

   val cond = Bool()
   val whenTrue, whenFalse = UInt(8 bits)
   val muxOutput  = Mux(cond, whenTrue, whenFalse)
   val muxOutput2 = cond ? whenTrue | whenFalse


Bitwise selection
-----------------

ビット単位の選択は、VHDL の ``when``構文のように見えます。

例
^^^^^^^

.. code-block:: scala

   val bitwiseSelect = UInt(2 bits)
   val bitwiseResult = bitwiseSelect.mux(
     0 -> (io.src0 & io.src1),
     1 -> (io.src0 | io.src1),
     2 -> (io.src0 ^ io.src1),
     default -> (io.src0)
   )

``mux`` は、ラッチの生成を防ぐためにすべての可能な値がカバーされているかをチェックします。
すべての可能な値がカバーされている場合、デフォルトのステートメントを追加してはいけません:
.. code-block:: scala

   val bitwiseSelect = UInt(2 bits)
   val bitwiseResult = bitwiseSelect.mux(
     0 -> (io.src0 & io.src1),
     1 -> (io.src0 | io.src1),
     2 -> (io.src0 ^ io.src1),
     3 -> (io.src0)
   )

``muxList(...)`` と ``muxListDc(...)`` は、入力としてタプルやマッピングのシーケンスを取る代替ビット単位のセレクタです。

``muxList`` は、ケースを生成するコードでより簡単に使用できるインターフェースを提供するため、 ``mux`` の直接的な代替として使用できます。
``mux`` と同じチェック動作を持ち、完全なカバレッジが必要であり、必要ない場合はデフォルトをリストに追加することを禁止します。

``muxtListDc`` は、カバーされていない値が重要でない場合に使用でき、これらは ``muxListDc`` を使用して未割り当てのままにすることができます。
必要に応じてデフォルトのケースが追加されます。このデフォルトのケースは、シミュレーション中に X を生成します。
一般的なコードで ``muxListDc(...)`` は良い代替手段です。

以下は、128 ビットの ``Bits`` を 32 ビットに分割する例です：

.. image:: /asset/picture/MuxList.png
   :align: center
   :width: 300px

.. code-block:: scala

   val sel  = UInt(2 bits)
   val data = Bits(128 bits)

   // 広いビットタイプを小さなチャンクに分割する方法、mux を使用します:
   val dataWord = sel.muxList(for (index <- 0 until 4)
                              yield (index, data(index*32+32-1 downto index*32)))

   // 同じことをする短い方法：
   val dataWord = data.subdivideIn(32 bits)(sel)

``muxListDc`` が設定可能な幅のベクトルからビットを選択する例：

.. code-block:: scala

  case class Example(width: Int = 3) extends Component {
    // デフォルトの幅のために 2 ビット幅
    val sel = UInt(log2Up(count) bit)
    val data = Bits(width*8 bit)
    // デフォルトの幅では 3 の不足したケースをカバーする必要がない
    val dataByte = sel.muxListDc(for(i <- 0 until count) yield (i, data(index*8, 8 bit)))
  }
