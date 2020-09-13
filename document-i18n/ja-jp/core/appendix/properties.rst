.. index:: ! soundness, type system
.. _soundness:

健全性
---------

WebAssemblyの :ref:`型システム <type-system>` は「健全（sound）」です。すなわちWebAssemblyセマンティクスにおいて「型安全性（type safety）」および「メモリ安全性（memory safety）」であることが暗に示されます。以下に例を示します。

* 宣言されたあらゆる型、および検証フェーズで派生した型は、実行時に尊重されます。
  例:「あらゆる個別の :ref:`ローカル <syntax-local>` 変数や :ref:`グローバル <syntax-global>` 変数は正しい型の値だけを含む」「あらゆる個別の :ref:`インストラクション <syntax-instr>` は期待された型を持つオペランドに対してのみ適用される」「あらゆる個別の :ref:`関数 <syntax-func>` :ref:`呼び出し <exec-invocation>` は、正しい型の結果を常に評価する（:ref:`トラップ <trap>` または乖離（diverge）しない場合）」

* プログラムで明示的に定義されない限り、いかなるメモリー位置に対する読み書きも行われない。
  例:「:ref:`ローカル <syntax-local>`」「:ref:`グローバル <syntax-global`」「:ref:`テーブル <syntax-table>` の要素」または「線形 :ref:`メモリー <syntax-mem>` 内の位置」

* 未定義の動作が存在しない。
  すなわち、:ref:`実行ルール <exec>` は、:ref:`有効な <valid>` プログラム内で発生する可能性のあるあらゆるケースをカバーし、かつそれらのルールが相互一貫する。

健全性は、特に関数の「カプセル化（encapsulation）」やモジュールのスコープといった追加の特性を確保するのにも役立ちます。いかなる :ref:`ローカル <syntax-local>` も、その関数の外部にはアクセスできず、（明示的に :ref:`エクスポート <syntax-export>` または :ref:`インポート <syntax-import>` されない限り）いかなる :ref:`モジュール <syntax-module>` コンポーネントもそのモジュール外にはアクセスできません。

WebAssemblyの :ref:`検証 <valid>` を定義する型付けルールは、WebAssemblyプログラムの「静的な」コンポーネントだけをカバーします。
健全性を精密に記述および証明するために、型付けルールは抽象 :ref:`ランタイム <syntax-runtime>` の「動的な」コンポーネント（すなわち「:ref:`ストア <syntax-store>`」「:ref:`設定 <syntax-config>`」「:ref:`管理インストラクション <syntax-instr-admin>`」）に拡張されなければなりません。 [#cite-pldi2017]_


.. index:: value, value type, result, result type, trap
.. _valid-val:
.. _valid-result:

値と結果
~~~~~~~~~~~~~~~~~~

以下のように、:ref:`値 <syntax-val>` は :ref:`値型 <syntax-valtype>` に分類され、:ref:`結果 <syntax-result>` は :ref:`結果型 <syntax-resulttype>` に分類できます。

:ref:`値 <syntax-val>` :math:`t.\CONST~c`
.............................................

* 値は :ref:`値型 <syntax-valtype>` :math:`t` で有効である。

.. math::
   \frac{
   }{
     \vdashval t.\CONST~c : t
   }


:ref:`結果 <syntax-result>` :math:`\val^\ast`
................................................

* :math:`\val^\ast` 内の個別の :ref:`値 <syntax-val>` :math:`\val_i` について以下が成り立つ。

  * 値 :math:`\val_i` はいくつかの :ref:`値型 <syntax-valtype>` :math:`t_i` で :ref:`有効 <valid-val>` である。

* :math:`t^\ast` をすべての :math:`t_i` の結合とする。

* それによって、結果は :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` で有効である。

.. math::
   \frac{
     (\vdashval \val : t)^\ast
   }{
     \vdashresult \val^\ast : [t^\ast]
   }


:ref:`結果 <syntax-result>` :math:`\TRAP`
............................................

* :ref:`値型 <syntax-valtype>` :math:`t^\ast` の任意のシーケンスにおいて、結果が :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` で有効である。

.. math::
   \frac{
   }{
     \vdashresult \TRAP : [t^\ast]
   }


.. _module-context:
.. _valid-store:

ストアの有効性
~~~~~~~~~~~~~~

以下の型付けルールは、あるランタイム :ref:`ストア <syntax-store>` :math:`S` が「有効となる」場合を記述します。
ひとつの有効なストアは「:ref:`関数 <syntax-funcinst>`」「:ref:`テーブル <syntax-tableinst>`」「:ref:`メモリー <syntax-meminst>`」「:ref:`グローバル <syntax-globalinst>`」「それ自身が（:math:`S` に相対的に）有効な :ref:`モジュール <syntax-moduleinst>`」から構成されなければなりません。

そのために、インスタンスの個別の種類は、その種類に対応する「:ref:`関数型 <syntax-functype>`」「 :ref:`テーブル型 <syntax-tabletype>`」「:ref:`メモリー型 <syntax-memtype>`」「:ref:`グローバル型 <syntax-globaltype>`」に分類されます。
モジュールインスタンスは「モジュールコンテキスト」に分類されます。これは通常の :ref:`コンテキスト <context>` の目的を変えて、あるモジュール内で定義される :ref:`インデックス空間 <syntax-index>` を記述するモジュール型としたものです。

.. index:: store, function instance, table instance, memory instance, global instance, function type, table type, memory type, global type

:ref:`ストア <syntax-store>` :math:`S`
.....................................

* :math:`S.\SFUNCS` 内の個別の :ref:`関数インスタンス <syntax-funcinst>` :math:`\funcinst_i` は、いくつかの :ref:`関数型 <syntax-functype>` :math:`\functype_i` で :ref:`有効 <valid-funcinst>` でなければならない。

* :math:`S.\STABLES` 内の個別の :ref:`テーブルインスタンス <syntax-tableinst>` :math:`\tableinst_i` は、いくつかの :ref:`テーブル型 <syntax-tabletype>` :math:`\tabletype_i` で :ref:`有効 <valid-tableinst>` でなければならない。

* :math:`S.\SMEMS` 内の個別の :ref:`メモリーインスタンス <syntax-meminst>` :math:`\meminst_i` は、いくつかの :ref:`メモリー型 <syntax-memtype>` :math:`\memtype_i` で :ref:`有効 <valid-meminst>` でなければならない。

* :math:`S.\SGLOBALS` 内の個別の :ref:`グローバルインスタンス <syntax-globalinst>` :math:`\globalinst_i` は、いくつかの :ref:`グローバル型 <syntax-globaltype>` :math:`\globaltype_i` で :ref:`有効 <valid-globalinst>` でなければならない。

* 以上を満たすことでストアは有効となる。

.. math::
   ~\\[-1ex]
   \frac{
     \begin{array}{@{}c@{}}
     (S \vdashfuncinst \funcinst : \functype)^\ast
     \qquad
     (S \vdashtableinst \tableinst : \tabletype)^\ast
     \\
     (S \vdashmeminst \meminst : \memtype)^\ast
     \qquad
     (S \vdashglobalinst \globalinst : \globaltype)^\ast
     \\
     S = \{
       \SFUNCS~\funcinst^\ast,
       \STABLES~\tableinst^\ast,
       \SMEMS~\meminst^\ast,
       \SGLOBALS~\globalinst^\ast \}
     \end{array}
   }{
     \vdashstore S \ok
   }


.. index:: function type, function instance
.. _valid-funcinst:

:ref:`関数インスタンス <syntax-funcinst>` :math:`\{\FITYPE~\functype, \FIMODULE~\moduleinst, \FICODE~\func\}`
.......................................................................................................................

* :ref:`関数型 <syntax-functype>` :math:`\functype` は :ref:`有効 <valid-functype>` でなければならない。

* :ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\moduleinst` は、いくつかの :ref:`コンテキスト <context>` :math:`C` で :ref:`有効 <valid-moduleinst>` でなければならない。

* :ref:`コンテキスト <context>` :math:`C` において、the :ref:`関数 <syntax-func>` :math:`\func` は :ref:`関数型 <syntax-functype>` :math:`\functype` で :ref:`有効 <valid-func>` でなければならない。

* 以上を満たすことで、関数インスタンスは :ref:`関数型 <syntax-functype>` :math:`\functype` で有効となる。

.. math::
   \frac{
     \vdashfunctype \functype \ok
     \qquad
     S \vdashmoduleinst \moduleinst : C
     \qquad
     C \vdashfunc \func : \functype
   }{
     S \vdashfuncinst \{\FITYPE~\functype, \FIMODULE~\moduleinst, \FICODE~\func\} : \functype
   }


.. index:: function type, function instance, host function
.. _valid-hostfuncinst:

:ref:`ホスト関数インスタンス <syntax-funcinst>` :math:`\{\FITYPE~\functype, \FIHOSTCODE~\X{hf}\}`
..................................................................................................

* :ref:`関数型 <syntax-functype>` :math:`\functype` は :ref:`有効 <valid-functype>` でなければならない。

* :math:`[t_1^\ast] \to [t_2^\ast]` を :ref:`関数型 <syntax-functype>` :math:`\functype` とする。

* :math:`S` を :ref:`拡張する <extend-store>` 個別の :ref:`有効 <valid-store>` な :ref:`ストア <syntax-store>` :math:`S_1`、および :ref:`型 <valid-val>` が :math:`t_1^\ast` と一致する :ref:`型 <syntax-val>` のあらゆる個別のシーケンス :math:`\val^\ast` で以下が成立する。

  * ストア :math:`S_1` で :math:`\X{hf}` を引数 :math:`\val^\ast` で :ref:`実行 <exec-invoke-host>` すると、空でない可能な結果セットを持つ。

  * その結果セットの個別の要素 :math:`R` について以下のいずれかが成立しなければならない。

    * :math:`R` が :math:`\bot` である（すなわち乖離（divergence））。

    * :math:`R` は、:math:`S_1` を :ref:`拡張する <extend-store>` :ref:`有効 <valid-store>` な :ref:`ストア <syntax-store>` :math:`S_2`、およびその :ref:`型 <valid-result>` が :math:`[t_2^\ast]` と一致する :ref:`結果 <syntax-result>` :math:`\result` で構成される。

* 以上が満たされることで、この関数インスタンスは :ref:`関数型 <syntax-functype>` :math:`\functype` で有効となる。

.. math::
   \frac{
     \begin{array}[b]{@{}l@{}}
     \vdashfunctype [t_1^\ast] \to [t_2^\ast] \ok \\
     \end{array}
     \quad
     \begin{array}[b]{@{}l@{}}
     \forall S_1, \val^\ast,~
       {\vdashstore S_1 \ok} \wedge
       {\vdashstoreextends S \extendsto S_1} \wedge
       {\vdashresult \val^\ast : [t_1^\ast]}
       \Longrightarrow {} \\ \qquad
       \X{hf}(S_1; \val^\ast) \supset \emptyset \wedge {} \\ \qquad
     \forall R \in \X{hf}(S_1; \val^\ast),~
       R = \bot \vee {} \\ \qquad\qquad
       \exists S_2, \result,~
       {\vdashstore S_2 \ok} \wedge
       {\vdashstoreextends S_1 \extendsto S_2} \wedge
       {\vdashresult \result : [t_2^\ast]} \wedge
       R = (S_2; \result)
     \end{array}
   }{
     S \vdashfuncinst \{\FITYPE~[t_1^\ast] \to [t_2^\ast], \FIHOSTCODE~\X{hf}\} : [t_1^\ast] \to [t_2^\ast]
   }

.. note::
   このルールは、ストアと引数に関する適切な事前条件が満たされる場合、ホスト関数の実行はストアと結果に関する適切な事後条件を満たさなければならないということを記述しています。この事後条件は、ホスト関数呼び出しのための :ref:`実行ルール <exec-invoke-host>` のひとつと一致します。

   この関数が呼び出されるどのストアでも、現在のストアの拡張が仮定されます。
   そうすることで、この関数自身が今後のストアにおける仮定を満たせるようになります。

.. index:: table type, table instance, limits, function address
.. _valid-tableinst:

:ref:`テーブルインスタンス <syntax-tableinst>` :math:`\{ \TIELEM~(\X{fa}^?)^n, \TIMAX~m^? \}`
..............................................................................................

* テーブル要素 :math:`(\X{fa}^?)^n` 内にある個別のオプショナルな :ref:`関数アドレス <syntax-funcaddr>` :math:`\X{fa}^?_i` は、以下のいずれかである。

  * :math:`\X{fa}^?_i` が空である。

  * the :ref:`外部値 <syntax-externval>` :math:`\EVFUNC~\X{fa}` は、いくつかの :ref:`外部値 <syntax-externtype>` :math:`\ETFUNC~\X{ft}` で :ref:`有効 <valid-externval-func>` でなければならない。

* :ref:`制限 <syntax-limits>` :math:`\{\LMIN~n, \LMAX~m^?\}` は、範囲 :math:`2^{32}` で :ref:`有効 <valid-limits>` でなければならない。

* 以上を満たすことで、テーブルインスタンスは :ref:`テーブル型 <syntax-tabletype>` :math:`\{\LMIN~n, \LMAX~m^?\}~\FUNCREF` で有効となる。

.. math::
   \frac{
     ((S \vdash \EVFUNC~\X{fa} : \ETFUNC~\functype)^?)^n
     \qquad
     \vdashlimits \{\LMIN~n, \LMAX~m^?\} : 2^{32}
   }{
     S \vdashtableinst \{ \TIELEM~(\X{fa}^?)^n, \TIMAX~m^? \} : \{\LMIN~n, \LMAX~m^?\}~\FUNCREF
   }


.. index:: memory type, memory instance, limits, byte
.. _valid-meminst:

:ref:`メモリーインスタンス <syntax-meminst>` :math:`\{ \MIDATA~b^n, \MIMAX~m^? \}`
..............................................................................

* :ref:`制限 <syntax-limits>` :math:`\{\LMIN~n, \LMAX~m^?\}` は範囲 :math:`2^{16}` で :ref:`有効 <valid-limits>` でなければならない。

* これを満たすことで、メモリーインスタンスは :ref:`メモリー型 <syntax-memtype>` :math:`\{\LMIN~n, \LMAX~m^?\}` で有効となる。

.. math::
   \frac{
     \vdashlimits \{\LMIN~n, \LMAX~m^?\} : 2^{16}
   }{
     S \vdashmeminst \{ \MIDATA~b^n, \MIMAX~m^? \} : \{\LMIN~n, \LMAX~m^?\}
   }


.. index:: global type, global instance, value, mutability
.. _valid-globalinst:

:ref:`グローバルインスタンス <syntax-globalinst>` :math:`\{ \GIVALUE~(t.\CONST~c), \GIMUT~\mut \}`
............................................................................................

* グローバルインスタンスは :ref:`グローバル型 <syntax-globaltype>` :math:`\mut~t` で有効となる。

.. math::
   \frac{
   }{
     S \vdashglobalinst \{ \GIVALUE~(t.\CONST~c), \GIMUT~\mut \} : \mut~t
   }


.. index:: external type, export instance, name, external value
.. _valid-exportinst:

:ref:`エクスポートインスタンス <syntax-exportinst>` :math:`\{ \EINAME~\name, \EIVALUE~\externval \}`
.......................................................................................................

* :ref:`外部値 <syntax-externval>` :math:`\externval` は、いくつかの :ref:`外部値 <syntax-externtype>` :math:`\externtype` で :ref:`有効 <valid-externval>` でなければならない。

* これを満たすことで、エクスポートインスタンスは有効となる。

.. math::
   \frac{
     S \vdashexternval \externval : \externtype
   }{
     S \vdashexportinst \{ \EINAME~\name, \EIVALUE~\externval \} \ok
   }


.. index:: module instance, context
.. _valid-moduleinst:

:ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\moduleinst`
...............................................................

* :math:`\moduleinst.\MITYPES` 内にある個別の :ref:`関数型 <syntax-functype>` :math:`\functype_i` は :ref:`有効 <valid-functype>` でなければならない。

* :math:`\moduleinst.\MIFUNCS` 内にある個別の :ref:`関数アドレス <syntax-funcaddr>` :math:`\funcaddr_i` について、:ref:`外部値 <syntax-externval>` :math:`\EVFUNC~\funcaddr_i` がいくつかの :ref:`外部値 <syntax-externtype>` :math:`\ETFUNC~\functype'_i` で :ref:`有効 <valid-externval-func>` でなければならない。

* :math:`\moduleinst.\MITABLES` 内にある個別の :ref:`テーブルアドレス <syntax-tableaddr>` :math:`\tableaddr_i` について、:ref:`外部値 <syntax-externval>` :math:`\EVTABLE~\tableaddr_i` がいくつかの :ref:`外部値 <syntax-externtype>` :math:`\ETTABLE~\tabletype_i` で :ref:`有効 <valid-externval-table>` でなければならない。

* :math:`\moduleinst.\MIMEMS` 内にある個別の :ref:`メモリーアドレス <syntax-memaddr>` :math:`\memaddr_i` について、:ref:`外部値 <syntax-externval>` :math:`\EVMEM~\memaddr_i` がいくつかの :ref:`外部値 <syntax-externtype>` :math:`\ETMEM~\memtype_i` で :ref:`有効 <valid-externval-mem>` でなければならない。

* :math:`\moduleinst.\MIGLOBALS` 内にある個別の :ref:`グローバルアドレス <syntax-globaladdr>` :math:`\globaladdr_i` について、:ref:`外部値 <syntax-externval>` :math:`\EVGLOBAL~\globaladdr_i` がいくつかの :ref:`外部値 <syntax-externtype>` :math:`\ETGLOBAL~\globaltype_i` で :ref:`有効 <valid-externval-global>` でなければならない。

* :math:`\moduleinst.\MIEXPORTS` 内にある個別の :ref:`エクスポートインスタンス <syntax-exportinst>` :math:`\exportinst_i` が :ref:`有効 <valid-exportinst>` でなければならない。

* :math:`\moduleinst.\MIEXPORTS` 内にある個別の :ref:`エクスポートインスタンス <syntax-exportinst>` :math:`\exportinst_i` について、:ref:`名前 <syntax-name>` :math:`\exportinst_i.\EINAME` が :math:`\moduleinst.\MIEXPORTS` 内で発生する他のどの名前とも重複してはならない。

* :math:`{\functype'}^\ast` を、すべての :math:`\functype'_i` を順序を維持して結合したものとする。

* :math:`\tabletype^\ast` を、すべての :math:`\tabletype_i` を順序を維持して結合したものとする。

* :math:`\memtype^\ast` を、すべての :math:`\memtype_i` を順序を維持して結合したものとする。

* :math:`\globaltype^\ast` を、すべての :math:`\globaltype_i` を順序を維持して結合したものとする。

* 以上を満たすことで、モジュールインスタンスは :ref:`コンテキスト <context>` :math:`\{\CTYPES~\functype^\ast, \CFUNCS~{\functype'}^\ast, \CTABLES~\tabletype^\ast, \CMEMS~\memtype^\ast, \CGLOBALS~\globaltype^\ast\}` で有効となる。

.. math::
   ~\\[-1ex]
   \frac{
     \begin{array}{@{}c@{}}
     (\vdashfunctype \functype \ok)^\ast
     \\
     (S \vdashexternval \EVFUNC~\funcaddr : \ETFUNC~\functype')^\ast
     \qquad
     (S \vdashexternval \EVTABLE~\tableaddr : \ETTABLE~\tabletype)^\ast
     \\
     (S \vdashexternval \EVMEM~\memaddr : \ETMEM~\memtype)^\ast
     \qquad
     (S \vdashexternval \EVGLOBAL~\globaladdr : \ETGLOBAL~\globaltype)^\ast
     \\
     (S \vdashexportinst \exportinst \ok)^\ast
     \qquad
     (\exportinst.\EINAME)^\ast ~\mbox{互いに素}
     \end{array}
   }{
     S \vdashmoduleinst \{
       \begin{array}[t]{@{}l@{~}l@{}}
       \MITYPES & \functype^\ast, \\
       \MIFUNCS & \funcaddr^\ast, \\
       \MITABLES & \tableaddr^\ast, \\
       \MIMEMS & \memaddr^\ast, \\
       \MIGLOBALS & \globaladdr^\ast \\
       \MIEXPORTS & \exportinst^\ast ~\} : \{
         \begin{array}[t]{@{}l@{~}l@{}}
         \CTYPES & \functype^\ast, \\
         \CFUNCS & {\functype'}^\ast, \\
         \CTABLES & \tabletype^\ast, \\
         \CMEMS & \memtype^\ast, \\
         \CGLOBALS & \globaltype^\ast ~\}
         \end{array}
       \end{array}
   }


.. index:: configuration, administrative instruction, store, frame
.. _frame-context:
.. _valid-config:

設定の有効性
~~~~~~~~~~~~~~~~~~~~~~

WebAssembly :ref:`型システム <valid>` をその :ref:`実行セマンティクス <exec>` と関連付けるために、:ref:`インストラクション用の型付けルール <valid-instr>` が :ref:`設定 <syntax-config>` :math:`S;T` に拡張されなければなりません。

「設定」と「スレッド」はそれらの :ref:`結果型 <syntax-resulttype>` によって分類されます。
ストア :math:`S` に加えて、スレッドも「戻り型」 :math:`\resulttype^?` で型付けされます。これはどの型を持つ |return| インストラクションを許すべきかを制御します。

この型は、管理インストラクションである |FRAME| 内部のインストラクションシーケンスを除いて非該当（ (:math:`\epsilon`)）となります。

最後に、:ref:`フレーム <syntax-frame>` は「フレームコンテキスト」で分類されます。フレームコンテキストは、あるフレームに関連付けられる :ref:`モジュールインスタンス <syntax-moduleinst>` （フレームが含む :ref:`ローカル <syntax-local>` を持つ）の :ref:`モジュールコンテキスト <module-context>` を拡張します。

.. index:: result type, thread

:ref:`設定 <syntax-config>` :math:`S;T`
.................................................

* :ref:`ストア <syntax-store>` :math:`S` は :ref:`有効 <valid-store>` でなければならない。

* 許可された戻り型が存在しない場合、:ref:`スレッド <syntax-thread>` :math:`T` はいくつかの :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` で :ref:`有効 <valid-thread>` でなければならない。

* 以上を満たすことで、設定は :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` で有効となる。

.. math::
   \frac{
     \vdashstore S \ok
     \qquad
     S; \epsilon \vdashthread T : [t^\ast]
   }{
     \vdashconfig S; T : [t^\ast]
   }


.. index:: thread, frame, instruction, result type, context
.. _valid-thread:

:ref:`スレッド <syntax-thread>` :math:`F;\instr^\ast`
....................................................

* :math:`\resulttype^?` を現在許可されている戻り型とする。

* :ref:`フレーム <syntax-frame>` :math:`F` は、ある :ref:`コンテキスト <context>` :math:`C` で :ref:`有効 <valid-frame>` でなければならない。

* :math:`C'` を :math:`C` と同じ :ref:`コンテキスト <context>` とするが、 |CRETURN| で :math:`\resulttype^?` を設定する。

* コンテキスト :math:`C'` において、インストラクションシーケンス :math:`\instr^\ast` はいくつかの型 :math:`[] \to [t^\ast]` で :ref:`有効 <valid-instr-seq>` でなければならない。

* 以上を満たすことで、スレッドは :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` で有効となる。

.. math::
   \frac{
     S \vdashframe F : C
     \qquad
     S; C,\CRETURN~\resulttype^? \vdashinstrseq \instr^\ast : [] \to [t^\ast]
   }{
     S; \resulttype^? \vdashthread F; \instr^\ast : [t^\ast]
   }


.. index:: frame, local, module instance, value, value type, context
.. _valid-frame:

:ref:`フレーム <syntax-frame>` :math:`\{\ALOCALS~\val^\ast, \AMODULE~\moduleinst\}`
.................................................................................

* :ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\moduleinst` は、いくつかの :ref:`モジュールコンテキスト <module-context>` :math:`C` で :ref:`有効 <valid-moduleinst>` でなければならない。

* :math:`\val^\ast` 内にある個別の :ref:`値 <syntax-val>` :math:`\val_i` は、いくつかの :ref:`値型 <syntax-valtype>` :math:`t_i` で :ref:`有効 <valid-val>` でなければならない。

* :math:`t^\ast` を、すべての :math:`t_i` を順序を維持して結合したものとする。

* :math:`C'` を :math:`C` と同じ :ref:`コンテキスト <context>` とするが、:ref:`値型 <syntax-valtype>` :math:`t^\ast` を |CLOCALS| ベクタの冒頭に追加する。

* 以上を満たすことで、フレームは :ref:`フレームコンテキスト <frame-context>` :math:`C'` で有効となる。

.. math::
   \frac{
     S \vdashmoduleinst \moduleinst : C
     \qquad
     (\vdashval \val : t)^\ast
   }{
     S \vdashframe \{\ALOCALS~\val^\ast, \AMODULE~\moduleinst\} : (C, \CLOCALS~t^\ast)
   }


.. index:: administrative instruction, value type, context, store
.. _valid-instr-admin:

管理インストラクション
~~~~~~~~~~~~~~~~~~~~~~~~~~~

:ref:`管理インストラクション <syntax-instr-admin>` の型付けルールは以下のように指定されます。
:ref:`コンテキスト <context>` :math:`C` に加えて、これらのインストラクションの型付けも、与えられた :ref:`ストア <syntax-store>` :math:`S` のもとで定義されます。
そのために、それ以前のあらゆる型付け判定 :math:`C \vdash \X{prop}` は、ストアに含めるために暗黙で :math:`S` をすべてのストアに追加することで :math:`S; C \vdash \X{prop}` と一般化されます。:math:`S` は事前に存在するルールによって変更されることは決してありませんが、以下で与えられる :ref:`管理インストラクション <valid-instr-admin>` 向けの追加ルール内でアクセスされます。

.. index:: trap

:math:`\TRAP`
.............

* このインストラクションは :ref:`値型 <syntax-valtype>` :math:`t_1^\ast` および :math:`t_2^\ast` の任意のシーケンスにおいて型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
   }{
     S; C \vdashadmininstr \TRAP : [t_1^\ast] \to [t_2^\ast]
   }


.. index:: function address, extern value, extern type, function type

:math:`\INVOKE~\funcaddr`
.........................

* :ref:`外部関数値 <syntax-externval>` :math:`\EVFUNC~\funcaddr` は  :ref:`外部関数型 <syntax-externtype>` :math:`\ETFUNC ([t_1^\ast] \to [t_2^\ast])` で :ref:`有効 <valid-externval-func>` でなければならない。

* これを満たすことで、このインストラクションは型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     S \vdashexternval \EVFUNC~\funcaddr : \ETFUNC~[t_1^\ast] \to [t_2^\ast]
   }{
     S; C \vdashadmininstr \INVOKE~\funcaddr : [t_1^\ast] \to [t_2^\ast]
   }


.. index:: element, table, table address, module instance, function index

:math:`\INITELEM~\tableaddr~o~x^n`
..................................

* :ref:`外部テーブル値 <syntax-externval>` :math:`\EVTABLE~\tableaddr` は、いくつかの :ref:`外部テーブル型 <syntax-externtype>` :math:`\ETTABLE~\limits~\FUNCREF` で :ref:`有効 <valid-externval-table>` でなければならない。

* インデックス :math:`o + n` は :math:`\limits.\LMIN` より小さいか等しくなければならない。

* :ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\moduleinst` は いくつかの :ref:`コンテキスト <context>` :math:`C` で :ref:`有効 <valid-moduleinst>` でなければならない。

* :math:`x^n` 内にある個別の :ref:`関数インデックス <syntax-funcidx>` :math:`x_i` は、コンテキスト :math:`C` 内で定義されなければならない。

* 以上を満たすことで、このインストラクションは有効となる。

.. math::
   \frac{
     S \vdashexternval \EVTABLE~\tableaddr : \ETTABLE~\limits~\FUNCREF
     \qquad
     o + n \leq \limits.\LMIN
     \qquad
     (C.\CFUNCS[x] = \functype)^n
   }{
     S; C \vdashadmininstr \INITELEM~\tableaddr~o~x^n \ok
   }


.. index:: data, memory, memory address, byte

:math:`\INITDATA~\memaddr~o~b^n`
................................

* :ref:`外部メモリー値 <syntax-externval>` :math:`\EVMEM~\memaddr` は、いくつかの :ref:`外部メモリー型 <syntax-externtype>` :math:`\ETMEM~\limits` で :ref:`有効 <valid-externval-mem>` でなければならない。

* インデックス :math:`o + n` は、:math:`\limits.\LMIN` を the :ref:`ページサイズ <page-size>` :math:`64\,\F{Ki}` で割ったものより小さいか等しくなければならない。

* 以上を満たすことで、このインストラクションは有効となる。

.. math::
   \frac{
     S \vdashexternval \EVMEM~\memaddr : \ETMEM~\limits
     \qquad
     o + n \leq \limits.\LMIN \cdot 64\,\F{Ki}
   }{
     S; C \vdashadmininstr \INITDATA~\memaddr~o~b^n \ok
   }


.. index:: label, instruction, result type

:math:`\LABEL_n\{\instr_0^\ast\}~\instr^\ast~\END`
..................................................

* インストラクションシーケンス :math:`\instr_0^\ast` はいくつかの型 :math:`[t_1^n] \to [t_2^*]` で :ref:`有効 <valid-instr-seq>` でなければならない。

* :math:`C'` は :math:`C` と同じ :ref:`コンテキスト <context>` とするが、:ref:`結果型 <syntax-resulttype>` :math:`[t_1^n]` を |CLABELS| ベクタの冒頭に追加する。

* コンテキスト :math:`C'` において、インストラクションシーケンス :math:`\instr^\ast` が型 :math:`[] \to [t_2^*]` で :ref:`有効 <valid-instr-seq>` でなければならない。

* 以上を満たすことで、複合インストラクションは型 :math:`[] \to [t_2^*]` で有効となる。

.. math::
   \frac{
     S; C \vdashinstrseq \instr_0^\ast : [t_1^n] \to [t_2^*]
     \qquad
     S; C,\CLABELS\,[t_1^n] \vdashinstrseq \instr^\ast : [] \to [t_2^*]
   }{
     S; C \vdashadmininstr \LABEL_n\{\instr_0^\ast\}~\instr^\ast~\END : [] \to [t_2^*]
   }


.. index:: frame, instruction, result type

:math:`\FRAME_n\{F\}~\instr^\ast~\END`
...........................................

* 戻り型 :math:`[t^n]` において、:ref:`スレッド <syntax-frame>` :math:`F; \instr^\ast` は :ref:`結果型 <syntax-resulttype>` :math:`[t^n]` で :ref:`有効 <valid-frame>` でなければならない。

* これを満たすことで、複合インストラクションは型 :math:`[] \to [t^n]` で有効となる。

.. math::
   \frac{
     S; [t^n] \vdashinstrseq F; \instr^\ast : [t^n]
   }{
     S; C \vdashadmininstr \FRAME_n\{F\}~\instr^\ast~\END : [] \to [t^n]
   }


.. index:: ! store extension, store
.. _extend:

ストアの拡張
~~~~~~~~~~~~~~~

プログラムは、:ref:`ストア <syntax-store>` およびそこに含まれるインスタンスを改変できます。
そのような変更は、すべて「アロケーションされたインスタンスを削除しない」「イミュータブルな定義を変更しない」といった特定の不変性を尊重しなければなりません。
そうした不変の性質がWebAssemblyの :ref:`インストラクション <exec-instr>` や :ref:`モジュール <exec-instantiation>` の実行セマンティクスに固有のものだとしても、:ref:`ホスト関数 <syntax-hostfunc>` が自動的にそれを守るわけではありません。その結果、そこで求められる不変性はホスト関数の :ref:`呼び出し <exec-invoke-host>` で明示的な制約として記述されなければなりません。
健全性は、the :ref:`エンベダー <embedder>` がそうした制約を遵守した場合にのみ維持されます。

必要な制約は、ストアの「拡張（extension）」記述によってコード化されます。あるストアがステート :math:`S` を拡張するステート :math:`S'` は、以下のルールが維持される場合に :math:`S \extendsto S'` と記述されます。

.. note::
   拡張は、新しいストアが有効であることを暗に示しません。ストアの有効性については :ref:`上述 <valid-store>` のように個別に定義されます。


.. index:: store, function instance, table instance, memory instance, global instance
.. _extend-store:

:ref:`ストア <syntax-store>` :math:`S`
.....................................

* :math:`S.\SFUNCS` の長さは縮小されてはならない。

* :math:`S.\STABLES` の長さは縮小されてはならない。

* :math:`S.\SMEMS` の長さは縮小されてはならない。

* :math:`S.\SGLOBALS` の長さは縮小されてはならない。

* 元の :math:`S.\SFUNCS` 内にある個別の :ref:`関数インスタンス <syntax-funcinst>` :math:`\funcinst_i` について、新しい関数インスタンスが古い関数インスタンスの :ref:`拡張 <extend-funcinst>` でなければならない。

* 元の :math:`S.\STABLES` 内にある個別の :ref:`テーブルインスタンス <syntax-tableinst>` :math:`\tableinst_i` について、新しいテーブルインスタンスが古いテーブルインスタンスの :ref:`拡張 <extend-tableinst>` でなければならない。

* 元の :math:`S.\SMEMS` 内にある個別の :ref:`メモリーインスタンス <syntax-meminst>` :math:`\meminst_i` について、新しいメモリーインスタンスが古いメモリーインスタンスの :ref:`拡張 <extend-meminst>` でなければならない。

* 元の :math:`S.\SGLOBALS` 内にある個別の :ref:`グローバルインスタンス <syntax-globalinst>` :math:`\globalinst_i` について、新しいグローバルインスタンスが古いグローバルインスタンスの :ref:`拡張 <extend-globalinst>` でなければならない。

.. math::
   \frac{
     \begin{array}{@{}ccc@{}}
     S_1.\SFUNCS = \funcinst_1^\ast &
     S_2.\SFUNCS = {\funcinst'_1}^\ast~\funcinst_2^\ast &
     (\funcinst_1 \extendsto \funcinst'_1)^\ast \\
     S_1.\STABLES = \tableinst_1^\ast &
     S_2.\STABLES = {\tableinst'_1}^\ast~\tableinst_2^\ast &
     (\tableinst_1 \extendsto \tableinst'_1)^\ast \\
     S_1.\SMEMS = \meminst_1^\ast &
     S_2.\SMEMS = {\meminst'_1}^\ast~\meminst_2^\ast &
     (\meminst_1 \extendsto \meminst'_1)^\ast \\
     S_1.\SGLOBALS = \globalinst_1^\ast &
     S_2.\SGLOBALS = {\globalinst'_1}^\ast~\globalinst_2^\ast &
     (\globalinst_1 \extendsto \globalinst'_1)^\ast \\
     \end{array}
   }{
     \vdashstoreextends S_1 \extendsto S_2
   }


.. index:: function instance
.. _extend-funcinst:

:ref:`関数インスタンス <syntax-funcinst>` :math:`\funcinst`
............................................................

* 関数インスタンスは無変更のままでなければならない。

.. math::
   \frac{
   }{
     \vdashfuncinstextends \funcinst \extendsto \funcinst
   }


.. index:: table instance
.. _extend-tableinst:

:ref:`テーブルインスタンス <syntax-tableinst>` :math:`\tableinst`
...........................................................

* :math:`\tableinst.\TIELEM` の長さは縮小されてはならない。

* :math:`\tableinst.\TIMAX` の値は無変更でなければならない。

.. math::
   \frac{
     n_1 \leq n_2
   }{
     \vdashtableinstextends \{\TIELEM~(\X{fa}_1^?)^{n_1}, \TIMAX~m\} \extendsto \{\TIELEM~(\X{fa}_2^?)^{n_2}, \TIMAX~m\}
   }


.. index:: memory instance
.. _extend-meminst:

:ref:`メモリーインスタンス <syntax-meminst>` :math:`\meminst`
........................................................

* :math:`\meminst.\MIDATA` の長さは縮小されてはならない。

* :math:`\meminst.\MIMAX` の値は無変更でなければならない。

.. math::
   \frac{
     n_1 \leq n_2
   }{
     \vdashmeminstextends \{\MIDATA~b_1^{n_1}, \MIMAX~m\} \extendsto \{\MIDATA~b_2^{n_2}, \MIMAX~m\}
   }


.. index:: global instance, value, mutability
.. _extend-globalinst:

:ref:`グローバルインスタンス <syntax-globalinst>` :math:`\globalinst`
..............................................................

* :ref:`改変可能性 <syntax-mut>` :math:`\globalinst.\GIMUT` は無変更でなければならない。

* :ref:`値 <syntax-val>` :math:`\globalinst.\GIVALUE` の :ref:`値型 <syntax-valtype>` は無変更でなければならない。

* :math:`\globalinst.\GIMUT` が |MCONST| の場合、:ref:`値 <syntax-val>` :math:`\globalinst.\GIVALUE` は無変更でなければならない。

.. math::
   \frac{
     \mut = \MVAR \vee c_1 = c_2
   }{
     \vdashglobalinstextends \{\GIVALUE~(t.\CONST~c_1), \GIMUT~\mut\} \extendsto \{\GIVALUE~(t.\CONST~c_2), \GIMUT~\mut\}
   }


.. index:: ! preservation, ! progress, soundness, configuration, thread, terminal configuration, instantiation, invocation, validity, module
.. _soundness-statement:

定理
~~~~~~~~

与えられた :ref:`有効な設定 <valid-config>` の定義において、以下の標準健全性定理が成り立ちます。 [#cite-cpp2018]_

**定理（保存）**: :ref:`設定 <syntax-config>` :math:`S;T` が :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` で :ref:`有効 <valid-config>` な場合（すなわち :math:`S;T \stepto S';T'`）、:math:`S';T'` は同じ結果型で有効な設定となる（すなわち :math:`\vdashconfig S';T' : [t^\ast]`）。
さらに、:math:`S'` は :math:`S` の :ref:`拡張 <extend-store>` である（すなわち :math:`\vdashstoreextends S \extendsto S'`）。

「終端」 :ref:`スレッド <syntax-thread>` とは、その :ref:`インストラクション <syntax-instr>` のシーケンスが :ref:`結果 <syntax-result>` となるスレッドのことである。
終端設定とは、そのスレッドが終端である設定のことである。

**定理（プログレス）**: :ref:`設定 <syntax-config>` :math:`S;T` が :ref:`有効 <valid-config>` な場合（すなわちいくつかの :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` において :math:`\vdashconfig S;T : [t^\ast]` となる)、「いずれか一方が終端である」または「いくつかの設定 :math:`S';T'` に進める」のいずれかとなる（すなわち :math:`S;T \stepto S';T'`）。

保存定理とプログレス定理から、WebAssembly型システムの健全性が直接導かれる。

**系（健全性）**: :ref:`設定 <syntax-config>` :math:`S;T` が :ref:`有効 <valid-config>` な場合（すなわちいくつかの :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` で :math:`\vdashconfig S';T' : [t^\ast]` となる）、乖離（diverge）するか有限のステップ数で終端設定 :math:`S';T'` に達する（すなわち :math:`S;T \stepto^\ast S';T'`）。これは同じ結果型（すなわち :math:`\vdashconfig S';T' : [t^\ast]`）で有効であり、ただし :math:`S'` は :math:`S` の :ref:`拡張 <extend-store>` （すなわち :math:`\vdashstoreextends S \extendsto S'`）である。

言い換えると、有効な設定内にある個別のスレッドはどれも、「永遠に実行続ける」「トラップする」「期待される型を持つ結果で終了する」のいずれかにしかなりません。
その結果、:ref:`有効なストア <valid-store>` が与えられた場合に、有効なモジュールの :ref:`インスタンス化 <exec-instantiation>` や :ref:`呼び出し <exec-invocation>` で定義される計算がクラッシュすることもなければ、本仕様で与えられる :ref:`実行 <exec>` セマンティクスでカバーできない形で振る舞いがおかしくなることもありません。

.. [#cite-pldi2017]
   この形式化および定理は、以下の記事から派生したものです。
   Andreas Haas, Andreas Rossberg, Derek Schuff, Ben Titzer, Dan Gohman, Luke Wagner, Alon Zakai, JF Bastien, Michael Holman. |PLDI2017|_. Proceedings of the 38th ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI 2017). ACM 2017.

.. [#cite-cpp2018]
   この形式化と健全性の証明を機械化したバージョンは、以下の記事に記載されています。
   Conrad Watt. |CPP2018|_. Proceedings of the 7th ACM SIGPLAN Conference on Certified Programs and Proofs (CPP 2018). ACM 2018.
