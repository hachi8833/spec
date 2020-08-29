型
-----

ほとんどの :ref:`型 <syntax-type>` は普遍的に有効です。
しかし :ref:`制限 <syntax-limits>` には制約（restrictions）が適用されますし、これらは検証フェーズでチェックされなければなりません。
さらに、:ref:`ブロック型 <syntax-blocktype>` は処理をより容易にするため、純粋な :ref:`関数型 <syntax-functype>` に変換されます。

.. index:: limits
   pair: validation; limits
   single: abstract syntax; limits
.. _valid-limits:

制限（limit）
~~~~~~

:ref:`制限 <syntax-limits>` は、指定された範囲内で意味のある束縛を持たなければなりません。

:math:`\{ \LMIN~n, \LMAX~m^? \}`
................................

* :math:`n` の値は :math:`k` より大きくなってはならない。

* 最大値 :math:`m^?` が空でない場合は以下のようになる。

  * その値は :math:`k` より大きくなってはならない。

  * その値は :math:`n` より小さくなってはならない。

* これによって制限は範囲 :math:`k` の中で有効となる。

.. math::
   \frac{
     n \leq k
     \qquad
     (m \leq k)^?
     \qquad
     (n \leq m)^?
   }{
     \vdashlimits \{ \LMIN~n, \LMAX~m^? \} : k
   }


.. index:: block type
   pair: validation; block type
   single: abstract syntax; block type
.. _valid-blocktype:

ブロック型（block type）
~~~~~~~~~~~

:ref:`ブロック型 <syntax-blocktype>` は2つの形式のいずれかで表現できます。2つの形式は以下のルールに沿って純粋な :ref:`関数型 <syntax-functype>` に変換されます。

:math:`\typeidx`
................

* 型 :math:`C.\CTYPES[\typeidx]` はそのコンテキスト内で定義されなければならない。

* これによってブロック型は :ref:`関数型 <syntax-functype>` :math:`C.\CTYPES[\typeidx]` として有効となる。

.. math::
   \frac{
     C.\CTYPES[\typeidx] = \functype
   }{
     C \vdashblocktype \typeidx : \functype
   }


:math:`[\valtype^?]`
....................

* このブロック型は :ref:`関数型 <syntax-functype>` :math:`[] \to [\valtype^?]` として有効となる。

.. math::
   \frac{
   }{
     C \vdashblocktype [\valtype^?] : [] \to [\valtype^?]
   }


.. index:: function type
   pair: validation; function type
   single: abstract syntax; function type
.. _valid-functype:

関数型（function type）
~~~~~~~~~~~~~~

:ref:`関数型 <syntax-functype>` は常に有効です。

:math:`[t_1^n] \to [t_2^m]`
...........................

* 以下の関数型は有効。

.. math::
   \frac{
   }{
     \vdashfunctype [t_1^\ast] \to [t_2^\ast] \ok
   }


.. index:: table type, element type, limits
   pair: validation; table type
   single: abstract syntax; table type
.. _valid-tabletype:

テーブル型（table type）
~~~~~~~~~~~

:math:`\limits~\elemtype`
.........................

* 制限 :math:`\limits` は範囲 :math:`2^{32}` の中で :ref:`有効 <valid-limits>` でなければならない。

* これにより、このテーブル型は有効となる。

.. math::
   \frac{
     \vdashlimits \limits : 2^{32}
   }{
     \vdashtabletype \limits~\elemtype \ok
   }


.. index:: memory type, limits
   pair: validation; memory type
   single: abstract syntax; memory type
.. _valid-memtype:

メモリー型（memory type）
~~~~~~~~~~~~

:math:`\limits`
...............

* 制限 :math:`\limits` は範囲 :math:`2^{16}` の中で :ref:`有効 <valid-limits>` でなければならない。

* これにより、このメモリー型は有効となる。

.. math::
   \frac{
     \vdashlimits \limits : 2^{16}
   }{
     \vdashmemtype \limits \ok
   }


.. index:: global type, value type, mutability
   pair: validation; global type
   single: abstract syntax; global type
.. _valid-globaltype:

グローバル型（global type）
~~~~~~~~~~~~

:math:`\mut~\valtype`
.....................

* 以下のグローバル型は有効。

.. math::
   \frac{
   }{
     \vdashglobaltype \mut~\valtype \ok
   }


.. index:: external type, function type, table type, memory type, global type
   pair: validation; external type
   single: abstract syntax; external type
.. _valid-externtype:

外部型（external type）
~~~~~~~~~~~~~~

:math:`\ETFUNC~\functype`
.........................

* :ref:`関数型 <syntax-functype>` :math:`\functype` は :ref:`有効 <valid-functype>` でなければならない。

* これによって外部型は有効となる。

.. math::
   \frac{
     \vdashfunctype \functype \ok
   }{
     \vdashexterntype \ETFUNC~\functype \ok
   }

:math:`\ETTABLE~\tabletype`
...........................

* :ref:`テーブル型 <syntax-tabletype>` :math:`\tabletype` は :ref:`有効 <valid-functype>` でなければならない。

* これによって外部型は有効となる。

.. math::
   \frac{
     \vdashtabletype \tabletype \ok
   }{
     \vdashexterntype \ETTABLE~\tabletype \ok
   }

:math:`\ETMEM~\memtype`
.......................

* :ref:`メモリー型 <syntax-memtype>` :math:`\memtype` は :ref:`有効 <valid-functype>` でなければならない。

* これによって外部型は有効となる。

.. math::
   \frac{
     \vdashmemtype \memtype \ok
   }{
     \vdashexterntype \ETMEM~\memtype \ok
   }

:math:`\ETGLOBAL~\globaltype`
.............................

* :ref:`グローバル型 <syntax-globaltype>` :math:`\globaltype` は :ref:`有効 <valid-functype>` でなければならない。

* これによって外部型は有効となる。

.. math::
   \frac{
     \vdashglobaltype \globaltype \ok
   }{
     \vdashexterntype \ETGLOBAL~\globaltype \ok
   }
