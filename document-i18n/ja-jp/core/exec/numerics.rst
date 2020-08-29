.. index:: value, integer, floating-point, bit width, determinism, NaN
.. _exec-op-partial:
.. _exec-numeric:

数値
--------

数値プリミティブは、あるビット幅 :math:`N` に対してインデックスされる演算子（operator）によって一般的な方法で定義されます。

演算子はいくつもの可能な結果を返す（さまざまな :ref:`NaN <syntax-nan>` 値など）可能性もあるため、中には「非決定的」な演算子もあります。
つまりテクニカルに言うと、個別の演算子が返すのは、許されている値のひとつの「集合（set）」です。
簡便のため、決定的な結果は普通の値で表現されます。これらはそれぞれ単集合（singleton set）として識別されると仮定されます。

演算子の中には、特定の入力に対する定義が存在しないために「パーシャル（partial）」なものもあります。
テクニカルに言うと、そのような入力の結果として空集合がひとつ返されます。

形式的記法においては、個別の演算子は優先順位の高い順に適用されるいくつかの方程式句（clause）によって定義されます。
すなわち、渡された引数に適用可能な最初の句が結果を定義することになります。
場合によっては、似たようないくつかの句が :math:`\pm` 記法や :math:`\mp` 記法でひとつに結合されます。
ひとつの句の中にそのようなプレースホルダがいくつもある場合、それらは一貫した方法で解決されなければなりません。すなわち、すべてについて上の記号または下の記号のいずれか一方のみを選ぶということです。

.. note::
   たとえば |fcopysign| 演算子は以下のように定義されます。

   .. math::
      \begin{array}{@{}lcll}
      \fcopysign_N(\pm p_1, \pm p_2) &=& \pm p_1 \\
      \fcopysign_N(\pm p_1, \mp p_2) &=& \mp p_1 \\
      \end{array}

   上の定義は、上のそれぞれの句を以下のような2つの句に展開したもののショートハンドとして理解されるべきです。

   .. math::
      \begin{array}{@{}lcll}
      \fcopysign_N(+ p_1, + p_2) &=& + p_1 \\
      \fcopysign_N(- p_1, - p_2) &=& - p_1 \\
      \fcopysign_N(+ p_1, - p_2) &=& - p_1 \\
      \fcopysign_N(- p_1, + p_2) &=& + p_1 \\
      \end{array}

.. _aux-trunc:

本仕様での記法は以下のとおりです。

* メタ変数 :math:`d` は、さまざまな単一ビットを表します。

* メタ変数 :math:`p` は、|NAN| や :math:`\infty` を含む浮動小数点値のさまざまな :ref:`大きさ <syntax-float>` を表します。

* メタ変数 :math:`q` は、|NAN| や :math:`\infty` を除く浮動小数点値の「有理数」のさまざまな :ref:`大きさ <syntax-float>` を表します。

* :math:`f^{-1}` 記法は、全単射関数（bijective function） :math:`f` の逆関数を表します。

* 有理数値の切り落とし（truncation）は :math:`\trunc(\pm q)` で表記されます。通常の数学的定義は以下のようになります。

  .. math::
     \begin{array}{lll@{\qquad}l}
     \trunc(\pm q) &=& \pm i & (\iff i \in \mathbb{N} \wedge +q - 1 < i \leq +q) \\
     \end{array}


.. index:: bit, integer, floating-point
.. _aux-bits:

表現形式（representation）
~~~~~~~~~~~~~~~

数字は、ひとつのビットシーケンスとしてのバイナリ表現形式に基づいています。

.. math::
   \begin{array}{lll@{\qquad}l}
   \bits_{\K{i}N}(i) &=& \ibits_N(i) \\
   \bits_{\K{f}N}(z) &=& \fbits_N(z) \\
   \end{array}

それぞれの関数は全単射であり、それゆえに逆関数を持ちます。

.. index:: Boolean
.. _aux-ibits:

整数（integer）
........

:ref:`整数 <syntax-int>` は、2を底とする符号なし数字として表現されます。

.. math::
   \begin{array}{lll@{\qquad}l}
   \ibits_N(i) &=& d_{N-1}~\dots~d_0 & (i = 2^{N-1}\cdot d_{N-1} + \dots + 2^0\cdot d_0) \\
   \end{array}

「:math:`\wedge`」「:math:`\vee`」「:math:`\veebar`」といったブーリアン演算子は、それらを点別に適用することで、等しい長さを持つビットシーケンスに持ち上げられます。

.. index:: IEEE 754, significand, exponent
.. _aux-fbias:
.. _aux-fsign:
.. _aux-fbits:

浮動小数点（floating-point）
..............

:ref:`浮動小数点値 <syntax-float>` は、|IEEE754|_ （セクション3.4）で定義されるバイナリ形式によって表現されます。

.. math::
   \begin{array}{lll@{\qquad}l}
   \fbits_N(\pm (1+m\cdot 2^{-M})\cdot 2^e) &=& \fsign({\pm})~\ibits_E(e+\fbias_N)~\ibits_M(m) \\
   \fbits_N(\pm (0+m\cdot 2^{-M})\cdot 2^e) &=& \fsign({\pm})~(0)^E~\ibits_M(m) \\
   \fbits_N(\pm \infty) &=& \fsign({\pm})~(1)^E~(0)^M \\
   \fbits_N(\pm \NAN(n)) &=& \fsign({\pm})~(1)^E~\ibits_M(n) \\[1ex]
   \fbias_N &=& 2^{E-1}-1 \\
   \fsign({+}) &=& 0 \\
   \fsign({-}) &=& 1 \\
   \end{array}

ただし :math:`M = \significand(N)` および :math:`E = \exponent(N)` となります。

.. index:: byte, little endian, memory
.. _aux-littleendian:
.. _aux-bytes:

ストレージ（storage）
.......

ある数字が :ref:`メモリー <syntax-mem>` に保存されると、|LittleEndian|_ バイトオーダーの :ref:`バイト <syntax-byte>` シーケンスに変換されます。

.. math::
   \begin{array}{lll@{\qquad}l}
   \bytes_t(i) &=& \littleendian(\bits_t(i)) \\[1ex]
   \littleendian(\epsilon) &=& \epsilon \\
   \littleendian(d^8~{d'}^\ast~) &=& \littleendian({d'}^\ast)~\ibits_8^{-1}(d^8) \\
   \end{array}

これらの関数も、逆関数を持つ全単射です。

.. index:: integer
.. _int-ops:

整数演算（integer operation）
~~~~~~~~~~~~~~~~~~

.. index:: sign, signed integer, unsigned integer, uninterpreted integer, two's complement
.. _aux-signed:

符号の解釈（sign interpretation）
...................

整数の演算子は |iN| 値に対して定義されます。
符号付きの解釈を用いる演算子は、その値を以下の定義を用いて変換し、その値が値の範囲の上半分に該当する場合は2の補数を取ります（すなわちMSBは :math:`1`）。

.. math::
   \begin{array}{lll@{\qquad}l}
   \signed_N(i) &=& i & (0 \leq i < 2^{N-1}) \\
   \signed_N(i) &=& i - 2^N & (2^{N-1} \leq i < 2^N) \\
   \end{array}

この関数は全単射であり、それゆえに逆関数を持ちます。


.. index:: Boolean
.. _aux-bool:

ブーリアンの解釈（boolean interpretation）
......................

述語の整数値結果（:ref:`テスト <syntax-testop>` 演算子や :ref:`相対 <syntax-relop>` 演算子など）は、条件に応じて値 :math:`1` または :math:`0` を生成する以下の補助関数の助けを借りて定義されます。

.. math::
   \begin{array}{lll@{\qquad}l}
   \bool(C) &=& 1 & (\iff C) \\
   \bool(C) &=& 0 & (\otherwise) \\
   \end{array}

.. _op-iadd:

:math:`\iadd_N(i_1, i_2)`
.........................

* :math:`i_1` と :math:`i_2` を加算した結果を法 :math:`2^N` で返す。

.. math::
   \begin{array}{@{}lcll}
   \iadd_N(i_1, i_2) &=& (i_1 + i_2) \mod 2^N
   \end{array}

.. _op-isub:

:math:`\isub_N(i_1, i_2)`
.........................

* :math:`i_1` から :math:`i_2` を減算した結果を法 :math:`2^N` で返す。

.. math::
   \begin{array}{@{}lcll}
   \isub_N(i_1, i_2) &=& (i_1 - i_2 + 2^N) \mod 2^N
   \end{array}

.. _op-imul:

:math:`\imul_N(i_1, i_2)`
.........................

* :math:`i_1` と :math:`i_2` を乗算した結果を法 :math:`2^N` で返す。

.. math::
   \begin{array}{@{}lcll}
   \imul_N(i_1, i_2) &=& (i_1 \cdot i_2) \mod 2^N
   \end{array}

.. _op-idiv_u:

:math:`\idivu_N(i_1, i_2)`
..........................

* :math:`i_2` が :math:`0` の場合、結果は未定義。

* それ以外の場合、:math:`i_1` を :math:`i_2` で除算した結果をゼロ方向に切り落として返す。

.. math::
   \begin{array}{@{}lcll}
   \idivu_N(i_1, 0) &=& \{\} \\
   \idivu_N(i_1, i_2) &=& \trunc(i_1 / i_2) \\
   \end{array}

.. note::
   この演算子は :ref:`パーシャル <exec-op-partial>` です。

.. _op-idiv_s:

:math:`\idivs_N(i_1, i_2)`
..........................

* :math:`j_1` を :math:`i_1` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` を :math:`i_2` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` が :math:`0` の場合、結果は未定義。

* 上の条件に該当せず、:math:`j_1` を :math:`j_2` で除算した結果が :math:`2^{N-1}` の場合、結果は未定義。

* それ以外の場合、:math:`i_1` を :math:`i_2` で除算した結果をゼロ方向に切り落として返す。

.. math::
   \begin{array}{@{}lcll}
   \idivs_N(i_1, 0) &=& \{\} \\
   \idivs_N(i_1, i_2) &=& \{\} \qquad\qquad (\iff \signed_N(i_1) / \signed_N(i_2) = 2^{N-1}) \\
   \idivs_N(i_1, i_2) &=& \signed_N^{-1}(\trunc(\signed_N(i_1) / \signed_N(i_2))) \\
   \end{array}

.. note::
   この演算子は :ref:`パーシャル <exec-op-partial>` です。
   :math:`0` による除算のほかに、:math:`(-2^{N-1})/(-1) = +2^{N-1}` の結果も :math:`N` ビット符号付き整数で表現できません。


.. _op-irem_u:

:math:`\iremu_N(i_1, i_2)`
..........................

* :math:`i_2` が :math:`0` の場合、結果は未定義。

* それ以外の場合、:math:`i_1` を :math:`i_2` で除算した余りを返す。

.. math::
   \begin{array}{@{}lcll}
   \iremu_N(i_1, 0) &=& \{\} \\
   \iremu_N(i_1, i_2) &=& i_1 - i_2 \cdot \trunc(i_1 / i_2) \\
   \end{array}

.. note::
   この演算子は :ref:`パーシャル <exec-op-partial>` です。

   両方の演算子が定義されている限りにおいて、:math:`i_1 = i_2\cdot\idivu(i_1, i_2) + \iremu(i_1, i_2)` となります。

.. _op-irem_s:

:math:`\irems_N(i_1, i_2)`
..........................

* :math:`j_1` を :math:`i_1` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` を :math:`i_2` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` が :math:`0` の場合、結果は未定義。

* それ以外の場合、:math:`i_1` を :math:`i_2` で除算した余りに、被除数 :math:`j_1` の符号を与えて返す。

.. math::
   \begin{array}{@{}lcll}
   \irems_N(i_1, 0) &=& \{\} \\
   \irems_N(i_1, i_2) &=& \signed_N^{-1}(j_1 - j_2 \cdot \trunc(j_1 / j_2)) \\
     && (\where j_1 = \signed_N(i_1) \wedge j_2 = \signed_N(i_2)) \\
   \end{array}

.. note::
   この演算子は :ref:`パーシャル <exec-op-partial>` です。

   両方の演算子が定義されている限りにおいて、:math:`i_1 = i_2\cdot\idivs(i_1, i_2) + \irems(i_1, i_2)` となります

.. _op-iand:

:math:`\iand_N(i_1, i_2)`
.........................

* :math:`i_1` と :math:`i_2` のビット論理積を返す。

.. math::
   \begin{array}{@{}lcll}
   \iand_N(i_1, i_2) &=& \ibits_N^{-1}(\ibits_N(i_1) \wedge \ibits_N(i_2))
   \end{array}

.. _op-ior:

:math:`\ior_N(i_1, i_2)`
........................

* :math:`i_1` と :math:`i_2` のビット論理和を返す。

.. math::
   \begin{array}{@{}lcll}
   \ior_N(i_1, i_2) &=& \ibits_N^{-1}(\ibits_N(i_1) \vee \ibits_N(i_2))
   \end{array}

.. _op-ixor:

:math:`\ixor_N(i_1, i_2)`
.........................

* :math:`i_1` と :math:`i_2` のビット排他的論理和を返す。

.. math::
   \begin{array}{@{}lcll}
   \ixor_N(i_1, i_2) &=& \ibits_N^{-1}(\ibits_N(i_1) \veebar \ibits_N(i_2))
   \end{array}

.. _op-ishl:

:math:`\ishl_N(i_1, i_2)`
.........................

* :math:`k` を :math:`i_2` （法 :math:`N`） とする。

* :math:`i_1` を左に :math:`k` ビットシフトした結果を返す（法 :math:`2^N`）。

.. math::
   \begin{array}{@{}lcll}
   \ishl_N(i_1, i_2) &=& \ibits_N^{-1}(d_2^{N-k}~0^k)
     & (\iff \ibits_N(i_1) = d_1^k~d_2^{N-k} \wedge k = i_2 \mod N)
   \end{array}

.. _op-ishr_u:

:math:`\ishru_N(i_1, i_2)`
..........................

* :math:`k` を :math:`i_2` （法 :math:`N`） とする。

* :math:`i_1` を右に :math:`k` ビットシフトした結果を返す（:math:`0` で拡張）。

.. math::
   \begin{array}{@{}lcll}
   \ishru_N(i_1, i_2) &=& \ibits_N^{-1}(0^k~d_1^{N-k})
     & (\iff \ibits_N(i_1) = d_1^{N-k}~d_2^k \wedge k = i_2 \mod N)
   \end{array}

.. _op-ishr_s:

:math:`\ishrs_N(i_1, i_2)`
..........................

* :math:`k` を :math:`i_2` （法 :math:`N`） とする。

* :math:`i_1` を右に :math:`k` ビットシフトした結果を返す（元の値のMSBで拡張）

.. math::
   \begin{array}{@{}lcll}
   \ishrs_N(i_1, i_2) &=& \ibits_N^{-1}(d_0^{k+1}~d_1^{N-k-1})
     & (\iff \ibits_N(i_1) = d_0~d_1^{N-k-1}~d_2^k \wedge k = i_2 \mod N)
   \end{array}

.. _op-irotl:

:math:`\irotl_N(i_1, i_2)`
..........................

* :math:`k` を :math:`i_2` （法 :math:`N`） とする。

* :math:`i_1` を左に :math:`k` ビットローテートした結果を返す。

.. math::
   \begin{array}{@{}lcll}
   \irotl_N(i_1, i_2) &=& \ibits_N^{-1}(d_2^{N-k}~d_1^k)
     & (\iff \ibits_N(i_1) = d_1^k~d_2^{N-k} \wedge k = i_2 \mod N)
   \end{array}

.. _op-irotr:

:math:`\irotr_N(i_1, i_2)`
..........................

* :math:`k` を :math:`i_2` （法 :math:`N`） とする。

* :math:`i_1` を右に :math:`k` ビットローテートした結果を返す。

.. math::
   \begin{array}{@{}lcll}
   \irotr_N(i_1, i_2) &=& \ibits_N^{-1}(d_2^k~d_1^{N-k})
     & (\iff \ibits_N(i_1) = d_1^{N-k}~d_2^k \wedge k = i_2 \mod N)
   \end{array}


.. _op-iclz:

:math:`\iclz_N(i)`
..................

* :math:`i` の上位ゼロビットの個数を返す。:math:`i` が :math:`0` の場合は、すべてビットが上位ゼロビットとみなされる。

.. math::
   \begin{array}{@{}lcll}
   \iclz_N(i) &=& k & (\iff \ibits_N(i) = 0^k~(1~d^\ast)^?)
   \end{array}


.. _op-ictz:

:math:`\ictz_N(i)`
..................

* :math:`i` の下位ゼロビットの個数を返す。:math:`i` が :math:`0` の場合は、すべてビットが下位ゼロビットとみなされる。

.. math::
   \begin{array}{@{}lcll}
   \ictz_N(i) &=& k & (\iff \ibits_N(i) = (d^\ast~1)^?~0^k)
   \end{array}


.. _op-ipopcnt:

:math:`\ipopcnt_N(i)`
.....................

* :math:`i` のうち非ゼロビットの個数を返す。

.. math::
   \begin{array}{@{}lcll}
   \ipopcnt_N(i) &=& k & (\iff \ibits_N(i) = (0^\ast~1)^k~0^\ast)
   \end{array}


.. _op-ieqz:

:math:`\ieqz_N(i)`
..................

* :math:`i` がゼロの場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \ieqz_N(i) &=& \bool(i = 0)
   \end{array}


.. _op-ieq:

:math:`\ieq_N(i_1, i_2)`
........................

* :math:`i_1` が :math:`i_2` と等しい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \ieq_N(i_1, i_2) &=& \bool(i_1 = i_2)
   \end{array}


.. _op-ine:

:math:`\ine_N(i_1, i_2)`
........................

* :math:`i_1` が :math:`i_2` と等しくない場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \ine_N(i_1, i_2) &=& \bool(i_1 \neq i_2)
   \end{array}


.. _op-ilt_u:

:math:`\iltu_N(i_1, i_2)`
.........................

* :math:`i_1` が :math:`i_2` より小さい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \iltu_N(i_1, i_2) &=& \bool(i_1 < i_2)
   \end{array}


.. _op-ilt_s:

:math:`\ilts_N(i_1, i_2)`
.........................

* :math:`j_1` を :math:`i_1` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` を :math:`i_2` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`i_1` が :math:`i_2` より小さい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \ilts_N(i_1, i_2) &=& \bool(\signed_N(i_1) < \signed_N(i_2))
   \end{array}


.. _op-igt_u:

:math:`\igtu_N(i_1, i_2)`
.........................

* :math:`i_1` が :math:`i_2` より大きい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \igtu_N(i_1, i_2) &=& \bool(i_1 > i_2)
   \end{array}


.. _op-igt_s:

:math:`\igts_N(i_1, i_2)`
.........................

* :math:`j_1` を :math:`i_1` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` を :math:`i_2` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`i_1` が :math:`i_2` より大きい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \igts_N(i_1, i_2) &=& \bool(\signed_N(i_1) > \signed_N(i_2))
   \end{array}


.. _op-ile_u:

:math:`\ileu_N(i_1, i_2)`
.........................

* :math:`i_1` が :math:`i_2` より小さいか等しい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \ileu_N(i_1, i_2) &=& \bool(i_1 \leq i_2)
   \end{array}


.. _op-ile_s:

:math:`\iles_N(i_1, i_2)`
.........................

* :math:`j_1` を :math:`i_1` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` を :math:`i_2` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`i_1` が :math:`i_2` より小さいか等しい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \iles_N(i_1, i_2) &=& \bool(\signed_N(i_1) \leq \signed_N(i_2))
   \end{array}


.. _op-ige_u:

:math:`\igeu_N(i_1, i_2)`
.........................

* :math:`i_1` が :math:`i_2` より大きいか等しい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \igeu_N(i_1, i_2) &=& \bool(i_1 \geq i_2)
   \end{array}


.. _op-ige_s:

:math:`\iges_N(i_1, i_2)`
.........................

* :math:`j_1` を :math:`i_1` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`j_2` を :math:`i_2` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`i_1` が :math:`i_2` より大きいか等しい場合は :math:`1` を返し、それ以外の場合は :math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \iges_N(i_1, i_2) &=& \bool(\signed_N(i_1) \geq \signed_N(i_2))
   \end{array}


.. _op-iextendn_s:

:math:`\iextendMs_N(i)`
.......................

* :math:`\extends_{M,N}(i)` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \iextendMs_{N}(i) &=& \extends_{M,N}(i) \\
   \end{array}


.. index:: floating-point, IEEE 754
.. _float-ops:

浮動小数点演算（floating-point operation）
~~~~~~~~~~~~~~~~~~

浮動小数演算は |IEEE754|_ 標準に従い、以下に準拠します。

* すべての演算子では、特に指定のない限り「最近点への丸め（round-to-nearest）」「偶数への丸め（round-to-even）」を用いる。
  デフォルトでない「directed rounding」属性はサポートされない。

* :ref:`NaN <syntax-nan>` ペイロードをオペランドから伝搬することの推奨に従うことは許されるが、必須ではない。

* すべての演算子は「non-stop」モードを用いる。浮動小数点の例外はそれ以外の場合観測可能にならない。
特に「alternate floating-point」例外ハンドリング属性と「ステータスフラグでの演算子」はいずれもサポートされない。
  シグナリングしない（quiet）NaNとシグナリングNaNの間には観測可能な差異は存在しない。

.. note::
   これらの制約のいくつかはWebAssemblyの今後のバージョンで解消される可能性があります。

.. index:: rounding
.. _aux-ieee:

丸め（rounding）
........

丸め（端数処理）は、常に |IEEE754|_ （セクション4.3.1）に対応する「最近点への偶数丸め（round-to-nearest ties-to-even: RNE）」となります。

ある浮動小数点の「厳密数（exact number）」は、与えられるビット幅 :math:`N` の :ref:`浮動小数点の数値 <syntax-float>` に正確に対応する有理数です。

与えられる浮動小数点のビット幅 :math:`N` における「限界数（limit number）」は、その大きさが :math:`2` の最小乗数で、幅 :math:`N` の浮動小数点数として正確には表現できない、正または負の数値です（大きさは :math:`N = 32` では :math:`2^{128}`、:math:`N = 64` では :math:`2^{1024}` ）。

ある「候補数（candidate number）」は、与えられるビット幅 :math:`N` における「厳密な浮動小数点数」または「正または負の限界数」のいずれか一方となります。


ある「候補ペア（candidate pair）」は、2つの間に候補数が存在しない、候補数のペア :math:`z_1,z_2` です。

ある「実数（real number）」は、以下のようにビット幅 :math:`N` の浮動小数点値に変換されます。

* :math:`r` が :math:`0` の場合、:math:`+0` を返す。

* 上に該当せず、:math:`r` が厳密な浮動小数点数の場合は、:math:`r` を返す。

* 上に該当せず、:math:`r` が正の上限より大きいか正の限界と等しい場合は、:math:`+\infty` を返す。

* 上に該当せず、:math:`r` が正の上限より小さいか負の限界と等しい場合は、:math:`-\infty` を返す。

* 上に該当せず、:math:`z_1` および :math:`z_2` が :math:`z_1 < r < z_2` を満たすひとつの候補ペアである場合は以下のようになる。

  * :math:`|r - z_1| < |r - z_2|` の場合は、:math:`z` be :math:`z_1` とする。

  * 上に該当せず、:math:`|r - z_1| > |r - z_2|` の場合は、 :math:`z` を :math:`z_2` とする。

  * 上に該当せず、:math:`|r - z_1| = |r - z_2|` かつ :math:`z_1` の :ref:`仮数部 <syntax-float>` が偶数の場合は、 :math:`z` を :math:`z_1` とする。

  * 上に該当しない場合は、:math:`z` を :math:`z_2` とする。

* :math:`z` が :math:`0` の場合は以下のようになる。

  * :math:`r < 0` の場合は :math:`-0` を返す。

  * 上に該当しない場合は :math:`+0` を返す。

* 上に該当せず、:math:`z` が限界数の場合は以下のようになる。

  * :math:`r < 0` の場合は :math:`-\infty` を返す。

  * 上に該当しない場合は :math:`+\infty` を返す。

* 上に該当しない場合は :math:`z` を返す。


.. math::
   \begin{array}{lll@{\qquad}l}
   \ieee_N(0) &=& +0 \\
   \ieee_N(r) &=& r & (\iff r \in \F{exact}_N) \\
   \ieee_N(r) &=& +\infty & (\iff r \geq +\F{limit}_N) \\
   \ieee_N(r) &=& -\infty & (\iff r \leq -\F{limit}_N) \\
   \ieee_N(r) &=& \F{closest}_N(r, z_1, z_2) & (\iff z_1 < r < z_2 \wedge (z_1,z_2) \in \F{candidatepair}_N) \\[1ex]
   \F{closest}_N(r, z_1, z_2) &=& \F{rectify}_N(r, z_1) & (\iff |r-z_1|<|r-z_2|) \\
   \F{closest}_N(r, z_1, z_2) &=& \F{rectify}_N(r, z_2) & (\iff |r-z_1|>|r-z_2|) \\
   \F{closest}_N(r, z_1, z_2) &=& \F{rectify}_N(r, z_1) & (\iff |r-z_1|=|r-z_2| \wedge \F{even}_N(z_1)) \\
   \F{closest}_N(r, z_1, z_2) &=& \F{rectify}_N(r, z_2) & (\iff |r-z_1|=|r-z_2| \wedge \F{even}_N(z_2)) \\[1ex]
   \F{rectify}_N(r, \pm \F{limit}_N) &=& \pm \infty \\
   \F{rectify}_N(r, 0) &=& +0 \qquad (r \geq 0) \\
   \F{rectify}_N(r, 0) &=& -0 \qquad (r < 0) \\
   \F{rectify}_N(r, z) &=& z \\
   \end{array}

ただし:

.. math::
   \begin{array}{lll@{\qquad}l}
   \F{exact}_N &=& \fN \cap \mathbb{Q} \\
   \F{limit}_N &=& 2^{2^{\exponent(N)-1}} \\
   \F{candidate}_N &=& \F{exact}_N \cup \{+\F{limit}_N, -\F{limit}_N\} \\
   \F{candidatepair}_N &=& \{ (z_1, z_2) \in \F{candidate}_N^2 ~|~ z_1 < z_2 \wedge \forall z \in \F{candidate}_N, z \leq z_1 \vee z \geq z_2\} \\[1ex]
   \F{even}_N((d + m\cdot 2^{-M}) \cdot 2^e) &\Leftrightarrow& m \mod 2 = 0 \\
   \F{even}_N(\pm \F{limit}_N) &\Leftrightarrow& \F{true} \\
   \end{array}


.. index:: NaN
.. _aux-nans:

NaNの伝搬（NaN propagation）
...............

浮動小数点演算子のひとつの結果が |fneg|、|fabs|、|fcopysign| 以外になる場合は :ref:`NaN <syntax-nan>` となり、その符号は非決定的となります。この ref:`ペイロード <syntax-payload>` は以下のように算出されます。

* その演算子に対するあらゆる NaN 入力のペイロードが :ref:`カノニカル <canonical-nan>` の場合（NaN入力が存在しないケースも含む）、その出力のペイロードもカノニカルとなる。

* それ以外の場合、ペイロードはあらゆる :ref:`算術的NaN <arithmetic-nan>` （MSBが :math:`1` でその他は無指定）の中から非決定的に選ばれる。

この非決定的な結果は、入力の集合から許可された出力の集合を生成する以下の補助関数で表現されます。

.. math::
   \begin{array}{lll@{\qquad}l}
   \nans_N\{z^\ast\} &=& \{ + \NAN(n), - \NAN(n) ~|~ n = \canon_N \}
     & (\iff \forall \NAN(n) \in z^\ast,~ n = \canon_N) \\
   \nans_N\{z^\ast\} &=& \{ + \NAN(n), - \NAN(n) ~|~ n \geq \canon_N \}
     & (\otherwise) \\
   \end{array}


.. _op-fadd:

:math:`\fadd_N(z_1, z_2)`
.........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合、:math:`\nans_N\{z_1, z_2\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号が逆の無限の場合は、:math:`\nans_N\{\}` をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号が等しい無限の場合は、その無限を返す。

* 上に該当しない場合、:math:`z_1` または :math:`z_2` が無限の場合は、その無限を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号が逆のゼロの場合は、その正のゼロを返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` がどちらも符号の等しいゼロの場合は、そのゼロを返す。

* 上に該当しない場合、:math:`z_1` または :math:`z_2` の一方がゼロの場合は、他方のオペランドを返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の「大きさ」が等しいが符号が互いに逆の場合は、正のゼロを返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` を加算した結果を、それを表現できる最も近い値に :ref:`丸めて <aux-ieee>` 返す。

.. math::
   \begin{array}{@{}lcll}
   \fadd_N(\pm \NAN(n), z_2) &=& \nans_N\{\pm \NAN(n), z_2\} \\
   \fadd_N(z_1, \pm \NAN(n)) &=& \nans_N\{\pm \NAN(n), z_1\} \\
   \fadd_N(\pm \infty, \mp \infty) &=& \nans_N\{\} \\
   \fadd_N(\pm \infty, \pm \infty) &=& \pm \infty \\
   \fadd_N(z_1, \pm \infty) &=& \pm \infty \\
   \fadd_N(\pm \infty, z_2) &=& \pm \infty \\
   \fadd_N(\pm 0, \mp 0) &=& +0 \\
   \fadd_N(\pm 0, \pm 0) &=& \pm 0 \\
   \fadd_N(z_1, \pm 0) &=& z_1 \\
   \fadd_N(\pm 0, z_2) &=& z_2 \\
   \fadd_N(\pm q, \mp q) &=& +0 \\
   \fadd_N(z_1, z_2) &=& \ieee_N(z_1 + z_2) \\
   \end{array}


.. _op-fsub:

:math:`\fsub_N(z_1, z_2)`
.........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合、:math:`\nans_N\{z_1, z_2\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` がどちらも符号の等しい無限の場合は、:math:`\nans_N\{\}` をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号が逆の無限の場合は、:math:`z_1` を返す。

* 上に該当しない場合、:math:`z_1` が無限の場合は、その無限を返す。

* 上に該当しない場合、:math:`z_2` が無限の場合は、符号を反転した無限を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` がどちらも符号の等しいゼロの場合は、正のゼロを返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号が等しいゼロの場合は、:math:`z_1` を返す。

* 上に該当しない場合、:math:`z_2` がゼロの場合は、:math:`z_1` を返す。

* 上に該当しない場合、:math:`z_1` がゼロの場合は、符号を反転した :math:`z_2` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値が等しい場合は、正のゼロを返す。

* 上に該当しない場合、:math:`z_1` から :math:`z_2` を減算した結果を、それを表現できる最も近い値に :ref:`丸めて <aux-ieee>` 返す。

.. math::
   \begin{array}{@{}lcll}
   \fsub_N(\pm \NAN(n), z_2) &=& \nans_N\{\pm \NAN(n), z_2\} \\
   \fsub_N(z_1, \pm \NAN(n)) &=& \nans_N\{\pm \NAN(n), z_1\} \\
   \fsub_N(\pm \infty, \pm \infty) &=& \nans_N\{\} \\
   \fsub_N(\pm \infty, \mp \infty) &=& \pm \infty \\
   \fsub_N(z_1, \pm \infty) &=& \mp \infty \\
   \fsub_N(\pm \infty, z_2) &=& \pm \infty \\
   \fsub_N(\pm 0, \pm 0) &=& +0 \\
   \fsub_N(\pm 0, \mp 0) &=& \pm 0 \\
   \fsub_N(z_1, \pm 0) &=& z_1 \\
   \fsub_N(\pm 0, \pm q_2) &=& \mp q_2 \\
   \fsub_N(\pm q, \pm q) &=& +0 \\
   \fsub_N(z_1, z_2) &=& \ieee_N(z_1 - z_2) \\
   \end{array}

.. note::
   NaNの非決定性のため、:math:`\fsub_N(z_1, z_2) = \fadd_N(z_1, \fneg_N(z_2))` は常に維持されます。

.. _op-fmul:

:math:`\fmul_N(z_1, z_2)`
.........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合、:math:`\nans_N\{z_1, z_2\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の一方がゼロで他方が無限の場合は、:math:`\nans_N\{\}` をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号が等しい無限の場合は、正の無限を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号が逆の無限の場合は、負の無限を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の一方が無限で他方がそれと同じ符号を持つ値の場合は、正の無限を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の一方が無限で他方がそれと逆の符号を持つ値の場合は、負の無限を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` がどちらも互いに符号が等しいゼロの場合、正のゼロを返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` がどちらも互いに符号が逆のゼロの場合、負のゼロを返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` を乗算した結果を、それを表現できる最も近い値に :ref:`丸めて <aux-ieee>` 返す。

.. math::
   \begin{array}{@{}lcll}
   \fmul_N(\pm \NAN(n), z_2) &=& \nans_N\{\pm \NAN(n), z_2\} \\
   \fmul_N(z_1, \pm \NAN(n)) &=& \nans_N\{\pm \NAN(n), z_1\} \\
   \fmul_N(\pm \infty, \pm 0) &=& \nans_N\{\} \\
   \fmul_N(\pm \infty, \mp 0) &=& \nans_N\{\} \\
   \fmul_N(\pm 0, \pm \infty) &=& \nans_N\{\} \\
   \fmul_N(\pm 0, \mp \infty) &=& \nans_N\{\} \\
   \fmul_N(\pm \infty, \pm \infty) &=& +\infty \\
   \fmul_N(\pm \infty, \mp \infty) &=& -\infty \\
   \fmul_N(\pm q_1, \pm \infty) &=& +\infty \\
   \fmul_N(\pm q_1, \mp \infty) &=& -\infty \\
   \fmul_N(\pm \infty, \pm q_2) &=& +\infty \\
   \fmul_N(\pm \infty, \mp q_2) &=& -\infty \\
   \fmul_N(\pm 0, \pm 0) &=& + 0 \\
   \fmul_N(\pm 0, \mp 0) &=& - 0 \\
   \fmul_N(z_1, z_2) &=& \ieee_N(z_1 \cdot z_2) \\
   \end{array}


.. _op-fdiv:

:math:`\fdiv_N(z_1, z_2)`
.........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合、:math:`\nans_N\{z_1, z_2\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` がどちらも無限の場合は、:math:`\nans_N\{\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` がどちらもゼロの場合は、:math:`\nans_N\{z_1, z_2\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` が無限で :math:`z_2` がそれと符号の等しい値の場合は、正の無限を返す。

* 上に該当しない場合、:math:`z_1` が無限で :math:`z_2` がそれと符号が逆の値の場合は、正の無限を返す。

* 上に該当しない場合、:math:`z_2` が無限で :math:`z_1` がそれと符号の等しい値の場合は、正のゼロを返す。

* 上に該当しない場合、:math:`z_2` が無限で :math:`z_1` がそれと符号が逆の値の場合は、負のゼロを返す。

* 上に該当しない場合、:math:`z_1` がゼロで :math:`z_2` がそれと符号が等しい値の場合は、正のゼロを返す。

* 上に該当しない場合、:math:`z_1` がゼロで :math:`z_2` がそれと符号が逆の値の場合は、負のゼロを返す。

* 上に該当しない場合、:math:`z_2` がゼロで :math:`z_1` がそれと符号が等しい値の場合は、正の無限を返す。

* 上に該当しない場合、:math:`z_2` がゼロで :math:`z_1` がそれと符号が逆の値の場合は、負の無限を返す。

* 上に該当しない場合、:math:`z_1` を :math:`z_2` で除算した結果を、それを表現できる最も近い値に :ref:`丸めて <aux-ieee>` 返す。

.. math::
   \begin{array}{@{}lcll}
   \fdiv_N(\pm \NAN(n), z_2) &=& \nans_N\{\pm \NAN(n), z_2\} \\
   \fdiv_N(z_1, \pm \NAN(n)) &=& \nans_N\{\pm \NAN(n), z_1\} \\
   \fdiv_N(\pm \infty, \pm \infty) &=& \nans_N\{\} \\
   \fdiv_N(\pm \infty, \mp \infty) &=& \nans_N\{\} \\
   \fdiv_N(\pm 0, \pm 0) &=& \nans_N\{\} \\
   \fdiv_N(\pm 0, \mp 0) &=& \nans_N\{\} \\
   \fdiv_N(\pm \infty, \pm q_2) &=& +\infty \\
   \fdiv_N(\pm \infty, \mp q_2) &=& -\infty \\
   \fdiv_N(\pm q_1, \pm \infty) &=& +0 \\
   \fdiv_N(\pm q_1, \mp \infty) &=& -0 \\
   \fdiv_N(\pm 0, \pm q_2) &=& +0 \\
   \fdiv_N(\pm 0, \mp q_2) &=& -0 \\
   \fdiv_N(\pm q_1, \pm 0) &=& +\infty \\
   \fdiv_N(\pm q_1, \mp 0) &=& -\infty \\
   \fdiv_N(z_1, z_2) &=& \ieee_N(z_1 / z_2) \\
   \end{array}


.. _op-fmin:

:math:`\fmin_N(z_1, z_2)`
.........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合、:math:`\nans_N\{z_1, z_2\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` または :math:`z_2` のいずれかが負の無限の場合、負の無限を返す。

* 上に該当しない場合、:math:`z_1` または :math:`z_2` のいずれかが正の無限の場合、他方の値を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号の異なるゼロの場合、負のゼロを返す。

* 上に該当しない場合、:math:`z_1` と :math:`z_2` のうち小さい方の値を返す。

.. math::
   \begin{array}{@{}lcll}
   \fmin_N(\pm \NAN(n), z_2) &=& \nans_N\{\pm \NAN(n), z_2\} \\
   \fmin_N(z_1, \pm \NAN(n)) &=& \nans_N\{\pm \NAN(n), z_1\} \\
   \fmin_N(+ \infty, z_2) &=& z_2 \\
   \fmin_N(- \infty, z_2) &=& - \infty \\
   \fmin_N(z_1, + \infty) &=& z_1 \\
   \fmin_N(z_1, - \infty) &=& - \infty \\
   \fmin_N(\pm 0, \mp 0) &=& -0 \\
   \fmin_N(z_1, z_2) &=& z_1 & (\iff z_1 \leq z_2) \\
   \fmin_N(z_1, z_2) &=& z_2 & (\iff z_2 \leq z_1) \\
   \end{array}


.. _op-fmax:

:math:`\fmax_N(z_1, z_2)`
.........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合、:math:`\nans_N\{z_1, z_2\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z_1` または :math:`z_2` のいずれかが正の無限の場合、正の無限を返す。

* 上に該当しない場合、:math:`z_1` または :math:`z_2` のいずれかが負の無限の場合、他方の値を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が互いに符号の異なるゼロの場合、負のゼロを返す。

* 上に該当しない場合、:math:`z_1` と :math:`z_2` のうち大きい方の値を返す。

.. math::
   \begin{array}{@{}lcll}
   \fmax_N(\pm \NAN(n), z_2) &=& \nans_N\{\pm \NAN(n), z_2\} \\
   \fmax_N(z_1, \pm \NAN(n)) &=& \nans_N\{\pm \NAN(n), z_1\} \\
   \fmax_N(+ \infty, z_2) &=& + \infty \\
   \fmax_N(- \infty, z_2) &=& z_2 \\
   \fmax_N(z_1, + \infty) &=& + \infty \\
   \fmax_N(z_1, - \infty) &=& z_1 \\
   \fmax_N(\pm 0, \mp 0) &=& +0 \\
   \fmax_N(z_1, z_2) &=& z_1 & (\iff z_1 \geq z_2) \\
   \fmax_N(z_1, z_2) &=& z_2 & (\iff z_2 \geq z_1) \\
   \end{array}


.. _op-fcopysign:

:math:`\fcopysign_N(z_1, z_2)`
..............................

* :math:`z_1` および :math:`z_2` の符号が同じ場合、:math:`z_1` を返す。

* 上に該当しない場合、:math:`z_1` の符号を反転して返す。

.. math::
   \begin{array}{@{}lcll}
   \fcopysign_N(\pm p_1, \pm p_2) &=& \pm p_1 \\
   \fcopysign_N(\pm p_1, \mp p_2) &=& \mp p_1 \\
   \end{array}


.. _op-fabs:

:math:`\fabs_N(z)`
..................

* :math:`z_1` がNaNの場合、正の :math:`z` を返す。

* 上に該当しない場合、:math:`z` が無限の場合は正の無限を返す。

* 上に該当しない場合、:math:`z` がゼロの場合は正のゼロを返す。

* 上に該当しない場合、:math:`z` が正の値の場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` の符号を反転して返す。

.. math::
   \begin{array}{@{}lcll}
   \fabs_N(\pm \NAN(n)) &=& +\NAN(n) \\
   \fabs_N(\pm \infty) &=& +\infty \\
   \fabs_N(\pm 0) &=& +0 \\
   \fabs_N(\pm q) &=& +q \\
   \end{array}


.. _op-fneg:

:math:`\fneg_N(z)`
..................

* :math:`z_1` がNaNの場合、:math:`z` の符号を反転して返す。

* 上に該当しない場合、:math:`z` が無限の場合は符号を反転した無限を返す。

* 上に該当しない場合、:math:`z` がゼロの場合は符号を反転したゼロを返す。

* 上に該当しない場合、:math:`z` の符号を反転して返す。

.. math::
   \begin{array}{@{}lcll}
   \fneg_N(\pm \NAN(n)) &=& \mp \NAN(n) \\
   \fneg_N(\pm \infty) &=& \mp \infty \\
   \fneg_N(\pm 0) &=& \mp 0 \\
   \fneg_N(\pm q) &=& \mp q \\
   \end{array}


.. _op-fsqrt:

:math:`\fsqrt_N(z)`
...................

* :math:`z` がNaNの場合、:math:`\nans_N\{z\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z` が負の無限の場合は :math:`\nans_N\{\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z` が正の無限の場合、正の無限を返す。

* 上に該当しない場合、:math:`z` がゼロの場合はそのゼロを返す。

* 上に該当しない場合、:math:`z` の符号が負の場合は :math:`\nans_N\{\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z` の平方根を返す。

.. math::
   \begin{array}{@{}lcll}
   \fsqrt_N(\pm \NAN(n)) &=& \nans_N\{\pm \NAN(n)\} \\
   \fsqrt_N(- \infty) &=& \nans_N\{\} \\
   \fsqrt_N(+ \infty) &=& + \infty \\
   \fsqrt_N(\pm 0) &=& \pm 0 \\
   \fsqrt_N(- q) &=& \nans_N\{\} \\
   \fsqrt_N(+ q) &=& \ieee_N\left(\sqrt{q}\right) \\
   \end{array}

.. _op-fceil:

:math:`\fceil_N(z)`
...................

* :math:`z` がNaNの場合、:math:`\nans_N\{z\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z` が無限の場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` がゼロの場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` が :math:`0` より小さいが :math:`-1` より大きい場合は負のゼロを返す。

* 上に該当しない場合、:math:`z` より小さくない最小の積分値を返す。

.. math::
   \begin{array}{@{}lcll}
   \fceil_N(\pm \NAN(n)) &=& \nans_N\{\pm \NAN(n)\} \\
   \fceil_N(\pm \infty) &=& \pm \infty \\
   \fceil_N(\pm 0) &=& \pm 0 \\
   \fceil_N(- q) &=& -0 & (\iff -1 < -q < 0) \\
   \fceil_N(\pm q) &=& \ieee_N(i) & (\iff \pm q \leq i < \pm q + 1) \\
   \end{array}

.. _op-ffloor:

:math:`\ffloor_N(z)`
....................

* :math:`z` がNaNの場合、:math:`\nans_N\{z\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z` が無限の場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` がゼロの場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` が :math:`0` より大きいが :math:`1` より小さい場合は正のゼロを返す。

* 上に該当しない場合、:math:`z` より大きくない最大の積分値を返す。

.. math::
   \begin{array}{@{}lcll}
   \ffloor_N(\pm \NAN(n)) &=& \nans_N\{\pm \NAN(n)\} \\
   \ffloor_N(\pm \infty) &=& \pm \infty \\
   \ffloor_N(\pm 0) &=& \pm 0 \\
   \ffloor_N(+ q) &=& +0 & (\iff 0 < +q < 1) \\
   \ffloor_N(\pm q) &=& \ieee_N(i) & (\iff \pm q - 1 < i \leq \pm q) \\
   \end{array}


.. _op-ftrunc:

:math:`\ftrunc_N(z)`
....................

* :math:`z` がNaNの場合、:math:`\nans_N\{z\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z` が無限の場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` がゼロの場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` が :math:`0` より大きいが :math:`1` より小さい場合は正のゼロを返す。

* 上に該当しない場合、:math:`z` が :math:`0` より小さいが :math:`-1` より大きい場合は負のゼロを返す。

* 上に該当しない場合、:math:`z` と符号が同じで、:math:`z` の「大きさ」より大きくない最大の積分値を返す。

.. math::
   \begin{array}{@{}lcll}
   \ftrunc_N(\pm \NAN(n)) &=& \nans_N\{\pm \NAN(n)\} \\
   \ftrunc_N(\pm \infty) &=& \pm \infty \\
   \ftrunc_N(\pm 0) &=& \pm 0 \\
   \ftrunc_N(+ q) &=& +0 & (\iff 0 < +q < 1) \\
   \ftrunc_N(- q) &=& -0 & (\iff -1 < -q < 0) \\
   \ftrunc_N(\pm q) &=& \ieee_N(\pm i) & (\iff +q - 1 < i \leq +q) \\
   \end{array}


.. _op-fnearest:

:math:`\fnearest_N(z)`
......................

* :math:`z` がNaNの場合、:math:`\nans_N\{z\}` の要素をひとつ返す。

* 上に該当しない場合、:math:`z` が無限の場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` がゼロの場合は :math:`z` を返す。

* 上に該当しない場合、:math:`z` が :math:`0` より大きいが :math:`0.5` より小さいか等しい場合は正のゼロを返す。

* 上に該当しない場合、:math:`z` が :math:`0` より大きいが :math:`-0.5` より大きいか等しい場合は負のゼロを返す。

* 上に該当しない場合、:math:`z` に最も近い積分値を返す。2つの値がほぼ等しい場合は偶数の積分値を返す。

.. math::
   \begin{array}{@{}lcll}
   \fnearest_N(\pm \NAN(n)) &=& \nans_N\{\pm \NAN(n)\} \\
   \fnearest_N(\pm \infty) &=& \pm \infty \\
   \fnearest_N(\pm 0) &=& \pm 0 \\
   \fnearest_N(+ q) &=& +0 & (\iff 0 < +q \leq 0.5) \\
   \fnearest_N(- q) &=& -0 & (\iff -0.5 \leq -q < 0) \\
   \fnearest_N(\pm q) &=& \ieee_N(\pm i) & (\iff |i - q| < 0.5) \\
   \fnearest_N(\pm q) &=& \ieee_N(\pm i) & (\iff |i - q| = 0.5 \wedge i~\mbox{even}) \\
   \end{array}


.. _op-feq:

:math:`\feq_N(z_1, z_2)`
........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が両方ともゼロの場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値が同じ場合は :math:`1` を返す。

* 上に該当しない場合、:math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \feq_N(\pm \NAN(n), z_2) &=& 0 \\
   \feq_N(z_1, \pm \NAN(n)) &=& 0 \\
   \feq_N(\pm 0, \mp 0) &=& 1 \\
   \feq_N(z_1, z_2) &=& \bool(z_1 = z_2) \\
   \end{array}

.. _op-fne:

:math:`\fne_N(z_1, z_2)`
........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` が両方ともゼロの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値が同じ場合は :math:`0` を返す。

* 上に該当しない場合、:math:`1` を返す。

.. math::
   \begin{array}{@{}lcll}
   \fne_N(\pm \NAN(n), z_2) &=& 1 \\
   \fne_N(z_1, \pm \NAN(n)) &=& 1 \\
   \fne_N(\pm 0, \mp 0) &=& 0 \\
   \fne_N(z_1, z_2) &=& \bool(z_1 \neq z_2) \\
   \end{array}

.. _op-flt:

:math:`\flt_N(z_1, z_2)`
........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値が同じ場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` が正の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` が負の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_2` が正の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_2` が負の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値がどちらもゼロの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` が :math:`z_2` より小さい場合は :math:`1` を返す。

* 上に該当しない場合、:math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \flt_N(\pm \NAN(n), z_2) &=& 0 \\
   \flt_N(z_1, \pm \NAN(n)) &=& 0 \\
   \flt_N(z, z) &=& 0 \\
   \flt_N(+ \infty, z_2) &=& 0 \\
   \flt_N(- \infty, z_2) &=& 1 \\
   \flt_N(z_1, + \infty) &=& 1 \\
   \flt_N(z_1, - \infty) &=& 0 \\
   \flt_N(\pm 0, \mp 0) &=& 0 \\
   \flt_N(z_1, z_2) &=& \bool(z_1 < z_2) \\
   \end{array}


.. _op-fgt:

:math:`\fgt_N(z_1, z_2)`
........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値が同じ場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` が正の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` が負の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_2` が正の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_2` が負の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値がどちらもゼロの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` が :math:`z_2` より大きい場合は :math:`1` を返す。

* 上に該当しない場合、:math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \fgt_N(\pm \NAN(n), z_2) &=& 0 \\
   \fgt_N(z_1, \pm \NAN(n)) &=& 0 \\
   \fgt_N(z, z) &=& 0 \\
   \fgt_N(+ \infty, z_2) &=& 1 \\
   \fgt_N(- \infty, z_2) &=& 0 \\
   \fgt_N(z_1, + \infty) &=& 0 \\
   \fgt_N(z_1, - \infty) &=& 1 \\
   \fgt_N(\pm 0, \mp 0) &=& 0 \\
   \fgt_N(z_1, z_2) &=& \bool(z_1 > z_2) \\
   \end{array}


.. _op-fle:

:math:`\fle_N(z_1, z_2)`
........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値が同じ場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` が正の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` が負の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_2` が正の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_2` が負の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値がどちらもゼロの場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` が :math:`z_2` より小さいか等しい場合は :math:`1` を返す。

* 上に該当しない場合、:math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \fle_N(\pm \NAN(n), z_2) &=& 0 \\
   \fle_N(z_1, \pm \NAN(n)) &=& 0 \\
   \fle_N(z, z) &=& 1 \\
   \fle_N(+ \infty, z_2) &=& 0 \\
   \fle_N(- \infty, z_2) &=& 1 \\
   \fle_N(z_1, + \infty) &=& 1 \\
   \fle_N(z_1, - \infty) &=& 0 \\
   \fle_N(\pm 0, \mp 0) &=& 1 \\
   \fle_N(z_1, z_2) &=& \bool(z_1 \leq z_2) \\
   \end{array}


.. _op-fge:

:math:`\fge_N(z_1, z_2)`
........................

* :math:`z_1` または :math:`z_2` の一方がNaNの場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値が同じ場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` が正の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` が負の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_2` が正の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z_2` が負の無限の場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` および :math:`z_2` の値がどちらもゼロの場合は :math:`1` を返す。

* 上に該当しない場合、:math:`z_1` が :math:`z_2` より大きいか等しい場合は :math:`1` を返す。

* 上に該当しない場合、:math:`0` を返す。

.. math::
   \begin{array}{@{}lcll}
   \fge_N(\pm \NAN(n), z_2) &=& 0 \\
   \fge_N(z_1, \pm \NAN(n)) &=& 0 \\
   \fge_N(z, z) &=& 1 \\
   \fge_N(+ \infty, z_2) &=& 1 \\
   \fge_N(- \infty, z_2) &=& 0 \\
   \fge_N(z_1, + \infty) &=& 0 \\
   \fge_N(z_1, - \infty) &=& 1 \\
   \fge_N(\pm 0, \mp 0) &=& 1 \\
   \fge_N(z_1, z_2) &=& \bool(z_1 \geq z_2) \\
   \end{array}


.. _convert-ops:

変換（conversion）
~~~~~~~~~~~

.. _op-extend_u:

:math:`\extendu_{M,N}(i)`
.........................

* :math:`i` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \extendu_{M,N}(i) &=& i \\
   \end{array}

.. note::
   抽象構文における符号なし拡張は、単に同じ値を再解釈します。


.. _op-extend_s:

:math:`\extends_{M,N}(i)`
.........................

* :math:`j` を :ref:`signed interpretation <aux-signed>` サイズ :math:`M` の :math:`i` とする。

* サイズ :math:`N` に相対的な、:math:`j` の2の補数を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \extends_{M,N}(i) &=& \signed_N^{-1}(\signed_M(i)) \\
   \end{array}


.. _op-wrap:

:math:`\wrap_{M,N}(i)`
......................

* :math:`i` の法 :math:`2^N` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \wrap_{M,N}(i) &=& i \mod 2^N \\
   \end{array}


.. _op-trunc_u:

:math:`\truncu_{M,N}(z)`
........................

* :math:`z` がNaNの場合、結果は未定義となる。

* 上に該当しない場合、:math:`z` が無限の場合結果は未定義となる。

* 上に該当しない場合、:math:`z` がひとつの数値で :math:`\trunc(z)` が対象となる型の範囲に収まる値の場合は、その値を返す。

* 上に該当しない場合、結果は未定義となる。

.. math::
   \begin{array}{lll@{\qquad}l}
   \truncu_{M,N}(\pm \NAN(n)) &=& \{\} \\
   \truncu_{M,N}(\pm \infty) &=& \{\} \\
   \truncu_{M,N}(\pm q) &=& \trunc(\pm q) & (\iff -1 < \trunc(\pm q) < 2^N) \\
   \truncu_{M,N}(\pm q) &=& \{\} & (\otherwise) \\
   \end{array}

.. note::
   この演算子は :ref:`パーシャル <exec-op-partial>` です。
   「NaN」「無限」「結果が範囲を超える値」については定義されません。


.. _op-trunc_s:

:math:`\truncs_{M,N}(z)`
........................

* :math:`z` がNaNの場合、結果は未定義となる。

* 上に該当しない場合、:math:`z` が無限の場合結果は未定義となる。

* 上に該当しない場合、:math:`z` がひとつの数値で :math:`\trunc(z)` が対象となる型の範囲に収まる値の場合は、その値を返す。

* 上に該当しない場合、結果は未定義となる。

.. math::
   \begin{array}{lll@{\qquad}l}
   \truncs_{M,N}(\pm \NAN(n)) &=& \{\} \\
   \truncs_{M,N}(\pm \infty) &=& \{\} \\
   \truncs_{M,N}(\pm q) &=& \trunc(\pm q) & (\iff -2^{N-1} - 1 < \trunc(\pm q) < 2^{N-1}) \\
   \truncs_{M,N}(\pm q) &=& \{\} & (\otherwise) \\
   \end{array}

.. note::
   この演算子は :ref:`パーシャル <exec-op-partial>` です。
   「NaN」「無限」「結果が範囲を超える値」については定義されません。

.. _op-trunc_sat_u:

:math:`\truncsatu_{M,N}(z)`
...........................

* :math:`z` がNaNの場合、:math:`0` を返す。

* 上に該当しない場合、:math:`z` が正の無限の場合は :math:`0` を返す。

* 上に該当しない場合、:math:`z` が負の無限の場合は :math:`2^N - 1` を返す。

* 上に該当しない場合、:math:`\trunc(z)` が :math:`0` より小さい場合は :math:`0` を返す。

* 上に該当しない場合、:math:`\trunc(z)` が :math:`2^N - 1` より大きい場合は :math:`2^N - 1` を返す。

* 上に該当しない場合、:math:`\trunc(z)` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \truncsatu_{M,N}(\pm \NAN(n)) &=& 0 \\
   \truncsatu_{M,N}(- \infty) &=& 0 \\
   \truncsatu_{M,N}(+ \infty) &=& 2^N - 1 \\
   \truncsatu_{M,N}(- q) &=& 0 & (\iff \trunc(- q) < 0) \\
   \truncsatu_{M,N}(+ q) &=& 2^N - 1 & (\iff \trunc(+ q) > 2^N - 1) \\
   \truncsatu_{M,N}(\pm q) &=& \trunc(\pm q) & (otherwise) \\
   \end{array}


.. _op-trunc_sat_s:

:math:`\truncsats_{M,N}(z)`
...........................

* :math:`z` がNaNの場合、:math:`0` を返す。

* 上に該当しない場合、:math:`z` が負の無限の場合は  :math:`-2^{N-1}` を返す。

* 上に該当しない場合、:math:`z` が正の無限の場合は :math:`2^{N-1} - 1` を返す。

* 上に該当しない場合、:math:`\trunc(z)` が :math:`-2^{N-1}` より大きい場合は :math:`-2^{N-1}` を返す。

* 上に該当しない場合、:math:`\trunc(z)` が :math:`2^{N-1} - 1` より小さい場合は :math:`2^{N-1} - 1` を返す。

* 上に該当しない場合、:math:`\trunc(z)` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \truncsats_{M,N}(\pm \NAN(n)) &=& 0 \\
   \truncsats_{M,N}(- \infty) &=& -2^{N-1} \\
   \truncsats_{M,N}(+ \infty) &=& 2^{N-1}-1 \\
   \truncsats_{M,N}(- q) &=& -2^{N-1} & (\iff \trunc(- q) < -2^{N-1}) \\
   \truncsats_{M,N}(+ q) &=& 2^{N-1} - 1 & (\iff \trunc(+ q) > 2^{N-1} - 1) \\
   \truncsats_{M,N}(\pm q) &=& \trunc(\pm q) & (otherwise) \\
   \end{array}


.. _op-promote:

:math:`\promote_{M,N}(z)`
.........................

* :math:`z` が :ref:`カノニカルNaN <canonical-nan>` の場合、:math:`\nans_N\{\}` の要素をひとつ返す（すなわちサイズ :math:`N` のカノニカルNaN）。

* 上に該当しない場合、:math:`z` がNaNの場合は :math:`\nans_N\{\pm \NAN(1)\}` の要素をひとつ返す（すなわちサイズ :math:`N` の任意の :ref:`算術的NaN <arithmetic-nan>`）。

* 上に該当しない場合、:math:`z` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \promote_{M,N}(\pm \NAN(n)) &=& \nans_N\{\} & (\iff n = \canon_N) \\
   \promote_{M,N}(\pm \NAN(n)) &=& \nans_N\{+ \NAN(1)\} & (\otherwise) \\
   \promote_{M,N}(z) &=& z \\
   \end{array}


.. _op-demote:

:math:`\demote_{M,N}(z)`
........................

* :math:`z` が :ref:`カノニカルNaN <canonical-nan>` の場合、:math:`\nans_N\{\}` の要素をひとつ返す（すなわちサイズ :math:`N` のカノニカルNaN）。

* 上に該当しない場合、:math:`z` がNaNの場合は :math:`\nans_N\{\pm \NAN(1)\}` の要素をひとつ返す（すなわちサイズ :math:`N` の任意の :ref:`算術的NaN <arithmetic-nan>`）。

* 上に該当しない場合、:math:`z` が無限の場合はその無限を返す。

* 上に該当しない場合、:math:`z` がゼロの場合はそのゼロを返す。

* 上に該当しない場合、:math:`\ieee_N(z)` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \demote_{M,N}(\pm \NAN(n)) &=& \nans_N\{\} & (\iff n = \canon_N) \\
   \demote_{M,N}(\pm \NAN(n)) &=& \nans_N\{+ \NAN(1)\} & (\otherwise) \\
   \demote_{M,N}(\pm \infty) &=& \pm \infty \\
   \demote_{M,N}(\pm 0) &=& \pm 0 \\
   \demote_{M,N}(\pm q) &=& \ieee_N(\pm q) \\
   \end{array}


.. _op-convert_u:

:math:`\convertu_{M,N}(i)`
..........................

* :math:`\ieee_N(i)` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \convertu_{M,N}(i) &=& \ieee_N(i) \\
   \end{array}


.. _op-convert_s:

:math:`\converts_{M,N}(i)`
..........................

* :math:`j` を :math:`i` の :ref:`符号付き解釈 <aux-signed>` とする。

* :math:`\ieee_N(j)` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \convertu_{M,N}(i) &=& \ieee_N(\signed_M(i)) \\
   \end{array}


.. _op-reinterpret:

:math:`\reinterpret_{t_1,t_2}(c)`
.................................

* :math:`d^\ast` を :math:`\bits_{t_1}(c)` のビットシーケンスとする。

* :math:`\bits_{t_2}(c') = d^\ast` における定数 :math:`c'` を返す。

.. math::
   \begin{array}{lll@{\qquad}l}
   \reinterpret_{t_1,t_2}(c) &=& \bits_{t_2}^{-1}(\bits_{t_1}(c)) \\
   \end{array}
