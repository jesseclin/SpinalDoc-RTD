
Hierarchy violation
===================

はじめに
------------

SpinalHDL は、信号が現在のコンポーネントの範囲外でアクセスされないようにチェックします。

次の信号は、コンポーネント内で読み取ることができます：

* 現在のコンポーネントで定義されたすべての方向のない信号
* 現在のコンポーネントのすべての入力/出力/双方向信号
* 子コンポーネントのすべての入力/出力/双方向信号

さらに、以下の信号はコンポーネント内で代入することができます：

* 現在のコンポーネントで定義されたすべての方向のない信号
* 現在のコンポーネントのすべての出力/双方向信号
* 子コンポーネントのすべての入力/双方向信号

もし ``HIERARCHY VIOLATION`` エラーが表示された場合、上記のルールのいずれかが違反されたことを意味します。

例
-------

以下のコード：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       // これは、現在のコンポーネント 'Toplevel' の 'in' シグナルです。
       val a = in UInt(8 bits)
     }
     val tmp = U"x42"
     io.a := tmp  // ERROR: 現在のコンポーネントの入力に割り当てを試みています。
   }

は、次のようにエラーを発生させます：

.. code-block:: text

   HIERARCHY VIOLATION : (toplevel/io_a : in UInt[8 bits]) is driven by (toplevel/tmp :  UInt[8 bits]), but isn't accessible in the toplevel component.
     ***
     Source file location of the `io.a := tmp` via the stack trace
     ***

修正方法は次の通りです：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a = out UInt(8 bits) // "in" から "out" に変更されました。
     }
     val tmp = U"x42"
     io.a := tmp  // 今、出力に割り当てています。
   }
