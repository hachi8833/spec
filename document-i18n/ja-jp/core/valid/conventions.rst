.. index:: ! validation, ! type system, function type, table type, memory type, globaltype, valtype, resulttype, index space, instantiation. module
.. _type-system:

記法
-----------

検証（validation）は、WebAssemblyモジュールが整形式（well-formed）であることをチェックします。
有効なモジュールだけが :ref:`インスタンス化 <exec-instantiation>` できます。

検証の正当性は、:ref:`モジュール <syntax-module>` の :ref:`抽象構文 <syntax>` 上の「型システム（type system）」によって定義されます。
抽象構文のひとつひとつについて、適用される制約を指定する型付けルールが存在します。
すべてのルールは、以下の2種類の「同等な」形式で与えられます。

1. 「散文記法」は、直感的な形で意味を記述します。
2. 「形式的記法」は、ルールを数学的な形式で記述します [#cite-pldi2017]_ 。

.. note::
   散文記法と形式的記法は同等なので、本仕様書を読む上で形式的記法の理解は必須では「ありません」。
   この形式主義によって、プログラミング言語のセマンティクスで広く用いられている簡潔な記法を提供し、数学的証明の対象としてふさわしい形になります。

どちらの記述においても、ルールは「宣言的」な形で定式化されます。
すなわち、ルールは制約を定式化するだけであって、アルゴリズムを定義しているわけではありません。
本仕様書に沿ってインストラクションシーケンスを型チェックするための健全かつ完備なアルゴリズムの骨格については、:ref:`付録 <algo-valid>` で提供されています。

.. index:: ! context, function type, table type, memory type, global type, value type, result type, index space, module, function
.. _context:

コンテキスト（context）
~~~~~~~~

個別の定義の正当性は「コンテキスト」に相対的な形で指定されます。つまり、それを取り巻く :ref:`モジュール <syntax-module>` に関する情報と、以下のスコープに該当する定義を収集します。

* *型*: 現在のモジュールで定義されている型のリスト。
* *関数*: 現在のモジュールで宣言されている関数をその関数型で表したリスト。
* *テーブル*: 現在のモジュールで宣言されているテーブルをそのテーブル型で表したリスト。
* *メモリー*: 現在のモジュールで宣言されているメモリーをそのメモリー型で表したリスト。
* *グローバル*: 現在のモジュールで宣言されているグローバルをそのグローバル型で表したリスト。
* *ローカル値*: 現在の関数（とパラメーター）で宣言されているローカル値をその値型で表したリスト。
* *ラベル*: 現在の位置からアクセス可能なラベルをその結果型で表したスタック。
* *戻り値*: 現在の関数が返す型を、オプションの結果型（つまり戻り値が許されていない場合は空になる）の独立した式として表したもの。

言い換えると、あるコンテキストに含まれるのは、そのインデックス空間内で定義されている個別のエントリを記述するそれぞれの :ref:`インデックス空間 <syntax-index>` に適した :ref:`型 <syntax-type>` のシーケンスです。
「ローカル値」「ラベル」「戻り値の型」は、:ref:`関数本体 <syntax-func>` 内の :ref:`インストラクション <syntax-instr>` を検証するときにだけ使われ、それ以外の場所では空のままとなります。
ラベルスタックは、あるインストラクションシーケンスでのバリデーションが進行するに連れて変化するコンテキストの唯一の部分となります。

より具体的には、コンテキストは抽象構文を用いて :ref:`記録 <notation-record>` :math:`C` として定義されます。

.. math::
   \begin{array}{llll}
   \production{(context)} & C &::=&
     \begin{array}[t]{l@{~}ll}
     \{ & \CTYPES & \functype^\ast, \\
        & \CFUNCS & \functype^\ast, \\
        & \CTABLES & \tabletype^\ast, \\
        & \CMEMS & \memtype^\ast, \\
        & \CGLOBALS & \globaltype^\ast, \\
        & \CLOCALS & \valtype^\ast, \\
        & \CLABELS & \resulttype^\ast, \\
        & \CRETURN & \resulttype^? ~\} \\
     \end{array}
   \end{array}

.. _notation-extend:

:math:`C.\K{field}` で書かれるフィールドアクセスに加えて、以下の記法がコンテキスト操作に採用されています。

* あるコンテキストをスペルアウトすると、空フィールドは省略されます。

* :math:`C,\K{field}\,A^\ast`： :math:`C` と同じコンテキストを表しますが、その :math:`\K{field}` コンポーネントシーケンスより前に要素 :math:`A^\ast` を持ちます。

.. note::
   ここでは、コンテキスト内にあるそれぞれの :ref:`インデックス空間 <syntax-index>` 内でインデックスを探索するのに :math:`C.\CLABELS[i]` のような :ref:`インデックス記法 <notation-index>` を用いています。
   コンテキストの拡張記法  :math:`C,\K{field}\,A` は、主に「相対的な」インデックス空間（:ref:`ラベルインデックス <syntax-labelidx>` など）をローカルで拡張するのに用いられます。
   それにより、この記法は新たな相対インデックス :math:`0` を導入して既存のものを移行し、それぞれのシーケンスの「前方」に追加するよう定義されています。

.. _valid-notation-textual:

散文記法（prose notation）
~~~~~~~~~~~~~~

検証は、:ref:`抽象構文 <syntax>` で関連する部分ごとに対するスタイル化されたルールによって指定されます。
このルールは、あるフェーズがいつ有効になるかを定義する制約を記述するだけではなく、型を用いた分類も行います。
以下の記法はこうしたルールの記述に用いられています。

* フェーズ :math:`A` は、個別のルールで表現された制約がすべて満たされた場合に限り、「型  :math:`T` で有効」となります。
  :math:`T` の形式は、フェーズ :math:`A` が何であるかによって変わります。

  .. note::
     たとえば、:math:`A` がある :ref:`関数 <syntax-func>` であれば、:math:`T` は :ref:`関数型 <syntax-functype>` となります。
     :ref:`グローバル <syntax-global>` である  :math:`A` なら、:math:`T` は :ref:`グローバル型 <syntax-globaltype>` となる、といった具合になります。

* これらのルールでは、与えられた  :ref:`コンテキスト <context>` :math:`C` を暗黙で仮定しています。

* ところによっては、コンテキストがローカルで拡張され、追加エントリを持つコンテキスト :math:`C'` になることがあります。「コンテキスト :math:`C'` のもとではナニナニである」という形式化は、それに続くステートメントが拡張済みコンテキストで具現化された仮定のもとで適用されなければならないことを表現するために採用されています。

.. index:: ! typing rules
.. _valid-notation:

形式的記法（formal notation）
~~~~~~~~~~~~~~~

.. note::
   本セクションでは、型付けルールを形式的に指定するための記法について簡単に説明するにとどめます。このテーマに関心のある方は、関連する教科書でさらに詳しい紹介文を読めます [#cite-tapl]_ 。

「フェーズ :math:`A` は、それに対応する型 :math:`T` を持つ」という命題は :math:`A : T` と書きます。
しかし一般に、型付けはコンテキスト :math:`C` によって変わってきます。
このことを明示的に表現するための完備な形式は、:math:`C \vdash A : T` という「判定（judgement）」で、これはコンテキスト :math:`C` にエンコードされた仮定のもとでは :math:`A : T` が成り立つことを表します。

形式的な型付けルールでは、型システムを指定するのにある標準的なアプローチを用いており、それらを「推論規則（deduction rule）」という形に落とし込んでいます。

あらゆるルールは以下の一般形式を持ちます。

.. math::
   \frac{
     \X{前提（premise）}_1 \qquad \X{前提（premise）}_2 \qquad \dots \qquad \X{前提（premise）}_n
   }{
     \X{結論（conclusion）}
   }

このようなルールは、ある重要な示唆として読み取れます。すなわち「前提がすべて成立すれば結論は成立する」ということです。
ルールによっては前提条件が存在しないものもありますが、これらは「公理（axiom）」と呼ばれ、無条件に結論が成立します。
この結論は、常に判定 :math:`C \vdash A : T` となるので、抽象構文において関連する構成 :math:`A` ごとに、それに対応するルールがひとつ存在することになります。

.. note::
   たとえば、:math:`\I32.\ADD` というインストラクションの型付けルールは以下の公理によって与えられます。

   .. math::
      \frac{
      }{
        C \vdash \I32.\ADD : [\I32~\I32] \to [\I32]
      }

   このインストラクションは、型 :math:`[\I32~\I32] \to [\I32`] において常に有効（valid）です
   （これは2つの |I32| 値を消費して1つを生成することを意味します）。
   このことは、その他の副条件から完全に独立しています。

   |LOCALGET| のようなインストラクションは以下のように型付けできます。

   .. math::
      \frac{
        C.\CLOCALS[x] = t
      }{
        C \vdash \LOCALGET~x : [] \to [t]
      }

   ここでは、コンテキストに即値  :ref:`ローカルインデックス <syntax-localidx>` :math:`x` が存在することが前提で強制されています。
   このインストラクションは、対応する型 :math:`t` から値を1つ生成します（値の消費は一切発生しません）。
   :math:`C.\CLOCALS[x]` が存在しない場合は前提が成立しないので、このインストラクションの型付けは違反となります。

   最後に、:ref:`構造化 <syntax-instr-control>` インストラクションでは以下のようにある再帰ルールが要求され、前提はそれ自体が型判定となっています。

   .. math::
      \frac{
        C \vdash \blocktype : [t_1^\ast] \to [t_2^\ast]
        \qquad
        C,\LABEL\,[t_2^\ast] \vdash \instr^\ast : [t_1^\ast] \to [t_2^\ast]
      }{
        C \vdash \BLOCK~\blocktype~\instr^\ast~\END : [t_1^\ast] \to [t_2^\ast]
      }

   |BLOCK| インストラクションは、その本体（body）内のインストラクションシーケンスが有効な場合にのみ有効となります。
   さらに、その結果型はブロックのアノテーション :math:`\blocktype` と一致しなければなりません。
   一致する場合は、|BLOCK| インストラクションの型はその本体の型と同じになります。
   その本体内では、結果型に対応する追加ラベルが利用できるようになり、これはその前提を示す追加ラベル情報でコンテキスト :math:`C` を拡張することで表現されます。

.. [#cite-pldi2017]
   このセマンティクスは以下の記事から派生したものです。
   Andreas Haas, Andreas Rossberg, Derek Schuff, Ben Titzer, Dan Gohman, Luke Wagner, Alon Zakai, JF Bastien, Michael Holman. |PLDI2017|_. Proceedings of the 38th ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI 2017). ACM 2017.

.. [#cite-tapl]
   例: Benjamin Pierce. |TAPL|_. The MIT Press 2002
