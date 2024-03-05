.. role:: raw-html-m2r(raw)
   :format: html

.. _state_machine:

State machine
=============

Introduction
------------

SpinalHDL では、VHDL/Verilog と同様に、列挙型と switch/case 文を使ってステートマシンを定義することができます。
しかし、SpinalHDL では専用の構文も使用できます。

以下のステートマシンは、次の例で実装されています:

.. image:: /asset/picture/fsm_simple.svg
   :align: center
   :width: 300

スタイル A:

.. code-block:: scala

   import spinal.lib.fsm._

   class TopLevel extends Component {
     val io = new Bundle {
       val result = out Bool()
     }

     val fsm = new StateMachine {
       val counter = Reg(UInt(8 bits)) init(0)
       io.result := False

       val stateA : State = new State with EntryPoint {
         whenIsActive(goto(stateB))
       }
       val stateB : State = new State {
         onEntry(counter := 0)
         whenIsActive {
           counter := counter + 1
           when(counter === 4) {
             goto(stateC)
           }
         }
         onExit(io.result := True)
       }
       val stateC : State = new State {
         whenIsActive(goto(stateA))
       }
     }
   }

スタイル B:

.. code-block:: scala

   import spinal.lib.fsm._

   class TopLevel extends Component {
     val io = new Bundle {
       val result = out Bool()
     }

     val fsm = new StateMachine {
       val stateA = new State with EntryPoint
       val stateB = new State
       val stateC = new State

       val counter = Reg(UInt(8 bits)) init(0)
       io.result := False

       stateA
         .whenIsActive(goto(stateB))

       stateB
         .onEntry(counter := 0)
         .whenIsActive {
           counter := counter + 1
           when(counter === 4) {
             goto(stateC)
           }
         }
         .onExit(io.result := True)

       stateC
         .whenIsActive(goto(stateA))
     }
   }

StateMachine
------------

``StateMachine`` クラスは、ステートマシンの基本クラスであり、ステートマシンのロジックを管理します。

.. code-block:: scala

   val myFsm = new StateMachine {
     // ステートの定義
   }

``StateMachine`` クラスはいくつかのアクセサ関数も提供しています。

.. list-table::
   :header-rows: 1
   :widths: 1 1 5

   * - 名前
     - 返り値	
     - 説明
   * - ``isActive(state)``
     - ``Bool``
     - 指定されたステートがアクティブかどうかを ``True`` で返します。
   * - ``isEntering(state)``
     - ``Bool``
     - 指定されたステートに遷移しているかどうかを ``True`` で返します。

Entry point
^^^^^^^^^^^

ステートは、EntryPoint トレイトを継承することで、ステートマシンのエントリーポイントとして定義できます。

.. code-block:: scala

   val stateA = new State with EntryPoint

または ``setEntry(state)`` メソッドを使用する:

.. code-block:: scala

   val stateA = new State
   setEntry(stateA)


Transitions
^^^^^^^^^^^

* 遷移は ``goto(nextState)`` 関数で表現され、次のサイクルでステートマシンを ``nextState`` に遷移させるようにスケジュールします
* ``exit()`` 関数は、次のサイクルでステートマシンをブートステートに遷移させるようにスケジュールします 
  (または ``StateFsm`` では、現在のネストされたステートマシンを終了させます)。

これらの関数は、ステートの定義内 (後述) または ``always { yourStatements }`` ブロック内で使用できます。
always ブロックは常に ``yourStatements`` を適用し、ステートよりも優先順位が高くなります。


State encoding
^^^^^^^^^^^^^^

デフォルトでは、FSM ステートベクトルは、
RTL が生成される言語/ツールのネイティブエンコーディング (Verilog または VHDL) を使用してエンコードされます。
このデフォルトは、 ``setEncoding(...)`` メソッドを使用して、 ``SpinalEnumEncoding`` またはカスタムエンコーディング 
``(State, BigInt)`` の可変長引数を設定することでオーバーライドできます。


.. code-block:: scala
   :caption: Using a ``SpinalEnumEncoding``
   
   val fsm = new StateMachine {
     setEncoding(binaryOneHot)

     ...
   }

.. code-block:: scala
   :caption: Using a custom encoding

   val fsm = new StateMachine {
     val stateA = new State with EntryPoint
     val stateB = new State
     ...
     setEncoding((stateA -> 0x23), (stateB -> 0x22))
   }

.. warning:: ``graySequential`` 列挙エンコーディングを使用する場合、
             FSM の遷移がステートベクトルで 1 ビットのみ変化するようにするためのチェックは行われません。
             エンコーディングはステート定義順序に従って行われるため、必要に応じて有効な遷移のみが行われるように設計者が確認する必要があります。

States
------

SpinalHDL では、以下の種類のステートを使用できます。

* ``State``: 基本的なステートクラスです。
* ``StateDelay``
* ``StateFsm``
* ``StateParallelFsm``
 
これらの各ステートは、以下の関数を提供し、ステートに関連するロジックを定義できます。

.. list-table::
   :header-rows: 1
   :widths: 1 10

   * - 名前
     - 説明
   * - .. code-block:: scala
     
          state.onEntry {
            yourStatements
          }
     - ステートマシンが現在の ``state`` にならず、次のサイクルで現在の ``state`` になる場合に ``yourStatements`` が実行されます。
   * - .. code-block:: scala
         
          state.onExit {
            yourStatements
          }
     - ステートマシンが現在の ``state`` にあり、次のサイクルで別の ``state`` になる場合に ``yourStatements`` が実行されます。
     
          state.whenIsActive {
            yourStatements
          }
     - ステートマシンが現在の ``state`` にある場合に ``yourStatements`` が実行されます。
   * - .. code-block:: scala
     
          state.whenIsNext {
            yourStatements
          }
     - ステートマシンが次のサイクルで現在の ``state`` になる場合に (すでに現在の ``state`` にいても) ``yourStatements`` が実行されます。

``new State`` ブロックの中では、 ``state.`` を記述する必要はありません。

.. image:: /asset/picture/fsm_stateb.svg
   :align: center
   :width: 300

.. code-block:: scala

   val stateB : State = new State {
     onEntry(counter := 0)
     whenIsActive {
       counter := counter + 1
       when(counter === 4) {
         goto(stateC)
       }
     }
     onExit(io.result := True)
   }

StateDelay
^^^^^^^^^^

``StateDelay`` は、一定時間ステートを維持し、その後に ``whenCompleted {...}`` ブロック内のステートメントを実行するステートです。
一般的には次のように使用します。

.. code-block:: scala

   val stateG : State = new StateDelay(cyclesCount=40) {
     whenCompleted {
       goto(stateH)
     }
   }

1行で書くこともできます:

.. code-block:: scala

   val stateG : State = new StateDelay(40) { whenCompleted(goto(stateH)) }

StateFsm
^^^^^^^^

``StateFsm`` は、ネストされたステートマシンを含むステートを定義します。
ネストされたステートマシンが終了すると、 ``whenCompleted { ... }`` ブロック内のステートメントが実行されます。

以下は StateFsm の定義例です。

.. code-block:: scala

   // internalFsm は以下で定義されている関数です 
   val stateC = new StateFsm(fsm=internalFsm()) {
     whenCompleted {
       goto(stateD)
     }
   }

   def internalFsm() = new StateMachine {
     val counter = Reg(UInt(8 bits)) init(0)

     val stateA : State = new State with EntryPoint {
       whenIsActive {
         goto(stateB)
       }
     }

     val stateB : State = new State {
       onEntry (counter := 0)
       whenIsActive {
         when(counter === 4) {
           exit()
         }
         counter := counter + 1
       }
     }
   }

この例では、 ``exit()`` 関数はステートマシンをブートステート (内部的に隠れているステート) に遷移させます。
これにより、``StateFsm`` はネストされたステートマシンの終了を認識します。

StateParallelFsm
^^^^^^^^^^^^^^^^

``StateParallelFsm`` を使用すると、複数のネストされた状態マシンを処理できます。
すべてのネストされた状態マシンが完了すると、 ``whenCompleted { ... }`` 内の文が実行されます。

例:

.. code-block:: scala

   val stateD = new StateParallelFsm (internalFsmA(), internalFsmB()) {
     whenCompleted {
       goto(stateE)
     }
   }

エントリ状態に関する注意
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

上記で定義されたエントリ状態の方法により、リセットと最初のクロックサンプリングの間に、状態マシンはブート状態になります。
最初のクロックサンプリング後に、定義されたエントリ状態がアクティブになります。
これにより、エントリ状態に正しく入ることができます（ ``onEntry`` 内の文を適用）、およびネストされた状態マシンを使用できます。

これは便利ですが、この機能をバイパスしてユーザー状態に直接ブートすることも可能です。

これを行うには、 ``new State`` を定義する代わりに `makeInstantEntry()` を使用します。
この関数は、リセット直後に直接アクティブになるブート状態を返します。

.. note::
   この状態の ``onEntry`` は、他の状態からこの状態に遷移するときにのみ呼び出され、ブート時には呼び出されません。

.. note::
   シミュレーション中、ブート状態は常に ``BOOT`` という名前です。

例:

.. code-block:: scala

    // 状態シーケンス：IDLE、STATE_A、STATE_B、...
    val fsm = new StateMachine {
      // IDLE はシミュレーションでは BOOT と名前が付けられます
      val IDLE = makeInstantEntry()
      val STATE_A, STATE_B, STATE_C = new State
      
      IDLE.whenIsActive(goto(STATE_A))
      STATE_A.whenIsActive(goto(STATE_B))
      STATE_B.whenIsActive(goto(STATE_C))
      STATE_C.whenIsActive(goto(STATE_B))
    }

.. code-block:: scala

    // 状態シーケンス：BOOT、IDLE、STATE_A、STATE_B、...
    val fsm = new StateMachine {
      val IDLE, STATE_A, STATE_B, STATE_C = new State
      setEntry(IDLE)
      
      IDLE.whenIsActive(goto(STATE_A))
      STATE_A.whenIsActive(goto(STATE_B))
      STATE_B.whenIsActive(goto(STATE_C))
      STATE_C.whenIsActive(goto(STATE_B))
    }
