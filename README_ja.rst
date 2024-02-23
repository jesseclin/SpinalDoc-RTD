=========
SpinalDoc
=========

これは、
`SpinalHDL <https://github.com/SpinalHDL/SpinalHDL>`_
のドキュメント リポジトリです。

`spinalhdl.github.io/SpinalDoc-RTD <https://spinalhdl.github.io/SpinalDoc-RTD/master/index.html>`_
で公開されています。

中国語版も次のリンクで
`spinalhdl-cn.github.io/SpinalDoc-RTD <https://spinalhdl-cn.github.io/SpinalDoc-RTD/zh_CN/index.html>`_.
あります。この継続的なローカリゼーション バージョンは、`Weblate <https://hosted.weblate.org/projects/spinaldoc-rtd/>`_ の助けを借りて維持されています。

API ドキュメントは次の場所にもあります。
`spinalhdl.github.io/SpinalHDL <https://spinalhdl.github.io/SpinalHDL/dev/spinal/index.html>`_.


このドキュメントの作成方法
===============================

With venv
---------

要件（システム）:

* make
* git

Pipenv を使用して仮想環境を作成します (必要なパッケージのインストールに Pipfile を使用します)。

.. code:: shell

   python3 -m venv .venv

その後、仮想環境を (bash で) アクティブ化し、依存関係をインストールできます。

.. code:: shell

   source .venv/bin/activate
   pip install -r requirements.txt

そして、通常の方法で「make」を使用できます

.. code:: shell

   make html     # for html
   make latex    # for latex
   make latexpdf # for latex (will require latexpdf installed)
   make          # list all the available output format

すべての出力は docs フォルダーにあります (html の場合: docs/html)


With Docker
-----------

要件（システム）:

* docker
* git

カスタム Docker イメージを作成するには (Python とその依存関係を使用):

.. code:: shell

   docker build -t spinaldoc-rtd .

次にドキュメントを構築します:

.. code:: shell

   docker run -it --rm -v $PWD:/docs spinaldoc-rtd


Docker でカスタム コマンドを実行して、たとえば以下をクリーンアップすることもできます:

.. code:: shell

   docker run -it --rm -v $PWD:/docs spinaldoc-rtd make clean

カスタム Docker イメージを作成して PDF (重い) を構築することもできます:

.. code:: shell

   docker build -f pdf.Dockerfile -t spinaldoc-pdf .

そしてそれを実行するには:

.. code:: shell

   docker run -it --rm -v $PWD:/docs spinaldoc-pdf


ネイティブ
------

要件（システム）:

* make
* git

要件（Python 3）:

* sphinx
* sphinx-rtd-theme
* sphinxcontrib-wavedrom
* sphinx-multiversion

要件をインストールした後、実行できます

.. code:: shell

   make html     # for html
   make latex    # for latex
   make latexpdf # for latex (will require latexpdf installed)
   make          # list all the available output format

ビルド複数バージョンのドキュメントを作成するには、

.. code:: shell

   sphinx-multiversion source docs/html

docs/html には、ブランチとタグごとにビルドされたドキュメントを含むフォルダーが存在します。