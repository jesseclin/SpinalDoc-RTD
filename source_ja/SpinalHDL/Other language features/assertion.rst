
Assertions
==========

Scala の実行時アサーションに加えて、以下の構文を使用してハードウェアアサーションを追加できます：

``assert(assertion : Bool, message : String = null, severity: AssertNodeSeverity = Error)``

重要度のレベルは次のとおりです:

.. list-table::
   :header-rows: 1
   :widths: 1 3

   * - 名前
     - 説明
   * - NOTE
     - 情報メッセージを報告するために使用されます
   * - WARNING
     - 異常なケースを報告するために使用されます
   * - ERROR
     - 発生すべきでない状況を報告するために使用されます
   * - FAILURE
     - 致命的な状況を報告し、シミュレーションを終了します


実用的な例として、ハンドシェイクプロトコルの ``ready`` が低いときに ``valid`` シグナルが落ちないことをチェックすることが挙げられます：

.. code-block:: scala

   class TopLevel extends Component {
     val valid = RegInit(False)
     val ready = in Bool()

     when(ready) {
       valid := False
     }
     // いくつかのロジック

     assert(
       assertion = !(valid.fall && !ready),
       message   = "Valid dropped when ready was low",
       severity  = ERROR
     )
   }
