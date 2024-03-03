
Flow
====

仕様
-------------

Flow インターフェースは、スレーブがバスを停止させることができないシンプルな有効/ペイロード プロトコルであり、
UART コントローラからのデータ、オンチップ メモリへの書き込みリクエストなどを表すために使用できます。

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 10 1

   * - シグナル
     - タイプ
     - ドライバー
     - 説明
     - これが発生したときに気にしない
   * - valid
     - Bool
     - マスター
     - ハイの場合 => インターフェース上にペイロードが存在します
     - 
   * - payload
     - T
     - マスター
     - トランザクションの内容
     - valid がローの場合


機能
---------

.. list-table::
   :header-rows: 1
   :widths: 1 10 1 1

   * - 構文
     - 説明
     - 戻り値
     - レイテンシー
   * - Flow(type : Data)
     - 指定されたタイプの Flow を作成します
     - Flow[T]
     - 
   * - master/slave Flow(type : Data)
     - | 指定されたタイプの Flow を作成します
       | 対応する in/out セットアップで初期化されます
     - Flow[T]
     - 
   * - x.m2sPipe()
     - | x によって駆動される Flow を返します
       | レジスタ ステージを通過し、有効/ペイロード パスをカットします
     - Flow[T]
     - 1
   * - | x << y
       | y >> x
     - y を x に接続します
     - 
     - 0
   * - | x <-< y
       | y >-> x
     - y を x に m2sPipe を介して接続します
     - 
     - 1
   * - x.throwWhen(cond : Bool)
     - | 条件が high の場合、x に接続された Flow を返します
       | トランザクションがドロップされます
     - Flow[T]
     - 0
   * - x.toReg()
     - valid が high の場合に ``payload`` で読み込まれるレジスタを返します
     - T
     - 
   * - x.setIdle()
     - Flow をアイドル状態に設定します： ``valid`` は ``False`` で、``payload`` については構いません。 
     -
     -
   * - x.push(newPayload: T)
     - 新しい有効なペイロードを ``Flow`` に割り当てます。 ``valid`` が ``True`` に設定されます。
     -
     -

Code example
------------

.. literalinclude:: /../examples/src/main/scala/spinaldoc/libraries/flow/FlowExample.scala
   :language: scala
   :start-at: case class FlowExample()
   :end-before: // end FlowExample

シミュレーション サポート
-------------------------

.. list-table::
  :header-rows: 1
  :widths: 1 5
  
  * - クラス
    - 使用法
  * - FlowMonitor
    - マスターおよびスレーブ側に使用され、Flow がデータを送信するとペイロード付きで関数を呼び出します。 
  * - FlowDriver
    - テストベンチ マスター側で使用され、値をドライブするために関数を呼び出します（利用可能な場合）。値が利用可能であったかどうかを返す必要があります。ランダムな遅延をサポートします。
  * - ScoreboardInOrder
    - リファレンス/DUT データを比較するためによく使用されます

.. literalinclude:: /../examples/src/main/scala/spinaldoc/libraries/flow/SimSupport.scala
   :language: scala
