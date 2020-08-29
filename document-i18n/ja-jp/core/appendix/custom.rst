.. index:: custom section, section, binary format

カスタムセクション
---------------

この付録では、WebAssemblyの :ref:`バイナリ形式 <binary>` で用いる専用の :ref:`カスタムセクション <binary-customsec>` を定義します。
そのようなセクションはWebAssemblyのセマンティクスに貢献することもない代わりに悪影響を及ぼすこともなく、他のカスタムセクションと同様、実装で無視される可能性があります。
ただし、カスタムセクションは実装で利用できる有用なメタデータを提供して、ユーザーエクスペリエンスを向上させたりコンパイルヒントを受け取ったりできます。

現時点で定義されている専用カスタムセクションは :ref:`名前セクション<binary-namesec>` ひとつだけです。

.. index:: ! name section, name, Unicode UTF-8
.. _binary-namesec:

名前セクション（name section）
~~~~~~~~~~~~

「名前セクション」は、その名前の文字列自身が :math:`\text{name}` である :ref:`カスタムセクション <binary-customsec>` です。
名前セクションの出現は、ひとつのモジュールあたり1回までとし、かつ :ref:`データセクション <binary-datasec>` より後ろに限るべきです。

このセクションの目的は、モジュール内で「印刷可能な名前」を定義にアタッチすることです。これは、たとえばデバッガで用いたり、モジュールの一部を :ref:`テキスト形式 <text>` でレンダリングする場合に利用できます。

.. note::
   すべての :ref:`名前 <binary-name>` は UTF-8 でエンコードされた  |Unicode|_ で表現されます。
   名前は一意である必要はありません。


.. _binary-namesubsection:

サブセクション（subsection）
...........

名前セクションの :ref:`データ <binary-customsec>` は「サブセクション」のシーケンスで構成されます。
個別のサブセクションは以下で構成されます。

* 1バイトのサブセクション「id」
* コンテンツの |U32| サイズ（バイトで表現）
* 実際の「コンテンツ」（その構造はサブセクションidに依存する）

.. math::
   \begin{array}{llcll}
   \production{name section} & \Bnamesec &::=&
     \Bsection_0(\Bnamedata) \\
   \production{name data} & \Bnamedata &::=&
     n{:}\Bname & (\iff n = \text{name}) \\ &&&
     \Bmodulenamesubsec^? \\ &&&
     \Bfuncnamesubsec^? \\ &&&
     \Blocalnamesubsec^? \\
   \production{name subsection} & \Bnamesubsection_N(\B{B}) &::=&
     N{:}\Bbyte~~\X{size}{:}\Bu32~~\B{B}
       & (\iff \X{size} = ||\B{B}||) \\
   \end{array}

以下のサブセクションidが用いられます。

==  ===========================================
id  サブセクション
==  ===========================================
 0  :ref:`モジュール名 <binary-modulenamesec>`
 1  :ref:`関数名 <binary-funcnamesec>`
 2  :ref:`ローカル名 <binary-localnamesec>`
==  ===========================================

個別のサブセクションの出現は1回までで、かつidの昇順とします。

.. index:: ! name map, index, index space
.. _binary-indirectnamemap:
.. _binary-namemap:

名前マップ（name map）
.........

ひとつの「名前マップ」は、与えられた :ref:`インデックス空間 <syntax-index>` 内の :ref:`インデックス <syntax-index>` に :ref:`名前 <syntax-name>` を割り当てます。
名前マップは、「インデックス」「名前」ペアのひとつの :ref:`ベクタ <binary-vec>` （インデックス値は昇順）で構成されます。
個別のインデックスは一意でなければなりませんが、割り当てられた名前は必ずしも一意である必要はありません。

.. math::
   \begin{array}{llclll}
   \production{name map} & \Bnamemap &::=&
     \Bvec(\Bnameassoc) \\
   \production{name association} & \Bnameassoc &::=&
     \Bidx~\Bname \\
   \end{array}

「間接名前マップ（indirect name map）」は、:ref:`名前 <syntax-name>` を2次元の :ref:`インデックス空間 <syntax-index>` に割り当てます。第2インデックスは第1インデックスによって「グループ化」されます。
間接名前マップは、第1の「インデックス」「名前」ペアをインデックス値の昇順に並べたひとつのベクタで構成され、個別の名前マップはそれによって第2インデックスを名前に対応付けます。
個別の第1インデックスは一意でなければならず、個別の名前マップに対する個別の第2インデックスも同様に一意でなければなりません。

.. math::
   \begin{array}{llclll}
   \production{indirect name map} & \Bindirectnamemap &::=&
     \Bvec(\Bindirectnameassoc) \\
   \production{indirect name association} & \Bindirectnameassoc &::=&
     \Bidx~\Bnamemap \\
   \end{array}


.. index:: module
.. _binary-modulenamesec:

モジュール名（module name）
............

「モジュール名サブセクション」のidは0です。
これは、モジュール自身に割り当てられる単一の :ref:`名前 <binary-name>` でシンプルに構成されます。

.. math::
   \begin{array}{llclll}
   \production{module name subsection} & \Bmodulenamesubsec &::=&
     \Bnamesubsection_0(\Bname) \\
   \end{array}


.. index:: function, function index
.. _binary-funcnamesec:

関数名（function name）
..............

「関数名サブセクション」のidは1です。
これは、関数名を :ref:`関数インデックス <syntax-funcidx>` に割り当てるひとつの :ref:`名前マップ <binary-namemap>` で構成されます。

.. math::
   \begin{array}{llclll}
   \production{function name subsection} & \Bfuncnamesubsec &::=&
     \Bnamesubsection_1(\Bnamemap) \\
   \end{array}


.. index:: function, local, function index, local index
.. _binary-localnamesec:

ローカル名（local name）
...........

「ローカル名サブセクション」のidは2です。
これは、:ref:`関数インデックス <syntax-funcidx>` でグループ化された :ref:`ローカルインデックス <syntax-localidx>` にローカル名を割り当てるひとつの :ref:`間接名前マップ <binary-indirectnamemap>` で構成されます。

.. math::
   \begin{array}{llclll}
   \production{local name subsection} & \Blocalnamesubsec &::=&
     \Bnamesubsection_2(\Bindirectnamemap) \\
   \end{array}
