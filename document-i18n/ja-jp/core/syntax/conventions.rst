.. index:: ! abstract syntax

記法
-----------

WebAssemblyは、具体的な表現形式を複数持つプログラミング言語です（:ref:`バイナリ形式 <binary>` と :ref:`テキスト形式 <text>`）。
どちらの形式も、共通の構造に対応付けられています。
表記を簡潔にするため、この構造は「抽象構文」の形で記述します。
本仕様書のあらゆる箇所の定義は、この抽象構文の形を取っています。

.. index:: ! grammar notation, notation
   single: abstract syntax; grammar
   pair: abstract syntax; notation
.. _grammar:

文法上の記法
~~~~~~~~~~~~~~~~

以下の記法は、抽象構文での文法ルールを定義するのに使われます。

* 終端シンボル（アトム）はサンセリフフォントで表記される: :math:`\K{i32}, \K{end}`

* 非終端シンボルはイタリックフォントで表記される: :math:`\X{valtype}, \X{instr}`

* :math:`A^n`： :math:`A` を :math:`n\geq 0` 回列挙したシーケンス。

* :math:`A^\ast`： :math:`A` の列挙回数が空の可能性もあるシーケンス。
  （:math:`A^n` のショートハンド、ただし :math:`n` は本筋と無関係）

* :math:`A^+`： :math:`A` の列挙回数が空でないシーケンス。
  （:math:`A^n` のショートハンド、ただし :math:`n \geq 1`）

* :math:`A^?`： :math:`A` があってもなくてもいい場合。
  （:math:`A^n` のショートハンド、ただし :math:`n \leq 1`）

* 積（production）は次のように表す: :math:`\X{sym} ::= A_1 ~|~ \dots ~|~ A_n`

* 大きな積は次のような複数の定義に分割される可能性もある。最初の定義が「:math:`\X{sym} ::= A_1 ~|~ \dots`」のように明示的に三点リーダーで終わり、続きが「:math:`\X{sym} ::= \dots ~|~ A_2`」のように三点リーダーで開始される。

* 積によっては、「":math:`(\iff \X{condition})`"」のように丸かっこで囲まれた副条件で拡張されることもある。これは、積の組み合わせ拡張を多くの個別のケースで表すショートハンドを提供する。

.. _notation-epsilon:
.. _notation-length:
.. _notation-index:
.. _notation-slice:
.. _notation-replace:
.. _notation-record:
.. _notation-project:
.. _notation-concat:
.. _notation-compose:

補助記法
~~~~~~~~~~~~~~~~~~

構文の構成を扱うときに、以下の記法も使われます。

* :math:`\epsilon`： 空のシーケンスを表す。

* :math:`|s|`： シーケンス :math:`s` の長さを表す。

* :math:`s[i]`： シーケンス :math:`s` の :math:`i` -1番目の要素を表す（開始は :math:`0`）。

* :math:`s[i \slice n]`： シーケンス :math:`s` のサブシーケンス :math:`s[i]~\dots~s[i+n-1]` を表す。

* :math:`s \with [i] = A`： :math:`s` と同じシーケンスを表すが、:math:`i` 番目の要素が :math:`A` で置き換わる点が異なる。

* :math:`s \with [i \slice n] = A^n`： :math:`s` と同じシーケンスを表すが、サブシーケンス :math:`s[i \slice n]` が :math:`A^n` で置き換わる点が異なる。

* :math:`\concat(s^\ast)`： :math:`s^\ast` にあるすべてのシーケンス :math:`s_i` を結合することで形成されるフラットなシーケンスを表す。

この他に、以下の記法も使われています。

* 「:math:`x^n`」記法（:math:`x` は非終端シンボル）： :math:`x` のさまざまなシーケンスを表すメタ変数として扱われる（:math:`x^\ast`、:math:`x^+`、:math:`x^?` でも同様）。

* あるシーケンス  :math:`x^n` が与えられると、:math:`(A_1~x~A_2)^n` と書かれたシーケンス内に出現する :math:`x` は、 :math:`x^n` と点対応すると仮定される（:math:`x^\ast`、:math:`x^+`、:math:`x^?` でも同様）。これは、あるひとつのシーケンス上での構文的な構成への対応付けを暗に表現する。

以下の形式を取る積は、フィールド :math:`\K{field}_i` の固定集合からそれぞれの「値」 :math:`A_i` への対応付けの「記録（record）」と解釈されます。

.. math::
   \X{r} ~::=~ \{ \K{field}_1~A_1, \K{field}_2~A_2, \dots \}

以下の記法は、そのような記録の操作に用いられます。

* :math:`r.\K{field}`： :math:`r` の :math:`\K{field}` コンポーネントの内容を表す。

* :math:`r \with \K{field} = A`： :math:`r` と同じ記録を表すが、:math:`\K{field}` の内容が :math:`A` で置き換えられる点が異なる。

* :math:`r_1 \compose r_2`： 個別のシーケンスを点対応で後ろに追加したのと同じフィールドを持つ、2つの記録の合成（composition）を表す。

  .. math::
     \{ \K{field}_1\,A_1^\ast, \K{field}_2\,A_2^\ast, \dots \} \compose \{ \K{field}_1\,B_1^\ast, \K{field}_2\,B_2^\ast, \dots \} = \{ \K{field}_1\,A_1^\ast~B_1^\ast, \K{field}_2\,A_2^\ast~B_2^\ast, \dots \}

* :math:`\bigcompose r^\ast`： 個別の記録のあるシーケンスの合成を表す。シーケンスが空の場合は、得られる記録のすべてのフィールドは空になる。


以下のシーケンスおよび記録の更新記法は、「パス（path）」 :math:`\X{pth} ::= ([\dots] \;| \;.\K{field})^+` でアクセスされるネストしたコンポーネントについても再帰的に一般化されます。

* :math:`s \with [i]\,\X{pth} = A` は :math:`s \with [i] = (s[i] \with \X{pth} = A)` のショートハンド。

* :math:`r \with \K{field}\,\X{pth} = A` は :math:`r \with \K{field} = (r.\K{field} \with \X{pth} = A)` のショートハンド。

ただし :math:`r \with~.\K{field} = A` は :math:`r \with \K{field} = A` のショートハンドです。

.. index:: ! vector
   pair: abstract syntax; vector
.. _syntax-vec:

ベクタ
~~~~~~~

「ベクタ（vector）」は、 :math:`A^n` (or :math:`A^\ast`) の形式を取る、束縛されたシーケンスです。:math:`A` は「値」と「複雑な構成」のいずれかです。
ベクタは最大 :math:`2^{32}-1` 個の要素を持てます。

.. math::
   \begin{array}{lllll}
   \production{vector} & \vec(A) &::=&
     A^n
     & (n < 2^{32} の場合)\\
   \end{array}
