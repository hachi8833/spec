.. index:: value
   pair: text format; value
.. _text-value:

値
------

このセクションにおける文法生成物は「レキシカル構文」を定義するので、:ref:`ホワイトスペース <text-space>` の利用は許されません。

.. index:: integer, unsigned integer, signed integer, uninterpreted integer
   pair: text format; integer
   pair: text format; unsigned integer
   pair: text format; signed integer
   pair: text format; uninterpreted integer
.. _text-sign:
.. _text-digit:
.. _text-hexdigit:
.. _text-num:
.. _text-hexnum:
.. _text-sint:
.. _text-uint:
.. _text-int:

整数（integer）
~~~~~~~~

あらゆる :ref:`整数 <syntax-int>` は、「10進数記法」または「16進記法」のどちらでも記述できます。
いずれの場合も、オプションで桁をアンダースコア文字で区切ることができます。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{sign} & \Tsign &::=&
     \epsilon \Rightarrow {+} ~~|~~
     \text{+} \Rightarrow {+} ~~|~~
     \text{-} \Rightarrow {-} \\
   \production{decimal digit} & \Tdigit &::=&
     \text{0} \Rightarrow 0 ~~|~~ \dots ~~|~~ \text{9} \Rightarrow 9 \\
   \production{hexadecimal digit} & \Thexdigit &::=&
     d{:}\Tdigit \Rightarrow d \\ &&|&
     \text{A} \Rightarrow 10 ~~|~~ \dots ~~|~~ \text{F} \Rightarrow 15 \\ &&|&
     \text{a} \Rightarrow 10 ~~|~~ \dots ~~|~~ \text{f} \Rightarrow 15
   \\[1ex]
   \production{decimal number} & \Tnum &::=&
     d{:}\Tdigit &\Rightarrow& d \\ &&|&
     n{:}\Tnum~~\text{\_}^?~~d{:}\Tdigit &\Rightarrow& 10\cdot n + d \\
   \production{hexadecimal number} & \Thexnum &::=&
     h{:}\Thexdigit &\Rightarrow& h \\ &&|&
     n{:}\Thexnum~~\text{\_}^?~~h{:}\Thexdigit &\Rightarrow& 16\cdot n + h \\
   \end{array}

整数リテラルで許される構文は、サイズと符号に依存します。
さらに、値はそれぞれの型に応じた範囲に収まらなければなりません。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{unsigned integer} & \TuN &::=&
     n{:}\Tnum &\Rightarrow& n & (\iff n < 2^N) \\ &&|&
     \text{0x}~~n{:}\Thexnum &\Rightarrow& n & (\iff n < 2^N) \\
   \production{signed integer} & \TsN &::=&
     {\pm}{:}\Tsign~~n{:}\Tnum &\Rightarrow& \pm n & (\iff -2^{N-1} \leq \pm n < 2^{N-1}) \\ &&|&
     {\pm}{:}\Tsign~~\text{0x}~~n{:}\Thexnum &\Rightarrow& \pm n & (\iff -2^{N-1} \leq \pm n < 2^{N-1}) \\
   \end{array}

:ref:`解釈されない整数 <syntax-int>` は「符号付き」「符号なし」のどちらでも記述でき、抽象構文上で「符号なし」として正規化されます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{uninterpreted integers} & \TiN &::=&
     n{:}\TuN &\Rightarrow& n \\ &&|&
     i{:}\TsN &\Rightarrow& n & (\iff i = \signed(n)) \\
   \end{array}


.. index:: floating-point number
   pair: text format; floating-point number
.. _text-frac:
.. _text-hexfrac:
.. _text-hexfloat:
.. _text-float:

浮動小数点（floating-point）
~~~~~~~~~~~~~~

:ref:`浮動小数点 <syntax-float>` 値は、「10進数記法」または「16進記法」のどちらでも記述できます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{decimal floating-point fraction} & \Tfrac &::=&
     d{:}\Tdigit &\Rightarrow& d/10 \\ &&|&
     d{:}\Tdigit~~\text{\_}^?~~p{:}\Tfrac &\Rightarrow& (d+p/10)/10 \\
   \production{hexadecimal floating-point fraction} & \Thexfrac &::=&
     h{:}\Thexdigit &\Rightarrow& h/16 \\ &&|&
     h{:}\Thexdigit~~\text{\_}^?~~p{:}\Thexfrac &\Rightarrow& (h+p/16)/16 \\
   \production{decimal floating-point number} & \Tfloat &::=&
     p{:}\Tnum~\text{.}^?
       &\Rightarrow& p \\ &&|&
     p{:}\Tnum~\text{.}~q{:}\Tfrac
       &\Rightarrow& p+q \\ &&|&
     p{:}\Tnum~\text{.}^?~(\text{E}~|~\text{e})~{\pm}{:}\Tsign~e{:}\Tnum
       &\Rightarrow& p\cdot 10^{\pm e} \\ &&|&
     p{:}\Tnum~\text{.}~q{:}\Tfrac~(\text{E}~|~\text{e})~{\pm}{:}\Tsign~e{:}\Tnum
       &\Rightarrow& (p+q)\cdot 10^{\pm e} \\
   \production{hexadecimal floating-point number} & \Thexfloat &::=&
     \text{0x}~p{:}\Thexnum~\text{.}^?
       &\Rightarrow& p \\ &&|&
     \text{0x}~p{:}\Thexnum~\text{.}~q{:}\Thexfrac
       &\Rightarrow& p+q \\ &&|&
     \text{0x}~p{:}\Thexnum~\text{.}^?~(\text{P}~|~\text{p})~{\pm}{:}\Tsign~e{:}\Tnum
       &\Rightarrow& p\cdot 2^{\pm e} \\ &&|&
     \text{0x}~p{:}\Thexnum~\text{.}~q{:}\Thexfrac~(\text{P}~|~\text{p})~{\pm}{:}\Tsign~e{:}\Tnum
       &\Rightarrow& (p+q)\cdot 2^{\pm e}
   \end{array}

リテラルの値は、対応する |IEEE754|_ 型で表現可能な範囲を超えてはいけません（つまり、ある数値は :math:`\pm\mbox{infinity}` にオーバーフローしてはいけません）が、表現可能な最も近い値に :ref:`丸められる <aux-ieee>` 可能性があります。

.. note::
   必要な型でサポートされている以上の有効ビットを持たない16進表記を用いると、丸め処理を防止できます。

浮動小数点値は、「infinity（無限）」または「カノニカルNaN」という定数で記述されることもあります。
さらに、明示的なペイロード値を提供することで任意のNaN値を表現できます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{floating-point value} & \TfN &::=&
     {\pm}{:}\Tsign~z{:}\TfNmag &\Rightarrow& \pm z \\
   \production{floating-point magnitude} & \TfNmag &::=&
     z{:}\Tfloat &\Rightarrow& \ieee_N(z) & (\iff \ieee_N(z) \neq \pm \infty) \\ &&|&
     z{:}\Thexfloat &\Rightarrow& \ieee_N(z) & (\iff \ieee_N(z) \neq \pm \infty) \\ &&|&
     \text{inf} &\Rightarrow& \infty \\ &&|&
     \text{nan} &\Rightarrow& \NAN(2^{\significand(N)-1}) \\ &&|&
     \text{nan{:}0x}~n{:}\Thexnum &\Rightarrow& \NAN(n) & (\iff 1 \leq n < 2^{\significand(N)}) \\
   \end{array}


.. index:: ! string, byte, character, ASCII, Unicode, UTF-8
   pair: text format; byte
   pair: text format; string
.. _text-byte:
.. _text-string:

文字列（string）
~~~~~~~

「文字列」は、テキストデータとバイナリデータのどちらも表現できるバイトのシーケンスを記述します。
文字列は引用符で囲まれ、「|ASCII|_ 制御文字」「引用符（:math:`\text{"}`）」「バックスラッシュ（:math:`\text{\backslash}`）」を除く任意の文字を含められます（「エスケープシーケンス」を用いるとこれらの文字も表現できます）。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{string} & \Tstring &::=&
     \text{"}~(b^\ast{:}\Tstringelem)^\ast~\text{"}
       &\Rightarrow& \concat((b^\ast)^\ast)
       & (\iff |\concat((b^\ast)^\ast)| < 2^{32}) \\
   \production{string element} & \Tstringelem &::=&
     c{:}\Tstringchar &\Rightarrow& \utf8(c) \\ &&|&
     \text{\backslash}~n{:}\Thexdigit~m{:}\Thexdigit
       &\Rightarrow& 16\cdot n+m \\
   \end{array}

文字列リテラル内の個別の文字は、そのUTF-8 |Unicode|_ （セクション2.5）エンコーディングに対応するバイトシーケンスで表現されます。例外として、16進エスケープシーケンス :math:`\textl\backslash hh\textr` は対応する値の生バイトを表現します。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{string character} & \Tstringchar &::=&
     c{:}\Tchar &\Rightarrow& c \qquad
       & (\iff c \geq \unicode{20} \wedge c \neq \unicode{7F} \wedge c \neq \text{"} \wedge c \neq \text{\backslash}) \\ &&|&
     \text{\backslash t} &\Rightarrow& \unicode{09} \\ &&|&
     \text{\backslash n} &\Rightarrow& \unicode{0A} \\ &&|&
     \text{\backslash r} &\Rightarrow& \unicode{0D} \\ &&|&
     \text{\backslash{"}} &\Rightarrow& \unicode{22} \\ &&|&
     \text{\backslash{'}} &\Rightarrow& \unicode{27} \\ &&|&
     \text{\backslash\backslash} &\Rightarrow& \unicode{5C} \\ &&|&
     \text{\backslash u\{}~n{:}\Thexnum~\text{\}}
       &\Rightarrow& \unicode{(n)} & (\iff n < \hex{D800} \vee \hex{E000} \leq n < \hex{110000}) \\
   \end{array}


.. index:: name, byte, character, character
   pair: text format; name
.. _text-name:

名前（name）
~~~~~

:ref:`名前 <syntax-name>` は、あるリテラル文字シーケンスを表します。
名前文字列は、|Unicode|_ （セクション2.5）で定義される有効なUTF-8エンコーディングでできていなければならず、Unicodeスカラー値の文字列として解釈されます。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{name} & \Tname &::=&
     b^\ast{:}\Tstring &\Rightarrow& c^\ast & (\iff b^\ast = \utf8(c^\ast)) \\
   \end{array}

.. note::
   16進エスケープをまったく含まない文字列は常に有効な名前です（ただしソーステキスト自体が正しくエンコードされていることが前提です）。

.. index:: ! identifiers
   pair: text format; identifiers
.. _text-idchar:
.. _text-id:

識別子（identifier）
~~~~~~~~~~~

:ref:`インデックス <syntax-index>` は「数値形式」「シンボル形式」のどちらでも与えられます。
インデックスの代わりとなるシンボリックな「識別子」は :math:`\text{\$}` で始まり、「スペース」「引用符」「カンマ」「セミコロン」「波かっこ」を含まない任意の印刷可能な |ASCII|_ 文字のシーケンスがその後ろに続きます。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{identifier} & \Tid &::=&
     \text{\$}~\Tidchar^+ \\
   \production{identifier character} & \Tidchar &::=&
     \text{0} ~~|~~ \dots ~~|~~ \text{9} \\ &&|&
     \text{A} ~~|~~ \dots ~~|~~ \text{Z} \\ &&|&
     \text{a} ~~|~~ \dots ~~|~~ \text{z} \\ &&|&
     \text{!} ~~|~~
     \text{\#} ~~|~~
     \text{\$} ~~|~~
     \text{\%} ~~|~~
     \text{\&} ~~|~~
     \text{'} ~~|~~
     \text{*} ~~|~~
     \text{+} ~~|~~
     \text{-} ~~|~~
     \text{.} ~~|~~
     \text{/} \\ &&|&
     \text{:} ~~|~~
     \text{<} ~~|~~
     \text{=} ~~|~~
     \text{>} ~~|~~
     \text{?} ~~|~~
     \text{@} ~~|~~
     \text{\backslash} ~~|~~
     \text{\hat{~~}} ~~|~~
     \text{\_} ~~|~~
     \text{\grave{~~}} ~~|~~
     \text{|} ~~|~~
     \text{\tilde{~~}} \\
   \end{array}

.. _text-id-fresh:

本仕様での記法
...........

一部の短縮形ルールを拡張するには、与えられたソーステキストに出現していない「フレッシュな」識別子の挿入が必要です。
構文的に有効であれば任意の識別子を利用できます。
