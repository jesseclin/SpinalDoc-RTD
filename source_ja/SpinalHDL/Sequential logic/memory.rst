RAM/ROM Memory
==============

SpinalHDL でメモリを作成するには、 ``Mem`` クラスを使用します。
これにより、メモリを定義し、読み取りポートと書き込みポートを追加できます。

次の表は、メモリをインスタンス化する方法を示しています：

.. list-table::
   :header-rows: 1
   :widths: 1 1

   * - 構文
     - 説明
   * - ``Mem(type : Data, size : Int)``
     - RAM を作成します
   * - ``Mem(type : Data, initialContent : Array[Data])``
     - ROM を作成します。FPGA を対象とする場合、メモリはブロック RAM として推論されるため、それに書き込みポートを作成することができます。


.. note::
   ROM を定義する場合、 ``initialContent`` 配列の要素はリテラル値のみである必要があります（演算子やリサイズ関数は含まれません）。例はこちらを参照してください。
   
.. note::
   RAM に初期値を与えるには、 ``init`` 関数を使用することもできます。
   
.. note::
   ライトマスクの幅は柔軟で、メモリワードをマスクの幅と同じ幅のスライスに分割します。
   たとえば、32ビットのメモリワードがある場合、4ビットのマスクが提供されると、バイトマスクになります。
   ワードビット数と同じだけのマスクビットが提供されると、ビットマスクになります。
   
.. note::
   ``Mem`` の操作はシミュレーション中に可能です。詳細については、セクション :ref:`シミュレーションにおけるメモリの読み込みと書き込み<simulation_of_memory>`
   を参照してください。

以下の表は、メモリにアクセスポートを追加する方法を示しています：

.. list-table::
   :header-rows: 1
   :widths: 1 30 1

   * - 構文
     - 説明
     - 戻り値
   * - mem(address) := data
     - 同期書き込み
     - 
   * - mem(x)
     - 非同期読み取り
     - T
   * - | mem.write(
       |  address
       |  data
       |  [enable]
       |  [mask]
       | )
     - | オプションのマスク付き同期書き込み。
       | ``enable`` が指定されていない場合、この関数が呼び出される条件スコープから自動的に推論されます。
     - 
   * - | mem.readAsync(
       |  address
       |  [readUnderWrite]
       | )
     - オプションの read-under-write ポリシーを持つ非同期読み込み
     - T
   * - | mem.readSync(
       |  address
       |  [enable]
       |  [readUnderWrite]
       |  [clockCrossing]
       | )
     - オプションの有効、 read-under-write ポリシー、および clockCrossing モードを持つ同期読み込み
     - T
   * - | mem.readWriteSync(
       |  address
       |  data
       |  enable
       |  write
       |  [mask]
       |  [readUnderWrite]
       |  [clockCrossing]
       | )
     - | 読み書きポートを推論します。
       | ``data`` は、 ``enable && write`` のときに書き込まれます。
       | ``enable`` が true のときに読み取りが行われ、読み取りデータを返します。 
     - T


.. note::
   もし Spinal に実装されていない特定のメモリポートが必要な場合は、そのメモリを BlackBox として指定することで抽象化することができます。
   
.. important::
   SpinalHDLにおけるメモリポートは暗黙的には推論されず、明示的に定義されます。
   VHDL や Verilog のようなコーディングテンプレートを使用して、合成ツールがメモリを推論するのを支援すべきではありません。

以下は、単純なデュアルポートRAM（ 32 ビット × 256 ）を推論する例です：   

.. code-block:: scala

   val mem = Mem(Bits(32 bits), wordCount = 256)
   mem.write(
     enable  = io.writeValid,
     address = io.writeAddress,
     data    = io.writeData
   )

   io.readData := mem.readSync(
     enable  = io.readValid,
     address = io.readAddress
   )

同期イネーブルの奇妙な動作
-----------------------------

イネーブル信号が `when` のような条件付きブロックで使用される場合、
アクセス条件として生成されるのはイネーブル信号のみであり、 `when` の条件は無視されます。

.. code-block:: scala

    val rom = Mem(Bits(10 bits), 32)
    when(cond){
      io.rdata := rom.readSync(io.addr, io.rdEna)
    }

上記の例では、条件 `cond` は詳細化されません。
以下のように、条件 `cond` を直接イネーブル信号に含めることを推奨します。

.. code-block:: scala

    io.rdata := rom.readSync(io.addr, io.rdEna & cond)

Read-under-write ポリシー
--------------------------------

このポリシーは、同じアドレスに同じサイクルで書き込みが行われた場合に、読み取りがどのように影響を受けるかを指定します。

.. list-table::
   :header-rows: 1
   :widths: 1 3

   * - 種類
     - 説明
   * - ``dontCare``
     - その場合に読み取り値を気にしない
   * - ``readFirst``
     - 読み取りは古い値（書き込み前の値）を取得します。
   * - ``writeFirst``
     - 読み取りは新しい値（書き込みによって提供された値）を取得します。


.. important::
   生成される VHDL/Verilog は常に ``readFirst`` モードであり、これは ``dontCare`` と互換性がありますが、
   ``writeFirst`` とは互換性がありません。 ``writeFirst`` を含むデザインを生成するには、
   :ref:`automatic memory blackboxing <automatic_memory_blackboxing>` を有効にする必要があります。

混合幅 RAM
---------------

これらの関数を使用して、メモリにアクセスするポートを、メモリ幅の2の累乗分数の幅で指定できます：

.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - 構文
     - 説明
   * - | mem.writeMixedWidth(
       |  address
       |  data
       |  [readUnderWrite]
       | )
     - ``mem.write`` と同様
   * - | mem.readAsyncMixedWidth(
       |  address
       |  data
       |  [readUnderWrite]
       | )
     - ``mem.readAsync`` と同様ですが、読み取った値を返す代わりに、 ``data`` 引数として与えられたシグナル/オブジェクトを駆動します。
   * - | mem.readSyncMixedWidth(
       |  address
       |  data
       |  [enable]
       |  [readUnderWrite]
       |  [clockCrossing]
       | )
     - ``mem.readSync`` と同様ですが、読み取った値を返す代わりに、 ``data`` 引数として与えられたシグナル/オブジェクトを駆動します。
   * - | mem.readWriteSyncMixedWidth(
       |  address
       |  data
       |  enable
       |  write
       |  [mask]
       |  [readUnderWrite]
       |  [clockCrossing]
       | )
     - ``mem.readWriteSync`` と同等です。


.. important::
   読み取り-書き込みポリシーの場合、この機能を使用するには、:ref:`自動メモリーブラックボックス化 <automatic_memory_blackboxing>` を有効にする必要があります。
   なぜなら、混合幅 RAM を推論するための普遍的な VHDL/Verilog 言語テンプレートがないからです。
   
.. _automatic_memory_blackboxing:

自動ブラックボックス化
-------------------------

通常の VHDL/Verilog を使用してすべての RAM 種類を推論することは不可能です。
そのため、SpinalHDL にはオプションの自動ブラックボックス化システムが統合されています。
このシステムは、RTL ネットリストに存在するすべてのメモリを調べ、それらをブラックボックスで置き換えます。
その後、生成されたコードは、メモリの機能（読み書き中の読み取りポリシーや混合幅ポートなど）を提供するために、
サードパーティの IP に依存します。

以下は、メモリのブラックボックス化をデフォルトで有効にする方法の例です：

.. code-block:: scala

   def main(args: Array[String]) {
     SpinalConfig()
       .addStandardMemBlackboxing(blackboxAll)
       .generateVhdl(new TopLevel)
   }

標準のブラックボックス化ツールが設計に十分でない場合は、 `Github issue <https://github.com/SpinalHDL/SpinalHDL/issues>`_ を作成することを躊躇しないでください。
また、独自のブラックボックス化ツールを作成する方法もあります。

ブラックボックス化ポリシー
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ブラックボックス化するメモリを選択し、ブラックボックス化が適切でない場合の対処方法を選択するために、複数のポリシーがあります：

.. list-table::
   :header-rows: 1
   :widths: 2 5

   * - 種類
     - 説明
   * - ``blackboxAll``
     - | すべてのメモリをブラックボックス化します。
       | ブラックボックス化できないメモリにエラーを投げる
   * - ``blackboxAllWhatsYouCan``
     - ブラックボックス化可能なすべてのメモリをブラックボックス化します。
   * - ``blackboxRequestedAndUninferable``
     - | ユーザーによって指定されたメモリと推論不可能なメモリ（ミックス幅など）をブラックボックス化します。
       | ブラックボックス化できないメモリに対してはエラーを投げます。
   * - ``blackboxOnlyIfRequested``
     - | ユーザーが指定したメモリをブラックボックス化します。
       | ブラックボックス化できないメモリに関してエラーを発生させます。


メモリを明示的にブラックボックス化するには、その ``generateAsBlackBox`` 関数を使用します。

.. code-block:: scala

   val mem = Mem(Rgb(rgbConfig), 1 << 16)
   mem.generateAsBlackBox()

``MemBlackboxingPolicy`` クラスを拡張して独自のブラックボックス化ポリシーを定義することもできます。

標準のメモリ ブラックボックス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下に、SpinalHDL で使用される標準ブラックボックスの VHDL 定義を示します：

.. code-block:: ada

   -- Simple asynchronous dual port (1 write port, 1 read port)
   component Ram_1w_1ra is
     generic(
       wordCount : integer;
       wordWidth : integer;
       technology : string;
       readUnderWrite : string;
       wrAddressWidth : integer;
       wrDataWidth : integer;
       wrMaskWidth : integer;
       wrMaskEnable : boolean;
       rdAddressWidth : integer;
       rdDataWidth : integer
     );
     port(
       clk : in std_logic;
       wr_en : in std_logic;
       wr_mask : in std_logic_vector;
       wr_addr : in unsigned;
       wr_data : in std_logic_vector;
       rd_addr : in unsigned;
       rd_data : out std_logic_vector
     );
   end component;

   -- 単純な同期デュアルポート（1つの書き込みポート、1つの読み取りポート）
   component Ram_1w_1rs is
     generic(
       wordCount : integer;
       wordWidth : integer;
       clockCrossing : boolean;
       technology : string;
       readUnderWrite : string;
       wrAddressWidth : integer;
       wrDataWidth : integer;
       wrMaskWidth : integer;
       wrMaskEnable : boolean;
       rdAddressWidth : integer;
       rdDataWidth : integer;
       rdEnEnable : boolean
     );
     port(
       wr_clk : in std_logic;
       wr_en : in std_logic;
       wr_mask : in std_logic_vector;
       wr_addr : in unsigned;
       wr_data : in std_logic_vector;
       rd_clk : in std_logic;
       rd_en : in std_logic;
       rd_addr : in unsigned;
       rd_data : out std_logic_vector
     );
   end component;

   -- シングルポート（1つの読み書きポート）
   component Ram_1wrs is
     generic(
       wordCount : integer;
       wordWidth : integer;
       readUnderWrite : string;
       technology : string
     );
     port(
       clk : in std_logic;
       en : in std_logic;
       wr : in std_logic;
       addr : in unsigned;
       wrData : in std_logic_vector;
       rdData : out std_logic_vector
     );
   end component;

   -- 真のデュアルポート（2つの読み書きポート）
   component Ram_2wrs is
     generic(
       wordCount : integer;
       wordWidth : integer;
       clockCrossing : boolean;
       technology : string;
       portA_readUnderWrite : string;
       portA_addressWidth : integer;
       portA_dataWidth : integer;
       portA_maskWidth : integer;
       portA_maskEnable : boolean;
       portB_readUnderWrite : string;
       portB_addressWidth : integer;
       portB_dataWidth : integer;
       portB_maskWidth : integer;
       portB_maskEnable : boolean
     );
     port(
       portA_clk : in std_logic;
       portA_en : in std_logic;
       portA_wr : in std_logic;
       portA_mask : in std_logic_vector;
       portA_addr : in unsigned;
       portA_wrData : in std_logic_vector;
       portA_rdData : out std_logic_vector;
       portB_clk : in std_logic;
       portB_en : in std_logic;
       portB_wr : in std_logic;
       portB_mask : in std_logic_vector;
       portB_addr : in unsigned;
       portB_wrData : in std_logic_vector;
       portB_rdData : out std_logic_vector
     );
   end component;

ブラックボックスには技術パラメータがあります。それを設定するには、対応するメモリ上で ``setTechnology`` 関数を使用できます。
現在、4種類の技術が可能です：

* ``auto``
* ``ramBlock``
* ``distributedLut``
* ``registerFile``

ブラックボックス化により、 ``SpinalConfig#setDevice(Device)`` がデバイスベンダーに設定されている場合、 HDL 属性を挿入できます。

生成される HDL 属性は次のようになります：

.. code-block:: verilog

   (* ram_style = "distributed" *)
   (* ramsyle = "no_rw_check" *)


SpinalHDL は、よく知られたベンダーやデバイスによって提供される多くの一般的なメモリタイプをサポートしようとしますが、
これは常に変化する状況であり、プロジェクトの要件はこの領域で非常に具体的な場合があります。

これが設計フローに重要である場合は、ベンダーのプラットフォームドキュメントを参照しながら、
期待される属性/ジェネリックの挿入を確認するために、出力 HDL を確認してください。

HDL 属性は、 ``addAttribute()`` の ``addAttribute`` メカニズムを使用して手動で追加することもできます。

