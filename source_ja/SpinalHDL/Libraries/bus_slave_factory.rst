.. role:: raw-html-m2r(raw)
   :format: html

.. _bus_slave_factory:

Bus Slave Factory
=================

Introduction
------------

多くの状況で、バスレジスタバンクを実装する必要があります。 ``BusSlaveFactory`` は、それらを定義するための抽象的でスムーズな方法を提供するツールです。

このツールの機能を見るために、単純な例として、:ref:`memory_mapped_uart <memory_mapped_uart>` を実装するために Apb3SlaveFactory の変形を使用したものを見てみましょう。また、メモリマッピング機能を含む :ref:`Timer <timer>` の別の例もあります。

``BusSlaveFactory`` ツールの内部実装に関する詳細なドキュメントは、:ref:`こちら <bus_slave_factory_implementation>` で見つけることができます。

Functionality
-------------

| ``BusSlaveFactory`` ツールには、多くの実装があります：AHB3-lite、APB3、APB4、AvalonMM、AXI-lite 3、AXI4、BMB、Wishbone、および PipelinedMemoryBus。
| このツールの各実装は、対応するバスのインスタンスを引数として受け取り、次のような機能を提供してハードウェアをメモリマッピングにマップします：

.. list-table::
   :header-rows: 1
   :widths: 2 1 10

   * - 名前
     - 戻り値
     - 説明
   * - busDataWidth
     - Int
     - バスのデータ幅を返す
   * - read(that,address,bitOffset)
     - 
     - バスが ``address`` を読み取るとき、レスポンスを ``bitOffset`` に ``that`` で満たす
   * - write(that,address,bitOffset)
     - 
     - バスが ``address`` を書き込むとき、 ``that`` を ``bitOffset`` からのバスのデータで割り当てる
   * - onWrite(address)(doThat)
     - 
     - ``address`` に書き込みトランザクションが発生したとき、 ``doThat`` を呼び出す
   * - onRead(address)(doThat)
     - 
     - ``address`` で読み取りトランザクションが発生したとき、 ``doThat`` を呼び出す
   * - nonStopWrite(that,bitOffset)
     - 
     - ``that`` を常にバスの書き込みデータから ``bitOffset`` で割り当てる
   * - readAndWrite(that,address,bitOffset)
     - 
     - ``that`` を ``address`` で読み書き可能にし、単語内の ``bitOffset`` に配置する 
   * - readMultiWord(that,address)
     - 
     - | ``that`` を ``address`` から読み取るメモリマッピングを作成する。 
       | ``that` が 1ワードよりも大きい場合、後続のアドレスにレジスタを拡張します
   * - writeMultiWord(that,address)
     - 
     - | ``that`` を ``address`` に書き込むメモリマッピングを作成します。 
       | ``that`` が 1ワードよりも大きい場合、後続のアドレスにレジスタを拡張します
   * - createWriteOnly(dataType,address,bitOffset)
     - T
     - ``address`` と単語内の ``bitOffset`` にタイプ ``dataType`` の書き込み専用レジスタを作成します
   * - createReadWrite(dataType,address,bitOffset)
     - T
     - ``address`` と単語内の ``bitOffset`` にタイプ ``dataType`` の読み書き可能なレジスタを作成します
   * - createAndDriveFlow(dataType,address,bitOffset)
     - Flow[T]
     - ``address`` と単語内の ``bitOffset`` に書き込み可能な ``Flow`` レジスタを作成します
   * - drive(that,address,bitOffset)
     - 
     - ``that`` を、単語内の ``bitOffset`` に配置された ``address`` で書き込み可能なレジスタでドライブします
   * - driveAndRead(that,address,bitOffset)
     - 
     - ``that`` を、単語内の ``bitOffset`` に配置された ``address`` で書き込み可能で読み取り可能なレジスタでドライブします
   * - driveFlow(that,address,bitOffset)
     - 
     - ``address`` で書き込みが発生した際に、単語内の ``bitOffset`` に配置されたデータを使用して、 ``that`` にトランザクションを送信します
   * - | readStreamNonBlocking(that,
       |                       address,
       |                       validBitOffset,
       |                       payloadBitOffset)
     - 
     - | ``address`` で読み込みが発生した際に、 ``that`` を読み取り、トランザクションを消費します。
       | valid <= validBitOffset ビット 
       | payload <= payloadBitOffset+widthOf(payload) ダウントゥ ``payloadBitOffset``」 
   * - | doBitsAccumulationAndClearOnRead(that,
       |                                  address,
       |                                  bitOffset)
     - 
     - | 毎サイクル、次のように内部レジスタをインスタンス化します： 
       | reg := reg | that。
       | その後、読み込みが発生すると、レジスタがクリアされます。このレジスタは、ワード内の ``address`` に配置され、 ``bitOffset`` に配置されています。 
