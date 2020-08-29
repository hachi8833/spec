.. index:: ! runtime
.. _syntax-runtime:

ランタイム構造
-----------------

WebAssemblyの抽象マシンを形成する :ref:`Store <store>` や :ref:`stack <stack>` といった「ランタイム構造」（:ref:`値 <syntax-val>` や :ref:`モジュールインスタンス <syntax-moduleinst>` など）は、追加の補助構文として精密にできています。

.. index:: ! value, constant, value type, integer, floating-point
   pair: abstract syntax; value
.. _syntax-val:

値（value）
~~~~~~

WebAssemblyの計算では、4つの基本的な :ref:`値型 <syntax-valtype>` の「値」を操作します。すなわち32ビット幅および64ビット幅の「:ref:`整数 <syntax-int>`」および「:ref:`浮動小数点データ <syntax-float>` 」です。

セマンティクスのほとんどの場所で、さまざまな型の値が使われます。
曖昧さを避けるため、値は型を明示的に示す抽象構文を用いて表現することにします。
以下のように、値を生成する |CONST| :ref:`インストラクション <syntax-const>` で同じ記法を再利用できると便利です。

.. math::
   \begin{array}{llcl}
   \production{(value)} & \val &::=&
     \I32.\CONST~\i32 \\&&|&
     \I64.\CONST~\i64 \\&&|&
     \F32.\CONST~\f32 \\&&|&
     \F64.\CONST~\f64
   \end{array}


.. index:: ! result, value, trap
   pair: abstract syntax; result
.. _syntax-result:

結果（result）
~~~~~~~

「結果」は、計算によって生じるものです。
結果は、:ref:`値 <syntax-val>` のシーケンスか :ref:`トラップ <syntax-trap>` になります。


.. math::
   \begin{array}{llcl}
   \production{(result)} & \result &::=&
     \val^\ast \\&&|&
     \TRAP
   \end{array}

.. note::
   WebAssemblyの現在のバージョンでは、結果に含まれる値は最大でも1つまでです。


.. index:: ! store, function instance, table instance, memory instance, global instance, module, allocation
   pair: abstract syntax; store
.. _syntax-store:
.. _store:

ストア（store）
~~~~~

「ストア」は、WebAssemblyプログラムによって操作されるあらゆるグローバルステートを表現します。
ストアは、抽象マシンの寿命がある間に  :ref:`アロケーション <alloc>` される「:ref:`関数 <syntax-funcinst>`」「 :ref:`テーブル <syntax-tableinst>`」「:ref:`メモリー <syntax-meminst>`」「:ref:`グローバル <syntax-globalinst>`」のあらゆるインスタンスのランタイム表現でできています。 [#gc]_

構文上は、それぞれのカテゴリごとに既存のインスタンスを列挙する :ref:`記録 <notation-record>` としてストアを定義します。

.. math::
   \begin{array}{llll}
   \production{(store)} & \store &::=& \{~
     \begin{array}[t]{l@{~}ll}
     \SFUNCS & \funcinst^\ast, \\
     \STABLES & \tableinst^\ast, \\
     \SMEMS & \meminst^\ast, \\
     \SGLOBALS & \globalinst^\ast ~\} \\
     \end{array}
   \end{array}

.. [#gc]
   実用上は、参照されなくなったストアからオブジェクトを削除するガベージコレクションのような手法が実装に適用される可能性もあります。
   しかし、そのような手法はセマンティクス上では観察できないため、本仕様書の範囲外とします。


本仕様での記法
..........

* メタ変数 :math:`S` は、コンテキストから明らかな場合はさまざまなストアを表します。

.. index:: ! address, store, function instance, table instance, memory instance, global instance, embedder
   pair: abstract syntax; function address
   pair: abstract syntax; table address
   pair: abstract syntax; memory address
   pair: abstract syntax; global address
   pair: function; address
   pair: table; address
   pair: memory; address
   pair: global; address
.. _syntax-funcaddr:
.. _syntax-tableaddr:
.. _syntax-memaddr:
.. _syntax-globaladdr:
.. _syntax-addr:

アドレス（address）
~~~~~~~~~

:ref:`ストア <syntax-store>` に含まれる「:ref:`関数インスタンス <syntax-funcinst>`」「:ref:`テーブルインスタンス <syntax-tableinst>`」「:ref:`メモリーインスタンス <syntax-meminst>`」「:ref:`グローバルインスタンス <syntax-globalinst>`」は、抽象的な「アドレス」によって参照されます。
アドレスは、ストアの個別のコンポーネントをシンプルに指し示します。

.. math::
   \begin{array}{llll}
   \production{(address)} & \addr &::=&
     0 ~|~ 1 ~|~ 2 ~|~ \dots \\
   \production{(function address)} & \funcaddr &::=&
     \addr \\
   \production{(table address)} & \tableaddr &::=&
     \addr \\
   \production{(memory address)} & \memaddr &::=&
     \addr \\
   \production{(global address)} & \globaladdr &::=&
     \addr \\
   \end{array}

:ref:`エンベダー <embedder>` は、アドレスに対応する :ref:`エクスポートされた <syntax-export>` ストアオブジェクトにidを割り当てることがあります。なお、このidはWebAssemblyコード自体の中では観察できません（:ref:`関数インスタンス <syntax-funcinst>` やイミュータブルな :ref:`グローバル <syntax-globalinst>` など）。

.. note::
   アドレスは「動的」なものであり、ランタイムのオブジェクトを一意にグローバル参照しますが、それとは対象的に :ref:`インデックス <syntax-index>` は「静的」なものであり、そのもととなる定義へのモジュールローカルな参照です。
   「メモリアドレス」 |memaddr| が記述するのは、あくまでそのストア内にあるメモリー「インスタンス」の抽象的なアドレスであり、あるメモリーインスタンス「内部」のオフセットではありません。

   割り当てられるストアオブジェクトの個数については特定の上限はありません。
   そのため、論理アドレスはいくらでも大きな自然数になる可能性があります。

.. index:: ! instance, function type, function instance, table instance, memory instance, global instance, export instance, table address, memory address, global address, index, name
   pair: abstract syntax; module instance
   pair: module; instance
.. _syntax-moduleinst:

モジュールインスタンス（module instance）
~~~~~~~~~~~~~~~~

ひとつの「モジュールインスタンス」は、ある :ref:`モジュール <syntax-module>` のランタイム表現です。
モジュールインスタンスは、モジュールを :ref:`インスタンス化 <exec-instantiation>` することで作成され、モジュールで「インポート」「定義」「エクスポート」されたあらゆるエンティティのランタイム表現がそこに集約されます。

.. math::
   \begin{array}{llll}
   \production{(module instance)} & \moduleinst &::=& \{
     \begin{array}[t]{l@{~}ll}
     \MITYPES & \functype^\ast, \\
     \MIFUNCS & \funcaddr^\ast, \\
     \MITABLES & \tableaddr^\ast, \\
     \MIMEMS & \memaddr^\ast, \\
     \MIGLOBALS & \globaladdr^\ast, \\
     \MIEXPORTS & \exportinst^\ast ~\} \\
     \end{array}
   \end{array}

それぞれのコンポーネントは、元になるモジュール内にある（インポートまたは定義された）個別の宣言に対応するランタイムインスタンスを、それぞれが持つ静的な :ref:`インデックス <syntax-index>` 順で参照します。
1つ以上の「:ref:`関数インスタンス <syntax-funcinst>`」「:ref:`テーブルインスタンス <syntax-tableinst>`」「:ref:`メモリーインスタンス <syntax-meminst>`」「:ref:`グローバルインスタンス <syntax-globalinst>`」が、:ref:`ストア <syntax-store>` 内でそれぞれに対応する :ref:`アドレス <syntax-addr>` を介して間接的に参照されます。

指定のモジュールインスタンスですべての :ref:`インスタンスをエクスポート <syntax-exportinst>` すると、それぞれが異なる :ref:`名前 <syntax-name>` を持つ点はセマンティクス上不変です。

.. index:: ! function instance, module instance, function, closure, module, ! host function, invocation
   pair: abstract syntax; function instance
   pair: function; instance
.. _syntax-hostfunc:
.. _syntax-funcinst:

関数インスタンス（function instance）
~~~~~~~~~~~~~~~~~~

ひとつの「関数インスタンス」は、ひとつの :ref:`関数 <syntax-func>` のランタイム表現です。
関数インスタンスは、元になる :ref:`モジュール <syntax-module>` の :ref:`モジュールインスタンス <syntax-moduleinst>` 上において、元の関数の効果的な「クロージャ（closure）」となります。
このモジュールインスタンスは、関数実行時に他の定義への参照を解決するのに用いられます。

.. math::
   \begin{array}{llll}
   \production{(function instance)} & \funcinst &::=&
     \{ \FITYPE~\functype, \FIMODULE~\moduleinst, \FICODE~\func \} \\ &&|&
     \{ \FITYPE~\functype, \FIHOSTCODE~\hostfunc \} \\
   \production{(host function)} & \hostfunc &::=& \dots \\
   \end{array}

「ホスト関数（host function）」は、WebAssemblyの外部で表現されるが、:ref:`インポート <syntax-import>` として :ref:`モジュール <syntax-module>` に渡される関数のことです。
ホスト関数の定義や振る舞いは本仕様書の範疇を超えます。
本仕様書における目的のため、ここではホスト関数の :ref:`呼び出し <exec-invoke-host>` は非決定論的に振る舞うと仮定しますが、その振る舞いはランタイムの一貫性を保つ特定の :ref:`制限 <exec-invoke-host>` の範囲内とします。

.. note::
   関数インスタンスはイミュータブルであり、関数インスタンスの同一性はWebAssemblyコードでは観察できません。
   しかし :ref:`エンベダー <embedder>` は関数インスタンスの :ref:`アドレス <syntax-funcaddr>` を区別する何らかの明示的または暗黙的な手段を提供する可能性があります。

.. index:: ! table instance, table, function address, table type, embedder, element segment
   pair: abstract syntax; table instance
   pair: table; instance
.. _syntax-funcelem:
.. _syntax-tableinst:

テーブルインスタンス（table instance）
~~~~~~~~~~~~~~~

「テーブルインスタンス」は、:ref:`テーブル <syntax-table>` のランタイム表現です。
テーブルインスタンスは「関数要素」のベクタをひとつ保持し、テーブル定義側の :ref:`テーブル型 <syntax-tabletype>` で指定されている場合はオプションとして最大サイズもひとつ保持します。

それぞれの関数要素は、「空」（初期化されていないテーブルエントリを表現する）であるか、:ref:`関数アドレス <syntax-funcaddr>` のいずれかになります。
関数要素の改変は、:ref:`要素セグメント <syntax-elem>` を介するか、:ref:`エンベダー <embedder>` が提供する外部の手段を用いることで可能となります。

.. math::
   \begin{array}{llll}
   \production{(table instance)} & \tableinst &::=&
     \{ \TIELEM~\vec(\funcelem), \TIMAX~\u32^? \} \\
   \production{(function element)} & \funcelem &::=&
     \funcaddr^? \\
   \end{array}

最大サイズが存在する場合は要素ベクタの長さが最大サイズを決して超えないという点は、セマンティクス上不変です。


.. note::
   ;wasWebAssemblyの今後のバージョンではこの他のテーブル要素が追加される可能性があります。

.. index:: ! memory instance, memory, byte, ! page size, memory type, embedder, data segment, instruction
   pair: abstract syntax; memory instance
   pair: memory; instance
.. _page-size:
.. _syntax-meminst:

メモリーインスタンス（memory instance）
~~~~~~~~~~~~~~~~

ひとつの「メモリーインスタンス」は、ひとつの線形 :ref:`メモリー <syntax-mem>` のランタイム表現です。
メモリーインスタンスは、:ref:`バイト <syntax-byte>` のベクタをひとつ保持し、メモリーの定義側で指定されている場合はオプションで最大サイズもひとつ保持します。

.. math::
   \begin{array}{llll}
   \production{(memory instance)} & \meminst &::=&
     \{ \MIDATA~\vec(\byte), \MIMAX~\u32^? \} \\
   \end{array}

ベクタの長さは常にWebAssemblyの「ページサイズ」の倍数となります。ページサイズは定数 :math:`65536`（省略形: :math:`64\,\F{Ki}`）で定義されます。
:ref:`メモリー型 <syntax-memtype>` と同様に、ひとつのメモリーインスタンスの最大サイズは、このページサイズを単位として与えられます。

このバイトは「:ref:`メモリーインストラクション <syntax-instr-memory>`」「:ref:`データセグメント <syntax-data>` の実行」「:ref:`エンベダー <embedder>` が提供する外部の手段」のいずれかによって改変可能です。

最大サイズが存在する場合はバイトベクタの長さが最大サイズを決して超えないという点は、セマンティクス上不変です。

.. index:: ! global instance, global, value, mutability, instruction, embedder
   pair: abstract syntax; global instance
   pair: global; instance
.. _syntax-globalinst:

グローバルインスタンス（global instance）
~~~~~~~~~~~~~~~~

ひとつの「グローバルインスタンス」は、ひとつの :ref:`グローバル <syntax-global>` 変数のランタイム表現です。
グローバルインスタンスは、個別の :ref:`値 <syntax-val>` と、ミュータブルであるかどうかを示すフラグをひとつ保持します。

.. math::
   \begin{array}{llll}
   \production{(global instance)} & \globalinst &::=&
     \{ \GIVALUE~\val, \GIMUT~\mut \} \\
   \end{array}

ミュータブルなグローバルの値は、「:ref:`変数インストラクション <syntax-instr-variable>`」「:ref:`エンベダー <embedder>` が提供する外部の手段」のいずれかによって改変可能です。

.. index:: ! export instance, export, name, external value
   pair: abstract syntax; export instance
   pair: export; instance
.. _syntax-exportinst:

エクスポートインスタンス（export instance）
~~~~~~~~~~~~~~~~

ひとつの「エクスポートインスタンス」は、ひとつの :ref:`エクスポート <syntax-export>` のランタイム表現です。
エクスポートインスタンスは、そのエクスポートの「:ref:`名前 <syntax-name>`」と、それに関連付けられる「:ref:`外部値 <syntax-externval>`」を定義します。

.. math::
   \begin{array}{llll}
   \production{(export instance)} & \exportinst &::=&
     \{ \EINAME~\name, \EIVALUE~\externval \} \\
   \end{array}


.. index:: ! external value, function address, table address, memory address, global address, store, function, table, memory, global
   pair: abstract syntax; external value
   pair: external; value
.. _syntax-externval:

外部値（external value）
~~~~~~~~~~~~~~~

ひとつの「外部値」は、インポートまたはエクスポートされるひとつのエンティティのランタイム表現です。
外部値はひとつの :ref:`アドレス <syntax-addr>` であり、:ref:`ストア <syntax-store>` で共有される「:ref:`関数インスタンス<syntax-funcinst>`」「:ref:`テーブルインスタンス <syntax-tableinst>`」「:ref:`メモリーインスタンス <syntax-meminst>`」「:ref:`グローバルインスタンス <syntax-globalinst>`」のいずれかを表します。

.. math::
   \begin{array}{llcl}
   \production{(external value)} & \externval &::=&
     \EVFUNC~\funcaddr \\&&|&
     \EVTABLE~\tableaddr \\&&|&
     \EVMEM~\memaddr \\&&|&
     \EVGLOBAL~\globaladdr \\
   \end{array}


本仕様での記法
...........

以下の補助記法は、外部値のシーケンスを定義します。
この記法では、順序を維持しながら特定の種類のエントリをフィルタします。

* :math:`\evfuncs(\externval^\ast) = [\funcaddr ~|~ (\EVFUNC~\funcaddr) \in \externval^\ast]`

* :math:`\evtables(\externval^\ast) = [\tableaddr ~|~ (\EVTABLE~\tableaddr) \in \externval^\ast]`

* :math:`\evmems(\externval^\ast) = [\memaddr ~|~ (\EVMEM~\memaddr) \in \externval^\ast]`

* :math:`\evglobals(\externval^\ast) = [\globaladdr ~|~ (\EVGLOBAL~\globaladdr) \in \externval^\ast]`


.. index:: ! stack, ! frame, ! label, instruction, store, activation, function, call, local, module instance
   pair: abstract syntax; frame
   pair: abstract syntax; label
.. _syntax-frame:
.. _syntax-label:
.. _frame:
.. _label:
.. _stack:

スタック（stack）
~~~~~

ほとんどの :ref:`インストラクション <syntax-instr>` は、:ref:`ストア <store>` のほかに暗黙の「スタック」ともやりとりを行います。
スタックには以下のようなエントリが含まれます。

* *値*: インストラクションの「オペランド」

* *ラベル*: 分岐の飛び先となるアクティブな :ref:`構造化制御インストラクション <syntax-instr-control>`

* *アクティベーション*: アクティブな :ref:`関数 <syntax-func>` 呼び出しの「コールフレーム」

これらのエントリは、プログラム実行中に任意の順序でスタック上に出現する可能性があります。
スタックのエントリは、抽象構文によって以下のように記述されます。

.. note::
   「オペランド」「制御構成体（control construct）」「呼び出し」で個別のスタックを用いてWebAssemblyのセマンティクスをモデリングすることは可能です。
   しかしこれらのスタックは互いに依存しているため、関連するスタックの高さについて追加のトラッキングが必要になります。
   本仕様書の目的上、インターリーブ表現の方がよりシンプルです。

値（value）
......

値は :ref:`値自身 <syntax-val>` によって表現されます。

ラベル（label）
......

ラベルは、引数の個数（arity） :math:`n` とラベルに関連付けられる分岐「ターゲット」を表し、ひとつの :ref:`インストラクション <syntax-instr>` シーケンスとして構文的に表現されます。

.. math::
   \begin{array}{llll}
   \production{(label)} & \label &::=&
     \LABEL_n\{\instr^\ast\} \\
   \end{array}

直感的には、:math:`\instr^\ast` は分岐が行われたときの実行の「継続」であり、元の制御構成が置き換わったものと言えます。

.. note::
   たとえば、あるループのラベルは以下のような形式になります。

   .. math::
      \LABEL_n\{\LOOP~\dots~\END\}

   このラベルへの分岐が実行されると、このループが実行され、効果的に冒頭から再実行されます。
   逆に、あるシンプルなブロックラベルは以下の形式になります。

   .. math::
      \LABEL_n\{\epsilon\}

   ここに分岐すると、空の「継続」はターゲットとなるブロックを終了し、以後のインストラクションを用いて実行を進められるようになります。

アクティベーション（activation）とフレーム（frame）
......................

アクティベーションフレームは、それぞれの関数の戻り値の個数 :math:`n` を持ち、（引数を含む） :ref:`ローカル <syntax-local>` の値を静的な :ref:`ローカルインデックス <syntax-localidx>` に対応する順序で保持し、その関数独自の :ref:`モジュールインスタンス <syntax-moduleinst>` への参照をひとつ持ちます。

.. math::
   \begin{array}{llll}
   \production{(activation)} & \X{activation} &::=&
     \FRAME_n\{\frame\} \\
   \production{(frame)} & \frame &::=&
     \{ \ALOCALS~\val^\ast, \AMODULE~\moduleinst \} \\
   \end{array}

ローカルの値は、それぞれに対応する :ref:`変数インストラクション <syntax-instr-variable>` によって改変されます。

.. _exec-expand:

本仕様での記法
...........

* メタ値 :math:`L` は、コンテキストから明らかな場合はさまざまなラベルを表します。

* メタ値 :math:`F` は、コンテキストから明らかな場合はさまざまなフレームを表します。

* 以下の補助記法の定義では、:ref:`ブロック型 <syntax-blocktype>` をひとつ取り、現在のフレームでそれが指す :ref:`関数型 <syntax-functype>` を探索します。

.. math::
   \begin{array}{lll}
   \expand_F(\typeidx) &=& F.\AMODULE.\MITYPES[\typeidx] \\
   \expand_F([\valtype^?]) &=& [] \to [\valtype^?] \\
   \end{array}


.. index:: ! administrative instructions, function, function instance, function address, label, frame, instruction, trap, call, memory, memory instance, table, table instance, element, data, segment
   pair:: abstract syntax; administrative instruction
.. _syntax-trap:
.. _syntax-invoke:
.. _syntax-init_elem:
.. _syntax-init_data:
.. _syntax-instr-admin:

管理インストラクション（administrative instruction）
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::
   このセクションは :ref:`形式的記法 <exec-notation>` にのみ関連します。

「:ref:`トラップ <trap>`」「:ref:`呼び出し <syntax-call>`」「:ref:`制御インストラクション <syntax-instr-control>`」の還元を表現するために、インストラクションの構文を拡張して以下の「管理インストラクション」を含めることにします。

.. math::
   \begin{array}{llcl}
   \production{(administrative instruction)} & \instr &::=&
     \dots \\ &&|&
     \TRAP \\ &&|&
     \INVOKE~\funcaddr \\ &&|&
     \INITELEM~\tableaddr~\u32~\funcidx^\ast \\ &&|&
     \INITDATA~\memaddr~\u32~\byte^\ast \\ &&|&
     \LABEL_n\{\instr^\ast\}~\instr^\ast~\END \\ &&|&
     \FRAME_n\{\frame\}~\instr^\ast~\END \\
   \end{array}

|TRAP| インストラクションはトラップの発生を表現します。
トラップは、ネストしたインストラクションシーケンスをさかのぼり、最終的にプログラム全体をひとつの |TRAP| インストラクションに還元することで、突然の終了をシグナリングします。

|INVOKE| インストラクションは :ref:`関数インスタンス <syntax-funcinst>` の呼び出しが差し迫っていることを表現し、そのインストラクションの :ref:`アドレス <syntax-funcaddr>` によって識別されます。
このインストラクションは、さまざまな形式の呼び出しの取り扱いを統一します。

|INITELEM| インストラクションと |INITDATA| インストラクションは、モジュールの :ref:`インスタンス化 <exec-instantiation>` の途中でそれぞれ :ref:`要素 <syntax-elem>` セグメントと :ref:`データ <syntax-data>` セグメントの初期化を実行します。

.. note::
   インスタンス化を個別の還元ステップに分割する理由は、スレッドのような将来の拡張と互換性のあるセマンティクスを提供するためです。

|LABEL| インストラクションと |FRAME| インストラクションは、「:ref:`スタック上 <exec-notation>`」の :ref:`ラベル <syntax-label>` と :ref:`フレーム <syntax-frame>` をそれぞれモデリングします。
さらにこの管理構文は、元の「:ref:`構造化制御インストラクション <syntax-instr-control>`」または「 :ref:`関数本体 <syntax-func>`」、および |END| マーカーで終わるそれらの :ref:`インストラクションシーケンス <syntax-instr-seq>`」を維持します。
このようにして、ネストの内側のインストラクションシーケンスの終了が、その外側のシーケンスの一部である場合に認識されます。

.. note::
   たとえば、|BLOCK| の :ref:`還元規則 <exec-block>` は以下のようになります。

   .. math::
      \BLOCK~[t^n]~\instr^\ast~\END \quad\stepto\quad
      \LABEL_n\{\epsilon\}~\instr^\ast~\END

   これは、そのブロックをひとつのラベルインストラクションに置き換えるもので、ラベルをスタックに「push」すると解釈できます。
   |END| に到達すると（すなわち内側のインストラクションシーケンスが空シーケンスに還元されると、または結果の値を表現する :math:`n` 個の |CONST| インストラクションのシーケンスに還元されると）、|LABEL| インストラクションはその :ref:`還元規則 <exec-label>` から丁重に排除されます。

   .. math::
      \LABEL_m\{\instr^\ast\}~\val^n~\END \quad\stepto\quad \val^n

   これは、そのラベルをスタックから削除して、ローカルで積算されたオペランド値だけをスタックに残すことと解釈できます。

.. commented out
   Both rules can be seen in concert in the following example:

   .. math::
      \begin{array}{@{}ll}
      & (\F32.\CONST~1)~\BLOCK~[]~(\F32.\CONST~2)~\F32.\NEG~\END~\F32.\ADD \\
      \stepto & (\F32.\CONST~1)~\LABEL_0\{\}~(\F32.\CONST~2)~\F32.\NEG~\END~\F32.\ADD \\
      \stepto & (\F32.\CONST~1)~\LABEL_0\{\}~(\F32.\CONST~{-}2)~\END~\F32.\ADD \\
      \stepto & (\F32.\CONST~1)~(\F32.\CONST~{-}2)~\F32.\ADD \\
      \stepto & (\F32.\CONST~{-}1) \\
      \end{array}


.. index:: ! block context, instruction, branch
.. _syntax-ctxt-block:

ブロックコンテキスト（block context）
..............

:ref:`分岐 <syntax-instr-control>` の還元を示すために、以下の「ブロックコンテキスト」構文が定義され、計算の次のステップが始まる場所をマーキングするひとつの「ホール（hole）」 :math:`[\_]` を囲むラベルのカウント :math:`k` でインデックス化されます。

.. math::
   \begin{array}{llll}
   \production{(block contexts)} & \XB^0 &::=&
     \val^\ast~[\_]~\instr^\ast \\
   \production{(block contexts)} & \XB^{k+1} &::=&
     \val^\ast~\LABEL_n\{\instr^\ast\}~\XB^k~\END~\instr^\ast \\
   \end{array}

この定義によって、:ref:`分岐 <syntax-br>` インストラクションや :ref:`return <syntax-return>` インストラクションを囲むアクティブなラベルをインデックス化できるようになります。

.. note::
   たとえば、シンブルな分岐の :ref:`還元 <exec-br>` は以下のように定義できます。

   .. math::
      \LABEL_0\{\instr^\ast\}~\XB^l[\BR~l]~\END \quad\stepto\quad \instr^\ast

   ここで、そのコンテキストのホール :math:`[\_]` は分岐インストラクションでインスタンス化されます。
   分岐が発生すると、「分岐先のラベル」とラベルを継続する「それに関連付けられたインストラクションシーケンス」でこのルールが置き換えられます。
   選択されたラベルは :ref:`ラベルインデックス <syntax-labelidx>` :math:`l` で識別され、これは、周辺にある飛び越えられなければならない |LABEL| インストラクションの個数に対応します。この個数は、ひとつのブロックコンテキストのインデックス内にエンコードされている個数と正確に一致します。

.. index:: ! configuration, ! thread, store, frame, instruction, module instruction
.. _syntax-thread:
.. _syntax-config:

設定（configuration）
..............

ひとつの「設定」は、現在の :ref:`ストア <syntax-store>` と、実行中のひとつの「スレッド（thread）」からなります。

ひとつのスレッドは、現在のある :ref:`フレーム <syntax-frame>` に対して相対的に操作を行う複数の :ref:`インストラクション <syntax-instr>` によるひとつの計算であり、その計算を実行する :ref:`モジュールインスタンス <syntax-moduleinst>` （すなわち現在の関数のオリジン）を参照します。

.. math::
   \begin{array}{llcl}
   \production{(configuration)} & \config &::=&
     \store; \thread \\
   \production{(thread)} & \thread &::=&
     \frame; \instr^\ast \\
   \end{array}

.. note::
   WebAssemblyの現在のバージョンはシングルスレッドですが、今後の設定ではマルチスレッドがサポートされる可能性があります。

.. index:: ! evaluation context, instruction, trap, label, frame, value
.. _syntax-ctxt-eval:

評価コンテキスト（evaluation context）
...................

最後に、以下の「評価コンテキスト」の定義および関連する構造化ルールによって、トラップの伝搬と同様に、インストラクションシーケンスや管理形式の内部で還元を行えるようになります。

.. math::
   \begin{array}{llll}
   \production{(evaluation contexts)} & E &::=&
     [\_] ~|~
     \val^\ast~E~\instr^\ast ~|~
     \LABEL_n\{\instr^\ast\}~E~\END \\
   \end{array}

.. math::
   \begin{array}{rcl}
   S; F; E[\instr^\ast] &\stepto& S'; F'; E[{\instr'}^\ast] \\
     && (\iff S; F; \instr^\ast \stepto S'; F'; {\instr'}^\ast) \\
   S; F; \FRAME_n\{F'\}~\instr^\ast~\END &\stepto& S'; F; \FRAME_n\{F''\}~\instr'^\ast~\END \\
     && (\iff S; F'; \instr^\ast \stepto S'; F''; {\instr'}^\ast) \\[1ex]
   S; F; E[\TRAP] &\stepto& S; F; \TRAP
     \qquad (\iff E \neq [\_]) \\
   S; F; \FRAME_n\{F'\}~\TRAP~\END &\stepto& S; F; \TRAP \\
   \end{array}

還元は、あるスレッドのインストラクションシーケンスがひとつの :ref:`結果 <syntax-result>` に還元される（つまり :ref:`値 <syntax-val>` のシーケンスまたは |TRAP| に還元される）と終了します。

.. note::
   評価コンテキストにおける制約は、:math:`E[\TRAP] = \TRAP` における :math:`[\_]` や :math:`\epsilon~[\_]~\epsilon` のようにルール化されます。

   評価コンテキストにおける還元の例として、以下のインストラクションシーケンスを考えてみます。

   .. math::
       (\F64.\CONST~x_1)~(\F64.\CONST~x_2)~\F64.\NEG~(\F64.\CONST~x_3)~\F64.\ADD~\F64.\MUL

   上は :math:`E[(\F64.\CONST~x_2)~\F64.\NEG]` に分解できます。ただし、

   .. math::
      E = (\F64.\CONST~x_1)~[\_]~(\F64.\CONST~x_3)~\F64.\ADD~\F64.\MUL

   さらにこれは、ホールの内容が還元ルールの左側にマッチする評価コンテキストにおいて「唯一」可能な選択肢です。
