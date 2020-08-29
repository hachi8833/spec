.. index:: ! implementation limitations, implementation
.. _impl:

実装制限
--------------------------

実装では、あるWebAssemblyのモジュールや実行の側面に対して多くの追加制約を課すのが普通です。
こうした制限は以下によって生じることがあります。

* 物理リソースの制限
* エンベダーやえんべダーの環境が課す制約
* 選択した実装戦略の制限

本セクションでは、許容されている制限をリストアップします。
制約は数値の制限という形式を取りますが、「最小限必要とされる制限の個数」も「具体的な固定数を前提とする制限」もありません。
しかしあらゆる実装は、共通のアプリケーションを有効できる「十分に多い」制限を備えることが期待されます。

.. note::
   WebAssemblyに適合する実装では、個別の「機能」を省略することは許されません。
   ただし、WebAssembly機能のサブセットが今後指定される可能性もあります。


構文上の制限
~~~~~~~~~~~~~~~~

.. index:: abstract syntax, module, type, function, table, memory, global, element, data, import, export, parameter, result, local, structured control instruction, instruction, name, Unicode, character
.. _impl-syntax:

構造（structure）
.........

実装は、あるモジュールに対して以下の制約を課すことができます。

* ひとつの :ref:`モジュール <syntax-module>` 内の :ref:`型 <syntax-type>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内（インポートを含む）の :ref:`関数 <syntax-func>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内（インポートを含む）の :ref:`テーブル <syntax-table>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内（インポートを含む）の :ref:`メモリー <syntax-mem>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内（インポートを含む）の :ref:`グローバル <syntax-global>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内の :ref:`要素セグメント <syntax-elem>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内の :ref:`データセグメント <syntax-data>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内の :ref:`インポート <syntax-import>` の個数
* ひとつの :ref:`モジュール <syntax-module>` 内の :ref:`エクスポート <syntax-export>` の個数
* ひとつの :ref:`関数型 <syntax-functype>` 内のパラメーターの個数
* ひとつの :ref:`関数型 <syntax-functype>` 内の結果の個数
* ひとつの :ref:`ブロック型 <syntax-blocktype>` 内のパラメーターの個数
* ひとつの :ref:`ブロック型 <syntax-blocktype>` 内の結果の個数
* ひとつの :ref:`関数 <syntax-func>` 内の :ref:`ローカル <syntax-local>` の個数
* ひとつの :ref:`関数 <syntax-func>` 本体のサイズ
* ひとつの :ref:`構造化制御インストラクション <syntax-instr-control>` のサイズ
* ひとつの :ref:`関数 <syntax-func>` 内の :ref:`構造化制御インストラクション <syntax-instr-control>` の個数
* :ref:`構造化制御インストラクション <syntax-instr-control>` のネストの深さ
* ひとつの |brtable| インストラクション内の :ref:`ラベルインデックス <syntax-labelidx>` の個数
* ひとつの :ref:`要素セグメント <syntax-elem>` の長さ
* ひとつの :ref:`データセグメント <syntax-data>` の長さ
* ひとつの :ref:`名前 <syntax-name>` の長さ
* ひとつの :ref:`名前 <syntax-name>` で使う :ref:`文字 <syntax-char>` の範囲

ある実装に与えられたモジュールがこの制限を超えた場合、実装はそのモジュールの「:ref:`検証 <valid>`」「コンパイル」「:ref:`インスタンス化 <exec-instantiation>`」を拒否してエンベダー固有のエラーにすることができます。

.. note::
   |Unicode|_ をサポートしない制約付き環境にある :ref:`エンベダー <embedder>` は、上の最後の項目を用いて :ref:`インポート <syntax-import>` や :ref:`エクスポート <syntax-export>` の名前を |ASCII|_ などの共通サブセットに制限できます。


.. index:: binary format, module, section, function, code
.. _impl-binary:

バイナリ形式（binary format）
.............

:ref:`バイナリ形式 <binary>` で与えられるモジュールについては、以下の制約が追加で課される可能性があります。

* ひとつの :ref:`モジュール <binary-module>` のサイズ
* :ref:`セクション <binary-section>` のサイズ
* 個別の関数の :ref:`コード <binary-code>` サイズ
* :ref:`セクション <binary-section>` の個数


.. index:: text format, source text, token, identifier, character, unicode
.. _impl-text:

テキスト形式（text format）
...........

:ref:`テキスト形式 <text>` で与えられるモジュールについては、以下の制約が追加で課される可能性があります。

* :ref:`ソーステキスト <source>` のサイズ
* 構文的要素のサイズ
* 個別の :ref:`トークン <text-token>` のサイズ
* :ref:`折りたたみインストラクション <text-foldedinstr>` のネストの深さ
* シンボル :ref:`識別子 <text-id>` の長さ
* :ref:`ソーステキスト <source>` 内で許容されるリテラル :ref:`文字 <text-char>` の範囲


.. index:: validation, function
.. _impl-valid:

検証（validation）
~~~~~~~~~~

実装は、個別の :ref:`関数 <syntax-func>` の :ref:`検証 <valid>` を、それらが初めて :ref:`呼び出される <exec-invoke>` まで先延ばしできます。

ある関数が無効であることが判明した場合、その関数の呼び出しと、以後の同じ関数の呼び出しはすべて  :ref:`トラップ <trap>` されます。

.. note::
   これは、関数をインタプリタまたはJIT（just-in-time）コンパイラとして実装できるようにするための仕様です。
   関数の本体冒頭が実行されるまでに関数を完全に検証しなければならない点は変わりません。


.. index:: execution, module instance, function instance, table instance, memory instance, global instance, allocation, frame, label, value
.. _impl-exec:

実行（execution）
~~~~~~~~~

あるWebAssemblyプログラムの :ref:`実行中 <exec>` に、以下の制約が課される可能性があります。

* アロケーションされる :ref:`モジュールインスタンス <syntax-moduleinst>` の個数
* アロケーションされる :ref:`関数インスタンス <syntax-funcinst>` の個数
* アロケーションされる :ref:`テーブルインスタンス <syntax-tableinst>` の個数
* アロケーションされる :ref:`メモリーインスタンス <syntax-meminst>` の個数
* アロケーションされる :ref:`グローバルインスタンス <syntax-globalinst>` の個数
* ひとつの :ref:`テーブルインスタンス <syntax-tableinst>` のサイズ
* ひとつの :ref:`メモリーインスタンス <syntax-meminst>` のサイズ
* :ref:`スタック <stack>` 上の :ref:`フレーム <syntax-frame>` の個数
* :ref:`スタック <stack>` 上の :ref:`ラベル <syntax-label>` の個数
* :ref:`スタック <stack>` 上の :ref:`値 <syntax-val>` の個数

計算の実行中に実装のランタイム制約を超えると、計算が終了してエンベダー固有のエラーを呼び出し側コードに通知する可能性があります。

上述の制限の一部はインスタンス化中に既に検証されているる可能性もあり、その場合実装は制限超過を :ref:`構文上の制限 <impl-syntax>` と同じ形で通知できます。

.. note::
   具体的な制限は固定されないのが普通ですが、何らかの事情に依存または相互依存したり、時間の経過とともに変化したり、他の実装やエンベダー固有の事情やイベントに依存する可能性もあります。

