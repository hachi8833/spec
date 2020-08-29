.. index:: value
   pair: binary format; value
.. _binary-value:

値
------


.. index:: byte
   pair: binary format; byte
.. _binary-byte:

バイト（byte）
~~~~~

:ref:`バイト <syntax-byte>` は、バイト自身をエンコードしたものです。

.. math::
   \begin{array}{llcll@{\qquad}l}
   \production{byte} & \Bbyte &::=&
     \hex{00} &\Rightarrow& \hex{00} \\ &&|&&
     \dots \\ &&|&
     \hex{FF} &\Rightarrow& \hex{FF} \\
   \end{array}


.. index:: integer, unsigned integer, signed integer, uninterpreted integer, LEB128, two's complement
   pair: binary format; integer
   pair: binary format; unsigned integer
   pair: binary format; signed integer
   pair: binary format; uninterpreted integer
.. _binary-sint:
.. _binary-uint:
.. _binary-int:

整数（integer）
~~~~~~~~

あらゆる :ref:`整数 <syntax-int>` は |LEB128|_ 可変長整数エンコーディングを用いて、符号なしか符号ありのいずれかにエンコードされます。

:ref:`符号なし整数 <syntax-uint>` は |UnsignedLEB128|_ 形式でエンコードされます。
追加の制約として、型 :math:`\uN` の値1個をエンコードするバイト数の合計は :math:`\F{ceil}(N/7)` を超えてはいけません。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{unsigned integer} & \BuN &::=&
     n{:}\Bbyte &\Rightarrow& n & (\iff n < 2^7 \wedge n < 2^N) \\ &&|&
     n{:}\Bbyte~~m{:}\BuX{(N\B{-7})} &\Rightarrow&
       2^7\cdot m + (n-2^7) & (\iff n \geq 2^7 \wedge N > 7) \\
   \end{array}

:ref:`符号あり整数 <syntax-sint>` は |SignedLEB128|_ でエンコードされ、2の補数表現を用います。
追加の制約として、型 :math:`\sN` の値1個をエンコードするバイト数の合計は :math:`\F{ceil}(N/7)` を超えてはいけません。

.. math::
   \begin{array}{llclll@{\qquad}l}
   \production{signed integer} & \BsN &::=&
     n{:}\Bbyte &\Rightarrow& n & (\iff n < 2^6 \wedge n < 2^{N-1}) \\ &&|&
     n{:}\Bbyte &\Rightarrow& n-2^7 & (\iff 2^6 \leq n < 2^7 \wedge n \geq 2^7-2^{N-1}) \\ &&|&
     n{:}\Bbyte~~m{:}\BsX{(N\B{-7})} &\Rightarrow&
       2^7\cdot m + (n-2^7) & (\iff n \geq 2^7 \wedge N > 7) \\
   \end{array}

:ref:`解釈されない整数 <syntax-int>` は符号あり整数としてエンコードされます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{uninterpreted integer} & \BiN &::=&
     n{:}\BsN &\Rightarrow& i & (\iff n = \signed_{\iN}(i))
   \end{array}

.. note::
   :math:`\uX{}` エンコーディングと :math:`\sX{}` エンコーディングの非終端バイトを表す積における副条件 :math:`N > 7` は、エンコーディングの長さを制約します。
   しかしこれらの束縛内でも「下位桁のゼロ埋め」は引き続き許容されます。
   たとえば  :math:`\hex{03}` と :math:`\hex{83}~\hex{00}` は、どちらも |u8| の値 :math:`3` を整形式でエンコーディングしたものです。
   同様に、:math:`\hex{7e}` と :math:`\hex{FE}~\hex{7F}` と :math:`\hex{FE}~\hex{FF}~\hex{7F}` は、いずれも |s16| の値 :math:`-2` を整形式でエンコーディングしたものです。

   終端バイトの値 :math:`n` における副条件はさらに制約を強め、「正の値では未使用ビットをすべて :math:`0` にしなければならない」「負の値では未使用ビットをすべて :math:`1` にしなければならない」よう強制します。
   たとえば、:math:`\hex{83}~\hex{10}` は |u8| エンコーディングとして無効です。
   同様に、:math:`\hex{83}~\hex{3E}` や :math:`\hex{FF}~\hex{7B}` も |s8| エンコーディングとして無効です。

.. index:: floating-point number, little endian
   pair: binary format; floating-point number
.. _binary-float:

浮動小数点（floating-point）
~~~~~~~~~~~~~~

:ref:`浮動小数点 <syntax-float>` 値は、|IEEE754|_ （セクション3.4）の ビットパターンによって、|LittleEndian|_ バイトオーダーで直接エンコーディングされます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{floating-point value} & \BfN &::=&
     b^\ast{:\,}\Bbyte^{N/8} &\Rightarrow& \bytes_{\fN}^{-1}(b^\ast) \\
   \end{array}


.. index:: name, byte, Unicode, ! UTF-8
   pair: binary format; name
.. _binary-utf8:
.. _binary-name:

名前（name）
~~~~~

1個の :ref:`名前 <syntax-name>` は、|Unicode|_ （セクション3.9）UTF-8エンコーディングされたその名前の文字シーケンスを含むバイトの :ref:`ベクタ <binary-vec>` 1個としてエンコードされます。

.. math::
   \begin{array}{llclllll}
   \production{name} & \Bname &::=&
     b^\ast{:}\Bvec(\Bbyte) &\Rightarrow& \name
       && (\iff \utf8(\name) = b^\ast) \\
   \end{array}

このエンコーディングを表現する補助的な |utf8| 関数は以下のように定義されます。

.. math::
   \begin{array}{@{}l@{}}
   \begin{array}{@{}lcl@{\qquad}l@{}}
   \utf8(c^\ast) &=& (\utf8(c))^\ast \\[1ex]
   \utf8(c) &=& b &
     (\begin{array}[t]{@{}c@{~}l@{}}
      \iff & c < \unicode{80} \\
      \wedge & c = b) \\
      \end{array} \\
   \utf8(c) &=& b_1~b_2 &
     (\begin{array}[t]{@{}c@{~}l@{}}
      \iff & \unicode{80} \leq c < \unicode{800} \\
      \wedge & c = 2^6(b_1-\hex{C0})+(b_2-\hex{80})) \\
      \end{array} \\
   \utf8(c) &=& b_1~b_2~b_3 &
     (\begin{array}[t]{@{}c@{~}l@{}}
      \iff & \unicode{800} \leq c < \unicode{D800} \vee \unicode{E000} \leq c < \unicode{10000} \\
      \wedge & c = 2^{12}(b_1-\hex{E0})+2^6(b_2-\hex{80})+(b_3-\hex{80})) \\
      \end{array} \\
   \utf8(c) &=& b_1~b_2~b_3~b_4 &
     (\begin{array}[t]{@{}c@{~}l@{}}
      \iff & \unicode{10000} \leq c < \unicode{110000} \\
      \wedge & c = 2^{18}(b_1-\hex{F0})+2^{12}(b_2-\hex{80})+2^6(b_3-\hex{80})+(b_4-\hex{80})) \\
      \end{array} \\
   \end{array} \\
   \where b_2, b_3, b_4 < \hex{C0} \\
   \end{array}

.. note::
   他の形式と異なり、名前の文字列の終端はゼロになりません。
