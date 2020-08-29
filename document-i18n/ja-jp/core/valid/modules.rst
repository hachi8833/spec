
モジュール
-------

:ref:`モジュール <syntax-module>` は、そこに含まれるコンポーネントがすべて有効な場合に有効になります。さらに、モジュールの定義のほとんどはそれ自体が適切な型で分類されています。

.. index:: function, local, function index, local index, type index, function type, value type, expression, import
   pair: abstract syntax; function
   single: abstract syntax; function
.. _valid-local:
.. _valid-func:

関数（function）
~~~~~~~~~

関数 :math:`\func` は :math:`[t_1^\ast] \to [t_2^\ast]` の形式を持つ :ref:`関数型<syntax-functype>` に分類されます。

:math:`\{ \FTYPE~x, \FLOCALS~t^\ast, \FBODY~\expr \}`
.....................................................

* 型 :math:`C.\CTYPES[x]` はそのコンテキスト内で定義されなければならない。

* :math:`[t_1^\ast] \to [t_2^\ast]` は :ref:`関数型<syntax-functype>` :math:`C.\CTYPES[x]` とすること。

* :math:`C'` の :ref:`コンテキスト <context>` は :math:`C` と同じだが、以下も加わる。

  * |CLOCALS| は :ref:`値型 <syntax-valtype>` :math:`t_1^\ast~t^\ast` のシーケンスに設定され、パラメーターやローカル値を結合する。

  * |CLABELS| は :ref:`結果型 <syntax-valtype>` :math:`[t_2^\ast]` だけを含む単独のシーケンスに設定される。

  * |CRETURN| は :ref:`結果型 <syntax-valtype>` :math:`[t_2^\ast]` に設定される。

* コンテキスト :math:`C'` において、式 :math:`\expr` は型 :math:`[t_2^\ast]` で有効でなければならない。

* これにより、この関数定義は型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     C.\CTYPES[x] = [t_1^\ast] \to [t_2^\ast]
     \qquad
     C,\CLOCALS\,t_1^\ast~t^\ast,\CLABELS~[t_2^\ast],\CRETURN~[t_2^\ast] \vdashexpr \expr : [t_2^\ast]
   }{
     C \vdashfunc \{ \FTYPE~x, \FLOCALS~t^\ast, \FBODY~\expr \} : [t_1^\ast] \to [t_2^\ast]
   }


.. index:: table, table type
   pair: validation; table
   single: abstract syntax; table
.. _valid-table:

テーブル（table）
~~~~~~

テーブル :math:`\table` は :ref:`テーブル型 <syntax-tabletype>` に分類されます。

:math:`\{ \TTYPE~\tabletype \}`
...............................

* :ref:`テーブル型 <syntax-tabletype>` :math:`\tabletype` は :ref:`有効 <valid-tabletype>` でなければならない。

* これにより、テーブル定義は型 :math:`\tabletype` で有効となる。

.. math::
   \frac{
     \vdashtabletype \tabletype \ok
   }{
     C \vdashtable \{ \TTYPE~\tabletype \} : \tabletype
   }


.. index:: memory, memory type
   pair: validation; memory
   single: abstract syntax; memory
.. _valid-mem:

メモリー（memory）
~~~~~~~~

メモリー :math:`\mem` は :ref:`メモリー型 <syntax-memtype>` に分類されます。

:math:`\{ \MTYPE~\memtype \}`
.............................

* :ref:`メモリー型 <syntax-memtype>` :math:`\memtype` は :ref:`有効 <valid-memtype>` でなければならない。

* これにより、メモリー定義は型 :math:`\memtype` で有効となる。

.. math::
   \frac{
     \vdashmemtype \memtype \ok
   }{
     C \vdashmem \{ \MTYPE~\memtype \} : \memtype
   }


.. index:: global, global type, expression
   pair: validation; global
   single: abstract syntax; global
.. _valid-global:

グローバル（global）
~~~~~~~

グローバル :math:`\global` は :math:`\mut~t` の形式を持つ :ref:`グローバル型 <syntax-globaltype>` に分類されます。

:math:`\{ \GTYPE~\mut~t, \GINIT~\expr \}`
.........................................

* :ref:`グローバル型 <syntax-globaltype>` :math:`\mut~t` は :ref:`有効 <valid-globaltype>` でなければならない。

* 式 :math:`\expr` は :ref:`結果型 <syntax-resulttype>` :math:`[t]` について :ref:`有効 <valid-expr>` でなければならない。

* 式 :math:`\expr` は :ref:`定数 <valid-constant>` でなければならない。

* これにより、グローバル定義は型 :math:`\mut~t` で有効となる。

.. math::
   \frac{
     \vdashglobaltype \mut~t \ok
     \qquad
     C \vdashexpr \expr : [t]
     \qquad
     C \vdashexprconst \expr \const
   }{
     C \vdashglobal \{ \GTYPE~\mut~t, \GINIT~\expr \} : \mut~t
   }


.. index:: element, table, table index, expression, function index
   pair: validation; element
   single: abstract syntax; element
   single: table; element
   single: element; segment
.. _valid-elem:

要素セグメント（element segment）
~~~~~~~~~~~~~~~~

要素セグメント :math:`\elem` は型に分類されません。

:math:`\{ \ETABLE~x, \EOFFSET~\expr, \EINIT~y^\ast \}`
......................................................

* テーブル :math:`C.\CTABLES[x]` はそのコンテキスト内で定義されなければならない。

* :math:`\limits~\elemtype` は :ref:`テーブル型 <syntax-tabletype>` :math:`C.\CTABLES[x]` とすること。

* :ref:`要素型 <syntax-elemtype>` :math:`\elemtype` は |FUNCREF| でなければならない。

* 式 :math:`\expr` は :ref:`結果型 <syntax-resulttype>` :math:`[\I32]` について :ref:`有効 <valid-expr>` でなければならない。

* 式 :math:`\expr` は :ref:`定数 <valid-constant>` でなければならない。

* :math:`y^\ast` にある個別の :math:`y_i` について、関数 :math:`C.\CFUNCS[y]` がそのコンテキスト内で定義されていなければならない。

* これにより、要素セグメントが有効となる。

.. math::
   \frac{
     C.\CTABLES[x] = \limits~\FUNCREF
     \qquad
     C \vdashexpr \expr : [\I32]
     \qquad
     C \vdashexprconst \expr \const
     \qquad
     (C.\CFUNCS[y] = \functype)^\ast
   }{
     C \vdashelem \{ \ETABLE~x, \EOFFSET~\expr, \EINIT~y^\ast \} \ok
   }


.. index:: data, memory, memory index, expression, byte
   pair: validation; data
   single: abstract syntax; data
   single: memory; data
   single: data; segment
.. _valid-data:

データセグメント（data segment）
~~~~~~~~~~~~~

データセグメント :math:`\data` は型に分類されません。

:math:`\{ \DMEM~x, \DOFFSET~\expr, \DINIT~b^\ast \}`
....................................................

* メモリー :math:`C.\CMEMS[x]` はそのコンテキスト内で定義されなければならない。

* 式 :math:`\expr` は :ref:`結果型 <syntax-resulttype>` :math:`[\I32]` について :ref:`有効 <valid-expr>` でなければならない。

* 式 :math:`\expr` は :ref:`定数 <valid-constant>` でなければならない。

* これにより、データセグメントが有効となる。

.. math::
   \frac{
     C.\CMEMS[x] = \limits
     \qquad
     C \vdashexpr \expr : [\I32]
     \qquad
     C \vdashexprconst \expr \const
   }{
     C \vdashdata \{ \DMEM~x, \DOFFSET~\expr, \DINIT~b^\ast \} \ok
   }


.. index:: start function, function index
   pair: validation; start function
   single: abstract syntax; start function
.. _valid-start:

開始関数（start function）
~~~~~~~~~~~~~~

開始関数宣言 :math:`\start` はどの型にも分類されません。

:math:`\{ \SFUNC~x \}`
......................

* 関数 :math:`C.\CFUNCS[x]` はそのコンテキスト内で定義されなければならない。

* :math:`C.\CFUNCS[x]` の型は :math:`[] \to []` でなければならない。

* これにより、開始関数が有効となる。

.. math::
   \frac{
     C.\CFUNCS[x] = [] \to []
   }{
     C \vdashstart \{ \SFUNC~x \} \ok
   }


.. index:: export, name, index, function index, table index, memory index, global index
   pair: validation; export
   single: abstract syntax; export
.. _valid-exportdesc:
.. _valid-export:

エクスポート（export）
~~~~~~~

エクスポート :math:`\export` およびエクスポートのデスクリプション :math:`\exportdesc` はそれらの :ref:`外部型 <syntax-externtype>` に分類されます。


:math:`\{ \ENAME~\name, \EDESC~\exportdesc \}`
..............................................

* エクスポートのデスクリプション :math:`\exportdesc` は :ref:`外部型 <syntax-externtype>` :math:`\externtype` について有効でなければならない。

* これにより、エクスポートが :ref:`外部型 <syntax-externtype>` :math:`\externtype` で有効となる。

.. math::
   \frac{
     C \vdashexportdesc \exportdesc : \externtype
   }{
     C \vdashexport \{ \ENAME~\name, \EDESC~\exportdesc \} : \externtype
   }


:math:`\EDFUNC~x`
.................

* 関数 :math:`C.\CFUNCS[x]` はそのコンテキスト内で定義されなければならない。

* これにより、エクスポートのデスクリプションは :ref:`外部型 <syntax-externtype>` :math:`\ETFUNC~C.\CFUNCS[x]` で有効となる。

.. math::
   \frac{
     C.\CFUNCS[x] = \functype
   }{
     C \vdashexportdesc \EDFUNC~x : \ETFUNC~\functype
   }


:math:`\EDTABLE~x`
..................

* テーブル :math:`C.\CTABLES[x]` はそのコンテキスト内で定義されなければならない。

* これにより、エクスポートのデスクリプションは :ref:`外部型 <syntax-externtype>` :math:`\ETTABLE~C.\CTABLES[x]` で有効となる。

.. math::
   \frac{
     C.\CTABLES[x] = \tabletype
   }{
     C \vdashexportdesc \EDTABLE~x : \ETTABLE~\tabletype
   }


:math:`\EDMEM~x`
................

* メモリー :math:`C.\CMEMS[x]` はそのコンテキスト内で定義されなければならない。

* これにより、エクスポートのデスクリプションは :ref:`外部型 <syntax-externtype>` :math:`\ETMEM~C.\CMEMS[x]` で有効となる。

.. math::
   \frac{
     C.\CMEMS[x] = \memtype
   }{
     C \vdashexportdesc \EDMEM~x : \ETMEM~\memtype
   }


:math:`\EDGLOBAL~x`
...................

* グローバル :math:`C.\CGLOBALS[x]` はそのコンテキスト内で定義されなければならない。

* これにより、エクスポートのデスクリプションは :ref:`外部型 <syntax-externtype>` :math:`\ETGLOBAL~C.\CGLOBALS[x]` で有効となる。

.. math::
   \frac{
     C.\CGLOBALS[x] = \globaltype
   }{
     C \vdashexportdesc \EDGLOBAL~x : \ETGLOBAL~\globaltype
   }


.. index:: import, name, function type, table type, memory type, global type
   pair: validation; import
   single: abstract syntax; import
.. _valid-importdesc:
.. _valid-import:

インポート（import）
~~~~~~~

インポート :math:`\import` およびインポートのデスクリプション :math:`\importdesc` は :ref:`外部型 <syntax-externtype>` に分類されます。

:math:`\{ \IMODULE~\name_1, \INAME~\name_2, \IDESC~\importdesc \}`
..................................................................

* インポートのデスクリプション :math:`\importdesc` は型 :math:`\externtype` について有効でなければならない。

* これにより、インポートが型 :math:`\externtype` で有効となる。

.. math::
   \frac{
     C \vdashimportdesc \importdesc : \externtype
   }{
     C \vdashimport \{ \IMODULE~\name_1, \INAME~\name_2, \IDESC~\importdesc \} : \externtype
   }


:math:`\IDFUNC~x`
.................

* 関数 :math:`C.\CTYPES[x]` はそのコンテキスト内で定義されなければならない。

* :math:`[t_1^\ast] \to [t_2^\ast]` は :ref:`関数型<syntax-functype>` :math:`C.\CTYPES[x]` とすること。

* これにより、インポートのデスクリプションが型 :math:`\ETFUNC~[t_1^\ast] \to [t_2^\ast]` で有効になる。

.. math::
   \frac{
     C.\CTYPES[x] = [t_1^\ast] \to [t_2^\ast]
   }{
     C \vdashimportdesc \IDFUNC~x : \ETFUNC~[t_1^\ast] \to [t_2^\ast]
   }


:math:`\IDTABLE~\tabletype`
...........................

* テーブル型 :math:`\tabletype` は :ref:`有効 <valid-tabletype>` でなければならない。

* これにより、インポートのデスクリプションが型 :math:`\ETTABLE~\tabletype` で有効になる。

.. math::
   \frac{
     \vdashtable \tabletype \ok
   }{
     C \vdashimportdesc \IDTABLE~\tabletype : \ETTABLE~\tabletype
   }


:math:`\IDMEM~\memtype`
.......................

* メモリー型 :math:`\memtype` は :ref:`有効 <valid-memtype>` でなければならない。

* これにより、インポートのデスクリプションが型 :math:`\ETMEM~\memtype` で有効になる。

.. math::
   \frac{
     \vdashmemtype \memtype \ok
   }{
     C \vdashimportdesc \IDMEM~\memtype : \ETMEM~\memtype
   }


:math:`\IDGLOBAL~\globaltype`
.............................

* グローバル型 :math:`\globaltype` は :ref:`有効 <valid-globaltype>` でなければならない。

* これにより、インポートのデスクリプションが型 :math:`\ETGLOBAL~\globaltype` で有効になる。

.. math::
   \frac{
     \vdashglobaltype \globaltype \ok
   }{
     C \vdashimportdesc \IDGLOBAL~\globaltype : \ETGLOBAL~\globaltype
   }


.. index:: module, type definition, function type, function, table, memory, global, element, data, start function, import, export, context
   pair: validation; module
   single: abstract syntax; module
.. _valid-module:

モジュール（module）
~~~~~~~

モジュールは、そのモジュールの :ref:`インポート <syntax-import>` の :ref:`外部型 <syntax-externtype>` から、そのモジュールの :ref:`エクスポート <syntax-export>` の :ref:`外部型 <syntax-externtype>` への対応付けによって分類されます。

ひとつのモジュールは、全体が「閉じています」。すなわち、モジュールを構成するコンポーネントは、そのモジュール自身の内部に出現する定義しか参照できません。
これによって、初期の :ref:`コンテキスト <context>` が必要ではなくなりました。
その代わりに、そのモジュールの内容を検証するコンテキスト :math:`C` は、そのモジュール内の定義から構成されます。

* 検証の対象となるモジュールを :math:`\module` とする。

* :math:`C` をある :ref:`コンテキスト <context>` とする。ただし、

  * :math:`C.\CTYPES` は :math:`\module.\MTYPES` である。

  * :math:`C.\CFUNCS` は :math:`\etfuncs(\X{it}^\ast)` である。これは :math:`\X{ft}^\ast` と、インポートの :ref:`外部型 <syntax-externtype>` :math:`\X{it}^\ast` と内部の :ref:`関数型<syntax-functype>` :math:`\X{ft}^\ast` を以下で決定されるとおりに結合したものである。

  * :math:`C.\CTABLES` は :math:`\ettables(\X{it}^\ast)` である。これは :math:`\X{tt}^\ast` と、インポートの :ref:`外部型 <syntax-externtype>` :math:`\X{it}^\ast` と内部の :ref:`テーブル型 <syntax-tabletype>` :math:`\X{tt}^\ast` を以下で決定されるとおりに結合したものである。

  * :math:`C.\CMEMS` は :math:`\etmems(\X{it}^\ast)` である。これは :math:`\X{mt}^\ast` と、インポートの :ref:`外部型 <syntax-externtype>` :math:`\X{it}^\ast` と内部の :ref:`メモリー型 <syntax-memtype>` :math:`\X{mt}^\ast` を以下で決定されるとおりに結合したものである。

  * :math:`C.\CGLOBALS` は :math:`\etglobals(\X{it}^\ast)` である。これは :math:`\X{gt}^\ast` と、インポートの :ref:`外部型 <syntax-externtype>` :math:`\X{it}^\ast` と内部の :ref:`グローバル型 <syntax-globaltype>` :math:`\X{gt}^\ast` を以下で決定されるとおりに結合したものである。

  * :math:`C.\CLOCALS` は空である。

  * :math:`C.\CLABELS` は空である。

  * :math:`C.\CRETURN` は空である。

* :math:`C'` を :ref:`コンテキスト <context>` とする。ただし :math:`C'.\CGLOBALS` はシーケンス :math:`\etglobals(\X{it}^\ast)` であり、その他のフィールドはすべて空である。

* コンテキスト :math:`C` において以下のようになる。

  * :math:`\module.\MTYPES` の中にある個別の :math:`\functype_i` について、:ref:`関数型<syntax-functype>` :math:`\functype_i` が :ref:`有効 <valid-functype>` でなければならない。

  * :math:`\module.\MFUNCS` の中にある個別の :math:`\func_i` について、定義 :math:`\func_i` が :ref:`関数型<syntax-functype>` :math:`\X{ft}_i` で :ref:`有効 <valid-func>` でなければならない。

  * :math:`\module.\MTABLES` の中にある個別の :math:`\table_i` について、定義 :math:`\table_i` が :ref:`テーブル型 <syntax-tabletype>` :math:`\X{tt}_i` で :ref:`有効 <valid-func>` でなければならない。

  * :math:`\module.\MMEMS` の中にある個別の :math:`\mem_i` について、定義 :math:`\mem_i` が :ref:`メモリー型 <syntax-memtype>` :math:`\X{mt}_i` で :ref:`有効 <valid-func>` でなければならない。

  * :math:`\module.\MGLOBALS` の中にある個別の :math:`\global_i` について、

    * コンテキスト :math:`C'` において、定義 :math:`\global_i` が :ref:`グローバル型 <syntax-globaltype>` :math:`\X{gt}_i` で :ref:`有効 <valid-func>` でなければならない。

  * :math:`\module.\MELEM` の中にある個別の :math:`\elem_i` について、セグメント :math:`\elem_i` が :ref:`有効 <valid-elem>` でなければならない。

  * :math:`\module.\MDATA` の中にある個別の :math:`\data_i` について、セグメント :math:`\data_i` が :ref:`有効 <valid-elem>` でなければならない。

  * :math:`\module.\MSTART` が空でない場合、:math:`\module.\MSTART` は :ref:`有効 <valid-start>` でなければならない。

  * :math:`\module.\MIMPORTS` の中にある個別の :math:`\import_i` について、セグメント :math:`\import_i` が :ref:`外部型 <syntax-externtype>` :math:`\X{it}_i` で :ref:`有効 <valid-import>` でなければならない.

  * :math:`\module.\MEXPORTS` の中にある個別の :math:`\export_i` について、セグメント :math:`\export_i` が :ref:`外部型 <syntax-externtype>` :math:`\X{et}_i` で :ref:`有効 <valid-import>` でなければならない.

* :math:`C.\CTABLES` の長さは :math:`1` より大きくなってはならない。

* :math:`C.\CMEMS` の長さは :math:`1` より大きくなってはならない。

* エクスポートされる名前 :math:`\export_i.\ENAME` はすべて異なっていなければならない。

* :math:`\X{ft}^\ast` は内部の :ref:`関数型<syntax-functype>` :math:`\X{ft}_i` をインデックス順に結合したものとする。

* :math:`\X{tt}^\ast` は内部の :ref:`テーブル型 <syntax-tabletype>` :math:`\X{tt}_i` をインデックス順に結合したものとする。

* :math:`\X{mt}^\ast` は内部の :ref:`メモリー型 <syntax-memtype>` :math:`\X{mt}_i` をインデックス順に結合したものとする。

* :math:`\X{gt}^\ast` は内部の :ref:`グローバル型 <syntax-globaltype>` :math:`\X{gt}_i` をインデックス順に結合したものとする。

* :math:`\X{it}^\ast` はインポートの :ref:`外部型 <syntax-externtype>` :math:`\X{it}_i` をインデックス順に結合したものとする。

* :math:`\X{et}^\ast` はエクスポートの :ref:`外部型 <syntax-externtype>` :math:`\X{et}_i` をインデックス順に結合したものとする。

* これにより、モジュールは :ref:`外部型 <syntax-externtype>` :math:`\X{it}^\ast \to \X{et}^\ast` で有効となる。

.. math::
   \frac{
     \begin{array}{@{}c@{}}
     (\vdashfunctype \functype \ok)^\ast
     \quad
     (C \vdashfunc \func : \X{ft})^\ast
     \quad
     (C \vdashtable \table : \X{tt})^\ast
     \quad
     (C \vdashmem \mem : \X{mt})^\ast
     \quad
     (C' \vdashglobal \global : \X{gt})^\ast
     \\
     (C \vdashelem \elem \ok)^\ast
     \quad
     (C \vdashdata \data \ok)^\ast
     \quad
     (C \vdashstart \start \ok)^?
     \quad
     (C \vdashimport \import : \X{it})^\ast
     \quad
     (C \vdashexport \export : \X{et})^\ast
     \\
     \X{ift}^\ast = \etfuncs(\X{it}^\ast)
     \qquad
     \X{itt}^\ast = \ettables(\X{it}^\ast)
     \qquad
     \X{imt}^\ast = \etmems(\X{it}^\ast)
     \qquad
     \X{igt}^\ast = \etglobals(\X{it}^\ast)
     \\
     C = \{ \CTYPES~\functype^\ast, \CFUNCS~\X{ift}^\ast~\X{ft}^\ast, \CTABLES~\X{itt}^\ast~\X{tt}^\ast, \CMEMS~\X{imt}^\ast~\X{mt}^\ast, \CGLOBALS~\X{igt}^\ast~\X{gt}^\ast \}
     \\
     C' = \{ \CGLOBALS~\X{igt}^\ast \}
     \qquad
     |C.\CTABLES| \leq 1
     \qquad
     |C.\CMEMS| \leq 1
     \qquad
     (\export.\ENAME)^\ast ~\F{disjoint}
     \end{array}
   }{
     \vdashmodule \{
       \begin{array}[t]{@{}l@{}}
         \MTYPES~\functype^\ast,
         \MFUNCS~\func^\ast,
         \MTABLES~\table^\ast,
         \MMEMS~\mem^\ast,
         \MGLOBALS~\global^\ast, \\
         \MELEM~\elem^\ast,
         \MDATA~\data^\ast,
         \MSTART~\start^?,
         \MIMPORTS~\import^\ast,
         \MEXPORTS~\export^\ast \} : \X{it}^\ast \to \X{et}^\ast \\
       \end{array}
   }

.. note::
   モジュール内にあるほとんどの定義（特に関数）は互いに再帰的になっています。
   その結果、このルールにある :ref:`コンテキスト <context>` :math:`C` の定義もまた再帰的になっています。
   これについては、そのモジュールに含まれる「関数」「テーブル」「メモリー」「グローバル定義」を検証した結果によって変わりますが、それ自体は :math:`C` に依存します。
   ただし、この再帰は定義のための装置に過ぎません。
   :math:`C` を構成するのに必要なあらゆる型は、モジュールに対するシンプルな事前パス（ここでは実際の検証は全く行われません）だけで簡単に決定できます。

   しかしながら、グローバル値は再帰的ではありません。
   モジュールのグローバル値を検証するための限定されたコンテキスト :math:`C'` を定義することの効用は、グローバル値の初期化式からはインポートしたグローバル値にしかアクセスできなくなることです。

.. note::
   テーブル数やメモリー数の制約については、WebAssemblyの今後のバージョンで解除される可能性があります。
