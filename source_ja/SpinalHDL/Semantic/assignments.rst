Assignments
===========

複数の代入演算子があります：

.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - シンボル
     - 説明
   * - ``:=``
     - 標準の代入演算子で、VHDL/Verilogの ``<=`` と同等です。
   * - ``\=``
     - VHDLの ``:=`` やVerilogの ``=`` と同等です。値は即座にその場で更新されます。組み合わせ信号にのみ機能し、レジスタでは機能しません。 
   * - ``<>``
     - 同じ型の2つの信号または2つのバンドルの間の自動接続。信号の方向（入力/出力）を使用して方向が推論されます。（ ``:=``\ と同様の動作）

マルチプレクサ（たとえば、 ``when`` を使用する場合は、:doc:`when_switch` を参照してください）の際に、
最後に有効な標準の代入 ``:=`` が勝利します。それ以外の場合、同じスコープから同じ代入先に2回代入すると、
代入の重複が発生します。SpinalHDL は、これが意図しない設計エラーであるとデフォルトで仮定し、
エラーでエラボレーションを停止します。特別なユースケースでは、代入の重複をプログラムでケースバイケースで許可することができます
（:doc:`../Design errors/assignment_overlap` を参照）。

.. code-block:: scala

   val a, b, c = UInt(4 bits)
   a := 0
   b := a
   //a := 1 // これにより、手動で代入がオーバーライドされる場合、代入が優先されます。
            // これは `assignment overlap` エラーを引き起こします。
   c := a

   var x = UInt(4 bits)
   val y, z = UInt(4 bits)
   x := 0
   y := x      // y は値 0 で x を読み取りました
   x \= x + 1
   z := x      // z は値 1 で x を読み取りました

   // UARTインターフェース間の自動接続。
   uartCtrl.io.uart <> io.uart

それは Bundle 割り当てもサポートしています（すべてのビット信号を適切な幅の Bits 型の単一のビットバスに変換し、
その広い形式を割り当て式で使用します）。 割り当て式の左側と右側の両方で、Scala のタプル構文の ``()`` を使用して複数の信号をまとめます。


.. code-block:: scala

   val a, b, c = UInt(4 bits)
   val d       = UInt(12 bits)
   val e       = Bits(10 bits)
   val f       = SInt(2  bits)
   val g       = Bits()

   (a, b, c) := B(0, 12 bits)
   (a, b, c) := d.asBits
   (a, b, c) := (e, f).asBits           // 両側
   g         := (a, b, c, e, f).asBits  // 右側

SpinalHDL では、信号の性質（組み合わせ/順次）は割り当て方ではなく、宣言で定義されることが重要です。
すべてのデータ型インスタンスは組み合わせ信号を定義し、 ``Reg(...)`` でラップされたデータ型インスタンスは順次（登録）信号を定義します。

It is important to understand that in SpinalHDL, the nature of a signal (combinational/sequential) is defined in its declaration, not by the way it is assigned.
All datatype instances will define a combinational signal, while a datatype instance wrapped with ``Reg(...)`` will define a sequential (registered) signal.

.. code-block:: scala

   val a = UInt(4 bits)              // Define a combinational signal
   val b = Reg(UInt(4 bits))         // Define a registered signal
   val c = Reg(UInt(4 bits)) init(0) // Define a registered signal which is
                                     //  set to 0 when a reset occurs


幅チェック
--------------

SpinalHDL は、代入文の左辺と右辺のビット数が一致するかどうかをチェックします。
指定されたBitVector（ ``Bits`` 、 ``UInt`` 、 ``SInt`` ）の幅を調整するための複数の方法があります：

.. list-table::
   :header-rows: 1
   :widths: 3 5

   * - リサイズ技術
     - 説明
   * - x := y.resized
     - x に、y のサイズから推定されたリサイズされたコピーを割り当てます。
   * - x := y.resize(newWidth)
     - x に、y のリサイズされたコピーを、 :code:`newWidth` ビットの幅で割り当てます。
   * - x := y.resizeLeft(newWidth)
     - x に、y のリサイズされたコピーを、:code:`newWidth` ビットの幅で割り当てます。必要に応じて、LSBでパディングします。

すべてのリサイズメソッドは、結果の幅が、元の幅の `y` に比べて広くなるか狭くなる可能性があります。
幅が広がる場合、余分なビットはゼロでパディングされます。

``x.resized`` による推論変換は、代入式の左側のターゲット幅に基づき、 ``y.resize(someWidth)`` と同じ意味論に従います。
式 ``x := y.resized`` は、 ``x := y.resize(x.getBitsWidth bits)`` と同等です。

例のコードスニペットでは代入文の使用が示されていますが、リサイズメソッドのファミリーは、通常の Scala メソッドと同様にチェーンすることができます。

Spinal が値を自動的にリサイズするケースが1つあります：

.. code-block:: scala

   // U(3) は 2 ビットの UInt を生成しますが、左側（8 ビット）と一致しません
   myUIntOf_8bits := U(3)

``U(3)`` は「弱い」ビット数推論信号であるため、SpinalHDL は自動的に幅を広げます。
これは、 ``U(3, 2 bits).resized`` と機能的に等価と見なすことができます。
ただし、SpinalHDL は必要な場合には適切な動作を行い、シナリオが縮小を必要とする場合にはエラーを引き続き報告します。
例えば、リテラルが 9 ビットを必要とする場合（例: ``U(0x100)`` ）には、 ``myUIntOf_8bits`` に代入しようとするとエラーが報告されます。


組み合わせループ
-------------------

SpinalHDL は、設計に組み合わせループ（ラッチ）がないことをチェックします。
もし組み合わせループが検出された場合、エラーが発生し、SpinalHDL はループのパスを出力します。


CombInit
--------

``CombInit`` は、信号とその現在の組み合わせ割り当てをコピーするために使用できます。
主なユースケースは、後でコピーを上書きできるようにすることで、元の信号に影響を与えないようにすることです。

CombInit
--------

``CombInit`` can be used to copy a signal and its current combinatorial assignments. The main use-case is to be able to overwrite the copied later, without impacting the original signal.

.. code-block:: scala

    val a = UInt(8 bits)
    a := 1

    val b = a
    when(sel) {
        b := 2
        // この時点で、a と b はそれぞれ2に評価されます。つまり、同じ信号を参照しています。
    }

    val c = UInt(8 bits)
    c := 1

    val d = CombInit(c)
    // ここでは、c と d はそれぞれ1に評価されます。
    when(sel) {
        d := 2
        // この時点で、c === 1 and d === 2.
    }

結果の Verilog を見ると、 ``b`` は存在しません。 ``b`` は参照によって ``a`` のコピーなので、これらの変数は同じ Verilog の wire を指定します。

.. code-block:: verilog

    always @(*) begin
      a = 8'h01;
      if(sel) begin
        a = 8'h02;
      end
    end

    assign c = 8'h01;
    always @(*) begin
      d = c;
      if(sel) begin
        d = 8'h02;
      end
    end

``CombInit`` は、返される値が入力を参照していないことを確認するために、特にヘルパー関数で役立ちます。

.. code-block:: scala

    // 注意: condition は、論理合成時(elaboration time)の定数です
    def invertedIf(b: Bits, condition: Boolean): Bits = if(condition) { ~b } else { CombInit(b) }

    val a2 = invertedIf(a1, c)

    when(sel) {
       a2 := 0
    }

``CombInit`` なしでは、もし ``c``が false の場合（ただし、 ``c``が true の場合は除く）、 ``a1``と ``a2`` は同じ信号を参照し、ゼロの代入も ``a1`` に適用されます。
``CombInit`` を使用すると、 ``c`` の値に関係なく、一貫した動作が得られます。
