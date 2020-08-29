モジュール
-------

モジュールのバイナリエンコーディングは、いくつもの「セクション」で編成されます。
ほとんどのセクションは、1個の :ref:`モジュール<syntax-module>` 記録（record）の1個のコンポーネントに対応します。例外として :ref:`関数定義 <syntax-func>` は2つのセクションに分割され、:ref:`コードセクション <binary-codesec>` 内の関数本体にある関数の型宣言を2つの :ref:`関数セクション <binary-funcsec>` に分離します。

.. note::
   関数を2つのセクションに分離することで、モジュール内の関数コンパイルを「パラレル化」および「ストリーミング」できるようになります。

.. index:: index, type index, function index, table index, memory index, global index, local index, label index
   pair: binary format; type index
   pair: binary format; function index
   pair: binary format; table index
   pair: binary format; memory index
   pair: binary format; global index
   pair: binary format; local index
   pair: binary format; label index
.. _binary-typeidx:
.. _binary-funcidx:
.. _binary-tableidx:
.. _binary-memidx:
.. _binary-globalidx:
.. _binary-localidx:
.. _binary-labelidx:
.. _binary-index:

インデックス（index）
~~~~~~~

あらゆる :ref:`インデックス <syntax-index>` は、それぞれに応じた値でエンコードされます。

.. math::
   \begin{array}{llclll}
   \production{type index} & \Btypeidx &::=& x{:}\Bu32 &\Rightarrow& x \\
   \production{function index} & \Bfuncidx &::=& x{:}\Bu32 &\Rightarrow& x \\
   \production{table index} & \Btableidx &::=& x{:}\Bu32 &\Rightarrow& x \\
   \production{memory index} & \Bmemidx &::=& x{:}\Bu32 &\Rightarrow& x \\
   \production{global index} & \Bglobalidx &::=& x{:}\Bu32 &\Rightarrow& x \\
   \production{local index} & \Blocalidx &::=& x{:}\Bu32 &\Rightarrow& x \\
   \production{label index} & \Blabelidx &::=& l{:}\Bu32 &\Rightarrow& l \\
   \end{array}


.. index:: ! section
   pair: binary format; section
.. _binary-section:

セクション（section）
~~~~~~~~

1個のセクションは以下から構成されます。

* 1バイトのセクション「id」
* セクションコンテンツの「|U32| サイズ」（単位はバイト）
* 実際の「コンテンツ」（構成はセクションidによって異なる）

どのセクションも必須ではなく「省略可能」です。省略されたセクションは、空のコンテンツで表現されるセクションと同等です。

以下のパラメーター化された文法ルールは、「id :math:`N`」と「文法 :math:`\B{B}` で記述されたコンテンツ」を持つ1個のセクションの一般的な構造を定義します。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{section} & \Bsection_N(\B{B}) &::=&
     N{:}\Bbyte~~\X{size}{:}\Bu32~~\X{cont}{:}\B{B}
       &\Rightarrow& \X{cont} & (\iff \X{size} = ||\B{B}||) \\ &&|&
     \epsilon &\Rightarrow& \epsilon
   \end{array}

ほとんどのセクションでは、コンテンツ :math:`\B{B}` を1個の :ref:`ベクタ <binary-vec>` にエンコードします。
これらにおいて、空の結果 :math:`\epsilon` は空のベクタと解釈されます。

.. note::
   未知の :ref:`カスタムセクション <binary-customsec>` を除き、:math:`\X{size}` はデコードで必須ではありませんが、バイナリ内を移動するときにセクションをスキップするのに利用できます。
   セクションのサイズが、バイナリコンテンツ :math:`\B{B}` の長さと一致しない場合、そのモジュールは無効となります。

以下のセクションidが用いられます。

==  ========================================
Id  Section
==  ========================================
 0  :ref:`カスタムセクション <binary-customsec>`
 1  :ref:`型セクション <binary-typesec>`
 2  :ref:`インポートセクション <binary-importsec>`
 3  :ref:`関数セクション <binary-funcsec>`
 4  :ref:`テーブルセクション <binary-tablesec>`
 5  :ref:`メモリーセクション <binary-memsec>`
 6  :ref:`グローバルセクション <binary-globalsec>`
 7  :ref:`エクスポートセクション <binary-exportsec>`
 8  :ref:`開始セクション <binary-startsec>`
 9  :ref:`要素セクション <binary-elemsec>`
10  :ref:`コードセクション <binary-codesec>`
11  :ref:`データセクション <binary-datasec>`
==  ========================================


.. index:: ! custom section
   pair: binary format; custom section
   single: section; custom
.. _binary-customsec:

カスタムセクション（custom section）
~~~~~~~~~~~~~~

「カスタムセクション」のidはゼロです。
このセクションは、デバッグ情報やサードパーティ拡張での利用が想定されており、WebAssemblyセマンティクスでは無視されます。
このセクションのコンテンツは、カスタムセクションをより細かく識別するための :ref:`名前 <syntax-name>` に、カスタム利用する「解釈されない」バイトシーケンスが付けられます。

.. math::
   \begin{array}{llclll}
   \production{custom section} & \Bcustomsec &::=&
     \Bsection_0(\Bcustom) \\
   \production{custom data} & \Bcustom &::=&
     \Bname~~\Bbyte^\ast \\
   \end{array}

.. note::
   実装側でカスタムセクションのデータを解釈する場合は、そのデータで発生するエラーやセクションの配置が原因でモジュールが無効化されないようにしなければなりません。

.. index:: ! type section, type definition
   pair: binary format; type section
   pair: section; type
.. _binary-typedef:
.. _binary-typesec:

型セクション（type section）
~~~~~~~~~~~~

「型セクション」のidは1です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MTYPES| コンポーネントを表現する1個以上の :ref:`関数型 <syntax-functype>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{type section} & \Btypesec &::=&
     \X{ft}^\ast{:\,}\Bsection_1(\Bvec(\Bfunctype)) &\Rightarrow& \X{ft}^\ast \\
   \end{array}


.. index:: ! import section, import, name, function type, table type, memory type, global type
   pair: binary format; import
   pair: section; import
.. _binary-import:
.. _binary-importdesc:
.. _binary-importsec:

インポートセクション（import section）
~~~~~~~~~~~~~~

「インポートセクション」のidは2です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MIMPORTS| コンポーネントを表現する1個以上の :ref:`インポート <syntax-import>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{import section} & \Bimportsec &::=&
     \X{im}^\ast{:}\Bsection_2(\Bvec(\Bimport)) &\Rightarrow& \X{im}^\ast \\
   \production{import} & \Bimport &::=&
     \X{mod}{:}\Bname~~\X{nm}{:}\Bname~~d{:}\Bimportdesc
       &\Rightarrow& \{ \IMODULE~\X{mod}, \INAME~\X{nm}, \IDESC~d \} \\
   \production{import description} & \Bimportdesc &::=&
     \hex{00}~~x{:}\Btypeidx &\Rightarrow& \IDFUNC~x \\ &&|&
     \hex{01}~~\X{tt}{:}\Btabletype &\Rightarrow& \IDTABLE~\X{tt} \\ &&|&
     \hex{02}~~\X{mt}{:}\Bmemtype &\Rightarrow& \IDMEM~\X{mt} \\ &&|&
     \hex{03}~~\X{gt}{:}\Bglobaltype &\Rightarrow& \IDGLOBAL~\X{gt} \\
   \end{array}


.. index:: ! function section, function, type index, function type
   pair: binary format; function
   pair: section; function
.. _binary-funcsec:

関数セクション（function section）
~~~~~~~~~~~~~~~~

「関数セクション」のidは3です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MFUNCS| コンポーネント内にある :ref:`関数 <syntax-func>` の |FTYPE| フィールドを表現する1個以上の :ref:`型インデックス <syntax-typeidx>` のベクタ1個にデコードされます。
それぞれの関数にある |FLOCALS| フィールドと |FBODY| フィールドは、別途 :ref:`コードセクション <binary-codesec>` でエンコードされます。

.. math::
   \begin{array}{llclll}
   \production{function section} & \Bfuncsec &::=&
     x^\ast{:}\Bsection_3(\Bvec(\Btypeidx)) &\Rightarrow& x^\ast \\
   \end{array}


.. index:: ! table section, table, table type
   pair: binary format; table
   pair: section; table
.. _binary-table:
.. _binary-tablesec:

テーブルセクション（table section）
~~~~~~~~~~~~~

「テーブルセクション」のidは4です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MTABLES| コンポーネントを表現する1個以上の :ref:`テーブル <syntax-table>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{table section} & \Btablesec &::=&
     \X{tab}^\ast{:}\Bsection_4(\Bvec(\Btable)) &\Rightarrow& \X{tab}^\ast \\
   \production{table} & \Btable &::=&
     \X{tt}{:}\Btabletype &\Rightarrow& \{ \TTYPE~\X{tt} \} \\
   \end{array}


.. index:: ! memory section, memory, memory type
   pair: binary format; memory
   pair: section; memory
.. _binary-mem:
.. _binary-memsec:

メモリーセクション（memory section）
~~~~~~~~~~~~~~

「メモリーセクション」のidは5です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MMEMS| コンポーネントを表現する1個以上の :ref:`メモリー <syntax-mem>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{memory section} & \Bmemsec &::=&
     \X{mem}^\ast{:}\Bsection_5(\Bvec(\Bmem)) &\Rightarrow& \X{mem}^\ast \\
   \production{memory} & \Bmem &::=&
     \X{mt}{:}\Bmemtype &\Rightarrow& \{ \MTYPE~\X{mt} \} \\
   \end{array}


.. index:: ! global section, global, global type, expression
   pair: binary format; global
   pair: section; global
.. _binary-global:
.. _binary-globalsec:

グローバルセクション（global section）
~~~~~~~~~~~~~~

「グローバルセクション」のidは6です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MGLOBALS| コンポーネントを表現する1個以上の :ref:`グローバル <syntax-global>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{global section} & \Bglobalsec &::=&
     \X{glob}^\ast{:}\Bsection_6(\Bvec(\Bglobal)) &\Rightarrow& \X{glob}^\ast \\
   \production{global} & \Bglobal &::=&
     \X{gt}{:}\Bglobaltype~~e{:}\Bexpr
       &\Rightarrow& \{ \GTYPE~\X{gt}, \GINIT~e \} \\
   \end{array}


.. index:: ! export section, export, name, index, function index, table index, memory index, global index
   pair: binary format; export
   pair: section; export
.. _binary-export:
.. _binary-exportdesc:
.. _binary-exportsec:

エクスポートセクション（export section）
~~~~~~~~~~~~~~

「エクスポートセクション」のidは7です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MEXPORTS| コンポーネントを表現する1個以上の :ref:`エクスポート <syntax-export>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{export section} & \Bexportsec &::=&
     \X{ex}^\ast{:}\Bsection_7(\Bvec(\Bexport)) &\Rightarrow& \X{ex}^\ast \\
   \production{export} & \Bexport &::=&
     \X{nm}{:}\Bname~~d{:}\Bexportdesc
       &\Rightarrow& \{ \ENAME~\X{nm}, \EDESC~d \} \\
   \production{export description} & \Bexportdesc &::=&
     \hex{00}~~x{:}\Bfuncidx &\Rightarrow& \EDFUNC~x \\ &&|&
     \hex{01}~~x{:}\Btableidx &\Rightarrow& \EDTABLE~x \\ &&|&
     \hex{02}~~x{:}\Bmemidx &\Rightarrow& \EDMEM~x \\ &&|&
     \hex{03}~~x{:}\Bglobalidx &\Rightarrow& \EDGLOBAL~x \\
   \end{array}


.. index:: ! start section, start function, function index
   pair: binary format; start function
   single: section; start
   single: start function; section
.. _binary-start:
.. _binary-startsec:

開始セクション（start section）
~~~~~~~~~~~~~

「開始セクション」のidは8です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MSTART| コンポーネントを表現する1個のオプショナルな :ref:`開始関数 <syntax-start>` にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{start section} & \Bstartsec &::=&
     \X{st}^?{:}\Bsection_8(\Bstart) &\Rightarrow& \X{st}^? \\
   \production{start function} & \Bstart &::=&
     x{:}\Bfuncidx &\Rightarrow& \{ \SFUNC~x \} \\
   \end{array}


.. index:: ! element section, element, table index, expression, function index
   pair: binary format; element
   pair: section; element
   single: table; element
   single: element; segment
.. _binary-elem:
.. _binary-elemsec:

要素セクション（element section）
~~~~~~~~~~~~~~~

「要素セクション」のidは9です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MELEM| コンポーネントを表現する1個以上の :ref:`要素セグメント <syntax-elem>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{element section} & \Belemsec &::=&
     \X{seg}^\ast{:}\Bsection_9(\Bvec(\Belem)) &\Rightarrow& \X{seg} \\
   \production{element segment} & \Belem &::=&
     x{:}\Btableidx~~e{:}\Bexpr~~y^\ast{:}\Bvec(\Bfuncidx)
       &\Rightarrow& \{ \ETABLE~x, \EOFFSET~e, \EINIT~y^\ast \} \\
   \end{array}


.. index:: ! code section, function, local, type index, function type
   pair: binary format; function
   pair: binary format; local
   pair: section; code
.. _binary-code:
.. _binary-func:
.. _binary-local:
.. _binary-codesec:

コードセクション（code section）
~~~~~~~~~~~~

「コードセクション」のidは10です。
このセクションは、1個の :ref:`モジュール <syntax-module>` において「:ref:`値型 <syntax-valtype>` ベクタと :ref:`式 <syntax-expr>` のペア」でできている「コード」エントリのベクタ1個にデコードされます。
これらのペアは、1個の :ref:`モジュール <syntax-module>` の |MFUNCS| コンポーネント内にある :ref:`関数 <syntax-func>` の |FLOCALS| フィールドと |FBODY| フィールドを表現します。
対応する関数の |FTYPE| フィールドは、別途 :ref:`関数セクション <binary-funcsec>` でエンコードされます。

個別のコードエントリのエンコーディングは以下で構成されます。

* その関数コードの |U32|「 サイズ」（単位はバイト）
* 以下で構成される実際の「関数コード」

  * 1個以上の「ローカル」宣言
  * :ref:`式 <binary-expr>` の形を取った関数「本体」

ローカル宣言はは、以下で構成される1個のベクトルに圧縮されます。

* |U32|「カウント」
* :ref:`値型 <binary-valtype>`

これは、同じ値型を持つ1個以上の「カウント」ローカルを記述します。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{code section} & \Bcodesec &::=&
     \X{code}^\ast{:}\Bsection_{10}(\Bvec(\Bcode))
       &\Rightarrow& \X{code}^\ast \\
   \production{code} & \Bcode &::=&
     \X{size}{:}\Bu32~~\X{code}{:}\Bfunc
       &\Rightarrow& \X{code} & (\iff \X{size} = ||\Bfunc||) \\
   \production{function} & \Bfunc &::=&
     (t^\ast)^\ast{:}\Bvec(\Blocals)~~e{:}\Bexpr
       &\Rightarrow& \concat((t^\ast)^\ast), e^\ast
         & (\iff |\concat((t^\ast)^\ast)| < 2^{32}) \\
   \production{locals} & \Blocals &::=&
     n{:}\Bu32~~t{:}\Bvaltype &\Rightarrow& t^n \\
   \end{array}

上の :math:`\X{code}` はさまざまなペア :math:`(\valtype^\ast, \expr)` を表します。メタ関数 :math:`\concat((t^\ast)^\ast)` は、:math:`(t^\ast)^\ast` の中にあるすべてのシーケンスを結合します。
得られたシーケンスの長さが :ref:`ベクタ <syntax-vec>` の最大サイズを超えるコードは、すべて無効です。

.. note::
   :ref:`セクション <binary-section>` の場合と同様に、コードの :math:`\X{size}` もデコードで必須ではありませんが、バイナリ内を移動するときにセクションをスキップするのに利用できます。
   コードのサイズが、対応する関数コードの長さと一致しない場合、そのモジュールは無効となります。

.. index:: ! data section, data, memory, memory index, expression, byte
   pair: binary format; data
   pair: section; data
   single: memory; data
   single: data; segment
.. _binary-data:
.. _binary-datasec:

データセクション（data section）
~~~~~~~~~~~~

「データセクション」のidは11です。
このセクションは、1個の :ref:`モジュール <syntax-module>` における |MDATA| コンポーネントを表現する1個以上の :ref:`データセグメント <syntax-data>` のベクタ1個にデコードされます。

.. math::
   \begin{array}{llclll}
   \production{data section} & \Bdatasec &::=&
     \X{seg}^\ast{:}\Bsection_{11}(\Bvec(\Bdata)) &\Rightarrow& \X{seg} \\
   \production{data segment} & \Bdata &::=&
     x{:}\Bmemidx~~e{:}\Bexpr~~b^\ast{:}\Bvec(\Bbyte)
       &\Rightarrow& \{ \DMEM~x, \DOFFSET~e, \DINIT~b^\ast \} \\
   \end{array}


.. index:: module, section, type definition, function type, function, table, memory, global, element, data, start function, import, export, context, version
   pair: binary format; module
.. _binary-magic:
.. _binary-version:
.. _binary-module:

モジュール（module）
~~~~~~~

1個の :ref:`モジュール <syntax-module>` のエンコードの冒頭には、4バイトのマジックナンバー（文字列 :math:`\text{\backslash0asm}`）を含む1個のプリアンブル（preamble: 序文）と1個のバージョンフィールドが置かれます。
WebAssemblyバイナリフォーマットの現在のバージョンは1です。

このプリアンブルの後ろに、:ref:`セクション <binary-section>` のシーケンスが置かれます。
:ref:`カスタムセクション <binary-customsec>` はこのシーケンスの任意の場所に挿入される可能性がありますが、その他のセクションは重複が許されず、事前に決められた順序で最大1回までしか出現できません。
どのセクションも空にできます。

:ref:`関数セクション <binary-funcsec>` と :ref:`コードセクション <binary-codesec>` で生成されるベクタの長さは（空の可能性も含めて）互いに一致しなければなりません。

.. math::
   \begin{array}{llcllll}
   \production{magic} & \Bmagic &::=&
     \hex{00}~\hex{61}~\hex{73}~\hex{6D} \\
   \production{version} & \Bversion &::=&
     \hex{01}~\hex{00}~\hex{00}~\hex{00} \\
   \production{module} & \Bmodule &::=&
     \Bmagic \\ &&&
     \Bversion \\ &&&
     \Bcustomsec^\ast \\ &&&
     \functype^\ast{:\,}\Btypesec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \import^\ast{:\,}\Bimportsec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \typeidx^n{:\,}\Bfuncsec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \table^\ast{:\,}\Btablesec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \mem^\ast{:\,}\Bmemsec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \global^\ast{:\,}\Bglobalsec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \export^\ast{:\,}\Bexportsec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \start^?{:\,}\Bstartsec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \elem^\ast{:\,}\Belemsec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \X{code}^n{:\,}\Bcodesec \\ &&&
     \Bcustomsec^\ast \\ &&&
     \data^\ast{:\,}\Bdatasec \\ &&&
     \Bcustomsec^\ast
     \quad\Rightarrow\quad \{~
       \begin{array}[t]{@{}l@{}}
       \MTYPES~\functype^\ast, \\
       \MFUNCS~\func^n, \\
       \MTABLES~\table^\ast, \\
       \MMEMS~\mem^\ast, \\
       \MGLOBALS~\global^\ast, \\
       \MELEM~\elem^\ast, \\
       \MDATA~\data^\ast, \\
       \MSTART~\start^?, \\
       \MIMPORTS~\import^\ast, \\
       \MEXPORTS~\export^\ast ~\} \\
      \end{array} \\
   \end{array}

ただし :math:`\X{code}^n` 内にある個別の :math:`t_i^\ast, e_i` について以下となる。

.. math::
   \func^n[i] = \{ \FTYPE~\typeidx^n[i], \FLOCALS~t_i^\ast, \FBODY~e_i \} ) \\

.. note::
   WebAssemblyバイナリ形式は、今後の形式の後方互換性を変更しなければならない場合にバージョンを上げる可能性があります。
   ただし、こうした変更はたとえ発生したとしても非常にまれであると予測されます。
   WebAssemblyバイナリ形式は前方互換性を意図しているため、今後そのような拡張はバージョンを上げずに行われるでしょう。
