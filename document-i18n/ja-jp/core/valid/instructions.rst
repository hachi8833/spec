.. index:: instruction, function type, context, value, operand stack, ! polymorphism
.. _valid-instr:

インストラクション
------------

:ref:`インストラクション <syntax-instr>` は、:ref:`関数型 <syntax-functype>` :math:`[t_1^\ast] \to [t_2^\ast]` に分類され、:ref:`オペランドスタック <stack>` の操作方法を記述します。
この型は、型 :math:`t_1^\ast` の引数値を持つ必要な入力スタックを記述します。インストラクションはこの入力スタックをpopして、型 :math:`t_2^\ast` の結果値を指定の出力スタックにpushします。

.. note::
   たとえば、インストラクション :math:`\I32.\ADD` は型 :math:`[\I32~\I32] \to [\I32]` を持ち、2つの |I32| 値を消費して1つを生成します。

型付けは :ref:`インストラクションシーケンス <valid-instr-seq>` :math:`\instr^\ast` に拡張されます。
こうしたシーケンスは、インストラクションの実行による累積的な効果によってオペランドスタックの型 :math:`t_1^\ast`の値を消費し、型 :math:`t_2^\ast` の新しい値をpushする場合、:ref:`関数型 <syntax-functype>` :math:`[t_1^\ast] \to [t_2^\ast]` を1つ持ちます。

.. _polymorphism:

一部のインストラクションでは、型付けルールによって型を完全には制約できないものもあるので、複数の型を許しています。
こうしたインストラクションは「ポリモーフィック（polymorphic）」と呼ばれます。
ポリモーフィズムの種類は2つに分かれます。

* *値ポリモーフィック（value-polymorphic）*:
  1つ以上の個別のオペランドにある :ref:`値型 <syntax-valtype>` :math:`t` に制約をかけない。
  |DROP| や |SELECT| などの :ref:`パラメーターインストラクション <valid-instr-parametric>` はすべてこれに該当する。


* *スタックポリモーフィック（stack-polymorphic）*:
  インストラクションの :ref:`関数型 <syntax-functype>` :math:`[t_1^\ast] \to [t_2^\ast]` 全体（あるいはそのほとんど）に制約をかけない。
  |UNREACHABLE| や |BR| や |BRTABLE| や |RETURN| などの「無条件制御転移」を実行する :ref:`制御インストラクション <valid-instr-control>` はすべてこれに該当する。

どちらの場合も、そのプログラムで周りを包囲するために挿入される制約が満たされている限り、制約がかかっていない型や型シーケンスを任意に選べます。

.. note::
   たとえば、|SELECT| インストラクションは型 :math:`[t~t~\I32] \to [t]` で、任意の可能な :ref:`値型 <syntax-valtype>` :math:`t` に対して有効です。その結果、以下の2つのインストラクションシーケンスはいずれも |SELECT| が |I32| や |F64| にそれぞれインスタンス化される型付けにおける :math:`t` で有効となります。

   .. math::
      (\I32.\CONST~1)~~(\I32.\CONST~2)~~(\I32.\CONST~3)~~\SELECT{}

   .. math::
      (\F64.\CONST~1.0)~~(\F64.\CONST~2.0)~~(\I32.\CONST~3)~~\SELECT{}

   |UNREACHABLE| インストラクションは、値型 :math:`t_1^\ast` と値型 :math:`t_2^\ast` で可能な任意のシーケンスにおける型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効です。

   その結果、以下は |UNREACHABLE| インストラクションで型 :math:`[] \to [\I32~\I32]` を仮定することで有効となります。

   .. math::
      \UNREACHABLE~~\I32.\ADD

   それと対照的に、以下は無効となります。

   .. math::
      \UNREACHABLE~~(\I64.\CONST~0)~~\I32.\ADD

   理由は、シーケンスを十分に型付けする |UNREACHABLE| インストラクション用に選べる可能な型が存在しないためです。

.. index:: numeric instruction
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-numeric:

数値インストラクション（numeric instruction）
~~~~~~~~~~~~~~~~~~~~

.. _valid-const:

:math:`t\K{.}\CONST~c`
......................

* このインストラクションは、型 :math:`[] \to [t]` で有効。

.. math::
   \frac{
   }{
     C \vdashinstr t\K{.}\CONST~c : [] \to [t]
   }


.. _valid-unop:

:math:`t\K{.}\unop`
...................

* このインストラクションは、型 :math:`[t] \to [t]` で有効。

.. math::
   \frac{
   }{
     C \vdashinstr t\K{.}\unop : [t] \to [t]
   }


.. _valid-binop:

:math:`t\K{.}\binop`
....................

* このインストラクションは、型 :math:`[t~t] \to [t]` で有効。

.. math::
   \frac{
   }{
     C \vdashinstr t\K{.}\binop : [t~t] \to [t]
   }


.. _valid-testop:

:math:`t\K{.}\testop`
.....................

* このインストラクションは、型 :math:`[t] \to [\I32]` で有効

.. math::
   \frac{
   }{
     C \vdashinstr t\K{.}\testop : [t] \to [\I32]
   }


.. _valid-relop:

:math:`t\K{.}\relop`
....................

* このインストラクションは、型 :math:`[t~t] \to [\I32]` で有効

.. math::
   \frac{
   }{
     C \vdashinstr t\K{.}\relop : [t~t] \to [\I32]
   }


.. _valid-cvtop:

:math:`t_2\K{.}\cvtop\K{\_}t_1\K{\_}\sx^?`
..........................................

* このインストラクションは、型 :math:`[t_1] \to [t_2]` で有効

.. math::
   \frac{
   }{
     C \vdashinstr t_2\K{.}\cvtop\K{\_}t_1\K{\_}\sx^? : [t_1] \to [t_2]
   }


.. index:: parametric instructions, value type, polymorphism
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-parametric:

パラメーターインストラクション（parametric instruction）
~~~~~~~~~~~~~~~~~~~~~~~

.. _valid-drop:

:math:`\DROP`
.............

* このインストラクションは、任意の :ref:`値型 <syntax-valtype>` :math:`t` における型 :math:`[t] \to []` で有効。

.. math::
   \frac{
   }{
     C \vdashinstr \DROP : [t] \to []
   }


.. _valid-select:

:math:`\SELECT`
...............

* このインストラクションは、任意の :ref:`値型 <syntax-valtype>` :math:`t` における型 :math:`[t~t~\I32] \to [t]` で有効。

.. math::
   \frac{
   }{
     C \vdashinstr \SELECT : [t~t~\I32] \to [t]
   }

.. note::
   |DROP| と |SELECT| はいずれも :ref:`値ポリモーフィック <polymorphism>` なインストラクションです。


.. index:: variable instructions, local index, global index, context
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-variable:

変数インストラクション（variable instruction）
~~~~~~~~~~~~~~~~~~~~~

.. _valid-local.get:

:math:`\LOCALGET~x`
...................

* ローカル値 :ref:`value-polymorphic <polymorphism>` はそのコンテキスト内で定義されなければならない。

* :math:`t` は :ref:`値型 <syntax-valtype>` :math:`C.\CLOCALS[x]` とすること。

* これにより、このインストラクションは型 :math:`[] \to [t]` で有効となる。

.. math::
   \frac{
     C.\CLOCALS[x] = t
   }{
     C \vdashinstr \LOCALGET~x : [] \to [t]
   }


.. _valid-local.set:

:math:`\LOCALSET~x`
...................

* ローカル値 :math:`C.\CLOCALS[x]` はそのコンテキスト内で定義されなければならない。

* :math:`t` は :ref:`値型 <syntax-valtype>` :math:`C.\CLOCALS[x]` とすること。

* これにより、このインストラクションは型 :math:`[t] \to []` で有効となる。

.. math::
   \frac{
     C.\CLOCALS[x] = t
   }{
     C \vdashinstr \LOCALSET~x : [t] \to []
   }


.. _valid-local.tee:

:math:`\LOCALTEE~x`
...................

* ローカル値 :math:`C.\CLOCALS[x]` はそのコンテキスト内で定義されなければならない。

* :math:`t` は :ref:`値型 <syntax-valtype>` :math:`C.\CLOCALS[x]` とすること。

* これにより、このインストラクションは型 :math:`[t] \to [t]` で有効となる。

.. math::
   \frac{
     C.\CLOCALS[x] = t
   }{
     C \vdashinstr \LOCALTEE~x : [t] \to [t]
   }


.. _valid-global.get:

:math:`\GLOBALGET~x`
....................

* グローバル値 :math:`C.\CGLOBALS[x]` はそのコンテキスト内で定義されなければならない。

* :math:`\mut~t` は :ref:`グローバル型 <syntax-globaltype>` :math:`C.\CGLOBALS[x]` とすること。

* これにより、このインストラクションは型 :math:`[] \to [t]` で有効となる。

.. math::
   \frac{
     C.\CGLOBALS[x] = \mut~t
   }{
     C \vdashinstr \GLOBALGET~x : [] \to [t]
   }


.. _valid-global.set:

:math:`\GLOBALSET~x`
....................

* グローバル値 :math:`C.\CGLOBALS[x]` はそのコンテキスト内で定義されなければならない。

* :math:`\mut~t` は :ref:`グローバル型 <syntax-globaltype>` :math:`C.\CGLOBALS[x]` とすること。

* ミュータブル化可能性 :math:`\mut` は |MVAR| でなければならない。

* これにより、このインストラクションは型 :math:`[t] \to []` で有効となる

.. math::
   \frac{
     C.\CGLOBALS[x] = \MVAR~t
   }{
     C \vdashinstr \GLOBALSET~x : [t] \to []
   }


.. index:: memory instruction, memory index, context
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-memarg:
.. _valid-instr-memory:

メモリーインストラクション（memory instruction）
~~~~~~~~~~~~~~~~~~~

.. _valid-load:

:math:`t\K{.}\LOAD~\memarg`
...........................

* メモリー :math:`C.\CMEMS[0]` はそのコンテキスト内で定義されなければならない。

* アラインメント :math:`2^{\memarg.\ALIGN}` は、:math:`t` の :ref:`ビット幅 <syntax-valtype>` を :math:`8` で割った値より大きくなってはならない。

* これにより、このインストラクションは型 :math:`[\I32] \to [t]` で有効となる。

.. math::
   \frac{
     C.\CMEMS[0] = \memtype
     \qquad
     2^{\memarg.\ALIGN} \leq |t|/8
   }{
     C \vdashinstr t\K{.load}~\memarg : [\I32] \to [t]
   }


.. _valid-loadn:

:math:`t\K{.}\LOAD{N}\K{\_}\sx~\memarg`
.......................................

* メモリー :math:`C.\CMEMS[0]` はそのコンテキスト内で定義されなければならない。

* アラインメント :math:`2^{\memarg.\ALIGN}` は、:math:`N/8` より大きくなってはならない。

* これにより、このインストラクションは型 :math:`[\I32] \to [t]` で有効となる。


.. math::
   \frac{
     C.\CMEMS[0] = \memtype
     \qquad
     2^{\memarg.\ALIGN} \leq N/8
   }{
     C \vdashinstr t\K{.load}N\K{\_}\sx~\memarg : [\I32] \to [t]
   }


:math:`t\K{.}\STORE~\memarg`
............................

* メモリー :math:`C.\CMEMS[0]` はそのコンテキスト内で定義されなければならない。

* アラインメント :math:`2^{\memarg.\ALIGN}` は、:math:`t` の :ref:`ビット幅 <syntax-valtype>` を :math:`8` で割った値より大きくなってはならない。

* これにより、このインストラクションは型 :math:`[\I32~t] \to []` で有効となる。

.. math::
   \frac{
     C.\CMEMS[0] = \memtype
     \qquad
     2^{\memarg.\ALIGN} \leq |t|/8
   }{
     C \vdashinstr t\K{.store}~\memarg : [\I32~t] \to []
   }


.. _valid-storen:

:math:`t\K{.}\STORE{N}~\memarg`
...............................

* メモリー :math:`C.\CMEMS[0]` はそのコンテキスト内で定義されなければならない。

* アラインメント :math:`2^{\memarg.\ALIGN}` は、:math:`N/8` より大きくなってはならない。

* これにより、このインストラクションは型 :math:`[\I32~t] \to []` で有効となる。

.. math::
   \frac{
     C.\CMEMS[0] = \memtype
     \qquad
     2^{\memarg.\ALIGN} \leq N/8
   }{
     C \vdashinstr t\K{.store}N~\memarg : [\I32~t] \to []
   }


.. _valid-memory.size:

:math:`\MEMORYSIZE`
...................

* メモリー :math:`C.\CMEMS[0]` はそのコンテキスト内で定義されなければならない。

* これにより、このインストラクションは型 :math:`[] \to [\I32]` で有効となる。

.. math::
   \frac{
     C.\CMEMS[0] = \memtype
   }{
     C \vdashinstr \MEMORYSIZE : [] \to [\I32]
   }


.. _valid-memory.grow:

:math:`\MEMORYGROW`
...................

* メモリー :math:`C.\CMEMS[0]` はそのコンテキスト内で定義されなければならない。

* これにより、このインストラクションは型 :math:`[\I32] \to [\I32]` で有効となる。

.. math::
   \frac{
     C.\CMEMS[0] = \memtype
   }{
     C \vdashinstr \MEMORYGROW : [\I32] \to [\I32]
   }


.. index:: control instructions, structured control, label, block, branch, block type, result type, label index, function index, type index, vector, polymorphism, context
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-label:
.. _valid-instr-control:

制御インストラクション（control instruction）
~~~~~~~~~~~~~~~~~~~~

.. _valid-nop:

:math:`\NOP`
............

* このインストラクションは型 :math:`[] \to []` で有効。

.. math::
   \frac{
   }{
     C \vdashinstr \NOP : [] \to []
   }


.. _valid-unreachable:

:math:`\UNREACHABLE`
....................

* このインストラクションは、:ref:`値型 <syntax-valtype>` :math:`t_1^\ast` と :math:`t_2^\ast` の任意のシーケンスにおける型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効。

.. math::
   \frac{
   }{
     C \vdashinstr \UNREACHABLE : [t_1^\ast] \to [t_2^\ast]
   }

.. note::
   |UNREACHABLE| インストラクションは :ref:`スタックポリモーフィック <polymorphism>` です。

.. _valid-block:

:math:`\BLOCK~\blocktype~\instr^\ast~\END`
..........................................

* :ref:`ブロック型 <syntax-blocktype>` は、何らかの :ref:`関数型 <syntax-functype>` :math:`[t_1^\ast] \to [t_2^\ast]` として :ref:`有効 <valid-blocktype>` でなければならない。

* :math:`C'` の :ref:`コンテキスト <context>` は :math:`C` と同じだが、|CLABELS| ベクタに :ref:`結果型 <syntax-resulttype>` :math:`[t_2^\ast]` が事前に追加されていること。

* インストラクションシーケンス :math:`\instr^\ast` は、コンテキスト :math:`C'` において型 :math:`[t_1^\ast] \to [t_2^\ast]` で :ref:`有効 <valid-instr-seq>` でなければならない。

* これにより、合成されたインストラクションは型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     C \vdashblocktype \blocktype : [t_1^\ast] \to [t_2^\ast]
     \qquad
     C,\CLABELS\,[t_2^\ast] \vdashinstrseq \instr^\ast : [t_1^\ast] \to [t_2^\ast]
   }{
     C \vdashinstr \BLOCK~\blocktype~\instr^\ast~\END : [t_1^\ast] \to [t_2^\ast]
   }

.. note::
   :ref:`記法 <notation-extend>` :math:`C,\CLABELS\,[t^\ast]` は、インデックス :math:`0` に新しいラベル型を挿入し、残りをすべてシフトします。

.. _valid-loop:

:math:`\LOOP~\blocktype~\instr^\ast~\END`
.........................................

* :ref:`ブロック型 <syntax-blocktype>` は何らかの :ref:`関数型 <syntax-functype>` :math:`[t_1^\ast] \to [t_2^\ast]` として :ref:`有効 <valid-blocktype>` でなければならない。

* :math:`C'` の :ref:`コンテキスト <context>` は :math:`C` と同じだが、|CLABELS| ベクタに :ref:`結果型 <syntax-resulttype>` :math:`[t_1^\ast]` が事前に追加されていること。

* インストラクションシーケンス :math:`\instr^\ast` は、コンテキスト :math:`C'` において型 :math:`[t_1^\ast] \to [t_2^\ast]` で :ref:`有効 <valid-instr-seq>` でなければならない。

* これにより、合成されたインストラクションは型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     C \vdashblocktype \blocktype : [t_1^\ast] \to [t_2^\ast]
     \qquad
     C,\CLABELS\,[t_1^\ast] \vdashinstrseq \instr^\ast : [t_1^\ast] \to [t_2^\ast]
   }{
     C \vdashinstr \LOOP~\blocktype~\instr^\ast~\END : [t_1^\ast] \to [t_2^\ast]
   }

.. note::
   :ref:`記法 <notation-extend>` :math:`C,\CLABELS\,[t^\ast]` は、インデックス :math:`0` に新しいラベル型を挿入し、残りをすべてシフトします。

.. _valid-if:

:math:`\IF~\blocktype~\instr_1^\ast~\ELSE~\instr_2^\ast~\END`
.............................................................

* :ref:`ブロック型 <syntax-blocktype>` は何らかの :ref:`関数型 <syntax-functype>` :math:`[t_1^\ast] \to [t_2^\ast]` として :ref:`有効 <valid-blocktype>` でなければならない。

* :math:`C'` の :ref:`コンテキスト <context>` は :math:`C` と同じだが、|CLABELS| ベクタに :ref:`結果型 <syntax-resulttype>` :math:`[t_2^\ast]` が事前に追加されていること。

* インストラクションシーケンス :math:`\instr^\ast` は、コンテキスト :math:`C'` において型 :math:`[t_1^\ast] \to [t_2^\ast]` で :ref:`有効 <valid-instr-seq>` でなければならない。

* インストラクションシーケンス :math:`\instr_2^\ast` は、コンテキスト :math:`C'` において型 :math:`[t_1^\ast] \to [t_2^\ast]` で :ref:`有効 <valid-instr-seq>` でなければならない。

* これにより、合成されたインストラクションは型 :math:`[t_1^\ast~\I32] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     C \vdashblocktype \blocktype : [t_1^\ast] \to [t_2^\ast]
     \qquad
     C,\CLABELS\,[t_2^\ast] \vdashinstrseq \instr_1^\ast : [t_1^\ast] \to [t_2^\ast]
     \qquad
     C,\CLABELS\,[t_2^\ast] \vdashinstrseq \instr_2^\ast : [t_1^\ast] \to [t_2^\ast]
   }{
     C \vdashinstr \IF~\blocktype~\instr_1^\ast~\ELSE~\instr_2^\ast~\END : [t_1^\ast~\I32] \to [t_2^\ast]
   }

.. note::
   :ref:`記法 <notation-extend>` :math:`C,\CLABELS\,[t^\ast]` は、インデックス :math:`0` に新しいラベル型を挿入し、残りをすべてシフトします。

.. _valid-br:

:math:`\BR~l`
.............

* ラベル :math:`C.\CLABELS[l]` はそのコンテキスト内で定義されなければならない。

* :math:`C.\CLABELS[l]` は :ref:`結果型 <syntax-resulttype>` :math:`C.\CLABELS[l]` とすること。

* これにより、このインストラクションは、:ref:`値型 <syntax-valtype>` :math:`t_1^\ast` と :math:`t_2^\ast` の任意のシーケンスにおける型 :math:`[t_1^\ast~t^\ast] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     C.\CLABELS[l] = [t^\ast]
   }{
     C \vdashinstr \BR~l : [t_1^\ast~t^\ast] \to [t_2^\ast]
   }

.. note::
   :ref:`コンテキスト <context>` :math:`C` 内の :ref:`ラベルインデックス <syntax-labelidx>` 空間は、:math:`C.\CLABELS[l]` が期待どおり相対的な探索を実行するよう、最も直近のラベルを最初に含みます。

   |BR| インストラクションは :ref:`スタックポリモーフィック <polymorphism>` です。

.. _valid-br_if:

:math:`\BRIF~l`
...............

* ラベル :math:`C.\CLABELS[l]` はそのコンテキスト内で定義されなければならない。

* :math:`[t^\ast]` は :ref:`結果型 <syntax-resulttype>` :math:`C.\CLABELS[l]` とすること。

* これにより、このインストラクションは型 :math:`[t^\ast~\I32] \to [t^\ast]` で有効となる。

.. math::
   \frac{
     C.\CLABELS[l] = [t^\ast]
   }{
     C \vdashinstr \BRIF~l : [t^\ast~\I32] \to [t^\ast]
   }

.. note::
   :ref:`コンテキスト <context>` :math:`C` 内の :ref:`ラベルインデックス <syntax-labelidx>` 空間は、:math:`C.\CLABELS[l]` が期待どおり相対的な探索を実行するよう、最も直近のラベルを最初に含みます。

.. _valid-br_table:

:math:`\BRTABLE~l^\ast~l_N`
...........................

* ラベル :math:`C.\CLABELS[l_N]` はそのコンテキスト内で定義されなければならない。

* :math:`[t^\ast]` は :ref:`結果型 <syntax-resulttype>` :math:`C.\CLABELS[l_N]` とすること。

* :math:`l^\ast` 内のすべての :math:`l_i` について、ラベル :math:`C.\CLABELS[l_i]` がそのコンテキスト内で定義されていなければならない。

* :math:`l^\ast` 内のすべての :math:`l_i` について、:math:`C.\CLABELS[l_i]` が :math:`[t^\ast]` でなければならない。

* これにより、このインストラクションは、:ref:`値型 <syntax-valtype>` :math:`t_1^\ast` と :math:`t_2^\ast` の任意のシーケンスにおける型 :math:`[t_1^\ast~t^\ast~\I32] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     (C.\CLABELS[l] = [t^\ast])^\ast
     \qquad
     C.\CLABELS[l_N] = [t^\ast]
   }{
     C \vdashinstr \BRTABLE~l^\ast~l_N : [t_1^\ast~t^\ast~\I32] \to [t_2^\ast]
   }

.. note::
   :ref:`コンテキスト <context>` :math:`C` 内の :ref:`ラベルインデックス <syntax-labelidx>` 空間は、:math:`C.\CLABELS[l]` が期待どおり相対的な探索を実行するよう、最も直近のラベルを最初に含みます。

   |BRTABLE| インストラクションは :ref:`スタックポリモーフィック <polymorphism>` です。

.. _valid-return:

:math:`\RETURN`
...............

* 戻り値の型 :math:`C.\CRETURN` はそのコンテキスト内で定義されなければならない。

* :math:`[t^\ast]` は :ref:`結果型 <syntax-resulttype>` :math:`C.\CRETURN` とすること。

* これにより、このインストラクションは、:ref:`値型 <syntax-valtype>` :math:`t_1^\ast` と :math:`t_2^\ast` の任意のシーケンスにおける型 :math:`[t_1^\ast~t^\ast] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     C.\CRETURN = [t^\ast]
   }{
     C \vdashinstr \RETURN : [t_1^\ast~t^\ast] \to [t_2^\ast]
   }

.. note::
   |RETURN| インストラクションは :ref:`スタックポリモーフィック <polymorphism>` です。

   :math:`C.\CRETURN` は、関数本体でない :ref:`式 <valid-expr>` を検証するときは不在となります（:math:`\epsilon` に設定される）。
   これは、関数が何も返さない場合に空の結果型（:math:`[\epsilon]`）に設定されるものとは異なります。

.. _valid-call:

:math:`\CALL~x`
...............

* 関数 :math:`C.\CFUNCS[x]` はそのコンテキスト内で定義されなければならない。

* これにより、このインストラクションは型 :math:`C.\CFUNCS[x]` で有効となる。

.. math::
   \frac{
     C.\CFUNCS[x] = [t_1^\ast] \to [t_2^\ast]
   }{
     C \vdashinstr \CALL~x : [t_1^\ast] \to [t_2^\ast]
   }


.. _valid-call_indirect:

:math:`\CALLINDIRECT~x`
.......................

* テーブル :math:`C.\CTABLES[0]` はそのコンテキスト内で定義されなければならない。

* :math:`\limits~\elemtype` は :ref:`テーブル型 <syntax-tabletype>` :math:`C.\CTABLES[0]` とすること。

* :ref:`要素型 <syntax-elemtype>` :math:`\elemtype` は |FUNCREF| でなければならない。

* 型 :math:`C.\CTYPES[x]` はそのコンテキスト内で定義されなければならない。

* :math:`[t_1^\ast] \to [t_2^\ast]` は :ref:`関数型 <syntax-functype>` :math:`C.\CTYPES[x]` とすること。

* これにより、このインストラクションは型 :math:`[t_1^\ast~\I32] \to [t_2^\ast]` で有効となる。

.. math::
   \frac{
     C.\CTABLES[0] = \limits~\FUNCREF
     \qquad
     C.\CTYPES[x] = [t_1^\ast] \to [t_2^\ast]
   }{
     C \vdashinstr \CALLINDIRECT~x : [t_1^\ast~\I32] \to [t_2^\ast]
   }


.. index:: instruction, instruction sequence
.. _valid-instr-seq:

インストラクションシーケンス（instruction sequence）
~~~~~~~~~~~~~~~~~~~~~

インストラクションシーケンスの型付けは再帰的に定義されます。


空のインストラクションシーケンス: :math:`\epsilon`
............................................

* 空のインストラクションシーケンスは、:ref:`値型 <syntax-valtype>` :math:`t^\ast` の任意のシーケンスにおける型 :math:`[t^\ast] \to [t^\ast]` で有効となる。

.. math::
   \frac{
   }{
     C \vdashinstrseq \epsilon : [t^\ast] \to [t^\ast]
   }


空でないインストラクションシーケンス: :math:`\instr^\ast~\instr_N`
............................................................

* インストラクションシーケンス :math:`\instr^\ast` は、:ref:`値型 <syntax-valtype>` :math:`t_1^\ast` と :math:`t_2^\ast` の何らかのシーケンスにおいて型 :math:`[t_1^\ast] \to [t_2^\ast]` で有効でなければならない。

* インストラクション :math:`\instr_N` は、:ref:`値型 <syntax-valtype>` :math:`t^\ast` と :math:`t_3^\ast` の何らかのシーケンスにおいて型 :math:`[t^\ast] \to [t_3^\ast]` で有効でなければならない

* :math:`t_2^\ast = t_0^\ast~t^\ast` となるような :ref:`値型 <syntax-valtype>` :math:`t_0^\ast` のシーケンスが1つ存在しなければならない。

* これにより、合成されたインストラクションシーケンスは型 :math:`[t_1^\ast] \to [t_0^\ast~t_3^\ast]` で有効となる。

.. math::
   \frac{
     C \vdashinstrseq \instr^\ast : [t_1^\ast] \to [t_0^\ast~t^\ast]
     \qquad
     C \vdashinstr \instr_N : [t^\ast] \to [t_3^\ast]
   }{
     C \vdashinstrseq \instr^\ast~\instr_N : [t_1^\ast] \to [t_0^\ast~t_3^\ast]
   }


.. index:: expression,result type
   pair: validation; expression
   single: abstract syntax; expression
   single: expression; constant
.. _valid-expr:

式（expression）
~~~~~~~~~~~

式 :math:`\expr` は、:math:`[t^\ast]` の形式を持つ :ref:`結果型 <syntax-resulttype>` に分類されます。


:math:`\instr^\ast~\END`
........................

* インストラクションシーケンス :math:`\instr^\ast` は、何らかの :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` において型 :math:`[] \to [t^\ast]` で有効でなければならない。

* これにより、式は :ref:`結果型 <syntax-resulttype>` :math:`[t^\ast]` において有効となる。

.. math::
   \frac{
     C \vdashinstrseq \instr^\ast : [] \to [t^\ast]
   }{
     C \vdashexpr \instr^\ast~\END : [t^\ast]
   }


.. index:: ! constant
.. _valid-constant:

定数式（constant expression）
....................

* 「定数」の式 :math:`\instr^\ast~\END` において、:math:`\instr^\ast` の中にあるすべてのインストラクションは定数でなければならない。

* 定数インストラクション :math:`\instr` は以下のいずれかを満たさなければならない。

  * :math:`t.\CONST~c` の形式

  * :math:`\GLOBALGET~x` の形式（:math:`C.\CGLOBALS[x]` の場合は :math:`\CONST~t` という形式の :ref:`グローバル型 <syntax-globaltype>` でなければならない）

.. math::
   \frac{
     (C \vdashinstrconst \instr \const)^\ast
   }{
     C \vdashexprconst \instr^\ast~\END \const
   }

.. math::
   \frac{
   }{
     C \vdashinstrconst t.\CONST~c \const
   }
   \qquad
   \frac{
     C.\CGLOBALS[x] = \CONST~t
   }{
     C \vdashinstrconst \GLOBALGET~x \const
   }

.. note::
   現時点では、:ref:`グローバル値 <syntax-global>` のイニシャライザとして現れる定数式は、それを含む |GLOBALGET| インストラクションは「インポートした」グローバル値の参照しか許されない形でより強い制約がかかっています。
   これは :ref:`モジュールの検証ルール <valid-module>` において :math:`C` コンテキストをそのように制約することで強制されています。

   定数式の定義は、WebAssemblyの今後のバージョンで拡張される可能性があります。
