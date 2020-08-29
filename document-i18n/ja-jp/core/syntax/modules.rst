.. index:: ! module, type definition, function type, function, table, memory, global, element, data, start function, import, export
   pair: abstract syntax; module
.. _syntax-module:

モジュール
-------

WebAssemblyプログラムは「モジュール」の集合体として編成されます。
モジュールは「デプロイ」「読み込み」「コンパイル」の単位です。
モジュールには1つ以上の「:ref:`型 <syntax-type>`」「:ref:`関数 <syntax-func>`」「:ref:`テーブル <syntax-table>`」「:ref:`メモリー <syntax-mem>`」「:ref:`グローバル <syntax-global>`」の定義が集約されます。
それらの他に「:ref:`インポート <syntax-import>`」「:ref:`エクスポート <syntax-export>`」も宣言でき、初期化ロジックを「:ref:`データ <syntax-data>` セグメント」や「:ref:`要素 <syntax-elem>` セグメント」や「:ref:`開始関数 <syntax-start>`」として提供することもできます。

.. math::
   \begin{array}{lllll}
   \production{module} & \module &::=& \{ &
     \MTYPES~\vec(\functype), \\&&&&
     \MFUNCS~\vec(\func), \\&&&&
     \MTABLES~\vec(\table), \\&&&&
     \MMEMS~\vec(\mem), \\&&&&
     \MGLOBALS~\vec(\global), \\&&&&
     \MELEM~\vec(\elem), \\&&&&
     \MDATA~\vec(\data), \\&&&&
     \MSTART~\start^?, \\&&&&
     \MIMPORTS~\vec(\import), \\&&&&
     \MEXPORTS~\vec(\export) \quad\} \\
   \end{array}

個別のベクタは空にすることもできます（つまりモジュール全体が空になることもあります）。

.. index:: ! index, ! index space, ! type index, ! function index, ! table index, ! memory index, ! global index, ! local index, ! label index, function, global, table, memory, local, parameter, import
   pair: abstract syntax; type index
   pair: abstract syntax; function index
   pair: abstract syntax; table index
   pair: abstract syntax; memory index
   pair: abstract syntax; global index
   pair: abstract syntax; local index
   pair: abstract syntax; label index
   pair: type; index
   pair: function; index
   pair: table; index
   pair: memory; index
   pair: global; index
   pair: local; index
   pair: label; index
.. _syntax-typeidx:
.. _syntax-funcidx:
.. _syntax-tableidx:
.. _syntax-memidx:
.. _syntax-globalidx:
.. _syntax-localidx:
.. _syntax-labelidx:
.. _syntax-index:

インデックス（index）
~~~~~~~

定義はゼロベースの「インデックス」で参照されます。
定義のクラスはそれぞれ独自の「インデックス空間（index space）」を持ち、以下のクラスに分類されます。

.. math::
   \begin{array}{llll}
   \production{type index} & \typeidx &::=& \u32 \\
   \production{function index} & \funcidx &::=& \u32 \\
   \production{table index} & \tableidx &::=& \u32 \\
   \production{memory index} & \memidx &::=& \u32 \\
   \production{global index} & \globalidx &::=& \u32 \\
   \production{local index} & \localidx &::=& \u32 \\
   \production{label index} & \labelidx &::=& \u32 \\
   \end{array}

:ref:`関数 <syntax-func>` と :ref:`テーブル <syntax-table>` と :ref:`メモリー <syntax-mem>` と :ref:`グローバル <syntax-global>` のインデックス空間では、同じモジュール内で宣言された :ref:`インポート <syntax-import>` がそれぞれのインデックス空間に含まれます。
それらのインポートのインデックスは、同じインデックス空間内のその他の定義のインデックスよりも優先されます。

:ref:`ローカル値 <syntax-local>` のインデックス空間は、:ref:`関数 <syntax-func>` 内部でのアクセスのみが許されます。その関数のパラメーターも含まれますが、これはローカル変数よりも優先されます。

ラベルインデックスは、あるインストラクションシーケンス内部の :ref:`構造化制御インストラクション <syntax-instr-control>` を参照します。

本仕様での記法
...........

* メタ変数 :math:`x` はさまざまなラベルインデックスを表します。

* メタ変数 :math:`x, y` は、その他のあらゆるインデックス空間でさまざまなラベルインデックスを表します。

.. index:: ! type definition, type index, function type
   pair: abstract syntax; type definition
.. _syntax-typedef:

型（type）
~~~~~

モジュールの |MTYPES| コンポーネントは、:ref:`関数型 <syntax-functype>` のベクタを1つ定義します。

あるモジュール内で用いられるすべての関数型はこのコンポーネントで定義されなければなりません。
これらは :ref:`型インデックス <syntax-typeidx>` で参照されます。

.. note::
   WebAssemblyの今後のバージョンでは、型定義の形式が追加される可能性があります。

.. index:: ! function, ! local, function index, local index, type index, value type, expression, import
   pair: abstract syntax; function
   pair: abstract syntax; local
.. _syntax-local:
.. _syntax-func:

関数（function）
~~~~~~~~~

モジュールの |MFUNCS| コンポーネントは、以下の構造を持つ「関数」のベクタを1つ定義します。

.. math::
   \begin{array}{llll}
   \production{function} & \func &::=&
     \{ \FTYPE~\typeidx, \FLOCALS~\vec(\valtype), \FBODY~\expr \} \\
   \end{array}

関数の |FTYPE| は、モジュール内で定義される :ref:`型 <syntax-type>` を参照することで関数のシグネチャを宣言します。
この関数のパラメーターは、関数の本体にあるゼロベースの :ref:`ローカルインデックス <syntax-localidx>` を介して参照されます。これらはミュータブルです。

|FLOCALS| は、ミュータブルなローカル変数やそれらの型のベクタを1つ宣言します。
これらの変数は、関数の本体にある :ref:`ローカルインデックス <syntax-localidx>` を介して参照されます。これらはミュータブルです。
最初のローカル変数のインデックスは、パラメーターを参照しない最小のインデックスです。

|FBODY| は、:ref:`インストラクション <syntax-expr>` のシーケンスで、これはその関数の型の :ref:`結果型 <syntax-resulttype>` と一致するスタックを生成しなければなりません。


関数は :ref:`関数インデックス <syntax-funcidx>` を介して参照されます。このインデックスは、関数 :ref:`インポート <syntax-import>` を参照しない最小のインデックスで始まります。

.. index:: ! table, table index, table type, limits, element, import
   pair: abstract syntax; table
.. _syntax-table:

テーブル（table）
~~~~~~

モジュールの |MTABLES| コンポーネントは、モジュールの :ref:`テーブル型 <syntax-tabletype>` で記述される「テーブル」のベクタを1つ定義します。

.. math::
   \begin{array}{llll}
   \production{table} & \table &::=&
     \{ \TTYPE~\tabletype \} \\
   \end{array}

1つのテーブルは、特定のテーブル :ref:`要素型 <syntax-elemtype>` の不透明な値のベクタです。
そのテーブル型の :ref:`制約 <syntax-limits>` の |LMIN| サイズはそのテーブルの初期サイズを指定しますが、|LMAX| が存在する場合は、後にテーブルが成長する場合のサイズに制約をかけます。

テーブルは :ref:`要素セグメント <syntax-elem>` を介して初期化されます。

テーブルは :ref:`テーブルインデックス <syntax-tableidx>` を介して参照され、テーブルの :ref:`インポート <syntax-import>` を参照しない最小のインデックスで始まります。
ほとんどの構成（construct）では、テーブルのインデックス :math:`0` を暗黙で参照します。

.. note::
   WebAssemblyの現在のバージョンでは、1つのモジュールで最大で1つのテーブルを定義またはインポートでき、すべての構成でテーブルのインデックス :math:`0` を暗黙で参照します。
   この制約は今後のバージョンで解除される可能性があります。

.. index:: ! memory, memory index, memory type, limits, page size, data, import
   pair: abstract syntax; memory
.. _syntax-mem:

メモリー（memory）
~~~~~~~~

モジュールの |MMEMS| コンポーネントは、モジュールの :ref:`メモリー型 <syntax-memtype>` で記述されたとおりに「線形メモリー（または単にメモリー）」のベクタを1つ定義します。

.. math::
   \begin{array}{llll}
   \production{memory} & \mem &::=&
     \{ \MTYPE~\memtype \} \\
   \end{array}

1つのメモリーは、解釈されない生のバイト列のベクタです。
そのメモリー型の :ref:`制約 <syntax-limits>` の |LMIN| サイズは、メモリーの初期サイズを指定しますが、|LMAX| が存在する場合は、後にメモリーが成長する場合のサイズに制約をかけます。
どちらも :ref:`ページサイズ <page-size>` を単位とします。

メモリーは :ref:`データセグメント <syntax-data>` を介して初期化されます。

メモリーは :ref:`メモリーインデックス <syntax-memidx>` を介して参照され、メモリーの :ref:`インポート <syntax-import>` を参照しない最小のインデックスで始まります。
ほとんどの構成（construct）では、メモリーのインデックス :math:`0` を暗黙で参照します。

.. note::
   WebAssemblyの現在のバージョンでは、1つのモジュールで最大で1つのメモリーを定義またはインポートでき、すべての構成でメモリーのインデックス :math:`0` を暗黙で参照します。
   この制約は今後のバージョンで解除される可能性があります。

.. index:: ! global, global index, global type, mutability, expression, constant, value, import
   pair: abstract syntax; global
.. _syntax-global:

グローバル（globals）
~~~~~~~

モジュールの |MGLOBALS| コンポーネントは「グローバル変数（または単にグローバル）」のベクタを1つ定義します。

.. math::
   \begin{array}{llll}
   \production{global} & \global &::=&
     \{ \GTYPE~\globaltype, \GINIT~\expr \} \\
   \end{array}

1つのグローバルごとに、:ref:`グローバル型 <syntax-globaltype>` で与えられる値が1つ保存されます。
そのグローバルの |GTYPE| では、グローバルがイミュータブルかミュータブル化も指定されます。
さらに、1つのグローバルごとに :ref:`定数 <valid-constant>` イニシャライザ :ref:`式 <syntax-expr>` で与えられる |GINIT| 値で初期化されます。

グローバルは :ref:`グローバルインデックス <syntax-globalidx>` を介して参照され、グローバルの :ref:`インポート <syntax-import>` を参照しない最小のインデックスで始まります。

.. index:: ! element, table, table index, expression, constant, function index, vector
   pair: abstract syntax; element
   single: table; element
   single: element; segment
.. _syntax-elem:

要素セグメント（element segment）
~~~~~~~~~~~~~~~~

テーブルの最初の内容は初期化されません。
モジュールの |MELEM| コンポーネントは、要素の静的な :ref:`ベクタ <syntax-vec>` を用いて、指定のオフセットに、あるテーブルの一部を初期化する「要素セグメント」のベクタを1つ定義します。

.. math::
   \begin{array}{llll}
   \production{element segment} & \elem &::=&
     \{ \ETABLE~\tableidx, \EOFFSET~\expr, \EINIT~\vec(\funcidx) \} \\
   \end{array}

|EOFFSET| は :ref:`定数 <valid-constant>` の :ref:`式 <syntax-expr>` で与えられます。

.. note::
   WebAssemblyの現在のバージョンでは、1つのモジュール内で許されているテーブル数は1つまでです。
   このため、有効な |tableidx| は :math:`0` だけです。

.. index:: ! data, memory, memory index, expression, constant, byte, vector
   pair: abstract syntax; data
   single: memory; data
   single: data; segment
.. _syntax-data:

データセグメント（data segment）
~~~~~~~~~~~~~

:ref:`メモリー <syntax-mem>` の初期の内容はゼロ値のバイト列です。
モジュールの |MDATA| コンポーネントは、:ref:`バイト <syntax-byte>` の静的な :ref:`ベクタ <syntax-vec>` を用いて、指定のオフセットに、あるテーブルの一部を初期化する「要素セグメント」のベクタを1つ定義します

.. math::
   \begin{array}{llll}
   \production{data segment} & \data &::=&
     \{ \DMEM~\memidx, \DOFFSET~\expr, \DINIT~\vec(\byte) \} \\
   \end{array}

|DOFFSET| は :ref:`定数 <valid-constant>` の :ref:`式 <syntax-expr>` で与えられます。

.. note::
   WebAssemblyの現在のバージョンでは、1つのモジュール内で許されているメモリー数は1つまでです。
   このため、有効な |tableidx| は :math:`0` だけです


.. index:: ! start function, function, function index, table, memory, instantiation
   pair: abstract syntax; start function
.. _syntax-start:

開始関数（start function）
~~~~~~~~~~~~~~

モジュールの |MSTART| コンポーネントは、モジュールが :ref:`インスタンス化 <exec-instantiation>` されるときに :ref:`テーブル <syntax-table>` や :ref:`メモリー <syntax-mem>` の初期化語に自動的に呼び出される「開始関数」の :ref:`関数インデックス <syntax-funcidx>` を宣言します。

.. math::
   \begin{array}{llll}
   \production{start function} & \start &::=&
     \{ \SFUNC~\funcidx \} \\
   \end{array}

.. note::
   開始関数は、モジュールのステートを初期化することを意図しています。
   この初期化が完了するまでは、モジュールやモジュールのエクスポートにアクセスできません。

.. index:: ! export, name, index, function index, table index, memory index, global index, function, table, memory, global, instantiation
   pair: abstract syntax; export
   single: function; export
   single: table; export
   single: memory; export
   single: global; export
.. _syntax-exportdesc:
.. _syntax-export:

エクスポート（exports）
~~~~~~~

モジュールの |MEXPORTS| コンポーネントは、モジュールの :ref:`初期化 <exec-instantiation>` が完了したときにホスト環境にアクセスできる「エクスポート」のセットを1つ定義します。

.. math::
   \begin{array}{llcl}
   \production{export} & \export &::=&
     \{ \ENAME~\name, \EDESC~\exportdesc \} \\
   \production{export description} & \exportdesc &::=&
     \EDFUNC~\funcidx \\&&|&
     \EDTABLE~\tableidx \\&&|&
     \EDMEM~\memidx \\&&|&
     \EDGLOBAL~\globalidx \\
   \end{array}

1つのエクスポートごとに、一意の :ref:`名前 <syntax-name>` がラベル付けされます。
エクスポート可能な定義は1つ以上の「:ref:`関数 <syntax-func>`」「:ref:`テーブル <syntax-table>`」「:ref:`メモリー <syntax-mem>`」「:ref:`グローバル <syntax-global>`」で、これらはそれぞれに対応するデスクリプタを介して参照されます。

本仕様での記法
...........

以下の補助記法は、エクスポートのシーケンス向けに定義されており、順序を維持しながら特定の種類のインデックスをフィルタで除外します。

* :math:`\edfuncs(\export^\ast) = [\funcidx ~|~ \EDFUNC~\funcidx \in (\export.\EDESC)^\ast]`

* :math:`\edtables(\export^\ast) = [\tableidx ~|~ \EDTABLE~\tableidx \in (\export.\EDESC)^\ast]`

* :math:`\edmems(\export^\ast) = [\memidx ~|~ \EDMEM~\memidx \in (\export.\EDESC)^\ast]`

* :math:`\edglobals(\export^\ast) = [\globalidx ~|~ \EDGLOBAL~\globalidx \in (\export.\EDESC)^\ast]`


.. index:: ! import, name, function type, table type, memory type, global type, index, index space, type index, function index, table index, memory index, global index, function, table, memory, global, instantiation
   pair: abstract syntax; import
   single: function; import
   single: table; import
   single: memory; import
   single: global; import
.. _syntax-importdesc:
.. _syntax-import:

インポート（imports）
~~~~~~~

モジュールの |MIMPORTS| コンポーネントは、:ref:`インスタンス化 <exec-instantiation>` で必要になる「インポート」のセットを1つ定義します。

.. math::
   \begin{array}{llll}
   \production{import} & \import &::=&
     \{ \IMODULE~\name, \INAME~\name, \IDESC~\importdesc \} \\
   \production{import description} & \importdesc &::=&
     \IDFUNC~\typeidx \\&&|&
     \IDTABLE~\tabletype \\&&|&
     \IDMEM~\memtype \\&&|&
     \IDGLOBAL~\globaltype \\
   \end{array}

1つのインポートごとに、2つのレベルの :ref:`名前 <syntax-name>` 空間がラベリングされます。ラベルは、1つの |IMODULE| 名と、そのモジュール内のエンティティを表す1つの |INAME| からなります。
インポート可能な定義は1つ以上の「:ref:`関数 <syntax-func>`」「:ref:`テーブル <syntax-table>`」「:ref:`メモリー <syntax-mem>`」「:ref:`グローバル <syntax-global>`」です。
1つのインポートごとに、インスタンス化の際に提供された定義が一致するように要求されるそれぞれの型を持つディスクリプタによって指定されます。

どのインポートも、それに対応する :ref:`インデックス空間 <syntax-index>` 内にインデックスを1つ定義します。
1つのインデックス空間ごとにあるインポートのインデックスは、モジュール自体に含まれる定義の最初のインデックスよりも前に配置されます。

.. note::
   エクスポート名と異なり、インポート名は必ずしも一意にならないことがあります。
   同じ |IMODULE|/|INAME| ペアを複数回インポートできますし、
   複数のインポートで型の記述が異なっている可能性すらあります（エンティティの種類が異なるなど）。
   そのようなインポートを持つモジュールでも、:ref:`エンベダー <embedder>` が解決とインポートの提供を許す方法によっては引き続きインスタンス化が可能です。
   しかし、エンベダーではそのようなオーバーロードのサポートが必須ではありませんし、WebAssemblyモジュール自体ではオーバーロードされた名前を実装できません。
