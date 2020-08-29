.. index:: validation, algorithm, instruction, module, binary format, opcode
.. _algo-valid:

検証アルゴリズム
--------------------

WebAssembly の :ref:`検証 <valid>` の仕様は純粋に「宣言的」です。これは、ある :ref:`モジュール <valid-module>` や :ref:`インストラクション <valid-instr>` シーケンスが有効になるために満たさなければならない制約を記述します。


本セクションでは、コード（すなわち :ref:`インストラクション <syntax-instr>` のシーケンス）を効果的に検証する健全かつ完備な「アルゴリズム」の骨格を簡単にスケッチします（それ以外の側面は実装に直結します）。

実際このアルゴリズムは、:ref:`バイナリ形式 <binary>` 内で発生するopcodeのフラットなシーケンスで表現され、実行は1パスのみです。
その結果、このアルゴリズムはデコーダーと直接統合できます。

この検証アルゴリズムは型付きの擬似コードで表現され、そのセマンティクスは意味が自明となるよう意図されています。


.. index:: value type, stack, label, frame, instruction

データ構造
~~~~~~~~~~~~~~~

このアルゴリズムでは「オペランドスタック（operand stack）」と「制御スタック（control stack）」という2つの独立したスタックを用います。
前者は :ref:`スタック <stack>` 上にあるオペランド値の :ref:`型 <syntax-valtype>` をトラッキングし、後者はそれを囲む :ref:`構造化制御インストラクション <syntax-instr-control>` とそれらに関連付けられる :ref:`ブロック <syntax-instr-control>` をトラッキングします。

.. code-block:: pseudo

   type val_type = I32 | I64 | F32 | F64

   type opd_stack = stack(val_type | Unknown)

   type ctrl_stack = stack(ctrl_frame)
   type ctrl_frame = {
     opcode : opcode
     start_types : list(val_type)
     end_types : list(val_type)
     height : nat
     unreachable : bool
   }

オペランドスタックは、個別の値の ref:`値型 <syntax-valtype>` を記録し、型が不明な場合は :code:`Unknown` を記録します。

コントロールスタックは、エンターした個別のブロックの「制御フレーム（control frame）」（元のopcodeを持つ）を記録し、オペランドスタックのトップにある型をブロックの冒頭と末尾に記録し（分岐とその結果をチェックするのに用いる）、オペランドスタックの高さをブロックの冒頭に記録し（そのオペランドが現在のブロックをアンダーフローしないことのチェックに用いる）、ブロックの残りの部分がunreachable（到達不能）かどうかを表すフラグ（分岐後の :ref:`スタックポリモーフィック <polymorphism>` 型付けを扱うのに用いる）を記録します。

オペランドと制御スタックは、アルゴリズムを表現する目的のため、グローバル変数と同様のシンプルさを維持します。

.. code-block:: pseudo

   var opds : opd_stack
   var ctrls : ctrl_stack

ただし、これらの変数はメインのチェック関数が直接操作するのではなく、補助関数のセットを介して操作されます。

.. code-block:: pseudo

   func push_opd(type : val_type | Unknown) =
     opds.push(type)

   func pop_opd() : val_type | Unknown =
     if (opds.size() = ctrls[0].height && ctrls[0].unreachable) return Unknown
     error_if(opds.size() = ctrls[0].height)
     return opds.pop()

   func pop_opd(expect : val_type | Unknown) : val_type | Unknown =
     let actual = pop_opd()
     if (actual = Unknown) return expect
     if (expect = Unknown) return actual
     error_if(actual =/= expect)
     return actual

   func push_opds(types : list(val_type)) = foreach (t in types) push_opd(t)
   func pop_opds(types : list(val_type)) = foreach (t in reverse(types)) pop_opd(t)

オペランドのpushは、対応する型をオペランドスタックに文字どおりpushします。

オペランドのpopは、そのオペランドスタックが現在のブロックをアンダーフローしていないことをチェックしてから、ひとつの型を削除します。
ただし1番目の関数では、ブロックが既知のオペランドを含まない箇所で特殊なケースが取り扱われますが、これはunreachableとしてマーキングされます。
unreachableは、そのスタックが :ref:`ポリモーフィック <polymorphism>` に型付けされている場合に、無条件分岐より後の部分で発生する可能性があります。
その場合、Unknown型がひとつ返されます。

2番目の関数はオペランドをpopしますが、これは期待される型をひとつ受け取り、実際のオペランド型をその型でチェックします。
いずれかがUnknownの場合、型は異なる可能性があります。
より具体的な型が返されます。

残りは、複数のオペランド型をpushまたはpopする集約関数です。

.. note::
   :code:`stack[i]` という記法は、スタックをトップから開始するインデックスを意味します（:code:`ctrls[0]` が最後にpushした要素にアクセスするように）。

制御スタックも、補助関数を介して同様に操作されます。

.. code-block:: pseudo

   func push_ctrl(opcode : opcode, in : list(val_type), out : list(val_type)) =
     let frame = ctrl_frame(opcode, in, out, opds.size(), false)
     ctrls.push(frame)
     push_opds(in)

   func pop_ctrl() : ctrl_frame =
     error_if(ctrls.is_empty())
     let frame = ctrls[0]
     pop_opds(frame.end_types)
     error_if(opds.size() =/= frame.height)
     ctrls.pop()
     return frame

   func label_types(frame : ctrl_frame) : list(val_types) =
     return (if frame.opcode == loop then frame.start_types else frame.end_types)

   func unreachable() =
     opds.resize(ctrls[0].height)
     ctrls[0].unreachable := true

制御フレームのpushは、ラベルの型と結果値の型を受け取ります。
それにより、オペランドスタックの現在の高さにそれらを記録する新しいフレーム記録をアロケーションし、そのブロックを到達可能（reachable）とマーキングします。

フレームのpopは、最初に制御スタックが空でないことをチェックします。
続いて、終了したブロックの末尾で期待される値の正しい型をオペランドスタックが含むことを検証してから、オペランドスタックからpopします。
その後、スタックが元の高さに縮んだことをチェックします。

ひとつの制御フレームに関連付けられる :ref:`ラベル <syntax-label>` の型は、そのフレームの冒頭または末尾のスタックの型であり、その由来となるopcodeによって決定されます。

最後に、現在のフレームはunreachableとマーキング可能です。
その場合、既存のオペランド型はすべてオペランドスタックから排除されます。これは、:code:`pop_opd` で :ref:`スタックポリモーフィズム <polymorphism>` を有効にするためのものです。

.. note::
   unreachableフラグが設定されている場合であっても、それに続くオペランドはオペランドスタックで引き続きpushまたはpopされます。
   これは、:math:`(\UNREACHABLE~(\I32.\CONST)~\I64.\ADD)` のように無効な :ref:`例 <polymorphism>` を検出するのに必要です。
   ただしポリモーフィックなスタックはアンダーフローできないので、その代わり必要に応じて :code:`Unknown` 型を生成します。

.. index:: opcode

opcodeシーケンスの検証
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

以下の関数は、スタックを操作する多数の代表的なインストラクションの検証を示しています。
その他のインストラクションについても類似の方法でチェックされます。

.. note::
   以下に記載されていないさまざまなインストラクションについては、:ref:`インデックス <syntax-index>` の利用をチェックするための検証 :ref:`コンテキスト <context>` も追加で存在する必要があります。
   この追加は容易なので、この説明では省略します。

.. code-block:: pseudo

   func validate(opcode) =
     switch (opcode)
       case (i32.add)
         pop_opd(I32)
         pop_opd(I32)
         push_opd(I32)

       case (drop)
         pop_opd()

       case (select)
         pop_opd(I32)
         let t1 = pop_opd()
         let t2 = pop_opd(t1)
         push_opd(t2)

       case (unreachable)
         unreachable()

       case (block t1*->t2*)
         pop_opds([t1*])
         push_ctrl(block, [t1*], [t2*])

       case (loop t1*->t2*)
         pop_opds([t1*])
         push_ctrl(loop, [t1*], [t2*])

       case (if t1*->t2*)
         pop_opd(I32)
         pop_opds([t1*])
         push_ctrl(if, [t1*], [t2*])

       case (end)
         let frame = pop_ctrl()
         push_opds(frame.end_types)

       case (else)
         let frame = pop_ctrl()
         error_if(frame.opcode =/= if)
         push_ctrl(else, frame.start_types, frame.end_types)

       case (br n)
         error_if(ctrls.size() < n)
         pop_opds(label_types(ctrls[n]))
         unreachable()

       case (br_if n)
         error_if(ctrls.size() < n)
         pop_opd(I32)
         pop_opds(label_types(ctrls[n]))
         push_opds(label_types(ctrls[n]))

       case (br_table n* m)
         error_if(ctrls.size() < m)
         foreach (n in n*)
           error_if(ctrls.size() < n || label_types(ctrls[n]) =/= label_types(ctrls[m]))
         pop_opd(I32)
         pop_opds(label_types(ctrls[m]))
         unreachable()

.. note::
   現在のWebAssemblyインストラクションセットにおいては、:code:`Unknown` 型のオペランドはスタック内で決して重複しないことが常に確定します。
   この言語が :code:`dup` などのスタックインストラクションで拡張されると、この部分が変わる可能性があります。
   そのような拡張が行われた場合、上述のアルゴリズムは :code:`Unknown` 型を正しい「型変数」に置き換えてすべての利用を一貫させる形に改める必要が生じるでしょう。

