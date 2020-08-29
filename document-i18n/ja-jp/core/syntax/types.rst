.. index:: ! type, validation, instantiation, execution
   pair: abstract syntax; type
.. _syntax-type:

型
-----

WebAssemblyにおけるさまざまなエンティティは型によって分類されます。
型のチェックは「:ref:`検証 <valid>`」や「:ref:`インスタンス化 <exec-instantiation>`」の間に行われますが、「:ref:`実行時 <syntax-call_indirect>`」の間に行われる可能性もあります。

.. index:: ! value type, integer, floating-point, IEEE 754, bit width
   pair: abstract syntax; value type
   pair: value; type
.. _syntax-valtype:

値型（value type）
~~~~~~~~~~~

「値型」は、WebAssemblyコードが計算する個別の値や、変数で使える個別の値を分類するものです。

.. math::
   \begin{array}{llll}
   \production{value type} & \valtype &::=&
     \I32 ~|~ \I64 ~|~ \F32 ~|~ \F64 \\
   \end{array}

「|I32|」型と「|I64|」型は、それぞれ32ビット整数と64ビット整数を分類します。
WebAssemblyにおける整数はもともと符号付きや符号なしを区別しないので、符号付きや符号なしの解釈は個別の操作によって決定されます。

「|F32|」型と「|F64|」型は、それぞれ32ビット浮動小数データと64ビット浮動小数データを分類します。
これらそれぞれに対応するバイナリ浮動小数点表現が存在しており、|IEEE754|_ 標準（セクション3.3）で定義されている「単精度」「倍精度」とも呼ばれています。

本仕様での記法
...........

* メタ変数 :math:`t` は、コンテキストから明らかな場合、さまざまな値型を表します。

* :math:`|t|` という記法は、その値型の「ビット幅」を表します。
  つまり :math:`|\I32| = |\F32| = 32` であり、:math:`|\I64| = |\F64| = 64` となります。

.. index:: ! result type, value type, instruction, execution, function
   pair: abstract syntax; result type
   pair: result; type
.. _syntax-resulttype:

結果型（result type）
~~~~~~~~~~~~

「結果型」は、:ref:`インストラクション <syntax-instr>` や :ref:`関数 <syntax-func>` を :ref:`実行 <exec-instr>` した結果を分類します。
これは値のシーケンスを角かっこで囲んだものです。

.. math::
   \begin{array}{llll}
   \production{result type} & \resulttype &::=&
     [\vec(\valtype)] \\
   \end{array}


.. index:: ! function type, value type, vector, function, parameter, result, result type
   pair: abstract syntax; function type
   pair: function; type
.. _syntax-functype:

関数型（function type）
~~~~~~~~~~~~~~

「関数型」は、:ref:`関数 <syntax-func>` のシグネチャ（signature）を分類します。パラメーターのベクタを結果のベクタに対応付けます。
関数型は :ref:`インストラクション <syntax-instr>` の入力や出力を分類するのにも用いられます。

.. math::
   \begin{array}{llll}
   \production{function type} & \functype &::=&
     \resulttype \to \resulttype \\
   \end{array}


.. index:: ! limits, memory type, table type
   pair: abstract syntax; limits
   single: memory; limits
   single: table; limits
.. _syntax-limits:

制限（limit）
~~~~~~

「制限」は、:ref:`メモリー型 <syntax-memtype>` や :ref:`テーブル型 <syntax-tabletype>` に関連するリサイズ可能なストレージサイズの範囲を分類します。

.. math::
   \begin{array}{llll}
   \production{limits} & \limits &::=&
     \{ \LMIN~\u32, \LMAX~\u32^? \} \\
   \end{array}

最大値が指定されない場合、対応するストレージは任意のサイズにまで成長する可能性があります。

.. index:: ! memory type, limits, page size, memory
   pair: abstract syntax; memory type
   pair: memory; type
   pair: memory; limits
.. _syntax-memtype:

メモリー型（memory type）
~~~~~~~~~~~~

「メモリー型」は、線形 :ref:`メモリー <syntax-mem>` とそのサイズの範囲を分類します。

.. math::
   \begin{array}{llll}
   \production{memory type} & \memtype &::=&
     \limits \\
   \end{array}

メモリー型の制限は、メモリーサイズの最小値に（オプションで最大値にも）制約をかけます。
この制限は :ref:`ページサイズ <page-size>` を単位として指定します。

.. index:: ! table type, ! element type, limits, table, element
   pair: abstract syntax; table type
   pair: abstract syntax; element type
   pair: table; type
   pair: table; limits
   pair: element; type
.. _syntax-elemtype:
.. _syntax-tabletype:

テーブル型（table type）
~~~~~~~~~~~

「テーブル型」は、サイズが範囲に収まっている「要素型」の要素に対する :ref:`テーブル <syntax-table>` を分類します。

.. math::
   \begin{array}{llll}
   \production{table type} & \tabletype &::=&
     \limits~\elemtype \\
   \production{element type} & \elemtype &::=&
     \FUNCREF \\
   \end{array}

メモリー型と同様に、テーブル型のサイズの最小値に（オプションで最大値にも）制約をかけられます。
この制約は、エントリー数の形で指定します。


要素型 |FUNCREF| は、あらゆる :ref:`関数型 <syntax-functype>` の無限和集合です。
従ってこの型のテーブルは、型の異なるさまざまな関数への参照を含みます。

.. note::
   WebAssemblyの今後のバージョンでは、これ以外の要素型も導入される可能性があります。

.. index:: ! global type, ! mutability, value type, global, mutability
   pair: abstract syntax; global type
   pair: abstract syntax; mutability
   pair: global; type
   pair: global; mutability
.. _syntax-mut:
.. _syntax-globaltype:

グローバル型（global type）
~~~~~~~~~~~~

グローバル型は :ref:`グローバル <syntax-global>` 変数を分類します。グローバル変数は、ミュータブルな値またはイミュータブルな値を保持します。

.. math::
   \begin{array}{llll}
   \production{global type} & \globaltype &::=&
     \mut~\valtype \\
   \production{mutability} & \mut &::=&
     \MCONST ~|~
     \MVAR \\
   \end{array}


.. index:: ! external type, function type, table type, memory type, global type, import, external value
   pair: abstract syntax; external type
   pair: external; type
.. _syntax-externtype:

外部型（external type）
~~~~~~~~~~~~~~

「外部型」は、:ref:`import <syntax-import>` や、対応するさまざまな型を持つ :ref:`外部値 <syntax-externval>` を分類します。

.. math::
   \begin{array}{llll}
   \production{external types} & \externtype &::=&
     \ETFUNC~\functype ~|~
     \ETTABLE~\tabletype ~|~
     \ETMEM~\memtype ~|~
     \ETGLOBAL~\globaltype \\
   \end{array}


本仕様での記法
...........

外部型のシーケンスを定義するのに、以下の補助記法も用います。
これらは特定の種類のエントリーを、順序を維持する形でフィルタします。

* :math:`\etfuncs(\externtype^\ast) = [\functype ~|~ (\ETFUNC~\functype) \in \externtype^\ast]`

* :math:`\ettables(\externtype^\ast) = [\tabletype ~|~ (\ETTABLE~\tabletype) \in \externtype^\ast]`

* :math:`\etmems(\externtype^\ast) = [\memtype ~|~ (\ETMEM~\memtype) \in \externtype^\ast]`

* :math:`\etglobals(\externtype^\ast) = [\globaltype ~|~ (\ETGLOBAL~\globaltype) \in \externtype^\ast]`
