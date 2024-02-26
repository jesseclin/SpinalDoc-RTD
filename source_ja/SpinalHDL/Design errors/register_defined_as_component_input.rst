
Register defined as component input
===================================

概要
------------

SpinalHDL では、レジスタを入力として持つコンポーネントを定義することは許されていません。
これは、ユーザーが登録されたシグナルで子コンポーネントの入力をドライブしようとしたときに予期せぬ動作を防ぐためのものです。
登録された入力が必要な場合は、 ``io`` バンドルで非登録の入力を宣言し、コンポーネントの本体でシグナルを登録する必要があります。

例
-------

次のコード：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a = in(Reg(UInt(8 bits)))
     }
   }

次のエラーが発生します：

.. code-block:: text

   REGISTER DEFINED AS COMPONENT INPUT : (toplevel/io_a : in UInt[8 bits]) is defined as a registered input of the toplevel component, but isn't allowed.
     ***
     Source file location of the toplevel/io_a definition via the stack trace
     ***

修正方法は以下の通りです：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a = in UInt(8 bits)
     }
   }

登録された ``a`` が必要な場合は、次のように行います：

.. code-block:: scala

   class TopLevel extends Component {
     val io = new Bundle {
       val a = in UInt(8 bits)
     }
     val a = RegNext(io.a)
   }
