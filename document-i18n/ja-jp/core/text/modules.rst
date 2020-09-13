モジュール
-------


.. index:: index, type index, function index, table index, memory index, global index, local index, label index
   pair: text format; type index
   pair: text format; function index
   pair: text format; table index
   pair: text format; memory index
   pair: text format; global index
   pair: text format; local index
   pair: text format; label index
.. _text-typeidx:
.. _text-funcidx:
.. _text-tableidx:
.. _text-memidx:
.. _text-globalidx:
.. _text-localidx:
.. _text-labelidx:
.. _text-index:

インデックス（index）
~~~~~~~

:ref:`インデックス <syntax-index>` は、対応する構成（construct）に束縛されるときに「生の数値形式」か「シンボリックな :ref:`識別子 <text-id>`」のいずれかの形で与えられます。
こうした識別子は、:ref:`識別子コンテキスト <text-context>` :math:`I` の適切な空間内で探索されます。

.. math::
   \begin{array}{llcllllllll}
   \production{type index} & \Ttypeidx_I &::=&
     x{:}\Tu32 &\Rightarrow& x \\&&|&
     v{:}\Tid &\Rightarrow& x & (\iff I.\ITYPES[x] = v) \\
   \production{function index} & \Tfuncidx_I &::=&
     x{:}\Tu32 &\Rightarrow& x \\&&|&
     v{:}\Tid &\Rightarrow& x & (\iff I.\IFUNCS[x] = v) \\
   \production{table index} & \Ttableidx_I &::=&
     x{:}\Tu32 &\Rightarrow& x \\&&|&
     v{:}\Tid &\Rightarrow& x & (\iff I.\ITABLES[x] = v) \\
   \production{memory index} & \Tmemidx_I &::=&
     x{:}\Tu32 &\Rightarrow& x \\&&|&
     v{:}\Tid &\Rightarrow& x & (\iff I.\IMEMS[x] = v) \\
   \production{global index} & \Tglobalidx_I &::=&
     x{:}\Tu32 &\Rightarrow& x \\&&|&
     v{:}\Tid &\Rightarrow& x & (\iff I.\IGLOBALS[x] = v) \\
   \production{local index} & \Tlocalidx_I &::=&
     x{:}\Tu32 &\Rightarrow& x \\&&|&
     v{:}\Tid &\Rightarrow& x & (\iff I.\ILOCALS[x] = v) \\
   \production{label index} & \Tlabelidx_I &::=&
     l{:}\Tu32 &\Rightarrow& l \\&&|&
     v{:}\Tid &\Rightarrow& l & (\iff I.\ILABELS[l] = v) \\
   \end{array}


.. index:: type definition, identifier
   pair: text format; type definition
.. _text-typedef:

型（type）
~~~~~

型定義は、1個のシンボリックな :ref:`型識別子 <text-id>` を束縛できます。

.. math::
   \begin{array}{llclll}
   \production{type definition} & \Ttype &::=&
     \text{(}~\text{type}~~\Tid^?~~\X{ft}{:}\Tfunctype~\text{)}
       &\Rightarrow& \X{ft} \\
   \end{array}


.. index:: type use
   pair: text format; type use
.. _text-typeuse:

型利用（type use）
~~~~~~~~~

1個の「型利用」は、1個の :ref:`型定義 <text-type>` への1個の参照です。
型利用はオプションで、インライン化された :ref:`パラメーター <text-param>` 宣言や :ref:`結果 <text-result>` 宣言を明示的に引数として取れます。
これによって、パラメーターの :ref:`ローカルインデックス <text-localidx>` に名前付けするシンボリックな :ref:`識別子 <text-id>` を束縛できるようになります。
インライン宣言が与えられる場合、それらの型と、参照される :ref:`関数型 <text-type>` は一致しなければなりません。

.. math::
   \begin{array}{llcllll}
   \production{type use} & \Ttypeuse_I &::=&
     \text{(}~\text{type}~~x{:}\Ttypeidx_I~\text{)}
       \quad\Rightarrow\quad x, I' \\ &&& \qquad
       (\iff \begin{array}[t]{@{}l@{}}
        I.\ITYPEDEFS[x] = [t_1^n] \to [t_2^\ast] \wedge
        I' = \{\ILOCALS~(\epsilon)^n\}) \\
        \end{array} \\[1ex] &&|&
     \text{(}~\text{type}~~x{:}\Ttypeidx_I~\text{)}
     ~~(t_1{:}\Tparam)^\ast~~(t_2{:}\Tresult)^\ast
       \quad\Rightarrow\quad x, I' \\ &&& \qquad
       (\iff \begin{array}[t]{@{}l@{}}
        I.\ITYPEDEFS[x] = [t_1^\ast] \to [t_2^\ast] \wedge
        I' = \{\ILOCALS~\F{id}(\Tparam)^\ast\} \idcwellformed) \\
        \end{array} \\
   \end{array}

ある |Ttypeuse| の合成属性（synthesized attribute）は、「使われている :ref:`型インデックス <syntax-typeidx>`」と「1個以上の可能なパラメーター識別子を含む、更新された :ref:`識別子コンテキスト <text-context>`」の2つからなります。
以下の補助関数は、パラメーターからオプションの識別子を抽出します。

.. math::
   \begin{array}{lcl@{\qquad\qquad}l}
   \F{id}(\text{(}~\text{param}~\Tid^?~\dots~\text{)}) &=& \Tid^? \\
   \end{array}

.. note::
   関数型が :math:`[] \to []` となる場合、2つの生成物はオーバーラップします。
   しかしこの場合、生じる結果が同じなので、どちらを選ぶかは重要ではありません。

   :math:`I'` の :ref:`整形式性 <text-context-wf>` 条件によって、そのパラメーターに含まれる1つ以上の識別子が重複しないようにします。

.. _text-typeuse-abbrev:

短縮形
.............

1個の |Ttypeuse| は、インラインの :ref:`パラメーター <text-param>` 宣言や :ref:`結果 <text-result>` 宣言で完全に置き換えることもできます。
この場合、:ref:`型インデックス <syntax-typeidx>` がひとつ自動挿入されます。

.. math::
   \begin{array}{llclll}
   \production{type use} &
     (t_1{:}\Tparam)^\ast~~(t_2{:}\Tresult)^\ast &\equiv&
     \text{(}~\text{type}~~x~\text{)}~~\Tparam^\ast~~\Tresult^\ast \\
   \end{array}

上の :math:`x` は、存在する最小の :ref:`型インデックス <syntax-typeidx>` を表し、現在のモジュール内にあるその定義は :ref:`function type <syntax-functype>` :math:`[t_1^\ast] \to [t_2^\ast]` です。
そのようなインデックスが存在しない場合、以下の形式で新しい :ref:`型定義 <text-type>` がモジュールの末尾に挿入されます。

.. math::
   \text{(}~\text{type}~~\text{(}~\text{func}~~\Tparam^\ast~~\Tresult^\ast~\text{)}~\text{)}

短縮形は出現順に拡張形に展開され、以前挿入された型定義を以後の拡張形で再利用します。

.. index:: import, name, function type, table type, memory type, global type
   pair: text format; import
.. _text-importdesc:
.. _text-import:

インポート（import）
~~~~~~~

インポート内のデスクリプタは、シンボリックな「関数」「テーブル」「メモリー」「グローバル」 :ref:`識別子 <text-id>` を束縛できます。

.. math::
   \begin{array}{llclll}
   \production{import} & \Timport_I &::=&
     \text{(}~\text{import}~~\X{mod}{:}\Tname~~\X{nm}{:}\Tname~~d{:}\Timportdesc_I~\text{)} \\ &&& \qquad
       \Rightarrow\quad \{ \IMODULE~\X{mod}, \INAME~\X{nm}, \IDESC~d \} \\[1ex]
   \production{import description} & \Timportdesc_I &::=&
     \text{(}~\text{func}~~\Tid^?~~x,I'{:}\Ttypeuse_I~\text{)}
       &\Rightarrow& \IDFUNC~x \\ &&|&
     \text{(}~\text{table}~~\Tid^?~~\X{tt}{:}\Ttabletype~\text{)}
       &\Rightarrow& \IDTABLE~\X{tt} \\ &&|&
     \text{(}~\text{memory}~~\Tid^?~~\X{mt}{:}\Tmemtype~\text{)}
       &\Rightarrow& \IDMEM~~\X{mt} \\ &&|&
     \text{(}~\text{global}~~\Tid^?~~\X{gt}{:}\Tglobaltype~\text{)}
       &\Rightarrow& \IDGLOBAL~\X{gt} \\
   \end{array}


短縮形
.............

短縮形のインポートでは、「:ref:`関数 <text-func>`」「:ref:`テーブル <text-table>`」「:ref:`メモリー <text-mem>`」「:ref:`グローバル <text-global>`」の定義をインラインで指定することもできます。詳しくは該当セクションを参照。

.. index:: function, type index, function type, identifier, local
   pair: text format; function
   pair: text format; local
.. _text-local:
.. _text-func:

関数（function）
~~~~~~~~~

関数定義は、シンボリックな「:ref:`関数識別子 <text-id>`」や、関数の :ref:`パラメーター <text-typeuse>` やローカルで用いる「:ref:`ローカル識別子 <text-id>`」を束縛できます。

.. math::
   \begin{array}{llclll}
   \production{function} & \Tfunc_I &::=&
     \text{(}~\text{func}~~\Tid^?~~x,I'{:}\Ttypeuse_I~~
     (t{:}\Tlocal)^\ast~~(\X{in}{:}\Tinstr_{I''})^\ast~\text{)} \\ &&& \qquad
       \Rightarrow\quad \{ \FTYPE~x, \FLOCALS~t^\ast, \FBODY~\X{in}^\ast~\END \} \\ &&& \qquad\qquad\qquad
       (\iff I'' = I' \compose \{\ILOCALS~\F{id}(\Tlocal)^\ast\} \idcwellformed) \\[1ex]
   \production{local} & \Tlocal &::=&
     \text{(}~\text{local}~~\Tid^?~~t{:}\Tvaltype~\text{)}
       \quad\Rightarrow\quad t \\
   \end{array}

ローカル :ref:`識別子コンテキスト <text-context>` :math:`I''` の定義では、以下の補助関数を用いてローカルからオプションの識別子を抽出します。

.. math::
   \begin{array}{lcl@{\qquad\qquad}l}
   \F{id}(\text{(}~\text{local}~\Tid^?~\dots~\text{)}) &=& \Tid^? \\
   \end{array}


.. note::
   :math:`I''` の :ref:`整形式性 <text-context-wf>` 条件によって、そのパラメーターに含まれる1つ以上の識別子が重複しないようにします。

.. index:: import, name
   pair: text format; import
.. index:: export, name, index, function index
   pair: text format; export
.. _text-func-abbrev:

短縮形
.............

複数の無名ローカルを単一の宣言に結合できます。

.. math::
   \begin{array}{llclll}
   \production{local} &
     \text{(}~~\text{local}~~\Tvaltype^\ast~~\text{)} &\equiv&
     (\text{(}~~\text{local}~~\Tvaltype~~\text{)})^\ast \\
   \end{array}

関数は、:ref:`インポート <text-import>` や :ref:`エクスポート <text-export>` のインラインとして定義できます。

.. math::
   \begin{array}{llclll}
   \production{module field} &
     \text{(}~\text{func}~~\Tid^?~~\text{(}~\text{import}~~\Tname_1~~\Tname_2~\text{)}~~\Ttypeuse~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{import}~~\Tname_1~~\Tname_2~~\text{(}~\text{func}~~\Tid^?~~\Ttypeuse~\text{)}~\text{)} \\[1ex] &
     \text{(}~\text{func}~~\Tid^?~~\text{(}~\text{export}~~\Tname~\text{)}~~\dots~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{export}~~\Tname~~\text{(}~\text{func}~~\Tid'~\text{)}~\text{)}~~
       \text{(}~\text{func}~~\Tid'~~\dots~\text{)}
       \\ & \qquad\qquad
       (\iff \Tid' = \Tid^? \neq \epsilon \vee \Tid' \idfresh) \\
   \end{array}

後者の短縮形は、別のインポートやエクスポートを含む「:math:`\dots`」を用いて繰り返し適用可能です。

.. index:: table, table type, identifier
   pair: text format; table
.. _text-table:

テーブル（table）
~~~~~~

テーブル定義は、1個のシンボリックな :ref:`テーブル識別子 <text-id>` を束縛できます。

.. math::
   \begin{array}{llclll}
   \production{table} & \Ttable_I &::=&
     \text{(}~\text{table}~~\Tid^?~~\X{tt}{:}\Ttabletype~\text{)}
       &\Rightarrow& \{ \TTYPE~\X{tt} \} \\
   \end{array}


.. index:: import, name
   pair: text format; import
.. index:: export, name, index, table index
   pair: text format; export
.. index:: element, table index, function index
   pair: text format; element
   single: table; element
   single: element; segment
.. _text-table-abbrev:

短縮形
.............

1個の :ref:`要素セグメント <text-elem>` は、1個のテーブル定義を用いてインラインで与えることができます。この場合そのオフセットは :math:`0` となり、その :ref:`テーブル型 <text-tabletype>` の :ref:`制限 <text-limits>` は、渡されるセグメントの長さを元に推測されます。

.. math::
   \begin{array}{llclll}
   \production{module field} &
     \text{(}~\text{table}~~\Tid^?~~\Telemtype~~\text{(}~\text{elem}~~x^n{:}\Tvec(\Tfuncidx)~\text{)}~~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{table}~~\Tid'~~n~~n~~\Telemtype~\text{)}~~
       \text{(}~\text{elem}~~\Tid'~~\text{(}~\text{i32.const}~~\text{0}~\text{)}~~\Tvec(\Tfuncidx)~\text{)}
       \\ & \qquad\qquad
       (\iff \Tid' = \Tid^? \neq \epsilon \vee \Tid' \idfresh) \\
   \end{array}

テーブルは、:ref:`インポート <text-import>` や :ref:`エクスポート <text-export>` のインラインとして定義できます。

.. math::
   \begin{array}{llclll}
   \production{module field} &
     \text{(}~\text{table}~~\Tid^?~~\text{(}~\text{import}~~\Tname_1~~\Tname_2~\text{)}~~\Ttabletype~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{import}~~\Tname_1~~\Tname_2~~\text{(}~\text{table}~~\Tid^?~~\Ttabletype~\text{)}~\text{)} \\[1ex] &
     \text{(}~\text{table}~~\Tid^?~~\text{(}~\text{export}~~\Tname~\text{)}~~\dots~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{export}~~\Tname~~\text{(}~\text{table}~~\Tid'~\text{)}~\text{)}~~
       \text{(}~\text{table}~~\Tid'~~\dots~\text{)}
       \\ & \qquad\qquad
       (\iff \Tid' = \Tid^? \neq \epsilon \vee \Tid' \idfresh) \\
   \end{array}

後者の短縮形は、別のインポートやエクスポートを含む「:math:`\dots`」を用いて繰り返し適用可能です。

.. index:: memory, memory type, identifier
   pair: text format; memory
.. _text-mem:

メモリー（memory）
~~~~~~~~

メモリー定義は、1個のシンボリックな :ref:`メモリー識別子 <text-id>` を束縛できます。

.. math::
   \begin{array}{llclll}
   \production{memory} & \Tmem_I &::=&
     \text{(}~\text{memory}~~\Tid^?~~\X{mt}{:}\Tmemtype~\text{)}
       &\Rightarrow& \{ \MTYPE~\X{mt} \} \\
   \end{array}


.. index:: import, name
   pair: text format; import
.. index:: export, name, index, memory index
   pair: text format; export
.. index:: data, memory, memory index, expression, byte, page size
   pair: text format; data
   single: memory; data
   single: data; segment
.. _text-mem-abbrev:

短縮形
.............

1個の :ref:`データセグメント <text-data>` は、1個のメモリー定義を用いてインラインで与えることができます。この場合そのオフセットは :math:`0` となり、その :ref:`メモリー型 <text-memtype>` の :ref:`制限 <text-limits>` は、渡されるセグメントの長さを元に推測されて :ref:`ページサイズ <page-size>` に丸められます。

.. math::
   \begin{array}{llclll}
   \production{module field} &
     \text{(}~\text{memory}~~\Tid^?~~\text{(}~\text{data}~~b^n{:}\Tdatastring~\text{)}~~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{memory}~~\Tid'~~m~~m~\text{)}~~
       \text{(}~\text{data}~~\Tid'~~\text{(}~\text{i32.const}~~\text{0}~\text{)}~~\Tdatastring~\text{)}
       \\ & \qquad\qquad
       (\iff \Tid' = \Tid^? \neq \epsilon \vee \Tid' \idfresh, m = \F{ceil}(n / 64\F{Ki})) \\
   \end{array}

メモリーは、:ref:`インポート <text-import>` や :ref:`エクスポート <text-export>` のインラインとして定義できます。

.. math::
   \begin{array}{llclll}
   \production{module field} &
     \text{(}~\text{memory}~~\Tid^?~~\text{(}~\text{import}~~\Tname_1~~\Tname_2~\text{)}~~\Tmemtype~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{import}~~\Tname_1~~\Tname_2~~\text{(}~\text{memory}~~\Tid^?~~\Tmemtype~\text{)}~\text{)}
       \\[1ex] &
     \text{(}~\text{memory}~~\Tid^?~~\text{(}~\text{export}~~\Tname~\text{)}~~\dots~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{export}~~\Tname~~\text{(}~\text{memory}~~\Tid'~\text{)}~\text{)}~~
       \text{(}~\text{memory}~~\Tid'~~\dots~\text{)}
       \\ & \qquad\qquad
       (\iff \Tid' = \Tid^? \neq \epsilon \vee \Tid' \idfresh) \\
   \end{array}

後者の短縮形は、別のインポートやエクスポートやインラインデータセグメントを含む「:math:`\dots`」を用いて繰り返し適用可能です。

.. index:: global, global type, identifier, expression
   pair: text format; global
.. _text-global:

グローバル（global）
~~~~~~~

グローバル定義は、1個のシンボリックな :ref:`グローバル識別子 <text-id>` を束縛できます。

.. math::
   \begin{array}{llclll}
   \production{global} & \Tglobal_I &::=&
     \text{(}~\text{global}~~\Tid^?~~\X{gt}{:}\Tglobaltype~~e{:}\Texpr_I~\text{)}
       &\Rightarrow& \{ \GTYPE~\X{gt}, \GINIT~e \} \\
   \end{array}


.. index:: import, name
   pair: text format; import
.. index:: export, name, index, global index
   pair: text format; export
.. _text-global-abbrev:

短縮形
.............

グローバルは、:ref:`インポート <text-import>` や :ref:`エクスポート <text-export>` のインラインとして定義できます。

.. math::
   \begin{array}{llclll}
   \production{module field} &
     \text{(}~\text{global}~~\Tid^?~~\text{(}~\text{import}~~\Tname_1~~\Tname_2~\text{)}~~\Tglobaltype~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{import}~~\Tname_1~~\Tname_2~~\text{(}~\text{global}~~\Tid^?~~\Tglobaltype~\text{)}~\text{)}
       \\[1ex] &
     \text{(}~\text{global}~~\Tid^?~~\text{(}~\text{export}~~\Tname~\text{)}~~\dots~\text{)} \quad\equiv \\ & \qquad
       \text{(}~\text{export}~~\Tname~~\text{(}~\text{global}~~\Tid'~\text{)}~\text{)}~~
       \text{(}~\text{global}~~\Tid'~~\dots~\text{)}
       \\ & \qquad\qquad
       (\iff \Tid' = \Tid^? \neq \epsilon \vee \Tid' \idfresh) \\
   \end{array}

後者の短縮形は、別のインポートやエクスポートを含む「:math:`\dots`」を用いて繰り返し適用可能です。

.. index:: export, name, index, function index, table index, memory index, global index
   pair: text format; export
.. _text-exportdesc:
.. _text-export:

エクスポート（export）
~~~~~~~

エクスポートの構文は、その :ref:`抽象構文 <syntax-export>` を直接反映しています。

.. math::
   \begin{array}{llclll}
   \production{export} & \Texport_I &::=&
     \text{(}~\text{export}~~\X{nm}{:}\Tname~~d{:}\Texportdesc_I~\text{)}
       &\Rightarrow& \{ \ENAME~\X{nm}, \EDESC~d \} \\
   \production{export description} & \Texportdesc_I &::=&
     \text{(}~\text{func}~~x{:}\Tfuncidx_I~\text{)}
       &\Rightarrow& \EDFUNC~x \\ &&|&
     \text{(}~\text{table}~~x{:}\Ttableidx_I~\text{)}
       &\Rightarrow& \EDTABLE~x \\ &&|&
     \text{(}~\text{memory}~~x{:}\Tmemidx_I~\text{)}
       &\Rightarrow& \EDMEM~x \\ &&|&
     \text{(}~\text{global}~~x{:}\Tglobalidx_I~\text{)}
       &\Rightarrow& \EDGLOBAL~x \\
   \end{array}


短縮形
.............

短縮形のエクスポートでは、「:ref:`関数 <text-func>`」「:ref:`テーブル <text-table>`」「:ref:`メモリー <text-mem>`」「:ref:`グローバル <text-global>`」の定義をインラインで指定することもできます。詳しくは該当セクションを参照。


.. index:: start function, function index
   pair: text format; start function
.. _text-start:

開始関数（start function）
~~~~~~~~~~~~~~

:ref:`開始関数 <syntax-start>` は、そのインデックスで定義されます。

.. math::
   \begin{array}{llclll}
   \production{start function} & \Tstart_I &::=&
     \text{(}~\text{start}~~x{:}\Tfuncidx_I~\text{)}
       &\Rightarrow& \{ \SFUNC~x \} \\
   \end{array}

.. note::
   1個のモジュール内に出現できる開始関数は最大1個までです。
   これは |Tmodule| 文法の適切な副条件によって確定されます。

.. index:: element, table index, expression, function index
   pair: text format; element
   single: table; element
   single: element; segment
.. _text-elem:

要素セグメント（element segment）
~~~~~~~~~~~~~~~~

要素セグメントは、初期化するテーブルを識別するオプションの :ref:`テーブルインデックス <text-tableidx>` を利用できるようにします。

.. math::
   \begin{array}{llclll}
   \production{element segment} & \Telem_I &::=&
     \text{(}~\text{elem}~~x{:}\Ttableidx_I~~\text{(}~\text{offset}~~e{:}\Texpr_I~\text{)}~~y^\ast{:}\Tvec(\Tfuncidx_I)~\text{)} \\ &&& \qquad
       \Rightarrow\quad \{ \ETABLE~x, \EOFFSET~e, \EINIT~y^\ast \} \\
   \end{array}

.. note::
   WebAssemblyの現在のバージョンにおけるテーブルインデックスは、「0」または「同じ値に解決されるシンボリックな :ref:`テーブル識別子 <text-id>`」のみが有効です。

短縮形
.............

オフセットの代わりに単一のインストラクションが短縮形として出現することがあります。

.. math::
   \begin{array}{llcll}
   \production{element offset} &
     \Tinstr &\equiv&
     \text{(}~\text{offset}~~\Tinstr~\text{)}
   \end{array}

また、このテーブルインデックスは省略可能です（デフォルトは :math:`\T{0}`）。

.. math::
   \begin{array}{llclll}
   \production{element segment} &
    \text{(}~\text{elem}~~\text{(}~\text{offset}~~\Texpr_I~\text{)}~~\dots~\text{)}
       &\equiv&
     \text{(}~\text{elem}~~0~~\text{(}~\text{offset}~~\Texpr_I~\text{)}~~\dots~\text{)}
   \end{array}

もうひとつの省略形として、:ref:`テーブル <text-table>` 定義をインラインの要素セグメントで指定することもできます。詳しくは該当セクションを参照。

.. index:: data, memory, memory index, expression, byte
   pair: text format; data
   single: memory; data
   single: data; segment
.. _text-datastring:
.. _text-data:

データセグメント（data segment）
~~~~~~~~~~~~~

データセグメントは、初期化するメモリーを識別するオプションの :ref:`メモリーインデックス <text-memidx>` で利用できます。
このデータは :ref:`文字列 <text-string>` として記述され、個別の文字列リテラルの空シーケンスにまで分割される可能性があります。

.. math::
   \begin{array}{llclll}
   \production{data segment} & \Tdata_I &::=&
     \text{(}~\text{data}~~x{:}\Tmemidx_I~~\text{(}~\text{offset}~~e{:}\Texpr_I~\text{)}~~b^\ast{:}\Tdatastring~\text{)} \\ &&& \qquad
       \Rightarrow\quad \{ \DMEM~x', \DOFFSET~e, \DINIT~b^\ast \} \\[1ex]
   \production{data string} & \Tdatastring &::=&
     (b^\ast{:}\Tstring)^\ast \quad\Rightarrow\quad \concat((b^\ast)^\ast) \\
   \end{array}

.. note::
   WebAssemblyの現在のバージョンにおけるメモリーインデックスは、「0」または「同じ値に解決されるシンボリックな :ref:`メモリー識別子 <text-id>`」のみが有効です。


短縮形
.............

オフセットの代わりに単一のインストラクションが短縮形として出現することがあります。

.. math::
   \begin{array}{llcll}
   \production{data offset} &
     \Tinstr &\equiv&
     \text{(}~\text{offset}~~\Tinstr~\text{)}
   \end{array}

また、このメモリーインデックスは省略可能です（デフォルトは :math:`\T{0}`）。

.. math::
   \begin{array}{llclll}
   \production{data segment} &
    \text{(}~\text{data}~~\text{(}~\text{offset}~~\Texpr_I~\text{)}~~\dots~\text{)}
       &\equiv&
     \text{(}~\text{data}~~0~~\text{(}~\text{offset}~~\Texpr_I~\text{)}~~\dots~\text{)}
   \end{array}

もうひとつの省略形として、:ref:`メモリー <text-mem>` 定義をインラインのデータセグメントで指定することもできます。詳しくは該当セクションを参照。


.. index:: module, type definition, function type, function, table, memory, global, element, data, start function, import, export, identifier context, identifier, name section
   pair: text format; module
   single: section; name
.. _text-modulefield:
.. _text-module:

モジュール（module）
~~~~~~~

1個のモジュールは、任意の順序で発生しうるフィールドのシーケンスで構成されます。
モジュールの定義やそれに対応する :ref:`識別子 <text-id>` は、それに先行するテキストを含め、すべてモジュール全体をスコープとします。

1個のモジュールは、オプションでそのモジュールに名前付けするために1個の :ref:`識別子 <text-id>` を束縛できます。
この名前の役割は、可読性のためのドキュメンテーションに過ぎません。

.. note::
   ツールでは、:ref:`バイナリ形式 <binary>` の :ref:`名前セクション <binary-namesec>` 内にこのモジュール名を含むことがあります。

.. math::
   \begin{array}{lll}
   \production{module} & \Tmodule &
   \begin{array}[t]{@{}cllll}
   ::=&
     \text{(}~\text{module}~~\Tid^?~~(m{:}\Tmodulefield_I)^\ast~\text{)}
       \quad\Rightarrow\quad \bigcompose m^\ast \\
       &\qquad (\iff I = \bigcompose \F{idc}(\Tmodulefield)^\ast \idcwellformed) \\
   \end{array} \\[1ex]
   \production{module field} & \Tmodulefield_I &
   \begin{array}[t]{@{}clll}
   ::=&
     \X{ty}{:}\Ttype &\Rightarrow& \{\MTYPES~\X{ty}\} \\ |&
     \X{im}{:}\Timport_I &\Rightarrow& \{\MIMPORTS~\X{im}\} \\ |&
     \X{fn}{:}\Tfunc_I &\Rightarrow& \{\MFUNCS~\X{fn}\} \\ |&
     \X{ta}{:}\Ttable_I &\Rightarrow& \{\MTABLES~\X{ta}\} \\ |&
     \X{me}{:}\Tmem_I &\Rightarrow& \{\MMEMS~\X{me}\} \\ |&
     \X{gl}{:}\Tglobal_I &\Rightarrow& \{\MGLOBALS~\X{gl}\} \\ |&
     \X{ex}{:}\Texport_I &\Rightarrow& \{\MEXPORTS~\X{ex}\} \\ |&
     \X{st}{:}\Tstart_I &\Rightarrow& \{\MSTART~\X{st}\} \\ |&
     \X{el}{:}\Telem_I &\Rightarrow& \{\MELEM~\X{el}\} \\ |&
     \X{da}{:}\Tdata_I &\Rightarrow& \{\MDATA~\X{da}\} \\
   \end{array}
   \end{array}

以下の制約は、:ref:`モジュール <syntax-module>` の合成（composition）に対するものです。:math:`m_1 \compose m_2` は以下が成立する場合に限って定義されます。

* :math:`m_1.\MSTART = \epsilon \vee m_2.\MSTART = \epsilon`

* :math:`m_1.\MFUNCS = m_1.\MTABLES = m_1.\MMEMS = m_1.\MGLOBALS = \epsilon \vee m_2.\MIMPORTS = \epsilon`

.. note::
   第1条件は、開始関数が最大1個までとなるようにします。
   第2条件は、すべての :ref:`インポート <text-import>` が通常の「:ref:`関数 <text-func>`」「:ref:`テーブル <text-table>`」「:ref:`メモリー <text-mem>`」「:ref:`グローバル <text-global>`」の定義より前に出現しなければならないよう強制し、それによってそれぞれの :ref:`インデックス空間 <syntax-index>` を維持します。

   |Tmodule| の文法内における :math:`I` の :ref:`整形式性 <text-context-wf>` 条件は、名前空間内の識別子が重複しないようにするものです。

初期状態の :ref:`識別志向テキスト <text-context>` :math:`I` の定義では、以下の補助定義を用います。これらは、関連する個別の定義を、1個の定義を持つ（空の可能性もある）単一のコンテキストに対応付けます。

.. math::
   \begin{array}{@{}lcl@{\qquad\qquad}l}
   \F{idc}(\text{(}~\text{type}~\Tid^?~\X{ft}{:}\Tfunctype~\text{)}) &=&
     \{\ITYPES~(\Tid^?), \ITYPEDEFS~\X{ft}\} \\
   \F{idc}(\text{(}~\text{func}~\Tid^?~\dots~\text{)}) &=&
     \{\IFUNCS~(\Tid^?)\} \\
   \F{idc}(\text{(}~\text{table}~\Tid^?~\dots~\text{)}) &=&
     \{\ITABLES~(\Tid^?)\} \\
   \F{idc}(\text{(}~\text{memory}~\Tid^?~\dots~\text{)}) &=&
     \{\IMEMS~(\Tid^?)\} \\
   \F{idc}(\text{(}~\text{global}~\Tid^?~\dots~\text{)}) &=&
     \{\IGLOBALS~(\Tid^?)\} \\
   \F{idc}(\text{(}~\text{import}~\dots~\text{(}~\text{func}~\Tid^?~\dots~\text{)}~\text{)}) &=&
     \{\IFUNCS~(\Tid^?)\} \\
   \F{idc}(\text{(}~\text{import}~\dots~\text{(}~\text{table}~\Tid^?~\dots~\text{)}~\text{)}) &=&
     \{\ITABLES~(\Tid^?)\} \\
   \F{idc}(\text{(}~\text{import}~\dots~\text{(}~\text{memory}~\Tid^?~\dots~\text{)}~\text{)}) &=&
     \{\IMEMS~(\Tid^?)\} \\
   \F{idc}(\text{(}~\text{import}~\dots~\text{(}~\text{global}~\Tid^?~\dots~\text{)}~\text{)}) &=&
     \{\IGLOBALS~(\Tid^?)\} \\
   \F{idc}(\text{(}~\dots~\text{)}) &=&
     \{\} \\
   \end{array}


短縮形
.............

ソースファイル内では、モジュール本体を囲むトップレベルの :math:`\T{(module}~\dots\T{)}` は省略可能です。

.. math::
   \begin{array}{llcll}
   \production{module} &
     \Tmodulefield^\ast &\equiv&
     \text{(}~\text{module}~~\Tmodulefield^\ast~\text{)}
   \end{array}
