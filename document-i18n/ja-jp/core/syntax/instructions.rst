.. index:: ! instruction, code, stack machine, operand, operand stack
   pair: abstract syntax; instruction
.. _syntax-instr:

インストラクション
------------

WebAssemblyコードは「インストラクション」のシーケンスでできています。
WebAssemblyの計算モデルは「スタックマシン」に基づいており、インストラクションは暗黙の「オペランドスタック」上で値を操作し、引数の値を消費（pop）して、結果の値を生成または返します（push）。

スタックから取得する動的なオペランドの他に、一部のインストラクションでは静的な「即値（immediate）」引数も持てます。これは :ref:`インデックス <syntax-index>` や型アノテーションでインストラクション自身の一部に含まれるのが典型的です。

インストラクションによっては、インストラクションのネストしたシーケンスを角かっこで囲む形で :ref:`構造化 <syntax-instr-control>` するものもあります。

以下のセクションでは、インストラクションをさまざまなカテゴリに分類します。

.. index:: ! numeric instruction, value, value type, integer, floating-point, two's complement
   pair: abstract syntax; instruction
.. _syntax-sx:
.. _syntax-const:
.. _syntax-iunop:
.. _syntax-ibinop:
.. _syntax-itestop:
.. _syntax-irelop:
.. _syntax-funop:
.. _syntax-fbinop:
.. _syntax-ftestop:
.. _syntax-frelop:
.. _syntax-instr-numeric:

数値インストラクション（numeric instruction）
~~~~~~~~~~~~~~~~~~~~

数値インストラクションは、特定の :ref:`型 <syntax-valtype>` を持つ数値の :ref:`値 <syntax-value>` に対する基本操作を提供します。
これらの操作は、ハードウェアで利用できる対応操作と多くの点で一致します。

.. math::
   \begin{array}{llcl}
   \production{width} & \X{nn}, \X{mm} &::=&
     \K{32} ~|~ \K{64} \\
   \production{signedness} & \sx &::=&
     \K{u} ~|~ \K{s} \\
   \production{instruction} & \instr &::=&
     \K{i}\X{nn}\K{.}\CONST~\xref{syntax/values}{syntax-int}{\iX{\X{nn}}} ~|~
     \K{f}\X{nn}\K{.}\CONST~\xref{syntax/values}{syntax-float}{\fX{\X{nn}}} \\&&|&
     \K{i}\X{nn}\K{.}\iunop ~|~
     \K{f}\X{nn}\K{.}\funop \\&&|&
     \K{i}\X{nn}\K{.}\ibinop ~|~
     \K{f}\X{nn}\K{.}\fbinop \\&&|&
     \K{i}\X{nn}\K{.}\itestop \\&&|&
     \K{i}\X{nn}\K{.}\irelop ~|~
     \K{f}\X{nn}\K{.}\frelop \\&&|&
     \K{i}\X{nn}\K{.}\EXTEND\K{8\_s} ~|~
     \K{i}\X{nn}\K{.}\EXTEND\K{16\_s} ~|~
     \K{i64.}\EXTEND\K{32\_s} \\&&|&
     \K{i32.}\WRAP\K{\_i64} ~|~
     \K{i64.}\EXTEND\K{\_i32}\K{\_}\sx ~|~
     \K{i}\X{nn}\K{.}\TRUNC\K{\_f}\X{mm}\K{\_}\sx \\&&|&
     \K{i}\X{nn}\K{.}\TRUNC\K{\_sat\_f}\X{mm}\K{\_}\sx \\&&|&
     \K{f32.}\DEMOTE\K{\_f64} ~|~
     \K{f64.}\PROMOTE\K{\_f32} ~|~
     \K{f}\X{nn}\K{.}\CONVERT\K{\_i}\X{mm}\K{\_}\sx \\&&|&
     \K{i}\X{nn}\K{.}\REINTERPRET\K{\_f}\X{nn} ~|~
     \K{f}\X{nn}\K{.}\REINTERPRET\K{\_i}\X{nn} \\&&|&
     \dots \\
   \production{integer unary operator} & \iunop &::=&
     \K{clz} ~|~
     \K{ctz} ~|~
     \K{popcnt} \\
   \production{integer binary operator} & \ibinop &::=&
     \K{add} ~|~
     \K{sub} ~|~
     \K{mul} ~|~
     \K{div\_}\sx ~|~
     \K{rem\_}\sx \\&&|&
     \K{and} ~|~
     \K{or} ~|~
     \K{xor} ~|~
     \K{shl} ~|~
     \K{shr\_}\sx ~|~
     \K{rotl} ~|~
     \K{rotr} \\
   \production{floating-point unary operator} & \funop &::=&
     \K{abs} ~|~
     \K{neg} ~|~
     \K{sqrt} ~|~
     \K{ceil} ~|~
     \K{floor} ~|~
     \K{trunc} ~|~
     \K{nearest} \\
   \production{floating-point binary operator} & \fbinop &::=&
     \K{add} ~|~
     \K{sub} ~|~
     \K{mul} ~|~
     \K{div} ~|~
     \K{min} ~|~
     \K{max} ~|~
     \K{copysign} \\
   \production{integer test operator} & \itestop &::=&
     \K{eqz} \\
   \production{integer relational operator} & \irelop &::=&
     \K{eq} ~|~
     \K{ne} ~|~
     \K{lt\_}\sx ~|~
     \K{gt\_}\sx ~|~
     \K{le\_}\sx ~|~
     \K{ge\_}\sx \\
   \production{floating-point relational operator} & \frelop &::=&
     \K{eq} ~|~
     \K{ne} ~|~
     \K{lt} ~|~
     \K{gt} ~|~
     \K{le} ~|~
     \K{ge} \\
   \end{array}

数値インストラクションは :ref:`値型 <syntax-valtype>` で分類されます。
型ごとに、多くのサブカテゴリで区別されます。

* *定数（constant）*: 静的な定数を1つ返す。

* *単項操作（unary operation）*: 1つのオペランドを消費し、それに対応する型を持つ結果を1つ生成する。

* *二項操作（binary operation）*: 2つのオペランドを消費し、それに対応する型を持つ結果を1つ生成する。

* *テスト（test）*: 対応する型を持つ1つのオペランドを消費し、ブーリアン整数値を1つ生成する。

* *比較（comparison）*: 対応する型をそれぞれ持つ2つのオペランドを消費し、ブーリアン整数値を1つ生成する。

* *変換（conversion）*: ある型をもつ値を1つ消費し、別の結果を生成する。
  （:math:`\K{\_}` の後ろにある型が変換前の型）

整数インストラクションの中には、2種類のフレーバーのいずれかを持つものもあります。符号付き記法 |sx| では、オペランドが  :ref:`符号なし <syntax-uint>` 整数または :ref:`符号あり <syntax-sint>` 整数のいずれかとして :ref:`解釈されます <aux-signed>`。
その他の整数インストラクションでは、2の補数を用いて符号付きと解釈した値は、符号の有無にかかわらず同じに振る舞うとみなされます。

.. _syntax-unop:
.. _syntax-binop:
.. _syntax-testop:
.. _syntax-relop:
.. _syntax-cvtop:

本仕様での記法
...........

場合によっては、以下の文法ショートハンドに沿って演算子をグループ化すると便利なこともあります。

.. math::
   \begin{array}{llll}
   \production{unary operator} & \unop &::=&
     \iunop ~|~
     \funop ~|~
     \EXTEND{N}\K{\_s} \\
   \production{binary operator} & \binop &::=& \ibinop ~|~ \fbinop \\
   \production{test operator} & \testop &::=& \itestop \\
   \production{relational operator} & \relop &::=& \irelop ~|~ \frelop \\
   \production{conversion operator} & \cvtop &::=&
     \WRAP ~|~
     \EXTEND ~|~
     \TRUNC ~|~
     \TRUNC\K{\_sat} ~|~
     \CONVERT ~|~
     \DEMOTE ~|~
     \PROMOTE ~|~
     \REINTERPRET \\
   \end{array}


.. index:: ! parametric instruction, value type
   pair: abstract syntax; instruction
.. _syntax-instr-parametric:

パラメーターインストラクション（parametric instruction）
~~~~~~~~~~~~~~~~~~~~~~~

ここに分類されるインストラクションは、操作するオペランドで任意の :ref:`値型 <syntax-valtype>` を利用できます。

.. math::
   \begin{array}{llcl}
   \production{instruction} & \instr &::=&
     \dots \\&&|&
     \DROP \\&&|&
     \SELECT
   \end{array}

|DROP| インストラクションは、単に1個のオペランドを破棄します。

|SELECT| インストラクションは、第3オペランドがゼロかそうでないかに応じて、第1オペランドと第2オペランドのいずれかを選択します。

.. index:: ! variable instruction, local, global, local index, global index
   pair: abstract syntax; instruction
.. _syntax-instr-variable:

変数インストラクション（variable instruction）
~~~~~~~~~~~~~~~~~~~~~

変数インストラクションは、:ref:`ローカル <syntax-local>` 変数や :ref:`グローバル <syntax-global>` 変数へのアクセスに関連します。

.. math::
   \begin{array}{llcl}
   \production{instruction} & \instr &::=&
     \dots \\&&|&
     \LOCALGET~\localidx \\&&|&
     \LOCALSET~\localidx \\&&|&
     \LOCALTEE~\localidx \\&&|&
     \GLOBALGET~\globalidx \\&&|&
     \GLOBALSET~\globalidx \\
   \end{array}

これらのインストラクションのうち、getは変数の値を取得し、setは変数に値を設定します。
|LOCALTEE| インストラクションは |LOCALSET| と似ていますが、自身の引数も返す点が異なります。

.. index:: ! memory instruction, memory, memory index, page size, little endian, trap
   pair: abstract syntax; instruction
.. _syntax-loadn:
.. _syntax-storen:
.. _syntax-memarg:
.. _syntax-instr-memory:

メモリーインストラクション（memory instruction）
~~~~~~~~~~~~~~~~~~~

このグループに分類されるインストラクションは、線形 :ref:`メモリー <syntax-mem>` に関連します。

.. math::
   \begin{array}{llcl}
   \production{memory immediate} & \memarg &::=&
     \{ \OFFSET~\u32, \ALIGN~\u32 \} \\
   \production{instruction} & \instr &::=&
     \dots \\&&|&
     \K{i}\X{nn}\K{.}\LOAD~\memarg ~|~
     \K{f}\X{nn}\K{.}\LOAD~\memarg \\&&|&
     \K{i}\X{nn}\K{.}\STORE~\memarg ~|~
     \K{f}\X{nn}\K{.}\STORE~\memarg \\&&|&
     \K{i}\X{nn}\K{.}\LOAD\K{8\_}\sx~\memarg ~|~
     \K{i}\X{nn}\K{.}\LOAD\K{16\_}\sx~\memarg ~|~
     \K{i64.}\LOAD\K{32\_}\sx~\memarg \\&&|&
     \K{i}\X{nn}\K{.}\STORE\K{8}~\memarg ~|~
     \K{i}\X{nn}\K{.}\STORE\K{16}~\memarg ~|~
     \K{i64.}\STORE\K{32}~\memarg \\&&|&
     \MEMORYSIZE \\&&|&
     \MEMORYGROW \\
   \end{array}

さまざまな :ref:`値型 <syntax-valtype>` で、|LOAD| インストラクションや |STORE| インストラクションでメモリーにアクセスできます。
これらのインストラクションはいずれも「メモリー即値（memory immediate）」 |memarg| を1つ取ります。メモリー即値はメモリアドレスの「オフセット」を1つと、期待する「アラインメント」（2の乗数で表現）を含みます。
整数の読み込みや保存では、オプションで「ストレージサイズ」も指定できます。これは、対応する値型の :ref:`ビット幅 <syntax-valtype>` より小さい値です。
読み込みの場合は、適切な振る舞いを選択するための符号拡張モード |sx| が必須です。

動的なアドレスオペランドには静的なアドレスオフセットが加えられ、33ビットの「実効アドレス（effective address）」が生成されます。この静的なアドレスオフセットは、メモリアクセス時のゼロベースのインデックスです。
すべての値は |LittleEndian|_ バイトオーダーで読み書きされます。
アクセスするメモリバイトのいずれかが、メモリの現在のサイズが暗に示すアドレス範囲を超えると :ref:`トラップ <trap>` が生成されます。

.. note::
   WebAssemblyの今後のバージョンでは、64ビットアドレス範囲を持つメモリーインストラクションが提供される可能性があります。

|MEMORYSIZE| インストラクションは、メモリの現在のサイズを返します。
|MEMORYGROW| インストラクションは、指定された増分でメモリを拡張して元のサイズを返すか、十分なメモリを割り当てられない場合は :math:`-1` を返します。
どちらのインストラクションも、:ref:`ページサイズ <page-size>` を単位として操作します。

.. note::
   WebAssemblyの現在のバージョンでは、すべてのメモリインストラクションは暗黙で :ref:`メモリー <syntax-mem>` :ref:`インデックス <syntax-memidx>` を :math:`0` として操作します。
   この制約は今後のバージョンで解除される可能性があります。

.. index:: ! control instruction, ! structured control, ! label, ! block, ! block type, ! branch, ! unwinding, result type, label index, function index, type index, vector, trap, function, table, function type, value type, type index
   pair: abstract syntax; instruction
   pair: abstract syntax; block type
   pair: block; type
.. _syntax-blocktype:
.. _syntax-nop:
.. _syntax-unreachable:
.. _syntax-block:
.. _syntax-loop:
.. _syntax-if:
.. _syntax-br:
.. _syntax-br_if:
.. _syntax-br_table:
.. _syntax-return:
.. _syntax-call:
.. _syntax-call_indirect:
.. _syntax-instr-seq:
.. _syntax-instr-control:

制御インストラクション（control instruction）
~~~~~~~~~~~~~~~~~~~~

このグループに分類されるインストラクションは、制御フローに影響します。

.. math::
   \begin{array}{llcl}
   \production{block type} & \blocktype &::=&
     \typeidx ~|~ \valtype^? \\
   \production{instruction} & \instr &::=&
     \dots \\&&|&
     \NOP \\&&|&
     \UNREACHABLE \\&&|&
     \BLOCK~\blocktype~\instr^\ast~\END \\&&|&
     \LOOP~\blocktype~\instr^\ast~\END \\&&|&
     \IF~\blocktype~\instr^\ast~\ELSE~\instr^\ast~\END \\&&|&
     \BR~\labelidx \\&&|&
     \BRIF~\labelidx \\&&|&
     \BRTABLE~\vec(\labelidx)~\labelidx \\&&|&
     \RETURN \\&&|&
     \CALL~\funcidx \\&&|&
     \CALLINDIRECT~\typeidx \\
   \end{array}

|NOP| インストラクションは「何もしない」ことを表します。

|UNREACHABLE| インストラクションは無条件に :ref:`トラップ <trap>` を生成します。

|BLOCK|、|LOOP|、|IF| インストラクションは「構造化」インストラクションです。
これらのインストラクションでは、「ブロック」と呼ばれるネストしたインストラクションシーケンスを角かっこで囲み、|END| で終了するか |ELSE| 疑似インストラクションで区切られます。
文法で規定されているように、ブロックは「正しくネスト」されていなければなりません。

構造化インストラクションは「入力」を消費し、そこに記述されている「ブロック型（block type）」に応じて「出力」をオペランドスタックに生成します。
これは、適切な :ref:`関数型 <syntax-functype>` を参照する :ref:`型インデックス <syntax-funcidx>` か、オプションの :ref:`値型 <syntax-valtype>` インライン（これは関数型 :math:`[] \to [\valtype^?]` のショートハンドです）のいずれかの形で渡されます。

構造化された制御インストラクションは、暗黙の「ラベル（label）」を導入します。
ラベルは、:ref:`ラベルインデックス <syntax-labelidx>` を用いて参照する分岐インストラクションで利用されます。
ラベルインデックスは、他の  :ref:`インデックス空間 <syntax-index>` と異なり、ネストの深さに対して相対的で、分岐インストラクションを参照しますが、インデックスを増分するとさらにその先を参照します。
このためラベルは、関連する構造化制御インストラクションの「内部から」しか参照できません。
つまり、分岐（branch）は外向きにしか行われず、分岐先の制御構造（control construct）ブロックから「breakする」ことも暗に示されています。
正確な影響についてはその制御構造に依存します。
|BLOCK| や |IF| は「前方へのジャンプ」であり、|END| にマッチしたら実行が再開されます。
|LOOP| は「後方へのジャンプ」であり、ループの先頭にジャンプします。

.. note::
   これは「構造化制御フロー」を強制するものです。
   直感的には、|BLOCK| や |IF| を対象とする分岐はC言語に似たほとんどの言語における :math:`\K{break}` 文と似た振る舞いを示しますが、|LOOP| を対象とする分岐は :math:`\K{continue}` 文と似た振る舞いになります。

分岐インストラクションはさまざまなフレーバーに分かれています。
|BR| インストラクションは無条件分岐を実行、|BRIF| インストラクションは条件分岐を実行、|BRTABLE| インストラクションはラベルベクタ（インストラクションを直接指します）へのインデックスオペランドを介した間接分岐を実行します（オペランドが境界を超えた場合はデフォルトの分岐先に分岐します）。
|RETURN| インストラクションは、ネストの最も外側のブロック（現在の関数本体を暗に示します）への無条件分岐のショートカットです。
分岐を取ると、対象の構造化制御インストラクションが入力された深さまでオペランドスタックを「巻き戻します」。
しかし、分岐はオペランドそのものも消費する場合があります。この場合、巻き戻し後にオペランドスタックに再びpushします。
前方分岐では、分岐先ブロックの型（終了したブロックによって生成された値を表します）の出力に対応するオペランドが必須となります。
後方分岐では、分岐先ブロックの型（再起動したブロックによって消費される値を表します）の出力に対応するオペランドが必須となります。

|CALL| インストラクションは、別の :ref:`関数 <syntax-func>` を呼び出し、スタックにある必要な引数を消費して、呼び出しの結果となる値を返します。
|CALLINDIRECT| インストラクションは、:ref:`テーブル <syntax-table>`にインデックスするオペランドを介して関数を間接的に呼び出します。
テーブルにはさまざまな型 |FUNCREF| の関数要素が含まれる可能性があるので、呼び出される側は、インストラクションの即値でインデックスされる :ref:`関数型 <syntax-functype>` に対して動的にチェックされ、型が一致しない場合は :ref:`トラップ <trap>` でabortされます。

.. note::
   WebAssemblyの現在のバージョンでは、|CALLINDIRECT| は暗黙で「 :ref:`テーブル <syntax-table>` の :ref:`インデックス <syntax-tableidx>` :math:`0` 」に対して操作を行います。
   この制約は今後のバージョンで解除される可能性があります。

.. index:: ! expression, constant, global, offset, element, data, instruction
   pair: abstract syntax; expression
   single: expression; constant
.. _syntax-expr:

式（expression）
~~~~~~~~~~~

:ref:`関数 <syntax-func>` の本体（body）、:ref:`グローバル <syntax-global>` な初期値、:ref:`要素 <syntax-elem>` セグメントや :ref:`データ <syntax-data>` セグメントのオフセットは「式」として与えられます。式は :ref:`インストラクション <syntax-instr>` のシーケンスで、|END| マーカーで終了します。

.. math::
   \begin{array}{llll}
   \production{expression} & \expr &::=&
     \instr^\ast~\END \\
   \end{array}

ところによっては、検証の :ref:`制約式 <valid-constant>` は「定数」が前提になっていることがあり、その場合は利用可能なインストラクションセットが制約されます。
