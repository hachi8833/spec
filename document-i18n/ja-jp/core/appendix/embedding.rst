.. index:: ! embedding, embedder, implementation, host
.. _embed:

埋め込み
---------

WebAssemblyの実装は「ホスト」環境に「埋め込まれる」のが典型的です。
「エンベダー（embedder）」は、そのようなホスト環境とWebAssemblyセマンティクスの間を本仕様書の本文で定義されているとおりにつなぎます。
エンベダーは、十分に定義された形でWebAssemblyセマンティクスとやりとりすることが期待されます。

本セクションでは、WebAssemblyセマンティクスへの適切なインターフェイスを、エンベダーがアクセスできるエントリポイントの形で定義します。
このインターフェイスは、エンベダーがWebAssembly仕様における他の機能を直接参照する必要がないという意味で「完備（complete）」であることが意図されています。

.. note::
   逆に、エンベダーはこのインターフェイスで定義されている機能へのアクセスをすべてホスト環境に提供する必要はありません。
   たとえば、実装は :ref:`テキスト形式 <text>` の :ref:`パース <embed-module-parse>` をサポートしなくても構いません。


型（type）
~~~~~

「:ref:`抽象構文 <syntax>`」や「:ref:`ランタイムの抽象マシン <syntax-runtime>`」由来の構文的クラス（syntactic class）は、そのクラスから可能なさまざまなオブジェクトの範囲内にある変数の名前としてエンベダーインターフェイスのデスクリプションの中で用いられます。
すなわち、これらの構文的クラスは型としても解釈されます。

数値パラメーターの場合、:math:`n:\u32` のような記法は、対応する値の範囲のほかに、シンボル名（symbolic name）を指定するのに使われます。

.. _embed-error:

エラー（error）
~~~~~~

あるインターフェイス操作の失敗は、補助的な構文的クラスで示されます。

.. math::
   \begin{array}{llll}
   \production{(error)} & \error &::=& \ERROR \\
   \end{array}

本セクションで明示的に指定するエラー条件のほかに、特定の :ref:`実装制限 <impl>` に達したときに実装がエラーを返すこともあります。

.. note::
   エラーは、この定義において抽象的かつ非特異的です。
   実装は、適切な分類や診断メッセージを伝達するためにエラーを洗練させることができます。


事前条件（pre-condition）と事後条件（post-condition）
~~~~~~~~~~~~~~~~~~~~~~~~

演算子によっては、それらの引数の「事前条件」や、それらの結果の「事後条件」を記述するものもあります。
事前条件を満たす責任はエンベダーにあります。
事前条件が満たされれば、事後条件はセマンティクスによって保証されます。

本仕様では、演算子ごとに明示的に記述される事前条件や事後条件のほかに、:ref:`ランタイムオブジェクト <syntax-runtime>` （ :math:`store`、:math:`\moduleinst`、:math:`\externval`、:ref:`アドレス <syntax-addr>`）について以下の記法も採用しています。

* あるパラメーターとして渡される個別のランタイムオブジェクトは、暗黙の事前条件に対して :ref:`有効 <valid-store>` でなければならない。

* 個別のランタイムオブジェクトは、暗黙の事後条件に対して :ref:`有効 <valid-store>` な結果を返さなければならない。

.. note::
   エンベダーは、ランタイムオブジェクトを抽象としてのみ取り扱い、かつここで定義されるインターフェイスを介して操作する限り、あらゆる暗黙の事前条件が自動的に満たされます。


.. index:: allocation, store
.. _embed-store:

ストア（store）
~~~~~

.. _embed-store-init:

:math:`\F{store\_init}() : \store`
..................................

1. 空の :ref:`ストア <syntax-store>` を返す。

.. math::
   \begin{array}{lclll}
   \F{store\_init}() &=& \{ \SFUNCS~\epsilon,~ \SMEMS~\epsilon,~ \STABLES~\epsilon,~ \SGLOBALS~\epsilon \} \\
   \end{array}



.. index:: module
.. _embed-module:

モジュール（module）
~~~~~~~

.. index:: binary format
.. _embed-module-decode:

:math:`\F{module\_decode}(\byte^\ast) : \module ~|~ \error`
...........................................................

1. :ref:`バイト <syntax-byte>` シーケンス :math:`\byte^\ast` の派生が :ref:`モジュールのバイナリ構文 <binary-module>` に基づいて :math:`\Bmodule` として存在し、:ref:`モジュール <syntax-module>` :math:`m` がひとつ生成された場合は、:math:`m` を返す。

2. それ以外の場合は :math:`\ERROR` を返す。

.. math::
   \begin{array}{lclll}
   \F{module\_decode}(b^\ast) &=& m && (\iff \Bmodule \stackrel\ast\Longrightarrow m{:}b^\ast) \\
   \F{module\_decode}(b^\ast) &=& \ERROR && (\otherwise) \\
   \end{array}


.. index:: text format
.. _embed-module-parse:

:math:`\F{module\_parse}(\char^\ast) : \module ~|~ \error`
..........................................................

1. :ref:`ソース <text-source>` :math:`\char^\ast` の派生が :ref:`モジュールのバイナリ構文 <binary-module>` に基づいて :math:`\Bmodule` として存在し、:ref:`モジュール <syntax-module>` :math:`m` がひとつ生成された場合は、:math:`m` を返す。

2. それ以外の場合は :math:`\ERROR` を返す。

.. math::
   \begin{array}{lclll}
   \F{module\_parse}(c^\ast) &=& m && (\iff \Tmodule \stackrel\ast\Longrightarrow m{:}c^\ast) \\
   \F{module\_parse}(c^\ast) &=& \ERROR && (\otherwise) \\
   \end{array}


.. index:: validation
.. _embed-module-validate:

:math:`\F{module\_validate}(\module) : \error^?`
................................................

1. :math:`\module` が :ref:`有効 <valid-module>` な場合は何も返さない。

2. それ以外の場合は :math:`\ERROR` を返す。

.. math::
   \begin{array}{lclll}
   \F{module\_validate}(m) &=& \epsilon && (\iff {} \vdashmodule m : \externtype^\ast \to {\externtype'}^\ast) \\
   \F{module\_validate}(m) &=& \ERROR && (\otherwise) \\
   \end{array}


.. index:: instantiation, module instance
.. _embed-module-instantiate:

:math:`\F{module\_instantiate}(\store, \module, \externval^\ast) : (\store, \moduleinst ~|~ \error)`
....................................................................................................

1. :math:`\store` 内の :math:`\module` を :ref:`外部値 <syntax-externval>` :math:`\externval^\ast` でインポートとして :ref:`インスタンス化 <exec-instantiation>` を試みる。

  a. 成功して :ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\moduleinst` ができたら、:math:`\X{result}` を :math:`\moduleinst` とする。

  b. それ以外の場合は :math:`\X{result}` を :math:`\ERROR` とする。

2. :math:`\X{result}` を持つ新しいストアを返す。

.. math::
   \begin{array}{lclll}
   \F{module\_instantiate}(S, m, \X{ev}^\ast) &=& (S', F.\AMODULE) && (\iff \instantiate(S, m, \X{ev}^\ast) \stepto^\ast S'; F; \epsilon) \\
   \F{module\_instantiate}(S, m, \X{ev}^\ast) &=& (S', \ERROR) && (\iff \instantiate(S, m, \X{ev}^\ast) \stepto^\ast S'; F; \TRAP) \\
   \end{array}

.. note::
   このストアは、エラーの場合にも変更される可能性があります。


.. index:: import
.. _embed-module-imports:

:math:`\F{module\_imports}(\module) : (\name, \name, \externtype)^\ast`
.......................................................................

1. 事前条件: :math:`\module` が外部インポート型 :math:`\externtype^\ast` および外部エクスポート型 :math:`{\externtype'}^\ast` で :ref:`有効 <valid-module>` であること。

2. :math:`\import^\ast` を :ref:`インポート <syntax-import>` :math:`\module.\MIMPORTS` とする。

3. アサーション: :math:`\import^\ast` の長さが :math:`\externtype^\ast` の長さと等しい。

4. :math:`\import^\ast` 内にある個別の :math:`\import_i` および :math:`\externtype^\ast` 内でそれに対応する :math:`\externtype_i` ごとに以下を行う。

  a. :math:`\X{result}_i` をトリプル :math:`(\import_i.\IMODULE、\import_i.\INAME, \externtype_i)` とする。

5. すべての :math:`\X{result}_i` をインデックス順に結合した結果を返す。

6. 事後条件: 個別の :math:`\externtype_i` が :ref:`有効 <valid-externtype>` であること。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{module\_imports}(m) &=& (\X{im}.\IMODULE, \X{im}.\INAME, \externtype)^\ast \\
     && \qquad (\iff \X{im}^\ast = m.\MIMPORTS \wedge {} \vdashmodule m : \externtype^\ast \to {\externtype'}^\ast) \\
   \end{array}


.. index:: export
.. _embed-module-exports:

:math:`\F{module\_exports}(\module) : (\name, \externtype)^\ast`
................................................................

1. 事前条件: :math:`\module` が外部インポート型 :math:`\externtype^\ast` および外部エクスポート型 :math:`{\externtype'}^\ast` で :ref:`有効 <valid-module>` であること。

2. :math:`\export^\ast` を :ref:`エクスポート <syntax-export>` :math:`\module.\MEXPORTS` とする。

3. アサーション: :math:`\export^\ast` の長さが :math:`\externtype^\ast` の長さと等しい。

4. :math:`\export^\ast` 内にある個別の :math:`\export_i` および :math:`{\externtype'}^\ast` 内でそれに対応する :math:`\externtype'_i` ごとに以下を行う。

  a. :math:`\X{result}_i` をペア :math:`(\export_i.\ENAME, \externtype'_i)` とする。

5. すべての :math:`\X{result}_i` をインデックス順に結合した結果を返す。

6. 事後条件: 個別の :math:`\externtype'_i` が :ref:`有効 <valid-externtype>` であること。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{module\_exports}(m) &=& (\X{ex}.\ENAME, \externtype')^\ast \\
     && \qquad (\iff \X{ex}^\ast = m.\MEXPORTS \wedge {} \vdashmodule m : \externtype^\ast \to {\externtype'}^\ast) \\
   \end{array}


.. index:: module, module instance
.. _embed-instance:

モジュールインスタンス（module instance）
~~~~~~~~~~~~~~~~

.. index:: export, export instance

.. _embed-instance-export:

:math:`\F{instance\_export}(\moduleinst, \name) : \externval ~|~ \error`
........................................................................

1. アサーション: :ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\moduleinst` が :ref:`有効 <valid-moduleinst>` であることによって、すべての :ref:`エクスポート名 <syntax-exportinst>` に重複が存在しない。

2. :ref:`名前 <syntax-name>` :math:`\exportinst_i.\EINAME` が :math:`\name` に等しい :math:`\exportinst_i` が :math:`\moduleinst.\MIEXPORTS` 内に存在する場合、以下を行う。

   a. :ref:`外部値 <syntax-externval>` :math:`\exportinst_i.\EIVALUE` を返す。

3. それ以外の場合は :math:`\ERROR` を返す。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{instance\_export}(m, \name) &=& m.\MIEXPORTS[i].\EIVALUE && (\iff m.\MEXPORTS[i].\EINAME = \name) \\
   \F{instance\_export}(m, \name) &=& \ERROR && (\otherwise) \\
   \end{array}


.. index:: function, host function, function address, function instance, function type, store
.. _embed-func:

関数（function）
~~~~~~~~~

.. _embed-func-alloc:

:math:`\F{func\_alloc}(\store, \functype, \hostfunc) : (\store, \funcaddr)`
...........................................................................

1. 事前条件: :math:`\functype` が :ref:`有効 <valid-functype>` であること。

2. :math:`\funcaddr` を、:math:`\store` 内に「:ref:`関数型 <syntax-functype>` :math:`\functype` とホスト関数コード :math:`\hostfunc`」という :ref:`ホスト関数をアロケーションした <alloc-func>` 結果とする。

3. :math:`\funcaddr` を持つ新しいストアを返す。

.. math::
   \begin{array}{lclll}
   \F{func\_alloc}(S, \X{ft}, \X{code}) &=& (S', \X{a}) && (\iff \allochostfunc(S, \X{ft}, \X{code}) = S', \X{a}) \\
   \end{array}

.. note::
   この操作は、:math:`\hostfunc` が型 :math:`\functype` の関数インスタンスで必要とされる :ref:`事前条件および事後条件 <exec-invoke-host>` を満たすことを仮定します。

   通常の（すなわちホスト関数でない）関数インスタンスは、:ref:`モジュールのインスタンス化 <embed-module-instantiate>` によってのみ間接的に作成可能です。

.. _embed-func-type:

:math:`\F{func\_type}(\store, \funcaddr) : \functype`
.....................................................

1. アサーション: :ref:`外部値 <syntax-externval>` :math:`\EVFUNC~\funcaddr` が :ref:`外部型 <syntax-externtype>` :math:`\ETFUNC~\functype` で :ref:`有効 <valid-externval>` である。

2. :math:`\functype` を返す。

3. 事後条件: :math:`\functype` が :ref:`有効 <valid-functype>` であること。

.. math::
   \begin{array}{lclll}
   \F{func\_type}(S, a) &=& \X{ft} && (\iff S \vdashexternval \EVFUNC~a : \ETFUNC~\X{ft}) \\
   \end{array}


.. index:: invocation, value, result
.. _embed-func-invoke:

:math:`\F{func\_invoke}(\store, \funcaddr, \val^\ast) : (\store, \val^\ast ~|~ \error)`
........................................................................................

1. :math:`\store` 内の関数 :math:`\funcaddr` を、:ref:`値 <syntax-val>` :math:`\val^\ast` を引数として :ref:`呼び出し <exec-invocation>` を試みる。

  a. 成功して結果が :ref:`値 <syntax-val>` :math:`{\val'}^\ast` になった場合は、:math:`\X{result}` を :math:`{\val'}^\ast` とする。

  b. それ以外の場合はトラップされる、すなわち :math:`\X{result}` を :math:`\ERROR` とする。

2. :math:`\X{result}` を持つ新しいストアを返す。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{func\_invoke}(S, a, v^\ast) &=& (S', {v'}^\ast) && (\iff \invoke(S, a, v^\ast) \stepto^\ast S'; F; {v'}^\ast) \\
   \F{func\_invoke}(S, a, v^\ast) &=& (S', \ERROR) && (\iff \invoke(S, a, v^\ast) \stepto^\ast S'; F; \TRAP) \\
   \end{array}

.. note::
   このストアは、エラーの場合にも変更される可能性があります。


.. index:: table, table address, store, table instance, table type, element, function address
.. _embed-table:

テーブル（table）
~~~~~~

.. _embed-table-alloc:

:math:`\F{table\_alloc}(\store, \tabletype) : (\store, \tableaddr)`
...................................................................

1. 事前条件: :math:`\tabletype` が :ref:`有効 <valid-tabletype>` であること。

2. :math:`\tableaddr` を、:math:`\store` 内に :ref:`テーブル型 <syntax-tabletype>` :math:`\tabletype` の :ref:`テーブルをアロケーションした <alloc-table>` 結果とする。

3. :math:`\tableaddr` を持つ新しいストアを返す。

.. math::
   \begin{array}{lclll}
   \F{table\_alloc}(S, \X{tt}) &=& (S', \X{a}) && (\iff \alloctable(S, \X{tt}) = S', \X{a}) \\
   \end{array}


.. _embed-table-type:

:math:`\F{table\_type}(\store, \tableaddr) : \tabletype`
........................................................

1. アサーション: the :ref:`外部値 <syntax-externval>` :math:`\EVTABLE~\tableaddr` が :ref:`外部型 <syntax-externtype>` :math:`\ETTABLE~\tabletype` で :ref:`有効 <valid-externval>` であること。

2. :math:`\tabletype` を返す。

3. 事後条件: :math:`\tabletype` が :ref:`有効 <valid-tabletype>` であること。

.. math::
   \begin{array}{lclll}
   \F{table\_type}(S, a) &=& \X{tt} && (\iff S \vdashexternval \EVTABLE~a : \ETTABLE~\X{tt}) \\
   \end{array}


.. _embed-table-read:

:math:`\F{table\_read}(\store, \tableaddr, i:\u32) : \funcaddr^? ~|~ \error`
............................................................................

1. :math:`\X{ti}` を :ref:`テーブルインスタンス <syntax-tableinst>` :math:`\store.\STABLES[\tableaddr]` とする。

2. :math:`i` が :math:`\X{ti}.\TIELEM` の長さより大きいか等しい場合、:math:`\ERROR` を返す。

3. それ以外の場合は :math:`\X{ti}.\TIELEM[i]` を返す。

.. math::
   \begin{array}{lclll}
   \F{table\_read}(S, a, i) &=& \X{fa}^? && (\iff S.\STABLES[a].\TIELEM[i] = \X{fa}^?) \\
   \F{table\_read}(S, a, i) &=& \ERROR && (\otherwise) \\
   \end{array}


.. _embed-table-write:

:math:`\F{table\_write}(\store, \tableaddr, i:\u32, \funcaddr^?) : \store ~|~ \error`
.......................................................................................

1. :math:`\X{ti}` を :ref:`テーブルインスタンス <syntax-tableinst>` :math:`\store.\STABLES[\tableaddr]` とする。

2. :math:`i` が :math:`\X{ti}.\TIELEM` の長さより大きいか等しい場合、:math:`\ERROR` を返す。

3. :math:`\X{ti}.\TIELEM[i]` をオプションの :ref:`関数アドレス <syntax-funcaddr>` :math:`\X{fa}^?` と置き換える。

4. 更新されたストアを返す。

.. math::
   \begin{array}{lclll}
   \F{table\_write}(S, a, i, \X{fa}^?) &=& S' && (\iff S' = S \with \STABLES[a].\TIELEM[i] = \X{fa}^?) \\
   \F{table\_write}(S, a, i, \X{fa}^?) &=& \ERROR && (\otherwise) \\
   \end{array}


.. _embed-table-size:

:math:`\F{table\_size}(\store, \tableaddr) : \u32`
..................................................

1. :math:`\store.\STABLES[\tableaddr].\TIELEM` の長さを返す。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{table\_size}(S, a) &=& n &&
     (\iff |S.\STABLES[a].\TIELEM| = n) \\
   \end{array}



.. _embed-table-grow:

:math:`\F{table\_grow}(\store, \tableaddr, n:\u32) : \store ~|~ \error`
.......................................................................

1. :ref:`テーブルインスタンス <syntax-tableinst>` :math:`\store.\STABLES[\tableaddr]` で要素 :math:`n` 個分の :ref:`成長 <grow-table>` を試みる。

   a. 成功した場合は、更新されたストアを返す。

   b. それ以外の場合は :math:`\ERROR` を返す。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{table\_grow}(S, a, n) &=& S' &&
     (\iff S' = S \with \STABLES[a] = \growtable(S.\STABLES[a], n)) \\
   \F{table\_grow}(S, a, n) &=& \ERROR && (\otherwise) \\
   \end{array}


.. index:: memory, memory address, store, memory instance, memory type, byte
.. _embed-mem:

メモリー（memory）
~~~~~~~~

.. _embed-mem-alloc:

:math:`\F{mem\_alloc}(\store, \memtype) : (\store, \memaddr)`
................................................................

1. 事前条件: :math:`\memtype` は :ref:`有効 <valid-memtype>` であること。

2. :math:`\memaddr` を、:math:`\store` 内に :ref:`メモリー型 <syntax-memtype>` :math:`\memtype` で :ref:`メモリーをアロケーションした <alloc-mem>` 結果とする。

3. :math:`\memaddr` を持つ新しいストアを返す。

.. math::
   \begin{array}{lclll}
   \F{mem\_alloc}(S, \X{mt}) &=& (S', \X{a}) && (\iff \allocmem(S, \X{mt}) = S', \X{a}) \\
   \end{array}


.. _embed-mem-type:

:math:`\F{mem\_type}(\store, \memaddr) : \memtype`
..................................................

1. アサーション: the :ref:`外部値 <syntax-externval>` :math:`\EVMEM~\memaddr` は :ref:`外部型 <syntax-externtype>` :math:`\ETMEM~\memtype` で :ref:`有効 <valid-externval>` である。

2. :math:`\memtype` を返す。

3. 事後条件: :math:`\memtype` は :ref:`有効 <valid-memtype>` であること。

.. math::
   \begin{array}{lclll}
   \F{mem\_type}(S, a) &=& \X{mt} && (\iff S \vdashexternval \EVMEM~a : \ETMEM~\X{mt}) \\
   \end{array}


.. _embed-mem-read:

:math:`\F{mem\_read}(\store, \memaddr, i:\u32) : \byte ~|~ \error`
..................................................................

1. :math:`\X{mi}` を :ref:`メモリーインスタンス <syntax-meminst>` :math:`\store.\SMEMS[\memaddr]` とする。

2. :math:`i` が :math:`\X{mi}.\MIDATA` の長さより大きいか等しい場合、:math:`\ERROR` を返す。

3. それ以外の場合は :ref:`バイト <syntax-byte>` :math:`\X{mi}.\MIDATA[i]` を返す。

.. math::
   \begin{array}{lclll}
   \F{mem\_read}(S, a, i) &=& b && (\iff S.\SMEMS[a].\MIDATA[i] = b) \\
   \F{mem\_read}(S, a, i) &=& \ERROR && (\otherwise) \\
   \end{array}


.. _embed-mem-write:

:math:`\F{mem\_write}(\store, \memaddr, i:\u32, \byte) : \store ~|~ \error`
...........................................................................

1. :math:`\X{mi}` を :ref:`メモリーインスタンス <syntax-meminst>` :math:`\store.\SMEMS[\memaddr]` とする。

2. :math:`\u32` が :math:`\X{mi}.\MIDATA` の長さより大きいか等しい場合、:math:`\ERROR` を返す。

3. :math:`\X{mi}.\MIDATA[i]` を :math:`\byte` で置き換える。

4. 更新されたストアを返す。

.. math::
   \begin{array}{lclll}
   \F{mem\_write}(S, a, i, b) &=& S' && (\iff S' = S \with \SMEMS[a].\MIDATA[i] = b) \\
   \F{mem\_write}(S, a, i, b) &=& \ERROR && (\otherwise) \\
   \end{array}


.. _embed-mem-size:

:math:`\F{mem\_size}(\store, \memaddr) : \u32`
..............................................

1. :math:`\store.\SMEMS[\memaddr].\MIDATA` の長さを :ref:`ページサイズ <page-size>` で割った結果を返す。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{mem\_size}(S, a) &=& n &&
     (\iff |S.\SMEMS[a].\MIDATA| = n \cdot 64\,\F{Ki}) \\
   \end{array}



.. _embed-mem-grow:

:math:`\F{mem\_grow}(\store, \memaddr, n:\u32) : \store ~|~ \error`
...................................................................

1. :ref:`メモリーインスタンス <syntax-meminst>` :math:`\store.\SMEMS[\memaddr]` で :math:`n` :ref:`ページ <page-size>` 分の :ref:`成長 <grow-table>` を試みる。

   a. 成功した場合は、更新されたストアを返す。

   b. それ以外の場合は :math:`\ERROR` を返す。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{mem\_grow}(S, a, n) &=& S' &&
     (\iff S' = S \with \SMEMS[a] = \growmem(S.\SMEMS[a], n)) \\
   \F{mem\_grow}(S, a, n) &=& \ERROR && (\otherwise) \\
   \end{array}



.. index:: global, global address, store, global instance, global type, value
.. _embed-global:

グローバル（global）
~~~~~~~

.. _embed-global-alloc:

:math:`\F{global\_alloc}(\store, \globaltype, \val) : (\store, \globaladdr)`
............................................................................

1. 事前条件: :math:`\globaltype` は :ref:`有効 <valid-globaltype>` であること。

2. :math:`\globaladdr` を、:math:`\store` 内で :ref:`グローバル型 <syntax-globaltype>` と初期値 :math:`\val` で :ref:`アロケーション <alloc-global>` した結果とする。

3. :math:`\globaladdr` を持つ新しいストアを返す。

.. math::
   \begin{array}{lclll}
   \F{global\_alloc}(S, \X{gt}, v) &=& (S', \X{a}) && (\iff \allocglobal(S, \X{gt}, v) = S', \X{a}) \\
   \end{array}


.. _embed-global-type:

:math:`\F{global\_type}(\store, \globaladdr) : \globaltype`
...........................................................

1. アサーション: :ref:`外部値 <syntax-externval>` :math:`\EVGLOBAL~\globaladdr` が :ref:`外部型 <syntax-externtype>` :math:`\ETGLOBAL~\globaltype` で  :ref:`有効 <valid-externval>` である。

2. :math:`\globaltype` を返す。

3. 事後条件: :math:`\globaltype` が :ref:`有効 <valid-globaltype>` であること。

.. math::
   \begin{array}{lclll}
   \F{global\_type}(S, a) &=& \X{gt} && (\iff S \vdashexternval \EVGLOBAL~a : \ETGLOBAL~\X{gt}) \\
   \end{array}


.. _embed-global-read:

:math:`\F{global\_read}(\store, \globaladdr) : \val`
....................................................

1. :math:`\X{gi}` を :ref:`グローバルインスタンス <syntax-globalinst>` :math:`\store.\SGLOBALS[\globaladdr]` とする。

2. :ref:`値 <syntax-val>` :math:`\X{gi}.\GIVALUE` を返す。

.. math::
   \begin{array}{lclll}
   \F{global\_read}(S, a) &=& v && (\iff S.\SGLOBALS[a].\GIVALUE = v) \\
   \end{array}


.. _embed-global-write:

:math:`\F{global\_write}(\store, \globaladdr, \val) : \store ~|~ \error`
........................................................................

1. :math:`\X{gi}` を :ref:`グローバルインスタンス <syntax-globalinst>` :math:`\store.\SGLOBALS[\globaladdr]` とする。

2. :math:`\X{gi}.\GIMUT` が :math:`\MVAR` でない場合、:math:`\ERROR` を返す。

3. :math:`\X{gi}.\GIVALUE` を :ref:`値 <syntax-val>` :math:`\val` で置き換える。

4. 更新されたストアを返す。

.. math::
   ~ \\
   \begin{array}{lclll}
   \F{global\_write}(S, a, v) &=& S' && (\iff S.\SGLOBALS[a].\GIMUT = \MVAR \wedge S' = S \with \SGLOBALS[a].\GIVALUE = v) \\
   \F{global\_write}(S, a, v) &=& \ERROR && (\otherwise) \\
   \end{array}
