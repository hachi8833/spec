.. index:: lexical format
.. _text-lexical:

レキシカル形式
--------------


.. index:: ! character, Unicode, ASCII, character, ! source text
   pair: text format; character
.. _source:
.. _text-source:
.. _text-char:

文字（character）
~~~~~~~~~~

テキスト形式は、「文字」のシーケンスでできている「ソーステキスト」に意味を与えます。
この文字は、有効な |Unicode|_ （セクション2.4）「スカラー値」で表現されると仮定します。

.. math::
   \begin{array}{llll}
   \production{source} & \Tsource &::=&
     \Tchar^\ast \\
   \production{character} & \Tchar &::=&
     \unicode{00} ~|~ \dots ~|~ \unicode{D7FF} ~|~ \unicode{E000} ~|~ \dots ~|~ \unicode{10FFFF} \\
   \end{array}

.. note::
   ソーステキストの :ref:`コメント <text-comment>` や :ref:`文字列 <text-string>` には任意のUnicode文字が含まれる可能性がありますが、以後説明する文法ではUnicodeの7ビット |ASCII|_ サブセットだけを用います。

.. index:: ! token, ! keyword, character, white space, comment, source text
   single: text format; token
.. _text-keyword:
.. _text-reserved:
.. _text-token:

トークン（token）
~~~~~~

ソーステキスト内の文字ストリームは、以下の文法で定義されるように、左から右へ「トークン」のシーケンスに分割されます。

.. math::
   \begin{array}{llll}
   \production{token} & \Ttoken &::=&
     \Tkeyword ~|~ \TuN ~|~ \TsN ~|~ \TfN ~|~ \Tstring ~|~ \Tid ~|~
     \text{(} ~|~ \text{)} ~|~ \Treserved \\
   \production{keyword} & \Tkeyword &::=&
     (\text{a} ~|~ \dots ~|~ \text{z})~\Tidchar^\ast
     \qquad (\mbox{その文法内でリテラル終端として出現する場合}) \\
   \production{reserved} & \Treserved &::=&
     \Tidchar^+ \\
   \end{array}

トークンは、入力文字ストリームから「最長一致」ルールに基づいて形成されます。
つまり次のトークンは、常に上述の構文によって認識可能な最長文字シーケンスで構成されます。
トークンは :ref:`ホワイトスペース <text-space>` で区切られることがありますが、文字列以外ではトークンにホワイトスペースを含めることはできません。

「キーワード」トークンのセットは、リテラル形式におけるあらゆる :ref:`終端シンボル <text-grammar>` の出現（本章の :ref:`構文的 <text-syntactic>` 生成物の :math:`\text{keyword}` など）によって暗黙に定義されます。

他のどのカテゴリにも該当しないトークンは、すべて「予約済み（reserved）」とみなされ、ソーステキストでは利用できません。

.. note::
   予約済みトークンのセットを定義する影響は、すべてのトークンは「丸かっこ」か「:ref:`ホワイトスペース <text-space>`」で区切られなければならないという点です。
   たとえば、:math:`\text{0\$x}` が単一の予約済みトークンだとします。
   その場合、これは「:math:`\text{0}`」「:math:`\text{\$x}`」という2つのトークンとしては認識されず、禁止されます。
   トークン化におけるこの性質は、予約済みトークンの定義がその他のトークンクラスとオーバーラップしても影響されません。

.. index:: ! white space, character, ASCII
   single: text format; white space
.. _text-format:
.. _text-space:

ホワイトスペース（white space）
~~~~~~~~~~~

「ホワイトスペース」とは、「リテラルのスペース文字」「書式文字（formatting character）」または「:ref:`コメント <text-comment>`」のシーケンスです。
利用が許されている書式文字は、 |ASCII|_ の「書式設定文字（format effector）」のサブセット、すなわち「水平タブ文字（:math:`\unicode{09}`)）」「LF（:math:`\unicode{0A}`）」「CR（:math:`\unicode{0D}`）」です。



.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{white space} & \Tspace &::=&
     (\text{~~} ~|~ \Tformat ~|~ \Tcomment)^\ast \\
   \production{format} & \Tformat &::=&
     \unicode{09} ~|~ \unicode{0A} ~|~ \unicode{0D} \\
   \end{array}

ホワイトスペースは :ref:`トークン <text-token>` の分離にのみ関連し、それ以外では無視されます。

.. index:: ! comment, character
   single: text format; comment
.. _text-comment:

コメント（comment）
~~~~~~~~

「コメント」は、ダブルセミコロン :math:`\Tcommentd` から行末までの「行コメント」か、:math:`\Tcommentl \dots \Tcommentr` という区切り文字で囲まれた「ブロックコメント」のいずれかです。
ブロックコメントはネスト可能です。

.. math::
   \begin{array}{llclll@{\qquad\qquad}l}
   \production{comment} & \Tcomment &::=&
     \Tlinecomment ~|~ \Tblockcomment \\
   \production{line comment} & \Tlinecomment &::=&
     \Tcommentd~~\Tlinechar^\ast~~(\unicode{0A} ~|~ \T{eof}) \\
   \production{line character} & \Tlinechar &::=&
     c{:}\Tchar & (c \neq \unicode{0A} ~\mbox{の場合}) \\
   \production{block comment} & \Tblockcomment &::=&
     \Tcommentl~~\Tblockchar^\ast~~\Tcommentr\\
   \production{block character} & \Tblockchar &::=&
     c{:}\Tchar & (c \neq \text{;} \wedge c \neq \text{(}~\mbox{の場合}) \\ &&|&
     \text{;} & (~\mbox{次の文字が}~\text{)}~\mbox{でない場合}) \\ &&|&
     \text{(} & (~\mbox{次の文字が}~\text{;}~\mbox{でない場合}) \\ &&|&
     \Tblockcomment \\
   \end{array}

上のダミートークン :math:`\T{eof}` は入力の終了を示します。
|Tblockchar| の生成における「look-ahead（先読み）」制約は文法のあいまいさを解消し、ブロックコメントのデリミタについて正しく囲む用法だけを許すようにします。

.. note::
   コメントの内側では、任意の書式設定文字や制御文字が許されます。
