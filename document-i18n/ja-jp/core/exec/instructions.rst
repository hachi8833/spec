.. index:: instruction, function type, store, validation
.. _exec-instr:

インストラクション
------------

WebAssembly計算（computation）は、個別の :ref:`インストラクション <syntax-instr>` を実行することで動作します。

.. index:: numeric instruction, determinism, trap, NaN, value, value type
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-numeric:

数値インストラクション（numeric instruction）
~~~~~~~~~~~~~~~~~~~~

数値インストラクションは、一般的な :ref:`数値演算子 <exec-numeric>` の観点から定義されます。
数値インストラクションと、その基礎となる演算子との対応付けは、以下の定義で表現されます。

.. math::
   \begin{array}{lll@{\qquad}l}
   \X{op}_{\K{i}N}(n_1,\dots,n_k) &=& \F{i}\X{op}_N(n_1,\dots,n_k) \\
   \X{op}_{\K{f}N}(z_1,\dots,z_k) &=& \F{f}\X{op}_N(z_1,\dots,z_k) \\
   \end{array}

:ref:`変換演算子 <exec-cvtop>` については以下のように定義されます。

.. math::
   \begin{array}{lll@{\qquad}l}
   \X{cvtop}^{\sx^?}_{t_1,t_2}(c) &=& \X{cvtop}^{\sx^?}_{|t_1|,|t_2|}(c) \\
   \end{array}

基礎となる演算子が「パーシャル」である箇所では、結果が未定義な場合に対応するインストラクションが :ref:`トラップ <trap>` します。
基礎となる演算子が「非決定的」である箇所では、可能な複数の :ref:`NaN <syntax-nan>` 値のいずれかが返される可能性があるため、対応するインストラクションも非決定的となります。

.. note::
   たとえば :math:`\I32.\ADD` インストラクションをオペランド :math:`i_1, i_2` に適用すると :math:`\ADD_{\I32}(i_1, i_2)` が呼び出されますが、これは上の定義を介して一般的な :math:`\iadd_{32}(i_1, i_2)` に対応付けられます。
   同様に、:math:`\I64.\TRUNC\K{\_}\F32\K{\_s}` インストラクションを :math:`z` に適用すると :math:`\TRUNC^{\K{s}}_{\F32,\I64}(z)` が呼び出されますが、これは一般的な :math:`\truncs_{32,64}(z)` に対応付けられます。

.. _exec-const:

:math:`t\K{.}\CONST~c`
......................

1. 値 :math:`t.\CONST~c` をスタックにpushする。

.. note::
   |CONST| インストラクションは :ref:`値 <syntax-val>` と一致するので、このインストラクションでは形式的な還元規則は必須ではありません。

.. _exec-unop:

:math:`t\K{.}\unop`
...................

1. アサーション: :ref:`バリデーション <valid-unop>` において :ref:`値型 <syntax-valtype>` :math:`t` の値がスタックのトップに存在する。

2. スタックから値 :math:`t.\CONST~c_1` をpopする。

3. :math:`\unop_t(c_1)` が定義されている場合、以下のようになる。

   a. :math:`c` を :math:`\unop_t(c_1)` の可能な計算結果とする。

   b. 値 :math:`t.\CONST~c` をスタックにpushする。


4. 上に該当しない場合、以下のようになる。

   a. トラップする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   (t\K{.}\CONST~c_1)~t\K{.}\unop &\stepto& (t\K{.}\CONST~c)
     & (\iff c \in \unop_t(c_1)) \\
   (t\K{.}\CONST~c_1)~t\K{.}\unop &\stepto& \TRAP
     & (\iff \unop_{t}(c_1) = \{\})
   \end{array}


.. _exec-binop:

:math:`t\K{.}\binop`
....................

1. アサーション: :ref:`バリデーション <valid-binop>` において :ref:`値型 <syntax-valtype>` :math:`t` の2つの値がスタックのトップに存在する。

2. 値 :math:`t.\CONST~c_2` をスタックからpopする。

3. 値 :math:`t.\CONST~c_1` をスタックからpopする

4. :math:`\binop_t(c_1, c_2)` が定義されている場合、以下のようになる。

   a. :math:`c` を :math:`\binop_t(c_1, c_2)` の可能な計算結果とする。

   b. 値 :math:`t.\CONST~c` をスタックにpushする。

5. 上に該当しない場合、以下のようになる。

   a. トラップする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   (t\K{.}\CONST~c_1)~(t\K{.}\CONST~c_2)~t\K{.}\binop &\stepto& (t\K{.}\CONST~c)
     & (\iff c \in \binop_t(c_1,c_2)) \\
   (t\K{.}\CONST~c_1)~(t\K{.}\CONST~c_2)~t\K{.}\binop &\stepto& \TRAP
     & (\iff \binop_{t}(c_1,c2) = \{\})
   \end{array}


.. _exec-testop:

:math:`t\K{.}\testop`
.....................

1. アサーション: :ref:`バリデーション <valid-testop>` において :ref:`値型 <syntax-valtype>` :math:`t` の値がスタックのトップに存在する。

2. スタックから値 :math:`t.\CONST~c_1` をpopする。

3. :math:`c` を :math:`\testop_t(c_1)` の可能な計算結果とする。

4. 値 :math:`\I32.\CONST~c` をスタックにpushする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   (t\K{.}\CONST~c_1)~t\K{.}\testop &\stepto& (\I32\K{.}\CONST~c)
     & (\iff c = \testop_t(c_1)) \\
   \end{array}


.. _exec-relop:

:math:`t\K{.}\relop`
....................

1. アサーション: :ref:`バリデーション <valid-relop>` において :ref:`値型 <syntax-valtype>` :math:`t` の2つの値がスタックのトップに存在する

2. 値 :math:`t.\CONST~c_2` をスタックからpopする。

3. 値 :math:`t.\CONST~c_1` をスタックからpopする。

4. :math:`c` を :math:`\relop_t(c_1, c_2)` の可能な計算結果とする。

5. 値 :math:`\I32.\CONST~c` をスタックにpushする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   (t\K{.}\CONST~c_1)~(t\K{.}\CONST~c_2)~t\K{.}\relop &\stepto& (\I32\K{.}\CONST~c)
     & (\iff c = \relop_t(c_1,c_2)) \\
   \end{array}


.. _exec-cvtop:

:math:`t_2\K{.}\cvtop\K{\_}t_1\K{\_}\sx^?`
..........................................

1. アサーション: :ref:`バリデーション <valid-cvtop>` において :ref:`値型 <syntax-valtype>` :math:`t_1` の値がスタックのトップに存在する。

2. 値 :math:`t_1.\CONST~c_1` をスタックからpopする。

3. :math:`\cvtop^{\sx^?}_{t_1,t_2}(c_1)` が定義されている場合、以下のようになる。

   a. :math:`c_2` を :math:`\cvtop^{\sx^?}_{t_1,t_2}(c_1)` の可能な計算結果とする。

   b. 値 :math:`t_2.\CONST~c_2` をスタックにpushする。

4. 上に該当しない場合、以下のようになる。

   a. トラップする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   (t_1\K{.}\CONST~c_1)~t_2\K{.}\cvtop\K{\_}t_1\K{\_}\sx^? &\stepto& (t_2\K{.}\CONST~c_2)
     & (\iff c_2 \in \cvtop^{\sx^?}_{t_1,t_2}(c_1)) \\
   (t_1\K{.}\CONST~c_1)~t_2\K{.}\cvtop\K{\_}t_1\K{\_}\sx^? &\stepto& \TRAP
     & (\iff \cvtop^{\sx^?}_{t_1,t_2}(c_1) = \{\})
   \end{array}


.. index:: parametric instructions, value
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-parametric:

パラメーターインストラクション（parametric instruction）
~~~~~~~~~~~~~~~~~~~~~~~

.. _exec-drop:

:math:`\DROP`
.............

1. アサーション: :ref:`バリデーション <valid-drop>` において値が1個スタックのトップに存在する。

2. 値 :math:`\val` をスタックからpopする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   \val~~\DROP &\stepto& \epsilon
   \end{array}


.. _exec-select:

:math:`\SELECT`
...............

1. アサーション: :ref:`バリデーション <valid-select>` において :ref:`値型 <syntax-valtype>` |I32| の値がスタックのトップに存在する。

2. 値 :math:`\I32.\CONST~c` をスタックからpopする。

3. アサーション: :ref:`バリデーション <valid-select>` において2つの値（同じ :ref:`値型 <syntax-valtype>`）がスタックのトップに存在する。

4. 値 :math:`\val_2` をスタックからpopする。

5. 値 :math:`\val_1` をスタックからpopする。

6. :math:`c` が :math:`0` でない場合、以下のようになる。

   a. 値 :math:`\val_1` をスタックにpushして戻す。

7. 上に該当しない場合、以下のようになる。

   a. 値 :math:`\val_2` をスタックにpushして戻す。

.. math::
   \begin{array}{lcl@{\qquad}l}
   \val_1~\val_2~(\I32\K{.}\CONST~c)~\SELECT &\stepto& \val_1
     & (\iff c \neq 0) \\
   \val_1~\val_2~(\I32\K{.}\CONST~c)~\SELECT &\stepto& \val_2
     & (\iff c = 0) \\
   \end{array}


.. index:: variable instructions, local index, global index, address, global address, global instance, store, frame, value
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-variable:

変数インストラクション（variable instruction）
~~~~~~~~~~~~~~~~~~~~~

.. _exec-local.get:

:math:`\LOCALGET~x`
...................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-local.get>` において :math:`F.\ALOCALS[x]` が存在する。

3. :math:`\val` を値 :math:`F.\ALOCALS[x]` とする。

4. 値 :math:`\val` をスタックにpushする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   F; (\LOCALGET~x) &\stepto& F; \val
     & (\iff F.\ALOCALS[x] = \val) \\
   \end{array}


.. _exec-local.set:

:math:`\LOCALSET~x`
...................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-local.set>` において :math:`F.\ALOCALS[x]` が存在する。

3. アサーション: :ref:`バリデーション <valid-local.set>` において値が1個スタックのトップに存在する。

4. 値 :math:`\val` をスタックからpopする。

5. :math:`F.\ALOCALS[x]` を値 :math:`\val` で置き換える。

.. math::
   \begin{array}{lcl@{\qquad}l}
   F; \val~(\LOCALSET~x) &\stepto& F'; \epsilon
     & (\iff F' = F \with \ALOCALS[x] = \val) \\
   \end{array}


.. _exec-local.tee:

:math:`\LOCALTEE~x`
...................

3. アサーション: :ref:`バリデーション <valid-local.tee>` において値が1個スタックのトップに存在する。

2. 値 :math:`\val` をスタックからpopする。

3. 値 :math:`\val` をスタックにpushする。

4. 値 :math:`\val` をスタックにpushする。

5. インストラクション :math:`(\LOCALSET~x)` を :ref:`実行 <exec-local.set>` する。

.. math::
   \begin{array}{lcl@{\qquad}l}
   \val~(\LOCALTEE~x) &\stepto& \val~\val~(\LOCALSET~x)
   \end{array}


.. _exec-global.get:

:math:`\GLOBALGET~x`
....................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-global.get>` において :math:`F.\AMODULE.\MIGLOBALS[x]` が存在する。

3. :math:`a` を :ref:`グローバルアドレス <syntax-globaladdr>` :math:`F.\AMODULE.\MIGLOBALS[x]` とする。

4. アサーション: :ref:`バリデーション <valid-global.get>` において :math:`S.\SGLOBALS[a]` が存在する。

5. :math:`\X{glob}` を :ref:`グローバルインスタンス <syntax-globalinst>` :math:`S.\SGLOBALS[a]` とする。

6. :math:`\val` を値 :math:`\X{glob}.\GIVALUE` とする。

7. 値 :math:`\val` をスタックにpushする。

.. math::
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; F; (\GLOBALGET~x) &\stepto& S; F; \val
   \end{array}
   \\ \qquad
     (\iff S.\SGLOBALS[F.\AMODULE.\MIGLOBALS[x]].\GIVALUE = \val) \\
   \end{array}


.. _exec-global.set:

:math:`\GLOBALSET~x`
....................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-global.set>` において :math:`F.\AMODULE.\MIGLOBALS[x]` が存在する。

3. :math:`a` を :ref:`グローバルアドレス <syntax-globaladdr>` :math:`F.\AMODULE.\MIGLOBALS[x]` とする。

4. アサーション: :ref:`バリデーション <valid-global.set>` において :math:`S.\SGLOBALS[a]` が存在する。

5. :math:`\X{glob}` を :ref:`グローバルインスタンス <syntax-globalinst>` :math:`S.\SGLOBALS[a]` とする。

6. アサーション: :ref:`バリデーション <valid-global.set>` において値が1個スタックのトップに存在する。

7. 値 :math:`\val` をスタックからpopする。

8. :math:`\X{glob}.\GIVALUE` を :math:`\val` で置き換える。

.. math::
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; F; \val~(\GLOBALSET~x) &\stepto& S'; F; \epsilon
   \end{array}
   \\ \qquad
   (\iff S' = S \with \SGLOBALS[F.\AMODULE.\MIGLOBALS[x]].\GIVALUE = \val) \\
   \end{array}

.. note::
   実際の :ref:`バリデーション <valid-global.set>` はそのグローバルがミュータブルとマークされるようにします。


.. index:: memory instruction, memory index, store, frame, address, memory address, memory instance, store, frame, value, integer, limits, value type, bit width
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-memarg:
.. _exec-instr-memory:

メモリーインストラクション（memory instruction）
~~~~~~~~~~~~~~~~~~~

.. note::
   ロードインストラクションやストアインストラクション内のアラインメント :math:`\memarg.\ALIGN` はセマンティクスに影響しません。
   これは、 メモリがアクセスされるオフセット :math:`\X{ea}` がプロパティ :math:`\X{ea} \mod 2^{\memarg.\ALIGN} = 0` を満たすことを意図しています。
   WebAssemblyの実装では、意図した用途に合わせて最適化するのにこのヒントを利用できます。
   このプロパティに違反するアラインされていないアクセスは引き続き許されますが、アノテーションに関わらず継続しなければなりません。
   ただし、一部のハードウェアでは本質的な速度低下が発生する可能性があります。


.. _exec-load:
.. _exec-loadn:

:math:`t\K{.}\LOAD~\memarg` および :math:`t\K{.}\LOAD{N}\K{\_}\sx~\memarg`
.......................................................................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-loadn>` において  :math:`F.\AMODULE.\MIMEMS[0]` が存在する。

3. :math:`a` を :ref:`メモリーアドレス <syntax-memaddr>` :math:`F.\AMODULE.\MIMEMS[0]` とする。

4. アサーション: :ref:`バリデーション <valid-loadn>` において :math:`S.\SMEMS[a]` が存在する。

5. :math:`\X{glob}` を :ref:`メモリーインスタンス <syntax-meminst>` :math:`S.\SMEMS[a]` とする。

6. アサーション: :ref:`バリデーション <valid-loadn>` において :ref:`値型 <syntax-valtype>` |I32| の値が1個スタックのトップに存在する。

7. 値 :math:`\I32.\CONST~i` をスタックからpopする。

8. :math:`\X{ea}` を整数 :math:`i + \memarg.\OFFSET` とする。

9. :math:`N` がインストラクションの一部にない場合、以下のようになる。

   a. :math:`N` を :ref:`値型 <syntax-valtype>` :math:`t` の :ref:`ビット幅 <syntax-valtype>` :math:`|t|` とする。

10. :math:`\X{ea} + N/8` が :math:`\X{mem}.\MIDATA` の長さより大きい場合、以下のようになる。

    a. トラップする。

11. :math:`b^\ast` をバイトシーケンス :math:`\X{mem}.\MIDATA[\X{ea} \slice N/8]` とする。

12. :math:`N` と :math:`\sx` がインストラクションの一部にない場合、以下のようになる。

    a. :math:`n` を :math:`\bytes_{\iN}(n) = b^\ast` となる整数とする。

    b. :math:`c` を計算 :math:`\extend\F{\_}\sx_{N,|t|}(n)` の結果とする。

13. 上に該当しない場合、以下のようになる。

    a. :math:`c` を :math:`\bytes_t(c) = b^\ast` となる定数とする。

14. 値 :math:`t.\CONST~c` をスタックにpushする。

.. math::
   ~\\[-1ex]
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~i)~(t.\LOAD~\memarg) &\stepto& S; F; (t.\CONST~c)
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & \X{ea} = i + \memarg.\OFFSET \\
     \wedge & \X{ea} + |t|/8 \leq |S.\SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA| \\
     \wedge & \bytes_t(c) = S.\SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA[\X{ea} \slice |t|/8])
     \end{array}
   \\[1ex]
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~i)~(t.\LOAD{N}\K{\_}\sx~\memarg) &\stepto&
     S; F; (t.\CONST~\extend\F{\_}\sx_{N,|t|}(n))
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & \X{ea} = i + \memarg.\OFFSET \\
     \wedge & \X{ea} + N/8 \leq |S.\SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA| \\
     \wedge & \bytes_{\iN}(n) = S.\SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA[\X{ea} \slice N/8])
     \end{array}
   \\[1ex]
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~k)~(t.\LOAD({N}\K{\_}\sx)^?~\memarg) &\stepto& S; F; \TRAP
   \end{array}
   \\ \qquad
     (\otherwise) \\
   \end{array}


.. _exec-store:
.. _exec-storen:

:math:`t\K{.}\STORE~\memarg` および :math:`t\K{.}\STORE{N}~\memarg`
................................................................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-storen>` において  :math:`F.\AMODULE.\MIMEMS[0]` が存在する。

3. :math:`a` を :ref:`メモリーアドレス <syntax-memaddr>` :math:`F.\AMODULE.\MIMEMS[0]` とする。

4. アサーション: :ref:`バリデーション <valid-storen>` において :math:`S.\SMEMS[a]` が存在する。

5. :math:`\X{glob}` を :ref:`メモリーインスタンス <syntax-meminst>` :math:`S.\SMEMS[a]` とする。

6. アサーション: :ref:`バリデーション <valid-storen>` において :ref:`値型 <syntax-valtype>` :math:`t` の値が1個スタックのトップに存在する。

7. 値 :math:`t.\CONST~c` をスタックからpopする。

8. アサーション: :ref:`バリデーション <valid-storen>` において :ref:`値型 <syntax-valtype>` |I32| の値が1個スタックのトップに存在する。

9. 値  :math:`\I32.\CONST~i` をスタックからpopする。

10. :math:`\X{ea}` を整数 :math:`i + \memarg.\OFFSET` とする。

11. :math:`N` がインストラクションの一部にない場合、以下のようになる。

    a. :math:`N` を :ref:`値型 <syntax-valtype>` :math:`t` の :ref:`ビット幅 <syntax-valtype>` :math:`|t|` とする。

12. :math:`\X{ea} + N/8` が :math:`\X{mem}.\MIDATA` の長さより大きい場合、以下のようになる。

    a. トラップする。

13. :math:`N` がインストラクションの一部にない場合、以下のようになる。

    a. :math:`n` を計算 :math:`\wrap_{|t|,N}(c)` の結果とする。

    b. :math:`b^\ast` をバイトシーケンス :math:`\bytes_{\iN}(n)` とする。

14. 上に該当しない場合、以下のようになる。

    a. :math:`b^\ast`をバイトシーケンス :math:`\bytes_t(c)` とする。

15. バイト :math:`\X{mem}.\MIDATA[\X{ea} \slice N/8]` を :math:`b^\ast` で置き換える。

.. math::
   ~\\[-1ex]
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~i)~(t.\CONST~c)~(t.\STORE~\memarg) &\stepto& S'; F; \epsilon
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & \X{ea} = i + \memarg.\OFFSET \\
     \wedge & \X{ea} + |t|/8 \leq |S.\SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA| \\
     \wedge & S' = S \with \SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA[\X{ea} \slice |t|/8] = \bytes_t(c)
     \end{array}
   \\[1ex]
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~i)~(t.\CONST~c)~(t.\STORE{N}~\memarg) &\stepto& S'; F; \epsilon
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & \X{ea} = i + \memarg.\OFFSET \\
     \wedge & \X{ea} + N/8 \leq |S.\SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA| \\
     \wedge & S' = S \with \SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA[\X{ea} \slice N/8] = \bytes_{\iN}(\wrap_{|t|,N}(c))
     \end{array}
   \\[1ex]
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~k)~(t.\CONST~c)~(t.\STORE{N}^?~\memarg) &\stepto& S; F; \TRAP
   \end{array}
   \\ \qquad
     (\otherwise) \\
   \end{array}


.. _exec-memory.size:

:math:`\MEMORYSIZE`
...................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-unop>` において :math:`F.\AMODULE.\MIMEMS[0]` が存在する。

3. :math:`a` を :ref:`メモリーアドレス <syntax-memaddr>` :math:`F.\AMODULE.\MIMEMS[0]` とする。

4. アサーション: :ref:`バリデーション <memory.size>` において :math:`S.\SMEMS[a]` が存在する。

5. :math:`\X{mem}` を :ref:`メモリーインスタンス <syntax-meminst>` :math:`S.\SMEMS[a]` とする。

6. :math:`\X{sz}` を、:math:`\X{mem}.\MIDATA` を :ref:`ページサイズ <page-size>` で割った長さとする。

7. Push the value :math:`\I32.\CONST~\X{sz}` to the stack.

.. math::
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; F; \MEMORYSIZE &\stepto& S; F; (\I32.\CONST~\X{sz})
   \end{array}
   \\ \qquad
     (\iff |S.\SMEMS[F.\AMODULE.\MIMEMS[0]].\MIDATA| = \X{sz}\cdot64\,\F{Ki}) \\
   \end{array}


.. _exec-memory.grow:

:math:`\MEMORYGROW`
...................

1. :math:`F` を :ref:`カレント <exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-memory.grow>` において :math:`F.\AMODULE.\MIMEMS[0]` が存在する。

3. :math:`a` を :ref:`メモリーアドレス <syntax-memaddr>` :math:`F.\AMODULE.\MIMEMS[0]` とする。

4. アサーション: :ref:`バリデーション <valid-memory.grow>` において :math:`S.\SMEMS[a]` が存在する。

5. :math:`\X{mem}` を :ref:`メモリーインスタンス <syntax-meminst>` :math:`S.\SMEMS[a]` とする。

6. :math:`\X{sz}` を、:math:`S.\SMEMS[a]` を :ref:`ページサイズ <page-size>` で割った長さとする。

7. アサーション: :ref:`バリデーション <valid-memory.grow>` において :ref:`値型 <syntax-valtype>` |I32| がスタックのトップに存在する。

8. 値 :math:`\I32.\CONST~n` をスタックからpopする。

9. :math:`\X{err}` を、:math:`\signed_{32}(\X{err})` が :math:`-1` となる |i32| 値 :math:`2^{32}-1` とする。

10. :ref:`メモリー成長 <grow-mem>` :math:`\X{mem}` を :math:`n` :ref:`ページ <page-size>` 分試行するか、11に進む。

   a. 成功した場合は、値 :math:`\I32.\CONST~\X{sz}` をスタックにpushする。

   b. 失敗した場合は、値 :math:`\I32.\CONST~\X{err}` をスタックにpushする。

11. または、値 :math:`\I32.\CONST~\X{err}` をスタックにpushする。

.. math::
   ~\\[-1ex]
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~n)~\MEMORYGROW &\stepto& S'; F; (\I32.\CONST~\X{sz})
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & F.\AMODULE.\MIMEMS[0] = a \\
     \wedge & \X{sz} = |S.\SMEMS[a].\MIDATA|/64\,\F{Ki} \\
     \wedge & S' = S \with \SMEMS[a] = \growmem(S.\SMEMS[a], n)) \\
     \end{array}
   \\[1ex]
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~n)~\MEMORYGROW &\stepto& S; F; (\I32.\CONST~\signed_{32}^{-1}(-1))
   \end{array}
   \end{array}

.. note::
   |MEMORYGROW| インストラクションは非決定的です。
   成功して古いメモリサイズ :math:`\X{sz}` を返すか、失敗して :math:`{-1}` を返すかのいずれかです。
   参照されたメモリーインスタンスに最大サイズが定義されていてそれを超える場合は、「必ず」失敗しなければなりません。
   ただし、それ以外の場合にも失敗する「可能性」があります。
   実践では、この選択は :ref:`エンベダー <embedder>` で利用可能な :ref:`リソース <impl-exec>` に依存します。


.. index:: control instructions, structured control, label, block, branch, result type, label index, function index, type index, vector, address, table address, table instance, store, frame
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-label:
.. _exec-instr-control:

制御インストラクション（control instruction）
~~~~~~~~~~~~~~~~~~~~

.. _exec-nop:

:math:`\NOP`
............

1. 何もしない。

.. math::
   \begin{array}{lcl@{\qquad}l}
   \NOP &\stepto& \epsilon
   \end{array}


.. _exec-unreachable:

:math:`\UNREACHABLE`
....................

1. トラップする。

.. math::
   \begin{array}{lcl@{\qquad}l}
   \UNREACHABLE &\stepto& \TRAP
   \end{array}


.. _exec-block:

:math:`\BLOCK~\blocktype~\instr^\ast~\END`
..........................................

1. アサーション: :ref:`バリデーション <valid-blocktype>` において :math:`\expand_F(\blocktype)` が定義されている。

2. :math:`[t_1^m] \to [t_2^n]` を :ref:`関数型 <syntax-functype>` :math:`\expand_F(\blocktype)` とする。

3. :math:`L` を、その引数の個数（arity）が :math:`n` でその継続がブロックの終了部であるラベルとする。

4. アサーション: :ref:`バリデーション <valid-block>` において、少なくとも値 :math:`m` がスタックのトップに存在する。

5. 値 :math:`\val^m` をスタックからpopする。

6. ラベル :math:`L` を持つブロック :math:`\val^m~\instr^\ast` に :ref:`enter <exec-instr-seq-enter>` する。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   F; \val^m~\BLOCK~\X{bt}~\instr^\ast~\END &\stepto&
     F; \LABEL_n\{\epsilon\}~\val^m~\instr^\ast~\END
     & (\iff \expand_F(\X{bt}) = [t_1^m] \to [t_2^n])
   \end{array}


.. _exec-loop:

:math:`\LOOP~\blocktype~\instr^\ast~\END`
.........................................

1. アサーション: :ref:`バリデーション <valid-blocktype>`, :math:`\expand_F(\blocktype)` が定義されている。

2. :math:`[t_1^m] \to [t_2^n]` を :ref:`関数型 <syntax-functype>` :math:`\expand_F(\blocktype)` とする。

3. :math:`L` を、その引数の個数（arity）が :math:`n` でその継続がブロックの開始部であるラベルとする。

4. アサーション: :ref:`バリデーション <valid-block>` において、少なくとも値 :math:`m` がスタックのトップに存在する。

5. :math:`\val^m` をスタックからpopする。

6. ラベル :math:`L` を持つブロック :math:`\val^m~\instr^\ast` に :ref:`enter <exec-instr-seq-enter>` する。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   F; \val^m~\LOOP~\X{bt}~\instr^\ast~\END &\stepto&
     F; \LABEL_m\{\LOOP~\X{bt}~\instr^\ast~\END\}~\val^m~\instr^\ast~\END
     & (\iff \expand_F(\X{bt}) = [t_1^m] \to [t_2^n])
   \end{array}


.. _exec-if:

:math:`\IF~\blocktype~\instr_1^\ast~\ELSE~\instr_2^\ast~\END`
.............................................................

1. アサーション: :ref:`バリデーション <valid-blocktype>` において :math:`\expand_F(\blocktype)` が定義されている。

2. :math:`[t_1^m] \to [t_2^n]` を :ref:`関数型 <syntax-functype>` :math:`\expand_F(\blocktype)` とする。

3. :math:`L` を、その引数の個数（arity）が :math:`n` でその継続が |IF| インストラクションの終了部であるラベルとする。

4. アサーション: :ref:`バリデーション <valid-if>` において、:ref:`値型 <syntax-valtype>` |I32| の値がスタックのトップに存在する。

5. 値 :math:`\I32.\CONST~c` をスタックからpopする。

6. アサーション: :ref:`バリデーション <valid-if>` において、少なくとも値 :math:`m` がスタックのトップに存在する。

7. 値 :math:`\val^m` をスタックからpopする。

8. :math:`c` がゼロでない場合、以下のようになる。

   a. ラベル :math:`L` を持つブロック :math:`\val^m~\instr_1^\ast` に :ref:`enter <exec-instr-seq-enter>` する。

9. 上に該当しない場合、以下のようになる。

   a. ラベル :math:`L` を持つブロック :math:`\val^m~\instr_2^\ast` に :ref:`enter <exec-instr-seq-enter>` する。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   F; \val^m~(\I32.\CONST~c)~\IF~\X{bt}~\instr_1^\ast~\ELSE~\instr_2^\ast~\END &\stepto&
     F; \LABEL_n\{\epsilon\}~\val^m~\instr_1^\ast~\END
     & (\iff c \neq 0 \wedge \expand_F(\X{bt}) = [t_1^m] \to [t_2^n]) \\
   F; \val^m~(\I32.\CONST~c)~\IF~\X{bt}~\instr_1^\ast~\ELSE~\instr_2^\ast~\END &\stepto&
     F; \LABEL_n\{\epsilon\}~\val^m~\instr_2^\ast~\END
     & (\iff c = 0 \wedge \expand_F(\X{bt}) = [t_1^m] \to [t_2^n]) \\
   \end{array}


.. _exec-br:

:math:`\BR~l`
.............

1. アサーション: :ref:`バリデーション <valid-br>` において、スタックが少なくとも :math:`l+1` 個のラベルを持つ。

2. :math:`L` をスタックの :math:`l` 番目（スタックのトップから開始し、ゼロから数える）に出現するラベルとする。

3. :math:`n` :math:`L` の引数の個数（arity）とする。

4. アサーション: :ref:`バリデーション <valid-br>` において、スタックのトップに少なくとも :math:`n` 個の値が存在する。

5. 値 :math:`\val^n` をスタックからpopする。

6. 以下を :math:`l+1` 回繰り返す。

   a. スタックのトップが1個の値の場合、以下を行う。

      i. その値をスタックからpopする。

   b. アサーション: :ref:`バリデーション <valid-br>` において、スタックのトップがラベルである。

   c. そのラベルをスタックからpopする。

7. 値 :math:`\val^n` をスタックにpushする。

8. :math:`L` の継続にジャンプする。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   \LABEL_n\{\instr^\ast\}~\XB^l[\val^n~(\BR~l)]~\END &\stepto& \val^n~\instr^\ast
   \end{array}


.. _exec-br_if:

:math:`\BRIF~l`
...............

1. アサーション: :ref:`バリデーション <valid-br_if>` において :ref:`値型 <syntax-valtype>` |I32| の値がスタックのトップに存在する。

2. 値 :math:`\I32.\CONST~c` をスタックからpopする。

3. :math:`c` がゼロでない場合、以下を実行する。

   a. インストラクション :math:`(\BR~l)` を :ref:`実行 <exec-br>` する。

4. 上に該当しない場合、以下のようになる。

   a. 何もしない。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   (\I32.\CONST~c)~(\BRIF~l) &\stepto& (\BR~l)
     & (\iff c \neq 0) \\
   (\I32.\CONST~c)~(\BRIF~l) &\stepto& \epsilon
     & (\iff c = 0) \\
   \end{array}


.. _exec-br_table:

:math:`\BRTABLE~l^\ast~l_N`
...........................

1. アサーション: :ref:`バリデーション <valid-if>` において :ref:`値型 <syntax-valtype>` |I32| の値がスタックのトップに存在する。

2. 値 :math:`\I32.\CONST~i` をスタックからpopする。

3. :math:`i` が :math:`l^\ast` の長さより小さい場合、以下を実行する。

   a. :math:`l_i` をラベル :math:`l^\ast[i]` とする。

   b. インストラクション :math:`(\BR~l_i)` を :ref:`実行 <exec-br>` する。

4. 上に該当しない場合、以下を実行する。

   a. インストラクション :math:`(\BR~l_N)` を :ref:`実行 <exec-br>` する。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   (\I32.\CONST~i)~(\BRTABLE~l^\ast~l_N) &\stepto& (\BR~l_i)
     & (\iff l^\ast[i] = l_i) \\
   (\I32.\CONST~i)~(\BRTABLE~l^\ast~l_N) &\stepto& (\BR~l_N)
     & (\iff |l^\ast| \leq i) \\
   \end{array}


.. _exec-return:

:math:`\RETURN`
...............

1. :math:`F` を :ref:`カレント<exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. :math:`n` を :math:`F` の引数の個数（arity）とする。

3. アサーション: :ref:`バリデーション <valid-return>` において少なくとも :math:`n` 個の値がスタックのトップに存在する。

4. 結果 :math:`\val^n` をスタックからpopする。

5. アサーション: :ref:`バリデーション <valid-return>` において、スタックに少なくとも1個の :ref:`フレーム <syntax-frame>` が存在する。

6. スタックのトップがフレームでない間、以下を実行する。

   a. トップ要素をスタックからpopする。

7. アサーション: スタックのトップがフレーム :math:`F` である。

8. そのフレームをスタックからpopする。

9. :math:`\val^n` をスタックにpushする。

10. そのフレームをpushした呼び出し元の次にあるインストラクションにジャンプする。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   \FRAME_n\{F\}~\XB^k[\val^n~\RETURN]~\END &\stepto& \val^n
   \end{array}


.. _exec-call:

:math:`\CALL~x`
...............

1. :math:`F` を :ref:`カレント<exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-call>` において :math:`F.\AMODULE.\MIFUNCS[x]` が存在する。

3. :math:`a` を :ref:`関数アドレス <syntax-funcaddr>` :math:`F.\AMODULE.\MIFUNCS[x]` とする。

4. アドレス :math:`a` にある関数インスタンスを :ref:`呼び出す <exec-invoke>` 。

.. math::
   \begin{array}{lcl@{\qquad}l}
   F; (\CALL~x) &\stepto& F; (\INVOKE~a)
     & (\iff F.\AMODULE.\MIFUNCS[x] = a)
   \end{array}


.. _exec-call_indirect:

:math:`\CALLINDIRECT~x`
.......................

1. :math:`F` を :ref:`カレント<exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. アサーション: :ref:`バリデーション <valid-call_indirect>` において :math:`F.\AMODULE.\MITABLES[0]` が存在する。

3. :math:`\X{ta}` を :ref:`テーブルアドレス <syntax-tableaddr>` :math:`F.\AMODULE.\MITABLES[0]` とする。

4. アサーション: :ref:`バリデーション <valid-call_indirect>` において :math:`S.\STABLES[\X{ta}]` が存在する。

5. :math:`\X{tab}` を :ref:`テーブルインスタンス <syntax-tableinst>` :math:`S.\STABLES[\X{ta}]` とする。

6. アサーション: :ref:`バリデーション <valid-call_indirect>` において :math:`F.\AMODULE.\MITYPES[x]` が存在する。

7. :math:`\X{ft}_{\F{expect}}` を :ref:`関数型 <syntax-functype>` :math:`F.\AMODULE.\MITYPES[x]` とする。

8. アサーション: :ref:`バリデーション <valid-call_indirect>` において :ref:`値型 <syntax-valtype>` |I32| の値が1個スタックのトップに存在する。

9. 値 :math:`\I32.\CONST~i` をスタックからpopする。

10. :math:`i` が :math:`\X{tab}.\TIELEM` の長さより小さくない場合、以下を実行する。

    a. トラップする。

11. :math:`\X{tab}.\TIELEM[i]` が初期化されていない場合、以下を実行する。

    a. トラップする。

12. :math:`a` を :ref:`関数アドレス <syntax-funcaddr>` :math:`\X{tab}.\TIELEM[i]` とする。

13. アサーション: :ref:`バリデーション <valid-call_indirect>` において :math:`S.\SFUNCS[a]` が存在する。

14. :math:`\X{f}` を :ref:`関数インスタンス <syntax-funcinst>` :math:`S.\SFUNCS[a]` とする。

15. :math:`\X{ft}_{\F{actual}}` を :ref:`関数型 <syntax-functype>` :math:`\X{f}.\FITYPE` とする。

16. :math:`\X{ft}_{\F{actual}}` と :math:`\X{ft}_{\F{expect}}` が異なる場合、以下を実行する。

    a. トラップする。

17. アドレス :math:`a` にある関数インスタンスを :ref:`呼び出す <exec-invoke>` 。

.. math::
   ~\\[-1ex]
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~i)~(\CALLINDIRECT~x) &\stepto& S; F; (\INVOKE~a)
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & S.\STABLES[F.\AMODULE.\MITABLES[0]].\TIELEM[i] = a \\
     \wedge & S.\SFUNCS[a] = f \\
     \wedge & F.\AMODULE.\MITYPES[x] = f.\FITYPE)
     \end{array}
   \\[1ex]
   \begin{array}{lcl@{\qquad}l}
   S; F; (\I32.\CONST~i)~(\CALLINDIRECT~x) &\stepto& S; F; \TRAP
   \end{array}
   \\ \qquad
     (\otherwise)
   \end{array}


.. index:: instruction, instruction sequence, block
.. _exec-instr-seq:

ブロック（block）
~~~~~~

以下の補助ルールは、1個の :ref:`ブロック <exec-instr-control>` を形成する1個の :ref:`インストラクションシーケンス <syntax-instr-seq>` を実行するセマンティクスを定義します。

.. _exec-instr-seq-enter:

ラベル :math:`L` を持つ :math:`\instr^\ast` にenterする
.................................................

1. :math:`L` をスタックにpushする。

2. インストラクションシーケンス :math:`\instr^\ast` の開始部にジャンプする。

.. note::
   ラベル :math:`L` は、構造化制御インストラクションに直接還元される :ref:`管理インストラクション <syntax-instr-admin>` 内に埋め込まれるため、インストラクションシーケンスにenterする形式的な還元規則は不要です。


.. _exec-instr-seq-exit:

ラベル :math:`L` を持つ :math:`\instr^\ast` からexitする
................................................

ジャンプやトラップによるabortなしでブロックの終了部に到達した場合、以下の手順が実行されます。

1. :math:`m` をスタックのトップにある値の個数とする。

2. 値 :math:`\val^m` をスタックからpopする。

3. アサーション: :ref:`バリデーション <valid-instr-seq>` において、ラベル :math:`L` が現在スタックのトップに存在する。

4. そのラベルをスタックからpopする。

5. :math:`\val^m` をスタックにpushして戻す。

6. ラベル :math:`L` に関連付けられる :ref:`構造化制御インストラクション <syntax-instr-control>` の |END| の次にジャンプする。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   \LABEL_n\{\instr^\ast\}~\val^m~\END &\stepto& \val^m
   \end{array}

.. note::
   This semantics also applies to the instruction sequence contained in a |LOOP| instruction.
   Therefore, execution of a loop falls off the end, unless a backwards branch is performed explicitly.


.. index:: ! call, function, function instance, label, frame

関数呼び出し（function call）
~~~~~~~~~~~~~~

以下の補助ルールは、いずれかの :ref:`呼び出しインストラクション <exec-instr-control>` を介して :ref:`関数インスタンス <syntax-funcinst>` を呼び出し、そこから戻るセマンティクスを定義します。


.. _exec-invoke:

:ref:`関数アドレス <syntax-funcaddr>` :math:`a` を呼び出す
.................................................................

1. アサーション: :ref:`バリデーション <valid-call>` において :math:`S.\SFUNCS[a]` が存在する。

2. :math:`f` を :ref:`関数インスタンス <syntax-funcinst>` :math:`S.\SFUNCS[a]` とする。

3. :math:`[t_1^n] \to [t_2^m]` を :ref:`関数型 <syntax-functype>` :math:`f.\FITYPE` とする。

4. :math:`t^\ast` を :ref:`値型s <syntax-valtype>` :math:`f.\FICODE.\FLOCALS` のリストとする。

5. :math:`\instr^\ast~\END` を :ref:`式 <syntax-expr>` :math:`f.\FICODE.\FBODY` とする。

6. アサーション: :ref:`バリデーション <valid-call>` において :math:`n` 個の値がスタックのトップに存在する。

7. 値 :math:`\val^n` をスタックからpopする。

8. :math:`\val_0^\ast` を型 :math:`t^\ast` のゼロ値のリストとする。

9. :math:`F` を :ref:`フレーム <syntax-frame>` :math:`\{ \AMODULE~f.\FIMODULE, \ALOCALS~\val^n~\val_0^\ast \}` とする。

10. :math:`m` 個の引数を持つ :math:`F` のアクティベーションをスタックにpushする。

11. :math:`L` を、引数の個数（arity）が :math:`m` 個で、その継続が関数の終了部である :ref:`ラベル <syntax-label>` とする。

12. ラベル :math:`L` を持つインストラクションシーケンス :math:`\instr^\ast` に :ref:`enter <exec-instr-seq-enter>` する。

.. math::
   ~\\[-1ex]
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; \val^n~(\INVOKE~a) &\stepto& S; \FRAME_m\{F\}~\LABEL_m\{\}~\instr^\ast~\END~\END
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & S.\SFUNCS[a] = f \\
     \wedge & f.\FITYPE = [t_1^n] \to [t_2^m] \\
     \wedge & f.\FICODE = \{ \FTYPE~x, \FLOCALS~t^k, \FBODY~\instr^\ast~\END \} \\
     \wedge & F = \{ \AMODULE~f.\FIMODULE, ~\ALOCALS~\val^n~(t.\CONST~0)^k \})
     \end{array} \\
   \end{array}


.. _exec-invoke-exit:

関数から戻る
.........................

ジャンプ（つまり |RETURN|）やトラップによるabortなしで関数の終了部に到達した場合、以下の手順が実行されます。

1. :math:`F` を :ref:`カレント<exec-notation-textual>` :ref:`フレーム <syntax-frame>` とする。

2. :math:`n` を :math:`F` のアクティベーションの引数の個数（arity）とする。

3. アサーション: :ref:`バリデーション <valid-instr-seq>` において :math:`n` 個の値がスタックのトップに存在する。

4. 結果 :math:`\val^n` をスタックからpopする。

5. アサーション: :ref:`バリデーション <valid-func>` において、フレーム :math:`F` が現在スタックのトップに存在する。

6. そのフレームをスタックからpopする。

7. :math:`\val^n` をスタックにpushして戻す。

8. 呼び出し元の次にあるインストラクションにジャンプする。

.. math::
   ~\\[-1ex]
   \begin{array}{lcl@{\qquad}l}
   \FRAME_n\{F\}~\val^n~\END &\stepto& \val^n
   \end{array}


.. index:: host function, store
.. _exec-invoke-host:

ホスト関数（host function）
..............

:ref:`ホスト関数 <syntax-hostfunc>` の呼び出しは「非決定的な」振る舞いです。
:ref:`トラップ <trap>` で終了する可能性も、通常どおり制御を戻す可能性もあります。
しかし後者においては、その :ref:`関数型 <syntax-functype>` に応じて、スタック上にあるWebAssembly :ref:`値 <syntax-val>` の個数や型が正しく消費および生成されなければなりません。

ホスト関数は :ref:`ストア <syntax-store>` を変更する可能性もあります。
しかし、ストアに対するあらゆる変更は元のストアの :ref:`拡張 <extend-store>` でなければなりません。すなわち、変更する対象はミュータブルなコンテンツだけでなければならず、インスタンスを削除してはいけません。
さらに、得られるストアは :ref:`有効 <valid-store>` でなければなりません。有効とはすなわち、ストア内のあらゆるデータとコードが十分型付けされているということです。

.. math::
   ~\\[-1ex]
   \begin{array}{l}
   \begin{array}{lcl@{\qquad}l}
   S; \val^n~(\INVOKE~a) &\stepto& S'; \result
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & S.\SFUNCS[a] = \{ \FITYPE~[t_1^n] \to [t_2^m], \FIHOSTCODE~\X{hf} \} \\
     \wedge & (S'; \result) \in \X{hf}(S; \val^n)) \\
     \end{array} \\
   \begin{array}{lcl@{\qquad}l}
   S; \val^n~(\INVOKE~a) &\stepto& S; \val^n~(\INVOKE~a)
   \end{array}
   \\ \qquad
     \begin{array}[t]{@{}r@{~}l@{}}
     (\iff & S.\SFUNCS[a] = \{ \FITYPE~[t_1^n] \to [t_2^m], \FIHOSTCODE~\X{hf} \} \\
     \wedge & \bot \in \X{hf}(S; \val^n)) \\
     \end{array} \\
   \end{array}

上の :math:`\X{hf}(S; \val^n)` は、引数 :math:`\val^n` を持つ現在のストア :math:`S` にあるホスト関数 :math:`\X{hf}` の、実装側で定義された実行を表します。
これはさまざまな結果を生み出しますが、その結果の個別の要素は、変更されたストア :math:`S'` とその :ref:`結果 <syntax-result>` のペアか、乖離を表す特殊値 :math:`\bot` のいずれかとなります。
1つのホスト関数は、少なくとも1つの引数に対応する結果が複数ある場合は非決定的となります。

WebAssemblyの実装をホスト関数の存在において :ref:`健全 <soundness>` であるためには、あらゆる :ref:`ホスト関数インスタンス <syntax-funcinst>` が :ref:`有効 <valid-hostfuncinst>` でなければなりません。ここで言う有効とは、適切な事前条件と事後条件に従うということです。すなわち、:ref:`有効なストア <valid-store>` :math:`S` のもとで、与えられた引数 :math:`\val^n` が対応するパラメータ型 :math:`t_1^n` と一致し、ホスト関数の実行が「空でない可能な結果」を生み出さなければならず、その結果が「乖離」または「有効なストア :math:`S'` で構成されている（すなわち :math:`S` の :ref:`拡張 <extend-store>` である）」のいずれかであること、そして結果が戻り値の型 :math:`t_2^m` に一致することです。
これらの記述についてはすべて :ref:`付録 <soundness>` で正確に記述されています。

.. note::
   ホスト関数は、ある :ref:`モジュール <syntax-module>` から :ref:`エクスポート <syntax-export>` された関数を :ref:`呼び出す <exec-invocation>` ことで、WebAssemblyにコールバックできます。
   ただし、そのような呼び出しによる影響は、そのホスト関数で許されている非決定的な振る舞いによって異なります。


.. index:: expression
   pair: execution; expression
   single: abstract syntax; expression
.. _exec-expr:

式（expression）
~~~~~~~~~~~

ひとつの :ref:`式 <syntax-expr>` の「評価」は、その式を含む :ref:`モジュールインスタンス <syntax-moduleinst>` を指す :ref:`カレント<exec-notation-textual>` :ref:`フレーム <syntax-frame>` から相対的になります。

1. その式のインストラクションシーケンス :math:`\instr^\ast` の開始部にジャンプする。

2. そのインストラクションシーケンスを実行する。

3. アサーション: :ref:`バリデーション <valid-expr>` において、スタックのトップに :ref:`値 <syntax-val>` が1個含まれる。

4. :ref:`値 <syntax-val>` :math:`\val` をスタックからpopする。

.. math::
   S; F; \instr^\ast \stepto S'; F'; \instr'^\ast
   \qquad (\iff S; F; \instr^\ast~\END \stepto S'; F'; \instr'^\ast~\END)

.. note::
   評価は、この還元を1個の値になるまで繰り返します。
   :ref:`関数 <syntax-func>` 本体を構成する式は、関数 :ref:`呼び出し <exec-invoke>` の間に実行されます。
