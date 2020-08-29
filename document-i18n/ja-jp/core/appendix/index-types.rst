.. index:: type
.. _index-type:

索引（型）
--------------

========================================  ===========================================  ===============================================================================
カテゴリ                                   コンストラクタ                                  バイナリOpcode
========================================  ===========================================  ===============================================================================
:ref:`型インデックス <syntax-typeidx>`      :math:`x`                                    （|Bs32| または |Bu32| の正の数値）
:ref:`値型 <syntax-valtype>`              |I32|                                        :math:`\hex{7F}` (-1 は |Bs7|)
:ref:`値型 <syntax-valtype>`              |I64|                                        :math:`\hex{7E}` (-2 は |Bs7|)
:ref:`値型 <syntax-valtype>`              |F32|                                        :math:`\hex{7D}` (-3 は |Bs7|)
:ref:`値型 <syntax-valtype>`              |F64|                                        :math:`\hex{7C}` (-4 は |Bs7|)
（予約）                                                                                :math:`\hex{7B}` .. :math:`\hex{71}`
:ref:`要素型 <syntax-elemtype>`           |FUNCREF|                                    :math:`\hex{70}` (-16 は |Bs7|)
（予約）                                                                                :math:`\hex{6F}` .. :math:`\hex{61}`
:ref:`関数型 <syntax-functype>`           :math:`[\valtype^\ast] \to [\valtype^\ast]`  :math:`\hex{60}` (-32 は |Bs7|)
（予約）                                                                                :math:`\hex{5F}` .. :math:`\hex{41}`
:ref:`結果型 <syntax-resulttype>`         :math:`[\epsilon]`                           :math:`\hex{40}` (-64 は |Bs7|)
:ref:`テーブル型 <syntax-tabletype>`       :math:`\limits~\elemtype`                    （なし）
:ref:`メモリー型 <syntax-memtype>`         :math:`\limits`                              （なし）
:ref:`グローバル型 <syntax-globaltype>`     :math:`\mut~\valtype`                       （なし）
========================================  ===========================================  ===============================================================================
