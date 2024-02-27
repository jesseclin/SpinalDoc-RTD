
Report
======

シミュレーションのために RTL にデバッグを追加することができます。以下の構文を使用してください：


.. code-block:: scala

    object Enum extends SpinalEnum{
        val MIAOU, RAWRR = newElement()
    }

    class TopLevel extends Component {
        val a = Enum.RAWRR()
        val b = U(0x42)
        val c = out(Enum.RAWRR())
        val d = out (U(0x42))
        report(Seq("miaou ", a, b, c, d))
    }

これにより、次の Verilog コードが生成されます：

.. code-block:: verilog

    $display("NOTE miaou %s%x%s%x", a_string, b, c_string, d);

SpinalHDL 1.4.4 からは、以下の構文もサポートされています：

.. code-block:: scala

    report(L"miaou $a $b $c $d")

REPORT_TIME オブジェクトを使用して現在のシミュレーション時間を表示できます。

.. code-block:: scala

    report(L"miaou $REPORT_TIME")

これにより、次の Verilog コードが生成されます：

.. code-block:: verilog

    $display("NOTE miaou %t", $time);
