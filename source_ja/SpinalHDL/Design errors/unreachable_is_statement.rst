
Unreachable is statement
========================

概要
------------

SpinalHDL は、 ``switch`` 文の中のすべての ``is`` ステートメントが到達可能であることを確認します。
到達可能とは、実行フロー上、そのステートメントに到達する可能性があることを意味します。

例
-------

以下のコード:


.. code-block:: scala

   class TopLevel extends Component {
     val sel = UInt(2 bits)
     val result = UInt(4 bits)
     switch(sel) {
       is(0){ result := 4 }
       is(1){ result := 6 }
       is(2){ result := 8 }
       is(3){ result := 9 }
       is(0){ result := 2 } // 重複した is ステートメント!
     }
   }

は、以下のようなエラーを出力します:

.. code-block:: text

   UNREACHABLE IS STATEMENT in the switch statement at
     ***
     Source file location of the is statement definition via the stack trace
     ***

修正方法:

.. code-block:: scala

   class TopLevel extends Component {
     val sel = UInt(2 bits)
     val result = UInt(4 bits)
     switch(sel) {
       is(0){ result := 4 }
       is(1){ result := 6 }
       is(2){ result := 8 }
       is(3){ result := 9 }
     }
   }
