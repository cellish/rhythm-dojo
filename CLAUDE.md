# CLAUDE.md — リズム道場

現状は `STATUS.md`、未完了の作業は `PLAN.md`、中身の一覧は `docs/INVENTORY.md`。

キーボード（またはタップ）で叩いてリズム感を練習するブラウザ用アプリ。**`index.html` 1ファイル**（HTML・CSS・JS をすべて含む。ビルドなし、外部読み込みなし）。

## 基本方針

- 画面の文言・コメント・報告・コミットは日本語。言葉は平易に。
- 外部のライブラリ・フォント・録音素材は使わない。音はすべて Web Audio API で合成する。加えるときは `THIRD_PARTY_NOTICES.md` を更新する。
- 著作権表示は「All rights reserved」（オープンソースにしない）。`LICENSE` を参照。
- 入力はキーボード中心（スマホは画面のパッド）。iPhone のマナーモードでは音なしでよい。
- 練習は易しい順のステップで進め、80点以上でクリア。
- 結果は文章を少なく、カード・メーター・グラフで見せる。詳しい説明は「くわしい説明を見る」の中へ。
- 新しいモードには、易しい順の練習ステップ（`STEPS`）を必ず付ける。
- 区切りや切り替えの前には、画面に予告（カウントダウン）を出す。

## コードの構成（`index.html` の `<script>`）

`/* ===== 見出し ===== */` のコメントで区切ってある。

| 区画 | 内容 |
|---|---|
| Audio・楽器音・叩いた音 | `ac()`, `playSound(t, name, f)`, 合成音（`pluck` ベース/ギター, `metal` シンバル, `piano`, `kick`/`snare`/`conga`, `percussion` 叩いた音） |
| Plan helpers | `Plan()`, `addBar()`, `winOf()` |
| Modes / ジャズ編 / クラシック編 / ポップス編 | `MODES`, `JAZZ_MODES`, `CLASSIC_MODES`, `POP_MODES`。伴奏の部品 `jazzBar`, `pianoBar`, `groove` |
| ステップ | `STEPS.<モードid>`（`ST(題, 補足, 設定値)`）。各編の配列より**前**に定義すること |
| UI | 編・タブ・設定欄・ステップ帯（`renderSetup`, `renderSteps`, `applyStep`）。`EDITIONS` |
| Session engine | `startSession`, `schedule`（先読み 0.15 秒）, `draw`, `tap`, 指揮の玉 `drawCond` |
| Evaluation / Results | `evaluate`, `stats`, `slopeOf`, `renderResult`（カード `K`、ひとこと、メーター、`groupChart`、`drawChart`） |
| その他 | テンポ変化の聴き分け（クイズ）、遅延補正、キー操作 |

### モードの書き方

```js
{ id:'p_xxx', steps:STEPS.p_xxx, name:'タブ名', title:'見出し', help:'練習中の説明', desc:'設定画面の説明',
  opts:[BPM_OPT(100), {key, label, type:'select'|'check'|'bpm', def, choices, note}],
  build(o){ /* Plan を返す */ } }
```

`build` が返す Plan：`events`（鳴らす音 `{t, sound, f}`）、`targets`（判定する打鍵 `{t, layer, win, group, ...}`）、`segments`（画面の文言 `{t, text, sub, cls:'warn'|'change'}`）、`vis`（拍の表示）、`rows`、`layers`、`dur`、`shade`/`bands`（グラフの帯）、必要なら `onTick`（途中で出題を作る）・`arcs`（指揮）。時刻はすべて開始からの秒。

- 判定の窓 `win` は、隣の目標との間隔の半分より小さくする（目安 `0.45 × 間隔`、上限 0.1〜0.15 秒）。
- モード固有の結果（カード・コメント）は `renderResult` の `if(mode.id === ...)` に足す。

## 確かめ方

1. 構文：`<script>` の中身を取り出して `node --check`。
2. 全モード・全ステップの組み立て：script の前半（UI より前）を node で `eval` し、各 `STEPS` の設定値が選択肢にあるか、`build` がエラーなく `targets` を返すかを調べる（`localStorage`/`document`/`window` はダミーでよい）。
3. 表示：ヘッドレス Chrome でスクリーンショット（`--virtual-time-budget`）。結果画面は `sess.taps` に作った打鍵を入れて `finish()` を呼ぶ。音の鳴りは目で確かめられないので、変更したら利用者に聴いてもらう。

## 公開

- GitHub：`cellish/rhythm-dojo`（`main`）。push は利用者が頼んだときだけ。
- Claude のアーティファクト版（slug `dc17a19e-e8ba-4713-b440-b0e3a386d0fb`）もある。更新は頼まれたときだけ。

## ドキュメント更新
- 作業の終わりに STATUS.md を上書き更新（50行以内、updated を更新）
- 完了した作業は PLAN.md から削除し、STATUS の「実装済み」に要約
- 外部素材を追加・削除したら THIRD_PARTY_NOTICES.md を更新
- OS・ブラウザ対応が変わったら STATUS の os: と対応状況を更新（未確認を「対応」と書かない）
- 詳細方針は道場シリーズ親フォルダの DOCS-POLICY.md を参照
