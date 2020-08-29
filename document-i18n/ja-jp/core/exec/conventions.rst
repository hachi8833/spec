.. index:: ! execution, stack, store

記法
-----------

WebAssemblyのコードは、あるモジュールを :ref:`インスタンス化 <exec-instantiation>` するときか、生成されたモジュール :ref:`インスタンス <syntax-moduleinst>` にある :ref:`エクスポートされた <syntax-export>` 関数を :ref:`呼び出す <exec-invocation>` ときに「実行」されます。

実行の振る舞いについては、「プログラムのステート」をモデリングする「抽象マシン」に基づいて定義されています。抽象マシンには、オペランド値や制御構成（constructs）を記録する「スタック」や、グローバルステートを保存する抽象的な「ストア」が含まれます。

インストラクションごとに、プログラムのステートに対する実行の結果を指定するルールが1つずつ存在します。
さらに、モジュールのインスタンス化を記述するルールも存在します。
:ref:`検証 <validation>` では、すべてのルールが以下の「同等な」2つの形式で与えられます。

1. *散文記法（prose）* では、直感的な形式で実行を記述します。
2. *形式的記法（formal notation）* では、数学的形式でルールを記述します。 [#cite-pldi2017]_

.. note::
   散文記法と形式的記法は同等なので、本仕様書を読む上で形式的記法の理解は必須では「ありません」。
   この形式主義によって、プログラミング言語のセマンティクスで広く用いられている簡潔な記法を提供し、数学的証明の対象としてふさわしい形になります。

.. _exec-notation-textual:

散文記法（prose notation）
~~~~~~~~~~~~~~

実行は、:ref:`抽象構文の <syntax>` :ref:`インストラクション <syntax-instr>` ごとに、スタイル化された段階的なルールで記述されます。
以下の記法はこれらのルールを記述するために採用されています。

* 実行ルールでは、:ref:`ストア <store>` :math:`S` が暗黙で与えられていることが仮定されています。

* 実行ルールでは、暗黙の :ref:`スタック <stack>` の存在も仮定されています。スタックは「:ref:`値 <syntax-value>`」「:ref:`ラベル <syntax-label>`」「:ref:`フレーム <syntax-frame>`」をpushまたはpopすることで変更されます。

* 特例のルールでは、スタックがフレームを1つ以上含むことが求められます。
  最も直近のフレームは「カレント」フレームとして参照されます。

* ストアとカレントフレームは、そこにある一部のコンポーネントを「置き換える」ことで改変されます。
  こうした置き換えはグローバルに適用されることが前提となっています。

* あるインストラクションを実行するときに「トラップ」が発生する可能性があります。トラップが発生すると計算全体が中断され、ストアの改変はそれ以上行われなくなります（その他の計算はその後引き続き初期化される可能性があります）。

* あるインストラクションを実行すると、最終的に指定のジャンプ先に「ジャンプ」することもあります。
  ジャンプは、次に実行するインストラクションを定義します。

* 実行では、:ref:`ブロック <syntax-instr-control>` を形成する :ref:`インストラクションシーケンス <syntax-instr-seq>` の実行と終了も行われます。

* :ref:`インストラクションシーケンス <syntax-instr-seq>` は、暗黙で順序どおりに実行されます（トラップやジャンプが派生した場合を除く）。

* ルールの多くには「アサーション（assertion）」が含まれており、プログラムのステートに関する重要な不変量を表現しています。


.. index:: ! reduction rules, configuration, evaluation context
.. _exec-notation:

形式的記法（formal notation）
~~~~~~~~~~~~~~~

.. note::
   本セクションでは、型付けルールを形式的に指定するための記法について簡単に説明するにとどめます。このテーマに関心のある方は、関連する教科書でさらに詳しい紹介文を読めます [#cite-tapl]_

形式的な型付けルールでは、型システムを指定するのにある標準的なアプローチを用いており、それらを「推論規則（deduction rule）」という形に落とし込んでいます。

あらゆるルールは以下の一般形式を持ちます。

.. math::
   \X{設定} \quad\stepto\quad \X{設定}

「設定（configuration）」は、プログラムのステートを統語論的に記述したものです。
1つのルールは、実行の1つの「ステップ（step）」に対応します。
与えられた設定に適用される最大でひとつの「還元規則（reduction rule）」が存在する限り、還元（すなわち実行）は「決定論的（deterministic）」になります。
WebAssemblyにはこれに関する例外はほとんど存在せず、本仕様書ではこれらのルールは明示的に記述されています。

WebAssemblyにおける1つの構成は、典型的にはタプル（tuple） :math:`(S; F; \instr^\ast)` になります。タプルは「現在の :ref:`ストア <store>` :math:`S`」「現在の関数の :ref:`コールフレーム <frame>` :math:`F`」「実行される :ref:`インストラクション <syntax-instr>` のシーケンス」からなります。
（より正確な定義については :ref:`後述 <syntax-config>` します）

記述の不要な煩雑さを避けるため、ストア :math:`S` やフレーム :math:`F` は還元規則で言及されない場合は省略します。

:ref:`スタック <stack>` については独立した表現形式はありません。
その代わり、その構成のインストラクションシーケンスの一部として簡便な方法で表現されます。
特に、:ref:`値 <syntax-val>` は |CONST| インストラクションと一致するよう定義されており、|CONST| インストラクションのシーケンスは、所定のサイズまで成長するオペランド「スタック」として解釈できます。

.. note::
   たとえば、:math:`\I32.\ADD` インストラクションの :ref:`還元規則 <exec-binop>` は以下のように与えられます。

   .. math::
      (\I32.\CONST~n_1)~(\I32.\CONST~n_2)~\I32.\ADD \quad\stepto\quad (\I32.\CONST~(n_1 + n_2) \mod 2^{32})

   このルールによって、2つの |CONST| インストラクションと |ADD| インストラクション自身がインストラクションストリームから削除されて1つの新しい |CONST| インストラクションに置き換わります。
   これは、スタックから2つの値をpopして、結果をスタックにpushしたと解釈できます。

   結果が生じない場合、インストラクションは以下の空シーケンスに還元されます。

   .. math::
      \NOP \quad\stepto\quad \epsilon

:ref:`ラベル <label>` と :ref:`フレーム <frame>` は、あるインストラクションシーケンスの一部として同じように :ref:`定義 <syntax-instr-admin>` されます。

還元の順序は、ある適切な :ref:`評価コンテキスト <syntax-ctxt-eval>` の定義によって決定されます。


適用する還元規則が他にない場合、還元は「終了（terminate）」します。
WebAssemblyの :ref:`型システム <type-system>` の :ref:`健全性 <soundness>` は、元となるインストラクションシーケンスが |CONST| インストラクションの1つのシーケンスにまで還元される（これは、得られたオペランドスタックの :ref:`値 <syntax-val>` として解釈できます）か、さもなければ :ref:`トラップ <syntax-trap>` が発生することを保証します。

.. note::
   たとえば以下のインストラクションシーケンスがあるとします。

   .. math::
      (\F64.\CONST~x_1)~(\F64.\CONST~x_2)~\F64.\NEG~(\F64.\CONST~x_3)~\F64.\ADD~\F64.\MUL

   上はステップを3つ進んだところで終了します。

   .. math::
      \begin{array}{ll}
      & (\F64.\CONST~x_1)~(\F64.\CONST~x_2)~\F64.\NEG~(\F64.\CONST~x_3)~\F64.\ADD~\F64.\MUL \\
      \stepto & (\F64.\CONST~x_1)~(\F64.\CONST~x_4)~(\F64.\CONST~x_3)~\F64.\ADD~\F64.\MUL \\
      \stepto & (\F64.\CONST~x_1)~(\F64.\CONST~x_5)~\F64.\MUL \\
      \stepto & (\F64.\CONST~x_6) \\
      \end{array}

   ただし、:math:`x_4 = -x_2` かつ :math:`x_5 = -x_2 + x_3` かつ :math:`x_6 = x_1 \cdot (-x_2 + x_3)`。

.. [#cite-pldi2017]
   このセマンティクスは以下の記事から派生したものです。
   Andreas Haas, Andreas Rossberg, Derek Schuff, Ben Titzer, Dan Gohman, Luke Wagner, Alon Zakai, JF Bastien, Michael Holman. |PLDI2017|_. Proceedings of the 38th ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI 2017). ACM 2017.

.. [#cite-tapl]
   例: Benjamin Pierce. |TAPL|_. The MIT Press 2002
