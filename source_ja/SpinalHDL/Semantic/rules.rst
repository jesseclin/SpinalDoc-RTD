Rules
=====

SpinalHDL の意味論を学ぶことは重要です。それによって、裏側で実際に何が起こっているのか、そしてそれをどのように制御するかが理解できます。

これらのセマンティクスは、複数のルールによって定義されます：

* シグナルとレジスタは互いに並行して動作します（ VHDL や Verilog と同様の並行動作です）
* 組み合わせ信号への代入は、常に真であるルールを表すのと同じです
* レジスタへの代入は、そのクロックドメインの各サイクルで適用されるルールを表すのと同じです
* 各シグナルについて、最後の有効な代入が優先されます
* ハードウェアの展開中に、それぞれのシグナルとレジスタを `OOP <https://en.wikipedia.org/wiki/Object-oriented_programming>`_ 的にオブジェクトとして操作できます


並行性
-----------

組み合わせ信号またはレジスタ信号を割り当てる順序には、動作に影響を与えません。

たとえば、次のコードの両方が等価です：

.. code-block:: scala

   val a, b, c = UInt(8 bits) // 3つの組み合わせ信号を定義します
   c := a + b  // c は 7 に設定されます
   b := 2      // b は 2 に設定されます
   a := b + 3  // a は 5 に設定されます

これは次のものと同等です：

.. code-block:: scala

   val a, b, c = UInt(8 bits) // 3つの組み合わせ信号を定義します
   b := 2      // b は 2 に設定されます
   a := b + 3  // a は 5 に設定されます
   c := a + b  // c は 7 に設定されます

一般的に、``:=`` 代入演算子を使用すると、左側の信号/レジスタに対する追加の新しいルールが指定されるのと同じです。

最後に有効な割り当てが勝ちます
-------------------------------

SpinalHDLの ``:=`` 演算子を使用して組み合わせ信号またはレジスタに複数回割り当てられた場合、
実行される最後の割り当てが勝ちます（したがって、その状態に対して結果として値を設定することができます）。

その時点で存在する状態に基づいて、上から下への評価が行われると言えます。
上流の信号入力がレジスタから駆動され、同期動作を持つ場合、各クロックサイクルで割り当てが新しい状態に基づいて再評価されると言えます。

割り当てステートメントがハードウェアで実行されない可能性があるいくつかの理由は、それが ``when(cond)`` 句でラップされているためかもしれません。

別の理由として、SpinalHDL コードが実行されなかった場合があります。それは機能がパラメータ化され、HDL コード生成中に無効にされたためです。
以下の ``paramIsFalse`` の使用を参照してください。

例えば：

.. code-block:: scala

   // 各クロックサイクルの評価はここから始まります
   val paramIsFalse = false
   val x, y = Bool()           // 2つの組み合わせ信号を定義します
   val result = UInt(8 bits)   // 組み合わせ信号を定義します

   result := 1
   when(x) {
     result := 2
     when(y) {
       result := 3
     }
   }
   if(paramIsFalse) {          // この割り当てが最後なので勝ちますが、エラボレーション時に評価が false になったため、ハードウェアに展開されませんでした。
     result := 4               // 上記の3つの := 割り当てはハードウェアに展開されます。
   }                           

これにより、次の真理値表が生成されます：


.. list-table::
   :header-rows: 1

   * - x
     - y
     - =>
     - result
   * - False
     - False
     - 
     - 1
   * - False
     - True
     - 
     - 1
   * - True
     - False
     - 
     - 2
   * - True
     - True
     - 
     - 3

Scala（OOPリファレンス+関数）における信号とレジスタの相互作用
-----------------------------------------------------------------------

SpinalHDL では、各ハードウェア要素がクラスのインスタンスでモデル化されます。
これは、参照を使用してインスタンスを操作できることを意味し、たとえば、それらを関数の引数として渡すことができます。

例えば、次のコードは、 ``inc`` が True のときにレジスタをインクリメントし、 
``clear`` が True のときにクリアするレジスタを実装しています（ ``clear`` は ``inc`` より優先されます）：

.. code-block:: scala

   val inc, clear = Bool()          // 2つの組み合わせ信号/ワイヤを定義します
   val counter = Reg(UInt(8 bits))  // 8ビットレジスタを定義します

   when(inc) {
     counter := counter + 1
   }
   when(clear) {
     counter := 0    // inc と clear が True の場合、この割り当てが勝ちます（最後の値の割り当てが勝つルール）
   }


同じ機能を実装するためには、以前の例を関数と組み合わせることができます：

.. code-block:: scala

   val inc, clear = Bool()
   val counter = Reg(UInt(8 bits))

   def setCounter(value : UInt): Unit = {
     counter := value
   }

   when(inc) {
     setCounter(counter + 1)  // カウンタをカウンタ + 1 に設定します
   }
   when(clear) {
     counter := 0
   }

関数内で条件チェックを組み込むこともできます：

.. code-block:: scala

   val inc, clear = Bool()
   val counter = Reg(UInt(8 bits))

   def setCounterWhen(cond : Bool,value : UInt): Unit = {
     when(cond) {
       counter := value
     }
   }

   setCounterWhen(cond = inc,   value = counter + 1)
   setCounterWhen(cond = clear, value = 0)


また、関数に何を割り当てるかを指定することもできます：

.. code-block:: scala
  
   val inc, clear = Bool()
   val counter = Reg(UInt(8 bits))

   def setSomethingWhen(something : UInt, cond : Bool, value : UInt): Unit = {
     when(cond) {
       something := value
     }
   }

   setSomethingWhen(something = counter, cond = inc,   value = counter + 1)
   setSomethingWhen(something = counter, cond = clear, value = 0)


すべての前述の例は、生成された RTL および SpinalHDL コンパイラの観点から厳密に等価です。
これは、SpinalHDL が Scala ランタイムとそこでインスタンス化されたオブジェクトにのみ関心を持ち、
Scala の構文自体には関心を持たないためです。

言い換えると、Scala の関数を使用してハードウェアを生成する場合、生成された RTL の観点/ SpinalHDL の観点からは、
その関数がインライン化されたかのようです。これは Scala のループにも当てはまり、生成された RTL では展開された形で表示されます。
