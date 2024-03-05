
VexRiscv (RV32IM CPU)
=====================

VexRiscv は、以下の機能を備えた FPGA フレンドリーな RISC-V ISA CPU 実装です：

* RV32IM 命令セット
* 5段階のパイプライン（フェッチ、デコード、実行、メモリ、ライトバック）
* すべての機能が有効な場合、1.44 DMIPS/MHz
* FPGA 向けに最適化されています
* オプションの MUL/DIV 拡張
* オプションの命令キャッシュとデータキャッシュ
* オプションの MMU
* Eclipse デバッグを介したオプションのデバッグ拡張（GDB >> openOCD >> JTAG 接続）
* riscv-privileged-v1.9.1 仕様のマシンモードとユーザーモードによるオプションの割り込みと例外処理
* シフト命令の2つの実装、シングルサイクル／ shiftNumber サイクル
* 各ステージにバイパスまたはインターロックハザードロジックを持つことができます
* FreeRTOSポート: https://github.com/Dolu1990/FreeRTOS-RISCV

詳細はこちらで入手できます： `https://github.com/SpinalHDL/VexRiscv <https://github.com/SpinalHDL/VexRiscv>`_
