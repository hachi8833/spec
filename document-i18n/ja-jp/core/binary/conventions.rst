.. index:: ! binary format, module, byte, file extension, abstract syntax

記法
-----------

WebAssembly :ref:`モジュール <module>` 向けのバイナリ形式は、:ref:`抽象構文 <syntax-module>` を濃縮した線形な「エンコーディング」です。
[#compression]_

この形式は「属性文法（attribute grammar）」によって定義され、終端シンボルのみ :ref:`バイト <syntax-byte>` となります。
1個のバイトシーケンスは、この文法によって生成された場合に限り、あるモジュールを整形式でエンコードしたものとなります。

この構文での個別の生成物は、正確に1個の「合成属性（synthesized attribute）」（個別のバイトシーケンスがエンコードする抽象構文）を持ちます。
したがって、この属性文法は関数の「デコード方法」（すなわちバイナリ形式での関数のパース）を暗黙で定義します。

わずかな例外を除いて、バイナリ構文は抽象構文の文法をほぼ鏡写しにしたものとなります。

.. note::
   抽象構文の句の中には、バイナリ形式で可能なエンコード方法が複数あるものもあります。
   たとえば数値は、オプションで上位桁をゼロで埋めているかのようにエンコードされることもあります。
   デコーダーの実装は、可能なあらゆるデコード方法をサポートしなければなりません。
   エンコーダーの実装は、許されているエンコード方法の中からどれを選んでも構いません。

バイナリ形式内にWebAssemblyモジュールを含むファイルの拡張子としては「:math:`\T{.wasm}`」が推奨され、|MediaType|_ としては「:math:`\T{application/wasm}`」が推奨されます。

.. [#compression]
   追加のエンコーディング層（圧縮の導入など）が、ここで定義される基本的な表現の上に定義される可能性があります。
   しかし、そのような層は現在の仕様の範疇からは外れます。

.. index:: grammar notation, notation, byte
   single: binary format; grammar
   pair: binary format; notation
.. _binary-grammar:

文法（grammar）
~~~~~~~

バイナリ形式向けの文法ルールの定義には、以下の記法を採用します。
これらは :ref:`抽象構文 <grammar>` で採用されている記法の写しです。
バイナリ構文のシンボルと抽象構文のシンボルを区別するために、前者の表記に :math:`\mathtt{typewriter}` フォントを採用します。

* 終端シンボルは、16進数記法 :math:`\hex{0F}` で表現される :ref:`バイト <syntax-byte>` である。

* 非終端シンボルはtypewriterフォント :math:`\B{valtype}, \B{instr}` で表記される。

* :math:`B^n` （:math:`n\geq 0`）は :math:`B` を繰り返す1個のシーケンスである。

* :math:`B^\ast` は :math:`B` を繰り返す1個のシーケンスだが、シーケンスが空の可能性もある（:math:`B^n` のショートハンドであり、:math:`n` が本筋に関係ない場合に用いられる）。

* :math:`B^?` は :math:`B` がある場合とない場合を表す（:math:`n \leq 1` である :math:`B^n` のショートハンド）。

* :math:`x{:}B` は非終端 :math:`B` と同じ言語を表すが、:math:`B` について合成された属性への変数 :math:`x` の束縛をも表す。

* 積（production）は :math:`\B{sym} ::= B_1 \Rightarrow A_1 ~|~ \dots ~|~ B_n \Rightarrow A_n` で記述される。ただし個別の :math:`A_i` はそのケースにおいて :math:`\B{sym}` について合成された属性を表す（通常は :math:`B_i` 内で束縛された属性変数から合成される）。

* 積によっては、丸かっこで囲まれた副条件で拡張されることもある。この副条件は、その積の適用範囲を制限する。これらは、積の組み合わせ拡張を多くの個別のケースで表すショートハンドを提供する。

.. note::
   たとえば、:ref:`値型 <syntax-valtype>` で用いる :ref:`バイナリ文法 <binary-valtype>` は以下のように与えられます。

   .. math::
     \begin{array}{llcll@{\qquad\qquad}l}
     \production{value types} & \Bvaltype &::=&
       \hex{7F} &\Rightarrow& \I32 \\ &&|&
       \hex{7E} &\Rightarrow& \I64 \\ &&|&
       \hex{7D} &\Rightarrow& \F32 \\ &&|&
       \hex{7C} &\Rightarrow& \F64 \\
     \end{array}

   これにより、たとえばバイト :math:`\hex{7F}` は型 |I32| をエンコードし、:math:`\hex{7E}` は型 |I64| をエンコードします。
   ひとつの値型をそれ以外のバイト値でエンコードすることは許されません。


   :ref:`制限 <syntax-limits>` の :ref:`バイナリ文法 <binary-valtype>` は以下のように定義されます。

   .. math::
      \begin{array}{llclll}
      \production{limits} & \Blimits &::=&
        \hex{00}~~n{:}\Bu32 &\Rightarrow& \{ \LMIN~n, \LMAX~\epsilon \} \\ &&|&
        \hex{01}~~n{:}\Bu32~~m{:}\Bu32 &\Rightarrow& \{ \LMIN~n, \LMAX~m \} \\
      \end{array}

   すなわち、ある制限ペアは「バイト :math:`\hex{00}` とそれに続く |U32| 値のエンコード」または「バイト :math:`\hex{01}` とそれに続くそれらのエンコーディング」のいずれかにエンコードされます。
   変数 :math:`n` と :math:`m` は、それぞれ |Bu32| 非終端属性を表し、ここではこれらがデコードされる実際の :ref:`符号なし整数 <syntax-uint>` となります。
   これにより、この完全な積の属性は、元の値で表現されていた制約を表す抽象構文となります。


.. _binary-notation:

補助記法
~~~~~~~~~~~~~~~~~~

バイナリエンコーディングを扱う場合、以下の記法も用います。

* :math:`\epsilon` は空のバイトシーケンスを表す。

* :math:`||B||` は、導出において積 :math:`B` から生成されるバイトシーケンスの長さを表す。


.. index:: vector
   pair: binary format; vector
.. _binary-vec:

ベクタ（vector）
~~~~~~~

:ref:`ベクタ <syntax-vec>` は、ベクタの長さ |Bu32| でエンコードされ、ベクタの要素シーケンスのエンコーディングがその後に続きます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{vector} & \Bvec(\B{B}) &::=&
     n{:}\Bu32~~(x{:}\B{B})^n &\Rightarrow& x^n \\
   \end{array}
