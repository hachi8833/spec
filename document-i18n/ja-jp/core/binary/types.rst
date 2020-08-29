.. index:: type
   pair: binary format; type
.. _binary-type:

型
-----

.. index:: value type
   pair: binary format; value type
.. _binary-valtype:

値型（value type）
~~~~~~~~~~~

:ref:`値型 <syntax-valtype>` は1バイトにエンコードされます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{value type} & \Bvaltype &::=&
     \hex{7F} &\Rightarrow& \I32 \\ &&|&
     \hex{7E} &\Rightarrow& \I64 \\ &&|&
     \hex{7D} &\Rightarrow& \F32 \\ &&|&
     \hex{7C} &\Rightarrow& \F64 \\
   \end{array}

.. note::
   値型は、:ref:`型インデックス <syntax-typeidx>` も許されているコンテキスト（:ref:`ブロック型 <binary-blocktype>` の場合など）に出現することがあります。
   このため型のバイナリ形式は、将来（正の）型インデックスと共存できるよう、小さな負の :math:`\sN` 値を |SignedLEB128|_ :ref:`エンコーディング <binary-sint>` したものに対応付けられます。

.. index:: result type, value type
   pair: binary format; result type
.. _binary-resulttype:

結果型（result type）
~~~~~~~~~~~~

:ref:`結果型 <syntax-resulttype>` は、それぞれの :ref:`値型 <binary-valtype>` に対応する :ref:`ベクタ <binary-vec>` によってエンコードされます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{result type} & \Bresulttype &::=&
     t^\ast{:\,}\Bvec(\Bvaltype) &\Rightarrow& [t^\ast] \\
   \end{array}


.. index:: function type, value type, result type
   pair: binary format; function type
.. _binary-functype:

関数型（function type）
~~~~~~~~~~~~~~

:ref:`関数型 <syntax-functype>` は、バイト :math:`\hex{60}` で始まり、その後ろにパラメーター型と結果型それぞれの :ref:`ベクタ <binary-vec>` を置いたものでエンコードされます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{function type} & \Bfunctype &::=&
     \hex{60}~~\X{rt}_1{:\,}\Bresulttype~~\X{rt}_2{:\,}\Bresulttype
       &\Rightarrow& \X{rt}_1 \to \X{rt}_2 \\
   \end{array}


.. index:: limits
   pair: binary format; limits
.. _binary-limits:

制限（limit）
~~~~~~

:ref:`制限 <syntax-limits>` は、最大値が存在するかどうかを示すフラグをその前に付けてエンコードされます。

.. math::
   \begin{array}{llclll}
   \production{limits} & \Blimits &::=&
     \hex{00}~~n{:}\Bu32 &\Rightarrow& \{ \LMIN~n, \LMAX~\epsilon \} \\ &&|&
     \hex{01}~~n{:}\Bu32~~m{:}\Bu32 &\Rightarrow& \{ \LMIN~n, \LMAX~m \} \\
   \end{array}


.. index:: memory type, limits, page size
   pair: binary format; memory type
.. _binary-memtype:

メモリー型（memory type）
~~~~~~~~~~~~

:ref:`メモリー型 <syntax-memtype>` は、:ref:`制限 <binary-limits>` 付きでエンコードされます。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{memory type} & \Bmemtype &::=&
     \X{lim}{:}\Blimits &\Rightarrow& \X{lim} \\
   \end{array}


.. index:: table type, element type, limits
   pair: binary format; table type
   pair: binary format; element type
.. _binary-elemtype:
.. _binary-tabletype:

テーブル型（table type）
~~~~~~~~~~~

:ref:`テーブル型 <syntax-tabletype>` は、その :ref:`制限 <binary-limits>` に :ref:`要素型 <syntax-elemtype>` を表す定数バイト1個を付けてエンコードされます。

.. math::
   \begin{array}{llclll}
   \production{table type} & \Btabletype &::=&
     \X{et}{:}\Belemtype~~\X{lim}{:}\Blimits &\Rightarrow& \X{lim}~\X{et} \\
   \production{element type} & \Belemtype &::=&
     \hex{70} &\Rightarrow& \FUNCREF \\
   \end{array}


.. index:: global type, mutability, value type
   pair: binary format; global type
   pair: binary format; mutability
.. _binary-mut:
.. _binary-globaltype:

グローバル型（global type）
~~~~~~~~~~~~

:ref:`グローバル型 <syntax-globaltype>` は、その :ref:`値型 <binary-valtype>` に :ref:`ミュータブル・イミュータブル <syntax-mut>` を示すフラグを付けてエンコードされます。

.. math::
   \begin{array}{llclll}
   \production{global type} & \Bglobaltype &::=&
     t{:}\Bvaltype~~m{:}\Bmut &\Rightarrow& m~t \\
   \production{mutability} & \Bmut &::=&
     \hex{00} &\Rightarrow& \MCONST \\ &&|&
     \hex{01} &\Rightarrow& \MVAR \\
   \end{array}
