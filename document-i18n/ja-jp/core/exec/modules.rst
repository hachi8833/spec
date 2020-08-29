モジュール
-------

モジュールの実行セマンティクスでは、主に :ref:`インスタンス化 <exec-instantiation>` が定義されます。インスタンス化では、1個のモジュールやそこに含まれる定義に対応するいくつものインスタンスを :ref:`アロケーション <alloc>` し、そこに含まれる :ref:`要素 <syntax-elem>` セグメントや :ref:`データ <syntax-data>` セグメントからいくつもの :ref:`テーブル <syntax-table>` や :ref:`メモリー <syntax-mem>` を初期化し、:ref:`開始関数 <syntax-start>` が存在する場合はそれを呼び出します。インスタンス化には、エクスポートされた関数の :ref:`呼び出し <exec-invocation>` も含まれます。

インスタンス化は、:ref:`インポートの型チェック <exec-import>` やインスタンスの :ref:`アロケーション <alloc>` のためのさまざまな補助的概念に依存しています。

.. index:: external value, external type, validation, import, store
.. _valid-externval:

外部型付け（external typing）
~~~~~~~~~~~~~~~

:ref:`インポート <syntax-import>` する :ref:`外部値 <syntax-externval>` をチェックする目的で、そのような値は :ref:`外部値 <syntax-externtype>` によって分類されます。
以下の補助的な型付けルールは、参照されるインスタンスが存在する :ref:`ストア <syntax-store>` :math:`S` に関連して、この型付け関係を指定します。

.. index:: function type, function address
.. _valid-externval-func:

:math:`\EVFUNC~a`
.................

* このストアエントリ :math:`S.\SFUNCS[a]` は、:ref:`関数インスタンス <syntax-funcinst>` :math:`\{\FITYPE~\functype, \dots\}` でなければならない。

* それによって :math:`\EVFUNC~a` は :ref:`外部型 <syntax-externtype>` :math:`\ETFUNC~\functype` で有効となる。

.. math::
   \frac{
     S.\SFUNCS[a] = \{\FITYPE~\functype, \dots\}
   }{
     S \vdashexternval \EVFUNC~a : \ETFUNC~\functype
   }


.. index:: table type, table address, limits
.. _valid-externval-table:

:math:`\EVTABLE~a`
..................

* このストアエントリ :math:`S.\STABLES[a]` は :ref:`テーブルインスタンス <syntax-tableinst>` :math:`\{\TIELEM~(\X{fa}^?)^n, \TIMAX~m^?\}` でなければならない。

* それによって :math:`\EVTABLE~a` は :ref:`外部型 <syntax-externtype>` :math:`\ETTABLE~(\{\LMIN~n, \LMAX~m^?\}~\FUNCREF)` で有効となる。

.. math::
   \frac{
     S.\STABLES[a] = \{ \TIELEM~(\X{fa}^?)^n, \TIMAX~m^? \}
   }{
     S \vdashexternval \EVTABLE~a : \ETTABLE~(\{\LMIN~n, \LMAX~m^?\}~\FUNCREF)
   }


.. index:: memory type, memory address, limits
.. _valid-externval-mem:

:math:`\EVMEM~a`
................

* このストアエントリ :math:`S.\SMEMS[a]` はいくつかの :math:`n` において :ref:`メモリーインスタンス <syntax-meminst>` :math:`\{\MIDATA~b^{n\cdot64\,\F{Ki}}, \MIMAX~m^?\}` でなければならない。

* それによって :math:`\EVMEM~a` は :ref:`外部型 <syntax-externtype>` :math:`\ETMEM~(\{\LMIN~n, \LMAX~m^?\})` で有効となる。

.. math::
   \frac{
     S.\SMEMS[a] = \{ \MIDATA~b^{n\cdot64\,\F{Ki}}, \MIMAX~m^? \}
   }{
     S \vdashexternval \EVMEM~a : \ETMEM~\{\LMIN~n, \LMAX~m^?\}
   }


.. index:: global type, global address, value type, mutability
.. _valid-externval-global:

:math:`\EVGLOBAL~a`
...................

* このストアエントリ :math:`S.\SGLOBALS[a]` は :ref:`グローバルインスタンス <syntax-globalinst>` :math:`\{\GIVALUE~(t.\CONST~c), \GIMUT~\mut\}` でなければならない。

* それによって :math:`\EVGLOBAL~a` は :ref:`外部型 <syntax-externtype>` :math:`\ETGLOBAL~(\mut~t)` で有効となる。

.. math::
   \frac{
     S.\SGLOBALS[a] = \{ \GIVALUE~(t.\CONST~c), \GIMUT~\mut \}
   }{
     S \vdashexternval \EVGLOBAL~a : \ETGLOBAL~(\mut~t)
   }


.. index:: ! matching, external type
.. _exec-import:
.. _match:

インポートのマッチング（import matching）
~~~~~~~~~~~~~~~

1個のモジュールを :ref:`インスタンス化 <exec-module>` する場合、個別のインポートを分類する :ref:`外部型 <syntax-externtype>` とその :ref:`型 <valid-externval>` が「一致する」 :ref:`外部値 <syntax-externval>` が提供されなければなりません。
これによって、いくつかのケースでは以下で定義されるシンプルなサブ型付け（subtyping）形式を利用できるようになります。

.. index:: limits
.. _match-limits:

制限（limit）
......



:ref:`制限 <syntax-limits>` :math:`\{ \LMIN~n_1, \LMAX~m_1^? \}` は、以下の場合に限って制限 :math:`\{ \LMIN~n_2, \LMAX~m_2^? \}` と一致します。

* :math:`n_1` が :math:`n_2` より大きいか等しい。

* 以下のいずれかを満たす。

  * :math:`m_2^?` が空。

* または

  * :math:`m_1^?` と :math:`m_2^?` がどちらも空ではない。

  * :math:`m_1` が :math:`m_2` より小さいか等しい。

.. math::
   ~\\[-1ex]
   \frac{
     n_1 \geq n_2
   }{
     \vdashlimitsmatch \{ \LMIN~n_1, \LMAX~m_1^? \} \matches \{ \LMIN~n_2, \LMAX~\epsilon \}
   }
   \quad
   \frac{
     n_1 \geq n_2
     \qquad
     m_1 \leq m_2
   }{
     \vdashlimitsmatch \{ \LMIN~n_1, \LMAX~m_1 \} \matches \{ \LMIN~n_2, \LMAX~m_2 \}
   }


.. _match-externtype:

.. index:: function type
.. _match-functype:

関数（function）
.........

1個の :ref:`外部型 <syntax-externtype>` :math:`\ETFUNC~\functype_1` は、以下の場合に限って :math:`\ETFUNC~\functype_2` と一致します。

* :math:`\functype_1` と :math:`\functype_2` が同じ。

.. math::
   ~\\[-1ex]
   \frac{
   }{
     \vdashexterntypematch \ETFUNC~\functype \matches \ETFUNC~\functype
   }


.. index:: table type, limits, element type
.. _match-tabletype:

テーブル（table）
......

1個の :ref:`外部型 <syntax-externtype>` :math:`\ETTABLE~(\limits_1~\elemtype_1)` は、以下の場合に限って :math:`\ETTABLE~(\limits_2~\elemtype_2)` と一致します。

* 制限 :math:`\limits_1` が :math:`\limits_2` と :ref:`一致 <match-limits>` する。

* :math:`\elemtype_1` と :math:`\elemtype_2` が同じ。

.. math::
   \frac{
     \vdashlimitsmatch \limits_1 \matches \limits_2
   }{
     \vdashexterntypematch \ETTABLE~(\limits_1~\elemtype) \matches \ETTABLE~(\limits_2~\elemtype)
   }


.. index:: memory type, limits
.. _match-memtype:

メモリー（memory）
........

1個の :ref:`外部型 <syntax-externtype>` :math:`\ETMEM~\limits_1` は、以下の場合に限って :math:`\ETMEM~\limits_2` と一致します。

* 制限 :math:`\limits_1` が :math:`\limits_2` と :ref:`一致 <match-limits>` する。

.. math::
   \frac{
     \vdashlimitsmatch \limits_1 \matches \limits_2
   }{
     \vdashexterntypematch \ETMEM~\limits_1 \matches \ETMEM~\limits_2
   }


.. index:: global type, value type, mutability
.. _match-globaltype:

グローバル（global）
.......

1個の :ref:`外部型 <syntax-externtype>` :math:`\ETGLOBAL~\globaltype_1` は、以下の場合に限って :math:`\ETGLOBAL~\globaltype_2` と一致します。

* :math:`\globaltype_1` と :math:`\globaltype_2` が同じ。

.. math::
   ~\\[-1ex]
   \frac{
   }{
     \vdashexterntypematch \ETGLOBAL~\globaltype \matches \ETGLOBAL~\globaltype
   }


.. index:: ! allocation, store, address
.. _alloc:

アロケーション（allocation）
~~~~~~~~~~

「:ref:`関数 <syntax-funcinst>`」「:ref:`テーブル <syntax-tableinst>`」「:ref:`メモリー <syntax-meminst>`」「:ref:`グローバル <syntax-globalinst>`」の新しいインスタンスは、以下の補助関数で定義されるように :ref:`ストア <syntax-store>` :math:`S` に「アロケーション」できます。

.. index:: function, function instance, function address, module instance, function type
.. _alloc-func:

:ref:`関数 <syntax-funcinst>`
..................................

1. :math:`\func` を「アロケーションする :ref:`モジュールインスタンス <syntax-moduleinst>`」とし、:math:`\moduleinst` を :ref:`関数 <syntax-func>` とする。

2. :math:`a` を :math:`S` における最初の自由な :ref:`関数アドレス <syntax-funcaddr>` とする。

3. :math:`\functype` を :ref:`関数型 <syntax-functype>` :math:`\moduleinst.\MITYPES[\func.\FTYPE]` とする。

4. :math:`\funcinst` を :ref:`関数インスタンス <syntax-funcinst>` :math:`\{ \FITYPE~\functype, \FIMODULE~\moduleinst, \FICODE~\func \}` とする。

5. :math:`\funcinst` を :math:`S` の |SFUNCS| の末尾に追加する。

6. :math:`a` を返す。

.. math::
   ~\\[-1ex]
   \begin{array}{rlll}
   \allocfunc(S, \func, \moduleinst) &=& S', \funcaddr \\[1ex]
   \mbox{ただし:} \hfill \\
   \funcaddr &=& |S.\SFUNCS| \\
   \functype &=& \moduleinst.\MITYPES[\func.\FTYPE] \\
   \funcinst &=& \{ \FITYPE~\functype, \FIMODULE~\moduleinst, \FICODE~\func \} \\
   S' &=& S \compose \{\SFUNCS~\funcinst\} \\
   \end{array}


.. index:: host function, function instance, function address, function type
.. _alloc-hostfunc:

:ref:`ホスト関数 <syntax-hostfunc>`
.......................................

1. :math:`\hostfunc` を「アロケーションする :ref:`関数型 <syntax-functype>`」とし、:math:`\functype` を :ref:`ホスト関数 <syntax-hostfunc>` とする。

2. :math:`a` を :math:`S` における最初の自由な :ref:`関数アドレス <syntax-funcaddr>` とする。

3. :math:`\funcinst` を :ref:`関数インスタンス <syntax-funcinst>` :math:`\{ \FITYPE~\functype, \FIHOSTCODE~\hostfunc \}` とする。

4. :math:`\funcinst` を :math:`S` の |SFUNCS| の末尾に追加する。

5. :math:`a` を返す。

.. math::
   ~\\[-1ex]
   \begin{array}{rlll}
   \allochostfunc(S, \functype, \hostfunc) &=& S', \funcaddr \\[1ex]
   \mbox{ただし:} \hfill \\
   \funcaddr &=& |S.\SFUNCS| \\
   \funcinst &=& \{ \FITYPE~\functype, \FIHOSTCODE~\hostfunc \} \\
   S' &=& S \compose \{\SFUNCS~\funcinst\} \\
   \end{array}

.. note::
   ホスト関数は、WebAssemblyセマンティクス自身によってアロケーションされることは決してありませんが、:ref:`エンベダー <embedder>` によってアロケーションされる可能性はあります。


.. index:: table, table instance, table address, table type, limits
.. _alloc-table:

:ref:`テーブル <syntax-tableinst>`
................................

1. :math:`\tabletype` を「アロケーションする :ref:`テーブル型 <syntax-tabletype>`」とする。

2. :math:`(\{\LMIN~n, \LMAX~m^?\}~\elemtype)` を :ref:`テーブル型 <syntax-tabletype>` :math:`\tabletype` の構造（structure）とする。

3. :math:`a` を :math:`S` における最初の自由な :ref:`テーブルアドレス <syntax-tableaddr>` とする。

4. :math:`\tableinst` を、:math:`n` 個の空要素を持つ :ref:`テーブルインスタンス <syntax-tableinst>` :math:`\{ \TIELEM~(\epsilon)^n, \TIMAX~m^? \}` とする。

5. :math:`\tableinst` を :math:`S` の |STABLES| の末尾に追加する。

6. :math:`a` を返す。

.. math::
   \begin{array}{rlll}
   \alloctable(S, \tabletype) &=& S', \tableaddr \\[1ex]
   \mbox{ただし:} \hfill \\
   \tabletype &=& \{\LMIN~n, \LMAX~m^?\}~\elemtype \\
   \tableaddr &=& |S.\STABLES| \\
   \tableinst &=& \{ \TIELEM~(\epsilon)^n, \TIMAX~m^? \} \\
   S' &=& S \compose \{\STABLES~\tableinst\} \\
   \end{array}


.. index:: memory, memory instance, memory address, memory type, limits, byte
.. _alloc-mem:

:ref:`メモリー <syntax-meminst>`
................................

1. :math:`\memtype` を「アロケーションする :ref:`メモリー型 <syntax-memtype>`」とする。

2. :math:`\{\LMIN~n, \LMAX~m^?\}` を :ref:`メモリー型 <syntax-memtype>` :math:`\memtype` の構造とする。

3. :math:`a` を :math:`S` における最初の自由な :ref:`メモリーアドレス <syntax-memaddr>` とする。

4. :math:`\meminst` を、ゼロ化した :ref:`バイト <syntax-byte>` を :math:`n` ページ含む :ref:`メモリーインスタンス <syntax-meminst>` :math:`\{ \MIDATA~(\hex{00})^{n \cdot 64\,\F{Ki}}, \MIMAX~m^? \}` とする。

5. :math:`\meminst` を :math:`S` の |SMEMS| の末尾に追加する。

6. :math:`a` を返す。

.. math::
   \begin{array}{rlll}
   \allocmem(S, \memtype) &=& S', \memaddr \\[1ex]
   \mbox{ただし:} \hfill \\
   \memtype &=& \{\LMIN~n, \LMAX~m^?\} \\
   \memaddr &=& |S.\SMEMS| \\
   \meminst &=& \{ \MIDATA~(\hex{00})^{n \cdot 64\,\F{Ki}}, \MIMAX~m^? \} \\
   S' &=& S \compose \{\SMEMS~\meminst\} \\
   \end{array}


.. index:: global, global instance, global address, global type, value type, mutability, value
.. _alloc-global:

:ref:`グローバル <syntax-globalinst>`
..................................

1. :math:`\globaltype` を「アロケーションする :ref:`グローバル型 <syntax-globaltype>`」とし、:math:`\val` をそのグローバルを初期化するときの :ref:`値 <syntax-val>` とする。

2. :math:`\mut~t` を :ref:`グローバル型 <syntax-globaltype>` :math:`\globaltype` の構造とする。

3. :math:`a` を :math:`S` の最初の自由な :ref:`グローバルアドレス <syntax-globaladdr>` とする。

4. :math:`\globalinst` を :ref:`グローバルインスタンス <syntax-globalinst>` :math:`\{ \GIVALUE~\val, \GIMUT~\mut \}` とする。

5. :math:`\globalinst` を :math:`S` の |SGLOBALS| の末尾に追加する。

6. :math:`a` を返す。

.. math::
   \begin{array}{rlll}
   \allocglobal(S, \globaltype, \val) &=& S', \globaladdr \\[1ex]
   \mbox{ただし:} \hfill \\
   \globaltype &=& \mut~t \\
   \globaladdr &=& |S.\SGLOBALS| \\
   \globalinst &=& \{ \GIVALUE~\val, \GIMUT~\mut \} \\
   S' &=& S \compose \{\SGLOBALS~\globalinst\} \\
   \end{array}


.. index:: table, table instance, table address, grow, limits
.. _grow-table:

:ref:`テーブル <syntax-tableinst>` を成長させる
........................................

1. :math:`\tableinst` を「成長させる :ref:`テーブルインスタンス <syntax-tableinst>`」とし、:math:`n` を「成長させる要素数」とする。

2. :math:`\X{len}` を、:math:`\tableinst.\TIELEM` の長さに追加される :math:`n` とする。

3. :math:`\X{len}` が :math:`2^{32}` より大きいか等しい場合は失敗する。

4. :math:`\tableinst.\TIMAX` が空でなく、その値が :math:`\X{len}` より小さい場合は失敗する。

5. :math:`n` 個の空要素を :math:`\tableinst.\TIELEM` に追加する。

.. math::
   \begin{array}{rllll}
   \growtable(\tableinst, n) &=& \tableinst \with \TIELEM = \tableinst.\TIELEM~(\epsilon)^n \\
     && (
       \begin{array}[t]{@{}r@{~}l@{}}
       \iff & \X{len} = n + |\tableinst.\TIELEM| \\
       \wedge & \X{len} < 2^{32} \\
       \wedge & (\tableinst.\TIMAX = \epsilon \vee \X{len} \leq \tableinst.\TIMAX)) \\
       \end{array} \\
   \end{array}


.. index:: memory, memory instance, memory address, grow, limits
.. _grow-mem:

:ref:`メモリー <syntax-meminst>` を成長させる
........................................

1. :math:`\meminst` を「成長させる :ref:`メモリーインスタンス <syntax-meminst>`」とし、:math:`n` を「成長させる :ref:`ページ <page-size>` 数」とする。

2. アサーション: :math:`\meminst.\MIDATA` の長さが :ref:`ページサイズ <page-size>` :math:`64\,\F{Ki}` で割り切れる。

3. :math:`\X{len}` を「:ref:`ページサイズ <page-size>` :math:`64\,\F{Ki}` で割り切れる長さ :math:`\meminst.\MIDATA` に追加される :math:`n`」とする。

4. :math:`\X{len}` が :math:`2^{16}` より大きい場合は失敗する。

5. :math:`\meminst.\MIMAX` が空でなく、その値が :math:`\X{len}` より小さい場合は失敗する。

6. 値 :math:`\hex{00}` を持つ :math:`64\,\F{Ki}` :ref:`バイト <syntax-byte>` を :math:`\meminst.\MIDATA` の末尾に :math:`n` 回追加する。

.. math::
   \begin{array}{rllll}
   \growmem(\meminst, n) &=& \meminst \with \MIDATA = \meminst.\MIDATA~(\hex{00})^{n \cdot 64\,\F{Ki}} \\
     && (
       \begin{array}[t]{@{}r@{~}l@{}}
       \iff & \X{len} = n + |\meminst.\MIDATA| / 64\,\F{Ki} \\
       \wedge & \X{len} \leq 2^{16} \\
       \wedge & (\meminst.\MIMAX = \epsilon \vee \X{len} \leq \meminst.\MIMAX)) \\
       \end{array} \\
   \end{array}


.. index:: module, module instance, function instance, table instance, memory instance, global instance, export instance, function address, table address, memory address, global address, function index, table index, memory index, global index, type, function, table, memory, global, import, export, external value, external type, matching
.. _alloc-module:

:ref:`モジュール <syntax-moduleinst>`
..................................

:ref:`モジュール <syntax-module>` のアロケーション関数では、そのモジュールの the :ref:`インポート <syntax-import>` ベクタに :ref:`一致 <match-externtype>` すると仮定される適切な :ref:`外部値 <syntax-externval>` リストと、そのモジュールの :ref:`グローバル <syntax-global>` を初期化する :ref:`値 <syntax-val>` のリストをそれぞれ必要とします。

1. :math:`\module` を「アロケーションする :ref:`モジュール <syntax-module>`」とし、:math:`\externval_{\F{im}}^\ast` を「そのモジュールのインポートを提供する :ref:`外部値 <syntax-externval>` のベクタ」とし、:math:`\val^\ast` を「そのモジュールの :ref:`グローバル <syntax-global>` を初期化する :ref:`値 <syntax-val>`」とする。

2. :math:`\module.\MFUNCS` 内の :ref:`関数 <syntax-func>` :math:`\func_i` ごとに以下を実行する。

   a. :math:`\funcaddr_i` を「以下で定義される :ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\moduleinst` を :ref:`アロケーション <alloc-func>` して得られる :ref:`関数アドレス <syntax-funcaddr>`」とする。

3. :math:`\module.\MTABLES` 内の :ref:`テーブル <syntax-table>` :math:`\table_i` ごとに以下を実行する。

   a. :math:`\tableaddr_i` を「:math:`\table_i.\TTYPE` を :ref:`アロケーション <alloc-table>` して得られる :ref:`テーブルアドレス <syntax-tableaddr>`」とする。

4. :math:`\module.\MMEMS` 内の :ref:`メモリー <syntax-mem>` :math:`\mem_i` ごとに以下を実行する。

   a. :math:`\memaddr_i` を「:math:`\mem_i.\MTYPE` を :ref:`アロケーション <alloc-mem>` して得られる :ref:`メモリーアドレス <syntax-memaddr>`」とする。

5. :math:`\module.\MGLOBALS` 内の :ref:`グローバル <syntax-global>` :math:`\global_i` ごとに以下を実行する。

   a. :math:`\globaladdr_i` を「:math:`\global_i.\GTYPE` を初期値 :math:`\val^\ast[i]` で :ref:`アロケーション <alloc-global>` して得られる :ref:`グローバルアドレス <syntax-globaladdr>`」とする。

6. :math:`\funcaddr^\ast` を「:ref:`関数アドレス <syntax-funcaddr>` :math:`\funcaddr_i` をインデックス順に結合したもの」とする。

7. :math:`\tableaddr^\ast` を「:ref:`テーブルアドレス <syntax-tableaddr>` :math:`\tableaddr_i` をインデックス順に結合したもの」とする。

8. :math:`\memaddr^\ast` を「:ref:`メモリーアドレス <syntax-memaddr>` :math:`\memaddr_i` をインデックス順に結合したもの」とする。

9. :math:`\globaladdr^\ast` を「:ref:`グローバルアドレス <syntax-globaladdr>` :math:`\globaladdr_i` をインデックス順に結合したもの」とする。

10. :math:`\funcaddr_{\F{mod}}^\ast` を「:math:`\externval_{\F{im}}^\ast` から抽出した :ref:`関数アドレス <syntax-funcaddr>` を :math:`\funcaddr^\ast` と結合したリスト」とする。

11. :math:`\tableaddr_{\F{mod}}^\ast` を「:math:`\externval_{\F{im}}` から抽出した :ref:`テーブルアドレス <syntax-tableaddr>` を :math:`\tableaddr^\ast` と結合したリスト」とする。

12. :math:`\memaddr_{\F{mod}}^\ast` を「:math:`\externval_{\F{im}}^\ast` から抽出した :ref:`メモリーアドレス <syntax-memaddr>` を :math:`\memaddr^\ast` と結合したリスト」とする。

13. :math:`\globaladdr_{\F{mod}}^\ast` を「:math:`\externval_{\F{im}}^\ast` から抽出した :ref:`グローバルアドレス <syntax-globaladdr>` を :math:`\globaladdr^\ast` と結合したリスト」とする。

14.  :math:`\module.\MEXPORTS` 内の :ref:`export <syntax-export>` :math:`\export_i` ごとに以下を実行する。

    a. :math:`\export_i` が :ref:`関数インデックス <syntax-funcidx>` :math:`x` 用の関数エクスポートである場合は、:math:`\externval_i` を :ref:`外部値 <syntax-externval>` :math:`\EVFUNC~(\funcaddr_{\F{mod}}^\ast[x])` とする。

    b. 上に該当しない場合、:math:`\export_i` が :ref:`テーブルインデックス <syntax-tableidx>` :math:`x` 用のテーブルエクスポートである場合は、:math:`\externval_i` を :ref:`外部値 <syntax-externval>` :math:`\EVTABLE~(\tableaddr_{\F{mod}}^\ast[x])` とする。

    c. 上に該当しない場合、:math:`\export_i` が :ref:`メモリーインデックス <syntax-memidx>` :math:`x` 用のメモリーエクスポートである場合は、:math:`\externval_i` を :ref:`外部値 <syntax-externval>` :math:`\EVMEM~(\memaddr_{\F{mod}}^\ast[x])` とする。

    d. 上に該当しない場合、:math:`\export_i` が :ref:`グローバルインデックス <syntax-globalidx>` :math:`x` 用のグローバルエクスポートである場合は、:math:`\externval_i` を :ref:`外部値 <syntax-externval>` :math:`\EVGLOBAL~(\globaladdr_{\F{mod}}^\ast[x])` とする。

    e. :math:`\exportinst_i` を :ref:`エクスポートインスタンス <syntax-exportinst>` :math:`\{\EINAME~(\export_i.\ENAME), \EIVALUE~\externval_i\}` とする。

15. :math:`\exportinst^\ast` を「:ref:`エクスポートインスタンス <syntax-exportinst>` :math:`\exportinst_i` をインデックス順に結合したもの」とする。

16. :math:`\moduleinst` を :ref:`モジュールインスタンス <syntax-moduleinst>` :math:`\{\MITYPES~(\module.\MTYPES),` :math:`\MIFUNCS~\funcaddr_{\F{mod}}^\ast,` :math:`\MITABLES~\tableaddr_{\F{mod}}^\ast,` :math:`\MIMEMS~\memaddr_{\F{mod}}^\ast,` :math:`\MIGLOBALS~\globaladdr_{\F{mod}}^\ast,` :math:`\MIEXPORTS~\exportinst^\ast\}` とする。

17. :math:`\moduleinst` を返す。


.. math::
   ~\\
   \begin{array}{rlll}
   \allocmodule(S, \module, \externval_{\F{im}}^\ast, \val^\ast) &=& S', \moduleinst    \end{array}

ただし:

.. math::
   \begin{array}{rlll}
   \moduleinst &=& \{~
     \begin{array}[t]{@{}l@{}}
     \MITYPES~\module.\MTYPES, \\
     \MIFUNCS~\evfuncs(\externval_{\F{im}}^\ast)~\funcaddr^\ast, \\
     \MITABLES~\evtables(\externval_{\F{im}}^\ast)~\tableaddr^\ast, \\
     \MIMEMS~\evmems(\externval_{\F{im}}^\ast)~\memaddr^\ast, \\
     \MIGLOBALS~\evglobals(\externval_{\F{im}}^\ast)~\globaladdr^\ast, \\
     \MIEXPORTS~\exportinst^\ast ~\}
     \end{array} \\[1ex]
   S_1, \funcaddr^\ast &=& \allocfunc^\ast(S, \module.\MFUNCS, \moduleinst) \\
   S_2, \tableaddr^\ast &=& \alloctable^\ast(S_1, (\table.\TTYPE)^\ast)
     \qquad\qquad\qquad~ (\where \table^\ast = \module.\MTABLES) \\
   S_3, \memaddr^\ast &=& \allocmem^\ast(S_2, (\mem.\MTYPE)^\ast)
     \qquad\qquad\qquad~ (\where \mem^\ast = \module.\MMEMS) \\
   S', \globaladdr^\ast &=& \allocglobal^\ast(S_3, (\global.\GTYPE)^\ast, \val^\ast)
     \qquad\quad~ (\where \global^\ast = \module.\MGLOBALS) \\
   \exportinst^\ast &=& \{ \EINAME~(\export.\ENAME), \EIVALUE~\externval_{\F{ex}} \}^\ast
     \quad (\where \export^\ast = \module.\MEXPORTS) \\[1ex]
   \evfuncs(\externval_{\F{ex}}^\ast) &=& (\moduleinst.\MIFUNCS[x])^\ast
     \qquad~ (\where x^\ast = \edfuncs(\module.\MEXPORTS)) \\
   \evtables(\externval_{\F{ex}}^\ast) &=& (\moduleinst.\MITABLES[x])^\ast
     \qquad (\where x^\ast = \edtables(\module.\MEXPORTS)) \\
   \evmems(\externval_{\F{ex}}^\ast) &=& (\moduleinst.\MIMEMS[x])^\ast
     \qquad (\where x^\ast = \edmems(\module.\MEXPORTS)) \\
   \evglobals(\externval_{\F{ex}}^\ast) &=& (\moduleinst.\MIGLOBALS[x])^\ast
     \qquad\!\!\! (\where x^\ast = \edglobals(\module.\MEXPORTS)) \\
   \end{array}

.. scratch
   Here, in slight abuse of notation, :math:(`\F{allocxyz}(S, \dots))^\ast` is taken to express multiple allocations with the updates to the store :math:`S` being threaded through, i.e.,

  .. math::
   \begin{array}{rlll}
   (\F{allocxyz}^\ast(S_0, \dots))^n &=& S_n, a^n \\[1ex]
   \mbox{where for all $i < n$:} \hfill \\
   S_{i+1}, a^n[i] &=& \F{allocxyz}(S_i, \dots)
   \end{array}

なお上の「:math:`\F{allocx}^\ast`」という記法は、オブジェクトの種類 :math:`X` の複数の :ref:`アロケーション <alloc>` のショートハンドであり、以下のように定義されます。

.. math::
   \begin{array}{rlll}
   \F{allocx}^\ast(S_0, X^n, \dots) &=& S_n, a^n \\[1ex]
   \mbox{where for all $i < n$:} \hfill \\
   S_{i+1}, a^n[i] &=& \F{allocx}(S_i, X^n[i], \dots)
   \end{array}

さらに3点ドット :math:`\dots` が（グローバルの場合と同様に）ひとつのシーケンス :math:`A^n` である場合は、このシーケンスの要素は点単位（pointwise）でアロケーション関数に渡されます。

.. note::
   モジュールアロケーションの定義は、それに関連付けられる関数のアロケーションを用いて互いに再帰的になります。理由は、得られるモジュールインスタンス :math:`\moduleinst` が（必要なクロージャを形成するために）関数アロケーターに1個の引数として渡されるからです。
   ある実装においては、二次ステップ内でいずれか一方を改変することでこの再帰を簡単に解除できます。

.. index:: ! instantiation, module, instance, store, trap
.. _exec-module:
.. _exec-instantiation:

インスタンス化（instantiation）
~~~~~~~~~~~~~

与えられた1個の :ref:`ストア <syntax-store>` :math:`S` において、1個の :ref:`モジュール <syntax-module>` :math:`\module` は、必要なインポートを提供する :ref:`外部値 <syntax-externval>` :math:`\externval^n` のリストを用いて以下のようにインスタンス化されます。

インスタンス化では、そのモジュールが :ref:`有効 <valid>` であることと、「提供されたインポート」と「宣言された型」が :ref:`一致する <match-externtype>` ことがチェックされ、一致しない場合はエラーで「失敗」します。
インスタンス化は、開始関数を実行したときの :ref:`トラップ <trap>` 内でも発生します。
そのような条件を通知する方法の定義は :ref:`エンベダー <embedder>` に依存します。

1. :math:`\module` が :ref:`有効 <valid-module>` でない場合は以下となる。

   a. 失敗する。

2. アサーション: :math:`\module` は、その :ref:`インポート <syntax-import>` に分類される :ref:`外部型 <syntax-externtype>` :math:`\externtype_{\F{im}}^m`  で :ref:`有効 <valid-module>` となる。

3. :ref:`インポート <syntax-import>` の個数 :math:`m` が、提供された :ref:`外部値 <syntax-externval>` の個数 :math:`n` と等しくない場合、以下となる。

   a. 失敗する。

4. :math:`\externval^n` 内の :ref:`外部値 <syntax-externval>` :math:`\externval_i` ごと、および :math:`\externtype_{\F{im}}^n` 内の :ref:`外部型 <syntax-externtype>` :math:`\externtype'_i` ごとに以下を実行する。

   a. :math:`\externval_i` が、ストア  :math:`S` 内にある :ref:`外部型 <syntax-externtype>` :math:`\externtype_i` で :ref:`有効 <valid-externval>` でない場合、以下となる。

      i. 失敗する。

   b. :math:`\externtype_i` が :math:`\externtype'_i` と :ref:`一致 <match-externtype>` しない場合、以下となる。

      i. 失敗する。

.. _exec-initvals:

5. :math:`\val^\ast` を「:math:`\module` と :math:`\externval^n` で決定される :ref:`グローバル <syntax-global>` initialization :ref:`値 <syntax-val>` のベクタ」とする。これは以下のように算出できる。

   a. :math:`\moduleinst_{\F{im}}` を「インポートしたグローバルのみで構成される補助モジュール :ref:`インスタンス <syntax-moduleinst>` :math:`\{\MIGLOBALS~\evglobals(\externval^n)\}`」とする。

   b. :math:`F_{\F{im}}` を補助 :ref:`フレーム <syntax-frame>` :math:`\{ \AMODULE~\moduleinst_{\F{im}}, \ALOCALS~\epsilon \}` とする。

   c. このフレーム :math:`F_{\F{im}}` をスタックにpushする。

   d. :math:`\module.\MGLOBALS` 内の :ref:`グローバル <syntax-global>` :math:`\global_i` ごとに以下を実行する。

      i. :math:`\val_i` を「初期化式 :math:`\global_i.\GINIT` を :ref:`評価 <exec-expr>` した結果」とする。

   e. アサーション: :ref:`バリデーション <valid-module>` において、フレーム :math:`F_{\F{im}}` が現在スタックのトップに存在する。

   f. フレーム :math:`F_{\F{im}}` をスタックからpopする。

6. :math:`\moduleinst` を「インポート :math:`\externval^n` とグローバルイニシャライザ :math:`\val^\ast` を用いてストア :math:`S` 内の :math:`\module` から :ref:`アロケーション <alloc-module>` した1個の新しいモジュールインスタンス」とし、:math:`S'` を「モジュールアロケーションによって生成された拡張済みストア」とする。

7. :math:`F` を :ref:`フレーム <syntax-frame>` :math:`\{ \AMODULE~\moduleinst, \ALOCALS~\epsilon \}` とする。

8. このフレーム :math:`F` をスタックにpushする。

9. :math:`\module.\MELEM` 内の :ref:`要素セグメント <syntax-elem>` :math:`\elem_i` ごとに以下を実行する。

    a. :math:`\X{eoval}_i` を「式 :math:`\elem_i.\EOFFSET` を :ref:`評価 <exec-expr>` した結果」とする。

    b. アサーション: :ref:`バリデーション <valid-elem>` において、:math:`\X{eoval}_i` が :math:`\I32.\CONST~\X{eo}_i` の形式を取る。

    c. :math:`\tableidx_i` を :ref:`テーブルインデックス <syntax-tableidx>` :math:`\elem_i.\ETABLE` とする。

    d. アサーション: :ref:`バリデーション <valid-elem>` において、:math:`\moduleinst.\MITABLES[\tableidx_i]` が存在する。

    e. :math:`\tableaddr_i` を :ref:`テーブルアドレス <syntax-tableaddr>` :math:`\moduleinst.\MITABLES[\tableidx_i]` とする。

    f. アサーション: :ref:`バリデーション <valid-elem>` において、:math:`S'.\STABLES[\tableaddr_i]` が存在する。

    g. :math:`\tableinst_i` を :ref:`テーブルインスタンス <syntax-tableinst>` :math:`S'.\STABLES[\tableaddr_i]` とする。

    h. :math:`\X{eend}_i` を「:math:`\X{eo}_i` に :math:`\elem_i.\EINIT` の長さを加えたもの」とする。

    i. :math:`\X{eend}_i` が :math:`\tableinst_i.\TIELEM` の長さより大きい場合は、以下となる。

       i. 失敗する。

10. :math:`\module.\MDATA` 内の :ref:`データセグメント <syntax-data>` :math:`\data_i` ごとに以下を実行する。

    a. :math:`\X{doval}_i` を「式 :math:`\data_i.\DOFFSET` を :ref:`評価 <exec-expr>` した結果」とする。

    b. アサーション: :ref:`バリデーション <valid-data>` において、:math:`\X{doval}_i` が :math:`\I32.\CONST~\X{do}_i` の形式を取る。

    c. :math:`\memidx_i` を :ref:`メモリーインデックス <syntax-memidx>` :math:`\data_i.\DMEM` とする。

    d. アサーション: :ref:`バリデーション <valid-data>` において、:math:`\moduleinst.\MIMEMS[\memidx_i]` が存在する。

    e. :math:`\memaddr_i` を :ref:`メモリーアドレス <syntax-memaddr>` :math:`\moduleinst.\MIMEMS[\memidx_i]` とする。

    f. アサーション: :ref:`バリデーション <valid-data>` において、:math:`S'.\SMEMS[\memaddr_i]` が存在する。

    g. :math:`\meminst_i` を :ref:`メモリーインスタンス <syntax-meminst>` :math:`S'.\SMEMS[\memaddr_i]` とする。

    h. :math:`\X{dend}_i` を「:math:`\X{do}_i` に :math:`\data_i.\DINIT` の長さを加えたもの」とする。

    i. If :math:`\X{dend}_i` が :math:`\meminst_i.\MIDATA` の長さより大きい場合、以下となる。

       i. 失敗する。

11. アサーション: :ref:`バリデーション <valid-module>` において、フレーム :math:`F` が現在スタックのトップに存在する。

12. このフレームをスタックからpopする。

13. :math:`\module.\MELEM` 内の :ref:`要素セグメント <syntax-elem>` :math:`\elem_i` ごとに以下を行う。

    a. :math:`\elem_i.\EINIT`（:math:`j = 0` で開始）内の :ref:`関数インデックス <syntax-funcidx>` :math:`\funcidx_{ij}` ごとに以下を行う。

       i. アサーション: :ref:`バリデーション <valid-elem>` において、:math:`\moduleinst.\MIFUNCS[\funcidx_{ij}]` が存在する。

       ii. :math:`\funcaddr_{ij}` を :ref:`関数アドレス <syntax-funcaddr>` :math:`\moduleinst.\MIFUNCS[\funcidx_{ij}]` とする。

       iii. :math:`\tableinst_i.\TIELEM[\X{eo}_i + j]` を :math:`\funcaddr_{ij}` で置き換える。

14. :math:`\module.\MDATA` 内の :ref:`データセグメント <syntax-data>` :math:`\data_i` ごとに以下を行う。

    a.  :math:`\data_i.\DINIT`（:math:`j = 0` で開始）内の :ref:`倍後 <syntax-byte>` :math:`b_{ij}` ごとに以下を行う。

       i. :math:`\meminst_i.\MIDATA[\X{do}_i + j]` を :math:`b_{ij}` で置き換える。

15. :ref:`開始関数 <syntax-start>` :math:`\module.\MSTART` が空でない場合、以下を行う。

    a. アサーション: :ref:`バリデーション <valid-start>` において、:math:`\moduleinst.\MIFUNCS[\module.\MSTART.\SFUNC]` が存在する。

    b. :math:`\funcaddr` を :ref:`関数アドレス <syntax-funcaddr>` :math:`\moduleinst.\MIFUNCS[\module.\MSTART.\SFUNC]` とする。

    c. :math:`\funcaddr` にある関数インスタンスを :ref:`呼び出す <exec-invoke>`。


.. math::
   ~\\
   \begin{array}{@{}rcll}
   \instantiate(S, \module, \externval^n) &=& S'; F;
     \begin{array}[t]{@{}l@{}}
     (\INITELEM~\tableaddr~\X{eo}~\elem.\EINIT)^\ast \\
     (\INITDATA~\memaddr~\X{do}~\data.\DINIT)^\ast \\
     (\INVOKE~\funcaddr)^? \\
     \end{array} \\
   &(\iff
     & \vdashmodule \module : \externtype_{\F{im}}^n \to \externtype_{\F{ex}}^\ast \\
     &\wedge& (S \vdashexternval \externval : \externtype)^n \\
     &\wedge& (\vdashexterntypematch \externtype \matches \externtype_{\F{im}})^n \\[1ex]
     &\wedge& \module.\MGLOBALS = \global^\ast \\
     &\wedge& \module.\MELEM = \elem^\ast \\
     &\wedge& \module.\MDATA = \data^\ast \\
     &\wedge& \module.\MSTART = \start^? \\[1ex]
     &\wedge& S', \moduleinst = \allocmodule(S, \module, \externval^n, \val^\ast) \\
     &\wedge& F = \{ \AMODULE~\moduleinst, \ALOCALS~\epsilon \} \\[1ex]
     &\wedge& (S'; F; \global.\GINIT \stepto^\ast S'; F; \val~\END)^\ast \\
     &\wedge& (S'; F; \elem.\EOFFSET \stepto^\ast S'; F; \I32.\CONST~\X{eo}~\END)^\ast \\
     &\wedge& (S'; F; \data.\DOFFSET \stepto^\ast S'; F; \I32.\CONST~\X{do}~\END)^\ast \\[1ex]
     &\wedge& (\X{eo} + |\elem.\EINIT| \leq |S'.\STABLES[\tableaddr].\TIELEM|)^\ast \\
     &\wedge& (\X{do} + |\data.\DINIT| \leq |S'.\SMEMS[\memaddr].\MIDATA|)^\ast
   \\[1ex]
     &\wedge& (\tableaddr = \moduleinst.\MITABLES[\elem.\ETABLE])^\ast \\
     &\wedge& (\memaddr = \moduleinst.\MIMEMS[\data.\DMEM])^\ast \\
     &\wedge& (\funcaddr = \moduleinst.\MIFUNCS[\start.\SFUNC])^?)
   \\[2ex]
   S; F; \INITELEM~a~i~\epsilon &\stepto&
     S; F; \epsilon \\
   S; F; \INITELEM~a~i~(x_0~x^\ast) &\stepto&
     S'; F; \INITELEM~a~(i+1)~x^\ast \\ &&
     (\iff S' = S \with \STABLES[a].\TIELEM[i] = F.\AMODULE.\MIFUNCS[x_0])
   \\[1ex]
   S; F; \INITDATA~a~i~\epsilon &\stepto&
     S; F; \epsilon \\
   S; F; \INITDATA~a~i~(b_0~b^\ast) &\stepto&
     S'; F; \INITDATA~a~(i+1)~b^\ast \\ &&
     (\iff S' = S \with \SMEMS[a].\MIDATA[i] = b_0)
   \end{array}

.. note::
   モジュールの :ref:`アロケーション <alloc-module>`、および :ref:`グローバル <syntax-global>` イニシャライザの :ref:`評価 <exec-expr>` は互いに再帰的となります。理由は、:ref:`値 <syntax-val>` :math:`\val^\ast` のグローバル初期化はモジュールアロケーターに渡されるからです。しかしこれはストア :math:`S'` や、アロケーションで返されるモジュールインスタンス :math:`\moduleinst` に依存します。
   ただし、この最近は単なる仕様上の装置に過ぎません。
   :ref:`バリデーション <valid-module>` のおかげで、初期状態のストアにあるグローバルイニシャライザを評価するシンプルなプリパス（pre-pass）から初期値を簡単に :ref:`決定 <exec-initvals>` できます。

   あらゆる失敗条件は、ストアで発生する観察可能な改変が行われる前にチェックされます。
   なおストアの改変はアトミックではありません。
   すなわちストアの改変は、他のスレッドによってインターリーブされる可能性のある個別のステップで発生します。

   :ref:`定数式 <valid-constant>` の :ref:`評価 <exec-expr>` はストアに影響しません。


.. index:: ! invocation, module, module instance, function, export, function address, function instance, function type, value, stack, trap, store
.. _exec-invocation:

呼び出し（invocation）
~~~~~~~~~~

ある :ref:`モジュール <syntax-module>` の :ref:`インスタンス化 <exec-instantiation>` が完了すると、エクスポートされているあらゆる外部関数を、「その :ref:`ストア <syntax-store>` :math:`S` 内にある :ref:`関数アドレス <syntax-funcaddr>`」および「引数 :ref:`値 <syntax-val>` の適切なリスト :math:`\val^\ast`」を介して「呼び出せる」ようになります。

引数が :ref:`関数型 <syntax-functype>` と一致しない場合、呼び出しが「失敗する」可能性があります。
呼び出しは :ref:`トラップ <trap>` によって発生することもあります。
そのような条件を通知する方法の定義は :ref:`エンベダー <embedder>` に依存します。

.. note::
   呼び出しが実行される前に :ref:`エンベダー <embedder>` APIが静的または動的に自身の型チェックを実行する場合、トラップ以外の失敗は発生しません。

呼び出しでは以下のステップが実行されます。

1. アサーション: :math:`S.\SFUNCS[\funcaddr]` が存在する。

2. :math:`\funcinst` を :ref:`関数インスタンス <syntax-funcinst>` :math:`S.\SFUNCS[\funcaddr]` とする。

3. :math:`[t_1^n] \to [t_2^m]` を :ref:`関数型 <syntax-functype>` :math:`\funcinst.\FITYPE` とする。

4. 与えられた引数値の長さ :math:`|\val^\ast|` が、期待される引数の個数 :math:`n` と異なる場合、以下となる。

   a. 失敗する。

5. :math:`t_1^n` 内の :ref:`値が <syntax-valtype>` :math:`t_i` ごとに、および :math:`\val^\ast` 内でそれらに対応する :ref:`値 <syntax-val>` :math:`val_i` ごとに以下を行う。

   a. :math:`\val_i` が一部の :math:`c_i` について :math:`t_i.\CONST~c_i` でない場合、以下となる。

      i. 失敗する。

6. :math:`F` をダミー :ref:`フレーム <syntax-frame>` :math:`\{ \AMODULE~\{\}, \ALOCALS~\epsilon \}` とする。

7. このフレーム :math:`F` をスタックにpushする。

8. 値 :math:`\val^\ast` をスタックにpushする。

9. アドレス :math:`\funcaddr` の関数インスタンスを :ref:`呼び出す <exec-invoke>`。

この関数から制御が戻ると、以下のステップが実行されます。

1. アサーション: :ref:`バリデーション <valid-func>` において、:ref:`値 <syntax-val>` :math:`m` がスタックのトップに存在する。

2. :math:`\val_{\F{res}}^m` をスタックからpopする。

呼び出しの結果として、値 :math:`\val_{\F{res}}^m` が返されます。

.. math::
   ~\\[-1ex]
   \begin{array}{@{}lcl}
   \invoke(S, \funcaddr, \val^n) &=& S; F; \val^n~(\INVOKE~\funcaddr) \\
     &(\iff & S.\SFUNCS[\funcaddr].\FITYPE = [t_1^n] \to [t_2^m] \\
     &\wedge& \val^n = (t_1.\CONST~c)^n \\
     &\wedge& F = \{ \AMODULE~\{\}, \ALOCALS~\epsilon \}) \\
   \end{array}
