.. _index-rules:

索引（セマンティックルール）
-----------------------


.. index:: validation
.. _index-valid:

静的構成体の型付け
~~~~~~~~~~~~~~~~~~~~~~~~~~~

=========================================================  ===============================================================================
構成体                                                        判定
=========================================================  ===============================================================================
:ref:`制限 <valid-limits>`                                   :math:`\vdashlimits \limits : k`
:ref:`関数型 <valid-functype>`                                :math:`\vdashfunctype \functype \ok`
:ref:`ブロック型 <valid-blocktype>`                            :math:`\vdashblocktype \blocktype \ok`
:ref:`テーブル型 <valid-tabletype>`                            :math:`\vdashtabletype \tabletype \ok`
:ref:`メモリー型 <valid-memtype>`                              :math:`\vdashmemtype \memtype \ok`
:ref:`グローバル型 <valid-globaltype>`                         :math:`\vdashglobaltype \globaltype \ok`
:ref:`外部型 <valid-externtype>`                             :math:`\vdashexterntype \externtype \ok`
:ref:`インストラクション <valid-instr>`                        :math:`S;C \vdashinstr \instr : \functype`
:ref:`インストラクションシーケンス <valid-instr-seq>`            :math:`S;C \vdashinstrseq \instr^\ast : \functype`
:ref:`式 <valid-expr>`                                       :math:`C \vdashexpr \expr : \resulttype`
:ref:`関数 <valid-func>`                                     :math:`C \vdashfunc \func : \functype`
:ref:`テーブル <valid-table>`                                :math:`C \vdashtable \table : \tabletype`
:ref:`メモリー <valid-mem>`                                  :math:`C \vdashmem \mem : \memtype`
:ref:`グローバル <valid-global>`                              :math:`C \vdashglobal \global : \globaltype`
:ref:`要素セグメント <valid-elem>`                             :math:`C \vdashelem \elem \ok`
:ref:`データセグメント <valid-data>`                           :math:`C \vdashdata \data \ok`
:ref:`開始関数 <valid-start>`                                :math:`C \vdashstart \start \ok`
:ref:`エクスポート <valid-export>`                            :math:`C \vdashexport \export : \externtype`
:ref:`エクスポートの記述 <valid-exportdesc>`                   :math:`C \vdashexportdesc \exportdesc : \externtype`
:ref:`インポート <valid-import>`                              :math:`C \vdashimport \import : \externtype`
:ref:`インポートの記述 <valid-importdesc>`                     :math:`C \vdashimportdesc \importdesc : \externtype`
:ref:`モジュール <valid-module>`                              :math:`\vdashmodule \module : \externtype^\ast \to \externtype^\ast`
=========================================================  ===============================================================================


.. index:: runtime

ランタイム構成体の型付け
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

=========================================================  ===============================================================================
構成体                                                        判定
=========================================================  ===============================================================================
:ref:`値 <valid-val>`                                       :math:`\vdashval \val : \valtype`
:ref:`結果 <valid-result>`                                  :math:`\vdashresult \result : \resulttype`
:ref:`外部値 <valid-externval>`                             :math:`S \vdashexternval \externval : \externtype`
:ref:`関数インスタンス <valid-funcinst>`                      :math:`S \vdashfuncinst \funcinst : \functype`
:ref:`テーブルインスタンス <valid-tableinst>`                   :math:`S \vdashtableinst \tableinst : \tabletype`
:ref:`メモリーインスタンス <valid-meminst>`                     :math:`S \vdashmeminst \meminst : \memtype`
:ref:`グローバルインスタンス <valid-globalinst>`                :math:`S \vdashglobalinst \globalinst : \globaltype`
:ref:`エクスポートインスタンス <valid-exportinst>`              :math:`S \vdashexportinst \exportinst \ok`
:ref:`モジュールインスタンス <valid-moduleinst>`                :math:`S \vdashmoduleinst \moduleinst : C`
:ref:`ストア <valid-store>`                                   :math:`\vdashstore \store \ok`
:ref:`設定 <valid-config>`                                  :math:`\vdashconfig \config \ok`
:ref:`スレッド <valid-thread>`                                :math:`S;\resulttype^? \vdashthread \thread : \resulttype`
:ref:`フレーム <valid-frame>`                                 :math:`S \vdashframe \frame : C`
=========================================================  ===============================================================================


定数性
~~~~~~~~~~~~

===============================================  ===============================================================================
構成体                                              判定
===============================================  ===============================================================================
:ref:`定数式 <valid-constant>`                     :math:`C \vdashexprconst \expr \const`
:ref:`定数インストラクション <valid-constant>`       :math:`C \vdashinstrconst \instr \const`
===============================================  ===============================================================================


インポートのマッチング
~~~~~~~~~~~~~~~

===============================================  ===============================================================================
構成体                                              判定
===============================================  ===============================================================================
:ref:`制限 <match-limits>`                        :math:`\vdashlimitsmatch \limits_1 \matches \limits_2`
:ref:`外部型 <match-externtype>`                  :math:`\vdashexterntypematch \externtype_1 \matches \externtype_2`
===============================================  ===============================================================================


ストア拡張
~~~~~~~~~~~~~~~

======================================================  ===============================================================================
構成体                                                      判定
======================================================  ===============================================================================
:ref:`関数インスタンス <extend-funcinst>`                   :math:`\vdashfuncinstextends \funcinst_1 \extendsto \funcinst_2`
:ref:`テーブルインスタンス <extend-tableinst>`               :math:`\vdashtableinstextends \tableinst_1 \extendsto \tableinst_2`
:ref:`メモリーインスタンス <extend-meminst>`                 :math:`\vdashmeminstextends \meminst_1 \extendsto \meminst_2`
:ref:`グローバルインスタンス <extend-globalinst>`            :math:`\vdashglobalinstextends \globalinst_1 \extendsto \globalinst_2`
:ref:`ストア <extend-store>`                              :math:`\vdashstoreextends \store_1 \extendsto \store_2`
======================================================  ===============================================================================


実行
~~~~~~~~~

===============================================  ===============================================================================
構成体                                              判定
===============================================  ===============================================================================
:ref:`インストラクション <exec-instr>`                :math:`S;F;\instr^\ast \stepto S';F';{\instr'}^\ast`
:ref:`式 <exec-expr>`                              :math:`S;F;\expr \stepto  S';F';\expr'`
===============================================  ===============================================================================
