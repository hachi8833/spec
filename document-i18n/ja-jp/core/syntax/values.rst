.. index:: ! value
   pair: abstract syntax; value
.. _syntax-value:

値
------

WebAssemblyプログラムは、プリミティブな「数値」を操作します。
さらに言うと、プログラムの定義上、イミュータブルな値シーケンスは（テキスト文字列やその他のベクタなどの）さらに複雑なデータを表現するようになります。


.. index:: ! byte
   pair: abstract syntax; byte
.. _syntax-byte:

バイト（byte）
~~~~~

最もシンプルな形式の値は、解釈されない生の「バイト」です。
バイトは、抽象構文において16進数リテラルで表現されます。

.. math::
   \begin{array}{llll}
   \production{byte} & \byte &::=&
     \hex{00} ~|~ \dots ~|~ \hex{FF} \\
   \end{array}


本仕様での記法
...........

* メタ変数 :math:`b` はさまざまなバイトを表します。

* バイトは、場合によっては自然数 :math:`n < 256` として解釈されることもあります。

.. index:: ! integer, ! unsigned integer, ! signed integer, ! uninterpreted integer, bit width, two's complement
   pair: abstract syntax; integer
   pair: abstract syntax; unsigned integer
   pair: abstract syntax; signed integer
   pair: abstract syntax; uninterpreted integer
   single: integer; unsigned
   single: integer; signed
   single: integer; uninterpreted
.. _syntax-sint:
.. _syntax-uint:
.. _syntax-int:

整数（integer）
~~~~~~~~

さまざまな異なる値を持つ「整数」のさまざまなクラスは「ビット幅 :math:`N`」によって区別され、「符号なし」「符号付き」のいずれかに属します。

.. math::
   \begin{array}{llll}
   \production{unsigned integer} & \uN &::=&
     0 ~|~ 1 ~|~ \dots ~|~ 2^N{-}1 \\
   \production{signed integer} & \sN &::=&
     -2^{N-1} ~|~ \dots ~|~ {-}1 ~|~ 0 ~|~ 1 ~|~ \dots ~|~ 2^{N-1}{-}1 \\
   \production{uninterpreted integer} & \iN &::=&
     \uN \\
   \end{array}

後者のクラスは、「解釈されない」整数を定義します。符号の有無の解釈はコンテキストによって変わります。
抽象構文においては符号付き値として表現されます。
ただし、 :ref:`convert <aux-signed>` のような操作を行うと、2の補数の解釈に基づいて符号付きの値になります。

.. note::
   本仕様書に出現する主な整数は、「|u32|」「|u64|」「|s32|」「|s64|」「|i8|」「|i16|」「|i32|」「|i64|」です。
   ただし、 :ref:`浮動小数 <syntax-float>` の定義などの補助的な構成では他のサイズも出現します。

本仕様での記法
...........

* メタ変数 :math:`m, n, i` はさまざまな整数を表します。

* 数値は、上述した文法のようなシンプルな演算で表されることもあります。:math:`2^N` のような「演算」と :math:`(1)^N` のような「シーケンス」を区別するために、後者を丸かっこで囲むことで区別します。

.. index:: ! floating-point, ! NaN, payload, significand, exponent, magnitude, canonical NaN, arithmetic NaN, bit width, IEEE 754
   pair: abstract syntax; floating-point number
   single: NaN; payload
   single: NaN; canonical
   single: NaN; arithmetic
.. _syntax-nan:
.. _syntax-payload:
.. _syntax-float:

浮動小数点（floating-point）
~~~~~~~~~~~~~~

「浮動小数点」データは32ビットまたは64ビットの値を表し、それぞれ |IEEE754|_ 標準（セクション3.3）に対応するバイナリ形式があります。

浮動小数点のどの値にも「符号（sign）」と「大きさ（magnitude）」がひとつずつあります。
大きさは、:math:`m_0.m_1m_2\dots m_M \cdot2^e` のような形式を取る「通常数（normal number）」（ :math:`e` は指数部、:math:`m` は仮数部を表し、最上位ビット（MSB）は :math:`m_0` は :math:`1`）か「非正規化数（subnormal number）」（指数部が可能な最小値に固定され、:math:`m_0` が :math:`0` となる）のいずれかで表現されます。非正規化数には、正の値、負の値、ゼロ値が含まれます。
仮数部（significand）は2進値なので、通常の数値は :math:`(1 + m\cdot 2^{-M}) \cdot 2^e` の形式で表現されます（ :math:`M` が :math:`m` のビット幅を表す点は非正規数と似ています）。

可能な大きさには、特殊な「:math:`\infty` （無限）」や「|NAN| （not-a-number）」も含まれます。
NaN値には、背後の  :ref:`バイナリ表現 <aux-fbits>` にある仮数部（mantissa）ビットを記述する「ペイロード」がひとつあります。
シグナリングのあるNaNとシグナリングのないNaNは区別されません。

.. math::
   \begin{array}{llcll}
   \production{floating-point value} & \fN &::=&
     {+} \fNmag ~|~ {-} \fNmag \\
   \production{floating-point magnitude} & \fNmag &::=&
     (1 + \uM\cdot 2^{-M}) \cdot 2^e & (-2^{E-1}+2 \leq e \leq 2^{E-1}-1 の場合) \\ &&|&
     (0 + \uM\cdot 2^{-M}) \cdot 2^e & (e = -2^{E-1}+2 の場合) \\ &&|&
     \infty \\ &&|&
     \NAN(n) & (1 \leq n < 2^M の場合) \\
   \end{array}

ただし、:math:`M = \significand(N)` と :math:`E = \exponent(N)` は以下とします。

.. _aux-significand:
.. _aux-exponent:

.. math::
   \begin{array}{lclllllcl}
   \significand(32) &=& 23 &&&&
   \exponent(32) &=& 8 \\
   \significand(64) &=& 52 &&&&
   \exponent(64) &=& 11 \\
   \end{array}

.. _canonical-nan:
.. _arithmetic-nan:
.. _aux-canon:

「カノニカルNaN」は :math:`\pm\NAN(\canon_N)` という浮動小数点値です（ただし :math:`\canon_N` はMSBが :math:`1` でその他が :math:`0` のペイロード）。

.. math::
   \canon_N = 2^{\significand(N)-1}

「算術的NaN」は :math:`\pm\NAN(n)` という浮動小数点値です。（ただし :math:`n \geq \canon_N` 、MSBが :math:`1` でその他は任意の値）。

.. note::
   抽象構文における非正規数は、仮数部の最上位の0によって区別されます。非正規数の指数部には、通常数で可能な最小の指数と同じ値が使われます。
   :ref:`バイナリ表現 <binary-float>` においてのみ、非正規数の指数部は通常数の指数部と異なる形でエンコードされます。

本仕様での記法
...........

* メタ変数 :math:`z` は、コンテキストによって明らかな場合はさまざまな浮動小数点値を表します。

.. index:: ! name, byte, Unicode, UTF-8, character, binary format
   pair: abstract syntax; name
.. _syntax-char:
.. _syntax-name:

名前（name）
~~~~~

「名前」は、「文字」のシーケンスです。この文字は、|Unicode|_ （セクション 2.4）で定義される「スカラー値」です。

.. math::
   \begin{array}{llclll}
   \production{name} & \name &::=&
     \char^\ast \qquad\qquad (|\utf8(\char^\ast)| < 2^{32} の場合) \\
   \production{character} & \char &::=&
     \unicode{00} ~|~ \dots ~|~ \unicode{D7FF} ~|~
     \unicode{E000} ~|~ \dots ~|~ \unicode{10FFFF} \\
   \end{array}

:ref:`バイナリ形式 <binary-name>` の制約により、名前の長さは名前の :ref:`UTF-8 <binary-utf8>` エンコーディングの長さによって束縛されます。

本仕様での記法
..........

* 文字（Unicodeスカラー値）は、場合によっては自然数 :math:`n < 1114112` と入れ替え可能です。
