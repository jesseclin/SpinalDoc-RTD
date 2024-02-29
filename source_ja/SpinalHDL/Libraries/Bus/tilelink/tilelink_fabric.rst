
tilelink.fabric.Node
===========================

tilelink.fabric.Node は、通常の TileLink ハードウェアインスタンス化の上に追加のレイヤーであり、
SoC レベルでのネゴシエーションとパラメータの伝播を処理します。

これは主に Fiber API に基づいており、エラボレーション時のファイバー（ユーザースペーススレッド）を作成することができ、
将来のパラメータの伝播/ネゴシエーションとハードウェアのエラボレーションをスケジュールすることができます。

ノードは3つの方法で作成できます：

- tilelink.fabric.Node.down()：下方向に接続できるノードを作成します（スレーブ側に向かって），CPU / DMA / ブリッジエージェントで使用されます。
- tilelink.fabric.Node()：中間ノードを作成します
- tilelink.fabric.Node.up()：上方向に接続できるノードを作成します（マスター側に向かって），周辺機器 / メモリ / ブリッジエージェントで使用されます

ノードには次のような属性が主にあります：

- bus：Handle[tilelink.Bus]；バスのハードウェアインスタンス
- m2s.proposed：Handle[tilelink.M2sSupport]；上向き接続によって提案される機能のセット
- m2s.supported：Handle[tilelink.M2sSupport]：下向き接続によってサポートされる機能のセット
- m2s.parameter：Handle[tilelink.M2sParameter]：最終的なバスパラメータ

これらはすべて Handles であることに注意してください。Handle は、SpinalHDL で値をファイバー間で共有する方法です。
ファイバーが Handle を読み取ると、まだ値がない場合、他のファイバーが Handle に値を提供するまでそのファイバーの実行がブロックされます。

また、s2m（名前付きm2sの逆）のような一連の属性もありますが、これはインターコネクトのスレーブ側から開始されたトランザクションのパラメーターを指定します（例：メモリの整合性）。

- 入門 : https://youtu.be/hVi9xOGuuek
- 詳細 : https://peertube.f-si.org/videos/watch/bcf49c84-d21d-4571-a73e-96d7eb89e907

トップレベルの例
-------------------

ここに、単純な架空の SoC トップレベルの例があります：

.. code-block:: scala

      val cpu = new CpuFiber()

      val ram = new RamFiber()
      ram.up at(0x10000, 0x200) of cpu.down // ラムを [0x10000-0x101FF] にマップします。ラムはそのサイズを自分で推定します

      val gpio = new GpioFiber()
      gpio.up at 0x20000 of cpu.down // gpio を[0x20000-0x20FFF]にマップします。4KB の範囲が内部で固定されています

また、以下のように、インターコネクトに中間ノードを定義することもできます：

.. code-block:: scala

      val cpu = new CpuFiber()

      val ram = new RamFiber()
      ram.up at(0x10000, 0x200) of cpu.down
        
      // ネームスペースを作成してきれいに保ちます
      val peripherals = new Area{
        // インターコネクトに中間ノードを作成します
        val access = tilelink.fabric.Node()
        access at 0x20000 of cpu.down

        val gpioA = new GpioFiber()
        gpioA.up at 0x0000 of access

        val gpioB = new GpioFiber()
        gpioB.up at 0x1000 of access
      }


Example GpioFiber
----------------------

GpioFiber は、32 ビットのトライステート配列を読み取り/ドライブできるシンプルな TileLink ペリフェラルです。

.. code-block:: scala

    import spinal.lib._
    import spinal.lib.bus.tilelink
    import spinal.core.fiber.Fiber
    class GpioFiber extends Area {
      // 上方向（マスター側のみ）に向かうノードを定義します
      val up = tilelink.fabric.Node.up()

      // "up"パラメータを指定し、ハードウェアを生成するためのエラボレーションスレッドを定義します
      val fiber = Fiber build new Area {
        // まず、up ノードがサポートする内容を定義します。m2 sはマスターからスレーブへのリクエストを意味します
        up.m2s.supported load tilelink.M2sSupport(
          addressWidth = 12,
          dataWidth = 32,
          // 転送は、up ノードがサポートするメモリトランザクションの種類を定義します。
          // ここでは、4バイトの get/putfull のみをサポートしています
          transfers = tilelink.M2sTransfers(
            get = tilelink.SizeRange(4),
            putFull = tilelink.SizeRange(4)
          )
        )
        // s2m はスレーブからマスターへのリクエストであり、これらはメモリ整合性の目的でのみ使用されます
        // したがって、ここでは必要なしと指定します
        up.s2m.none()

        // 最後にハードウェアを生成できます
        // 最初に32ビットのTriStateArrayを定義します（Arrayは各ピンに独自のwriteEnableビットを持つことを意味します）
        val pins = master(TriStateArray(32 bits)) 
        
        // tilelink.SlaveFactory は、タイルリンクバスからハードウェアを制御するために必要なロジックを簡単に生成するユーティリティです。
        val factory = new tilelink.SlaveFactory(up.bus, allowBurst = false)
        
        // SlaveFactory API を使用して、ピンを読み取る/ドライブするためのハードウェアを生成します
        val writeEnableReg = factory.drive(pins.writeEnable, 0x0) init (0)
        val writeReg = factory.drive(pins.write, 0x4) init(0)
        factory.read(pins.read, 0x8)
      }
    }

Example RamFiber
----------------------

RamFiber は、通常の TileLink Ram コンポーネントの統合レイヤーです。

.. code-block:: scala

    import spinal.lib.bus.tilelink
    import spinal.core.fiber.Fiber
    class RamFiber() extends Area {
      val up = tilelink.fabric.Node.up()

      val thread = Fiber build new Area {
        // ここでサポートされるパラメータは、マスターが理想的にサポートする内容に依存します。
        // tilelink.Ram はすべての addressWidth / dataWidth / バースト長 / get / put アクセスをサポートしています
        // ただし、アトミック/整合性はサポートしていません。したがって、使用されるものを取り、すべての get / put リクエストに制限します
        up.m2s.supported load up.m2s.proposed.intersect(M2sTransfers.allGetPut)
        up.s2m.none()

        // ここでは、接続されたマスターのメモリマッピングを見て、ラムがどれだけのバイトを必要とするかを推論します
        val bytes = up.ups.map(e => e.mapping.value.highestBound - e.mapping.value.lowerBound + 1).max.toInt
        
        // 最後に通常のハードウェアを生成します
        val logic = new tilelink.Ram(up.bus.p.node, bytes)
        logic.io.up << up.bus
      }
    }

Example CpuFiber
----------------------

CpuFiber は、マスターの統合の架空の例です。

.. code-block:: scala

    import spinal.lib.bus.tilelink
    import spinal.core.fiber.Fiber

    class CpuFiber extends Area {
      // 下方向（スレーブ側のみ）に向かうノードを定義します
      val down = tilelink.fabric.Node.down()

      val fiber = Fiber build new Area {
        // ここで、バスパラメータを特定の構成に強制します
        down.m2s forceParameters tilelink.M2sParameters(
          addressWidth = 32,
          dataWidth = 64,
          // このノードを使用して各マスターのトラフィックを定義します。（1つのマスター=> 1つの M2sAgent）
          // この場合、CpuFiber のみです。
          masters = List(
            tilelink.M2sAgent(
              name = CpuFiber.this, // オリジナルのエージェントへの参照。
              // エージェントは、異なる目的のために複数のソース ID のセットを使用できます
              // ここでは、すべてのセットのソース ID の使用方法を定義します
              // この場合、get/putFull リクエストの送信に ID [0-3]を使用するとします
              mapping = List(
                tilelink.M2sSource(
                  id = SizeMapping(0, 4),
                  emits = M2sTransfers(
                    get = tilelink.SizeRange(1, 64), //get アクセスは [1, 64] の任意の 2の累乗サイズであることを意味します
                    putFull = tilelink.SizeRange(1, 64)
                  )
                )
              )
            )
          )
        )

        // CPU はスレーブ発行リクエスト（メモリ整合性）をサポートしていないとします
        down.s2m.supported load tilelink.S2mSupport.none()

        // それからいくつかのハードウェアを生成できます（この例では何も役に立ちません）
        down.bus.a.setIdle()
        down.bus.d.ready := True
      }
    }

Tilelink の特異性の 1つは、マスターがマッピングされていないメモリ空間にリクエストを送信しないと仮定していることです。
マスターがどのメモリアクセスを行うことが許可されているかを特定するために、spinal.lib.system.tag.MemoryConnection.getMemoryTransfers ツールを次のように使用できます：

.. code-block:: scala

        val mappings = spinal.lib.system.tag.MemoryConnection.getMemoryTransfers(down)
        // ここでは、単にstdout に値を出力していますが、代わりにそれからハードウェアを生成することができます。
        for(mapping <- mappings){
          println(s"- ${mapping.where} -> ${mapping.transfers}")
        }

この Cpu のファイバーで、次の soc で実行すると：

.. code-block:: scala

      val cpu = new CpuFiber()

      val ram = new RamFiber()
      ram.up at(0x10000, 0x200) of cpu.down
        
      // クリーンに保つためのペリフェラル名前空間を作成します
      val peripherals = new Area{
        // インターコネクトに中間ノードを作成します
        val access = tilelink.fabric.Node()
        access at 0x20000 of cpu.down

        val gpioA = new GpioFiber()
        gpioA.up at 0x0000 of access

        val gpioB = new GpioFiber()
        gpioB.up at 0x1000 of access
      }

次のようになります：

.. code-block:: 

    - toplevel/ram_up mapped=SM(0x10000, 0x200) through=List(OT(0x10000))  -> GF
    - toplevel/peripherals_gpioA_up mapped=SM(0x20000, 0x1000) through=List(OT(0x20000), OT(0x0))  -> GF
    - toplevel/peripherals_gpioB_up mapped=SM(0x21000, 0x1000) through=List(OT(0x20000), OT(0x1000))  -> GF

- "through=" は、ターゲットに到達するために行われたアドレス変換のチェーンを指定します。
- "SM" は、SizeMapping(address, size) を意味します。
- "OT" は、OffsetTransformer(offset) を意味します。

また、ノードに PMA (Physical Memory Attributes) を追加し、この getMemoryTransfers ユーティリティを介してそれらを取得することもできます。

現在定義されている PMA は次のとおりです：

.. code-block:: 

  object MAIN          extends PMA
  object IO            extends PMA
  object CACHABLE      extends PMA // 中間エージェントがリージョンのキャッシュコピーを持っている場合があります
  object TRACEABLE     extends PMA // リージョンが別のマスターによってキャッシュされているかもしれませんが、整合性が提供されています
  object UNCACHABLE    extends PMA // リージョンはまだキャッシュされていませんが、可能な限りキャッシュする必要があります
  object IDEMPOTENT    extends PMA // 読み取りは最後に入れたコンテンツを返しますが、コンテンツをキャッシュしてはいけません
  object EXECUTABLE    extends PMA // エージェントがこのリージョンからコードを取得できるようにします
  object VOLATILE      extends PMA // コンテンツは書き込みなしに変更される可能性があります
  object WRITE_EFFECTS extends PMA // 書き込みに副作用があり、したがって組み合わせたり遅延させたりしてはいけません
  object READ_EFFECTS  extends PMA // 読み取りに副作用があり、したがって仮定的に発行してはいけません

getMemoryTransfers ユーティリティは、専用の SpinalTag に依存しています：

.. code-block:: 

    trait MemoryConnection extends SpinalTag {
      def up : Nameable with SpinalTagReady // システムのマスター側に向かう側
      def down : Nameable with SpinalTagReady // システムのスレーブ側に向かう側
      def mapping : AddressMapping // マスターアドレスからスレーブのメモリマッピングを指定します（トランスフォーマーが適用される前）
      def transformers : List[AddressTransformer]  // この接続で行われた変更のリスト（オフセット、インタリーブなど）
      def sToM(downs : MemoryTransfers, args : MappedNode) : MemoryTransfers = downs // スレーブ MemoryTransfers の機能をマスターのものに変換します
    }

この SpinalTag は、与えられたメモリバス接続の両端に適用でき、エラボレーション時にこの接続を検出可能にします。
これに関して良い点の 1つは、バスに依存しないことです。つまり、TileLink 固有のものではありません。


Example WidthAdapter
---------------------

幅適応器はブリッジのシンプルな例です。

.. code-block:: 

    class WidthAdapterFiber() extends Area{
      val up = Node.up()
      val down = Node.down()

      // MemoryConnection グラフを構築します
      new MemoryConnection {
        override def up = up
        override def down = down
        override def transformers = Nil
        override def mapping = SizeMapping(0, BigInt(1) << WidthAdapterFiber.this.up.m2s.parameters.addressWidth)
        populate()
      }

      // データ幅パラメータを交渉し、ハードウェアを生成するファイバー
      val logic = Fiber build new Area{
        // まず、パラメータ提案を下方向に伝播させ、下方向が同意することを期待します
        down.m2s.proposed.load(up.m2s.proposed)

        // 次に、実際にサポートされているものを上方向に伝播させますが、データ幅の不一致に注意します
        up.m2s.supported load down.m2s.supported.copy(
          dataWidth = up.m2s.proposed.dataWidth
        )

        // 次に、最終的なバスパラメータを下方向に伝播させますが、データ幅の不一致に注意します
        down.m2s.parameters load up.m2s.parameters.copy(
          dataWidth = down.m2s.supported.dataWidth
        )

        // s2m パラメータに変更はありません
        up.s2m.from(down.s2m)

        // 最後に、ハードウェアを生成します
        val bridge = new tilelink.WidthAdapter(up.bus.p, down.bus.p)
        bridge.io.up << up.bus
        bridge.io.down >> down.bus
      }
    }


