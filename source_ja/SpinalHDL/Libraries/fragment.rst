
Fragment
========

仕様
-------------

``Fragment`` バンドルは、 "大きな" ものを複数の "小さな" フラグメントを使用して送信する概念です。例えば：

* 幅×高さのトランザクションで送信される画像は、 ``Stream[Fragment[Pixel]]`` 上で送信されます。
* フロー制御のないコントローラから受信した UART パケットは、 ``Flow[Fragment[Bits]]`` 上で送信されます。
* AXI リードバーストは、 ``Stream[Fragment[AxiReadResponse]]`` で送信される可能性があります。

``Fragment`` バンドルで定義されたシグナルは次のとおりです：

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 5

   * - シグナル
     - タイプ
     - ドライバー
     - 説明
   * - fragment
     - T
     - マスター
     - 現在のトランザクションの "ペイロード" 
   * - last
     - Bool
     - マスター
     - ``Fragment`` が現在のパケットの最後のものである場合に高い


この仕様と先行例からわかるように、 ``Fragment`` の概念はトランザクションがどのように送信されるかを指定していません
（Stream、Flow、または他の通信プロトコルを使用できます）。
それは現在のトランザクションが最初のものか、最後のものか、または特定のパケットの途中にあるものかを知るための十分な情報 (\ ``last``\ ) のみを追加します。

.. note::
   このプロトコルは \'first\' ビットを伝送しなかったため、どこでも 'RegNextWhen（bus.last、bus.fire）init（True）' を実行することで生成できます。
   
機能
---------

``Stream[Fragment[T]]`` および ``Flow[Fragment[T]]``\ のための、以下の機能が提供されています：

.. list-table::
   :header-rows: 1
   :widths: 1 1 20

   * - 構文
     - 戻り値	
     - 説明
   * - x.first
     - Bool
     - 次のトランザクションまたは現在のトランザクションが/される最初のパケットの場合に True を返します
   * - x.tail
     - Bool
     - 次のトランザクションまたは現在のトランザクションが/される最初でないパケットの場合に True を返します
   * - x.isFirst
     - Bool
     - トランザクションが存在し、パケットの最初の場合に True を返します
   * - x.isTail
     - Bool
     - トランザクションが存在し、パケットの最初でも最後でもない場合に True を返します
   * - x.isLast
     - Bool
     - トランザクションが存在し、パケットの最後の場合に True を返します


``Stream[Fragment[T]]`` のために、以下の機能もアクセス可能です：

.. list-table::
   :header-rows: 1
   :widths: 1 1 1

   * - 構文
     - 戻り値	
     - 説明
   * - x.insertHeader(header : T)
     - Stream[Fragment[T]]
     - ``x`` 上の各パケットに ``header`` を追加し、結果のバスを返します
