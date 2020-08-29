.. index:: ! text format, Unicode, UTF-8, S-expression, identifier, file extension, abstract syntax

記法
-----------

WebAssembly :ref:`モジュール <module>` のテキスト形式は、モジュールの :ref:`抽象構文 <syntax-module>` をレンダリングして |SExpressions|_ にしたものです。

:ref:`バイナリ形式 <binary>` の場合と同様、テキスト形式も「属性文法（attribute grammar）」で定義されます。
1つのテキスト文字列は、この文法によって生成された場合に限り、モジュールの整形式記述となります。

この構文での個別の生成物は、最大1個の「合成属性（synthesized attribute）」（個別のバイトシーケンスがエンコードする抽象構文）を持ちます。
したがって、この属性文法は関数の「パース(parsing)」を暗黙で定義します。

生成物によっては、束縛された :ref:`識別子 <text-id>` を記録する、継承された属性として :ref:`コンテキスト <text-context>` をひとつ受け取るものもあります。

わずかな例外を除いて、バイナリ構文は抽象構文の文法をほぼ鏡写しにしたものとなります。
しかしバイナリ構文は、コアとなる構文の「シンタックスシュガー」となる「短縮形」も多数定義しています。
テキスト形式のWebAssemblyモジュールを含むファイルの拡張子は、「:math:`\T{.wat}`」が推奨されます。
この拡張子を持つファイルは、|Unicode|_ （セクション2.5）で規定されているUTF-8でエンコードされていると仮定されます。

.. index:: grammar notation, notation, Unicode
   single: text format; grammar
   pair: text format; notation
.. _text-grammar:

文法
~~~~~~~

以下の記法は、テキスト形式の文法ルールの定義に採用されています。
これは、:ref:`抽象構文 <grammar>` や the :ref:`バイナリ形式 <binary>` で用いられている記法の写しです。
テキスト構文のシンボルと抽象構文のシンボルを区別するために、前者の表記に :math:`\mathtt{typewriter}` フォントを採用します。

* 終端シンボルは、引用符でくくられたリテラル文字、または |Unicode|_ スカラー値のいずれかで表現される（:math:`\text{module}` と :math:`\unicode{0A}`）（リテラル表記されるすべての文字は Unicodeの 7ビット |ASCII|_ サブセットによってあいまいさなしで表現される）。

* 非終端シンボルはtypewriterフォント :math:`\T{valtype}, \T{instr}` で表記される。

* :math:`T^n` （:math:`n\geq 0`）は :math:`T` を繰り返す1個のシーケンスである。

* :math:`T^\ast` は :math:`T` を繰り返す1個のシーケンスだが、シーケンスが空の可能性もある（:math:`T^n` のショートハンドであり、:math:`n` が本筋に関係ない場合に用いられる）。

* :math:`T^+` は :math:`T` を1回以上の繰り返す1個のシーケンスである（:math:`n \geq 1` である :math:`T^n` のショートハンド）。

* :math:`T^?` は :math:`T` がある場合とない場合を表す（:math:`n \leq 1` である :math:`T^n` のショートハンド）。

* :math:`x{:}T` は非終端 :math:`T` と同じ言語を表すが、:math:`T` について合成された属性への変数 :math:`x` の束縛をも表す。

* 積（production）は :math:`\T{sym} ::= T_1 \Rightarrow A_1 ~|~ \dots ~|~ T_n \Rightarrow A_n` で記述される。ただし個別の :math:`A_i` はそのケースにおいて :math:`\T{sym}` について合成された属性を表す（通常は :math:`T_i` 内で束縛された属性変数から合成される）。

* 積によっては、丸かっこで囲まれた副条件で拡張されることもある。この副条件は、その積の適用範囲を制限する。これらは、積の組み合わせ拡張を多くの個別のケースで表すショートハンドを提供する。

.. _text-syntactic:

* 「レキシカルな（lexical: 意味に踏み込まない字面的な）」生成物と「構文的な（syntactic）」生成物は区別される。後者については、文法にスペース文字が含まれている場所であればどこにでも任意の :ref:`ホワイトスペース <text-space>` を配置できる。:ref:`レキシカル構文 <text-lexical>` を定義する生成物と、:Ref:`値 <text-value>` を定義する構文はレキシカルとみなされ、それ以外はすべて構文的とみなされる。

.. note::
   たとえば、:ref:`値型 <syntax-valtype>` の :ref:`テキスト文法 <text-valtype>` は以下のように与えられます。

   .. math::
     \begin{array}{llcll@{\qquad\qquad}l}
     \production{value types} & \Tvaltype &::=&
       \text{i32} &\Rightarrow& \I32 \\ &&|&
       \text{i64} &\Rightarrow& \I64 \\ &&|&
       \text{f32} &\Rightarrow& \F32 \\ &&|&
       \text{f64} &\Rightarrow& \F64 \\
     \end{array}

   :ref:`制限 <syntax-limits>` の :ref:`テキスト文法 <text-valtype>` は以下のように定義されます。

   .. math::
      \begin{array}{llclll}
      \production{limits} & \Tlimits &::=&
        n{:}\Tu32 &\Rightarrow& \{ \LMIN~n, \LMAX~\epsilon \} \\ &&|&
        n{:}\Tu32~~m{:}\Tu32 &\Rightarrow& \{ \LMIN~n, \LMAX~m \} \\
      \end{array}

   変数 :math:`n` と :math:`m` は、それぞれ |Tu32| 非終端属性を表し、ここではこれらがデコードされる実際の :ref:`符号なし整数 <syntax-uint>` となります。
   これにより、この完全な積の属性は、元の値で表現されていた制約を表す抽象構文となります。

.. index:: ! abbreviations, rewrite rule
.. _text-abbreviations:

短縮形（abbreviation）
~~~~~~~~~~~~~

テキスト構文では、:ref:`抽象構文 <syntax>` と直接対応するコア文法のほかに、利便性および読みやすさのため「短縮形」も多数定義されています。

短縮形は、その拡張形（expansion）をコア構文に指定する「書き換えルール」によって定義されます。

.. math::
   \X{短縮形構文} \quad\equiv\quad \X{拡張形構文}

これらの拡張形は、抽象構文を構成するコア文法を適用する前に、再帰的かつ出現順に適用することが仮定されます。

.. index:: ! identifier context, identifier, index, index space
.. _text-context-wf:
.. _text-context:

コンテキスト（context）
~~~~~~~~

テキスト形式では、:ref:`インデックス <syntax-index>` の代わりにシンボリックな :ref:`識別子 <text-id>` の利用が許されています。
これらの識別子を具体的なインデックスに解決するために、一部の文法生成物は「識別子コンテキスト（identifier context）」 :math:`I` でインデックス化されます。これは、:ref:`インデックス空間 <syntax-index>` ごとに宣言される1つ以上の識別子を記録します。
これに加えて、コンテキストはそのモジュール内で定義される型も記録し、:ref:`関数 <text-func>` で :ref:`パラメータ <text-param>` インデックスが算出されるようにします。

識別子コンテキストを、以下のように抽象構文で :ref:`記録 <notation-record>` :math:`I` として定義すると便利です。

.. math::
   \begin{array}{llll}
   \production{(identifier context)} & I &::=&
     \begin{array}[t]{l@{~}ll}
     \{ & \ITYPES & (\Tid^?)^\ast, \\
        & \IFUNCS & (\Tid^?)^\ast, \\
        & \ITABLES & (\Tid^?)^\ast, \\
        & \IMEMS & (\Tid^?)^\ast, \\
        & \IGLOBALS & (\Tid^?)^\ast, \\
        & \ILOCALS & (\Tid^?)^\ast, \\
        & \ILABELS & (\Tid^?)^\ast, \\
        & \ITYPEDEFS & \functype^\ast ~\} \\
     \end{array}
   \end{array}

インデックス空間ごとに、そのようなコンテキストは、定義されたインデックスに割り当てられた :ref:`識別子 <text-id>` のリストを含みます。
無名インデックスはこれらのリストで空エントリ（:math:`\epsilon`）に関連付けられます。

識別子コンテキストは、識別子がインデックス空間内で重複していない場合に「整形式（well-formed）」となります。


本仕様での記法
...........

記述の不要な煩雑さを避けるため、空のコンポーネントは識別子コンテキストの表記では省略します。
たとえば、記録 :math:`\{\}` は、そのコンポーネントがすべて空である :ref:`識別子コンテキスト <text-context>` のショートハンドです。

.. index:: vector
   pair: text format; vector
.. _text-vec:

ベクタ（vector）
~~~~~~~

:ref:`ベクタ <syntax-vec>` はひとつの純粋なシーケンスとして記述しますが、それらのシーケンスの長さには制約があります。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{vector} & \Tvec(\T{A}) &::=&
     (x{:}\T{A})^n &\Rightarrow& x^n & (\iff n < 2^{32}) \\
   \end{array}
