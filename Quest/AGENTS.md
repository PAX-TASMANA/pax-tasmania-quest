# AGENTS.md — Pax Tasmana! 開発のための作業ルール

このリポジトリで作業するAIエージェント（Cursor等）向けの規約。まず `README.md` を読んで全体像・現状・次タスクを把握すること。

## プロジェクトの性質
- **単一の自己完結HTML**: `pax-tasmania-quest.html` にHTML/CSS/JSが全部入っている。フレームワーク・ビルド・依存パッケージなし。
- 画像は `game-assets/*.png` を相対パスで読む。プレビューは **Live Server** か `python3 -m http.server` で（`file://` 直開きは画像が出ないことがある）。

## 作業の約束
1. **差分（ターゲット）編集**で直す。ファイル全文を作り直さない（2000行近くあり、巻き込み事故・トランケーションの元）。
2. **既存機能を壊さない**: タイトル / マップモード / クエスト / 名所・移動（ワープ） / 屋内シーン / バトル / 仲間化 / エサ袋。変更後に主要動線を一通り確認する。
3. コードは1つのIIFE内。**関数名・変数名で検索**して該当箇所を特定する（行番号に依存しない）。
4. 変更は必ず**実ブラウザ（Live Server）で動作確認**してから「完了」とする。コンソールエラー0を確認。
5. **日本語で対話**する。
6. **大きな作り替えは段階的に。着手前にユーザーへ方針を相談**する（一気に仕様を確定させず、フェーズを区切って1つずつ固める進め方を好む）。

## よく使う入口（詳細は README のアーキテクチャ早見）
- 画面遷移: `screen` / `overlay` / `applyUI()`
- 移動: `loop()` / `nextDir()` / `tryStep()` / `onArrive()` / `moveTarget`
- 描画: `renderField` / `renderInterior` / `renderBattle` / `drawSprite` / `drawMonster`(`MON_PAL`/`shape`)
- バトル: `startBattle` / `battleAdvance` / `chooseCmd` / `decideAfterHero` / `MOBS`
- 仲間化: `searchHere` / `HABITAT_TERR` / `doTame` / `bag`
- データ: `PLACES`（16名所）/ `DEFAULT_SKILLS`（必殺技）

## 次タスク（README の TODO 表参照・要約）

優先: **D**（2e ラスボス＋エンディング）→ **C バックログ**（名所ごとの屋内/風景バリエーション）。

**ボツ**: フェリー/海戦、バス移動。

## 注意
- 開発用の **`b`キー**（フィールドで即ダミーバトル）はリリース前に外す。
