# Pax Tasmana! - The Quest -

タスマニアを舞台にした、ドラクエ/FF風のブラウザRPG。**単一の自己完結HTML**（`pax-tasmania-quest.html`）で動く。タスマニアの実在16名所を巡り、外来の悪い力で「魔物化」した在来動物を倒して元に戻し、各動物を生息地で正しいエサを与えて仲間にし、最後は巨大ラスボスを倒して友達にする、というモンスター仲間化RPG。

> このREADMEは、開発を引き継ぐ人/AIが最初に読む前提のメモです。日本語で進めます。

---

## 実行方法（プレビュー）

ローカルにサーバを立てて開く（`game-assets/` の画像を相対パスで読むため、`file://` で直接開くと画像が出ないことがある）。

```bash
cd "PAX TASMANA!/Quest"
python3 -m http.server 8000
# → ブラウザで http://localhost:8000/pax-tasmania-quest.html
```

または VS Code / Cursor の **Live Server** 拡張で `Quest/pax-tasmania-quest.html` を右クリック →「Open with Live Server」。

スマホ風の操作（dパッド/タップ）も実装済みなので、デスクトップ・モバイル両方で動く。

---

## ファイル構成

- `pax-tasmania-quest.html` … **本体（これが全て）**。HTML/CSS/JSが1ファイルに同梱。
- `game-assets/*.png` … PAXキャラ＝タスマニア動物10体のスプライト（透過・トリミング済み）。
  `wombi / harri / platti / spiki / possi / thylas / dotti / spotti / poppi / fairi`
- `../index.html` … ブランド用ランディング（親フォルダ。本ゲームとは別）。
- 親フォルダの `PAX ...`, `Resources` 等 … 素材・ロゴ等の元データ。ゲーム実行には不要。

---

## アーキテクチャ早見

> ⚠️ 行番号は変動するので**関数名・変数名で検索**して参照すること。コードは1つの即時関数（IIFE）内にまとまっている。

### 画面ステート
- `screen` … `'title' | 'modeselect' | 'mapmode' | 'field' | 'interior' | 'battle'`
- `overlay` … `'' | 'dialog' | 'places' | 'tameoffer'`（モーダル系）
- `started` … クエスト開始フラグ
- `player` … `{tx,ty,fromX,fromY,dir,moving,t,char,faceRight}`（タイル座標＋移動補間）
- `party` … 仲間の配列（`[0]`が主人公）。`hist`/`faceMem` が隊列追従用
- `bag` … エサ袋 `{肉,草,きのこ,魚,虫}`（HUDの🍖ピル `bagWrap`/`bagN`）
- `mobs` … フィールド徘徊中の魔物シンボル配列、`moveTarget` … タップ移動の目的タイル

### 主要関数
- `generateWorld()` … タスマニア多角形をラスタライズしてタイル生成、名所配置
- `loop(ts)` … メインループ。移動処理＋各画面の描画を振り分け
  - `nextDir()` … 次に進む向き（押下キー優先→`moveTarget`へ貪欲移動）
  - `tryStep(dir)` … 1タイル移動開始（`walkable`判定）。`onArrive()` … 到着時の処理（名所に入る/出口で出る）
- `renderField()` / `renderInterior()` / `renderBattle()` … 各画面描画
- `drawSprite(id,cx,feet,th,hop,faceRight)` … 動物スプライト描画（向き反転対応）
- `drawMonster(shape,cx,feet,sz,shake)` … 魔物描画。`MON_PAL`（配色）＋`shape`で `bird/kingfisher/snake/lizard/frog/roo/fungus` を描き分け
- バトル: `startBattle(enemy,mobRef)` / `battleAdvance` / `chooseCmd` / `chooseFood` / `decideAfterHero` / `enemyTurn` / `endBattle`。メッセージはキュー方式（`bstep`）
- 仲間化: `searchHere()`（名所周辺で `調べる`＝Space すると隠れた動物を発見）＋ `HABITAT_TERR`（動物→地形 forest/grass/coast/water）＋ `doTame()`
- 移動手段: `doWarp` / `startTravel` / `openPlacesMenu` / `buildPlacesMenu`（名所・移動メニューからワープ）

### データ
- `PLACES` … 16名所。各 `{id,jp,en,lon,lat,desc,mei,url,animal?,food?,habitat?}`。`url` は Discover Tasmania の該当ページ（新タブで開く）。
- `MOBS` … 魔物化した在来動物7種（`{name,orig,food,maxhp,atk,shape}`）。撃破で `orig`(元の動物)に戻り、`food` をドロップ。
- `DEFAULT_SKILLS` / `SKILLS` … 10体の必殺技マスタ（`name` + 演出 `fx`）。⚡ボタンで編集 UI。詳細は `skills-table.md`

### 操作
- 移動: 矢印/WASD・dパッド・**フィールド/屋内をタップ→その場所へ自動で歩く**
- 調べる/決定: Space・決定ボタン / 名所・移動: 📖メニュー（J キー）

---

## 現状（実装・確認済み）

- **フェーズ1（世界）**: タイトル、60×54マップ、実在16名所、屋内シーン（フクロウのヒント＋案内板→Discover Tasmania）、名所・移動メニュー（ワープ）、隊列追従、マップモード。
- **フェーズ2**:
  - **2a バトル基盤** … FF風サイドビュー・ターン制。コマンド こうげき/エサ/にげる、HP制。
  - **2b 魔物エンカウント** … シンボル接触式。撃破で在来動物に戻して解放＋エサdrop。エサ袋(HUD)。
  - **2c 仲間化** … 生息地の正しい地形を `調べる` と隠れた動物を発見→正しいエサで加入。Possiはきのこで幻覚演出(`halluUntil`)。
  - **仕上げ修正** … 歩きを1タイルずつ滑らかに＋タップ移動、出入口クリーン化、バトルの白フラッシュ除去＋レイアウト調整、魔物の見た目を種類ごとに描き分け。
  - **2d 海戦（試作・撤回済）** … フェリー/海戦はボツ。**魚は陸の魔物ドロップ（案1）** に戻した。

---

## 次にやること（TODO）

> 優先度の目安: **A 体感・操作** → **B バトル** → **C ワールド見た目** → **D コンテンツ完成** → **R リリース前**

### A. 体感・操作（優先）

| ID | 内容 | メモ |
|----|------|------|
| A1 | ~~**カメラを滑らかに**~~ ✅ | `cam` + `updateCamera(dt)` で補間追従 |
| A2 | ~~**隊列スプライト統一**~~ ✅ | `SPR_H=52` で主人公・仲間・バトル共通 |
| A3 | ~~**全員ぴょんぴょん（No Exception）**~~ ✅ | `partyHop(k)` — 移動中ホップ＋待機中もバウンス |
| A4 | ~~**バス**~~ ❌ 却下 | 機能ごと削除。名所・移動はワープのみ |

### B. バトル

| ID | 内容 | メモ |
|----|------|------|
| B1 | ~~**攻撃アニメ**~~ ✅ | `drawBattleFx` — ウンコ/スライド/バースト/斬撃 |
| B2 | ~~**仲間も攻撃参加**~~ ✅ | `partyStrike()` で全員連続攻撃 |
| B3 | ~~**必殺技名の文案修正**~~ ✅ | `DEFAULT_SKILLS` 表 + ⚡ 編集 UI + `skills-table.md` |

### C. ワールド・名所の見た目

| ID | 内容 | メモ |
|----|------|------|
| C1 | ~~**街は街らしく**~~ ✅ | フィールド `drawPlaceMarker` + 屋外 `drawOutdoorDecor` |
| C2 | ~~**名所ごとにそれらしい作り**~~ ✅ | `PLACE_KIND` / `renderOutdoor()` — town/port/park/historic/island |
| C3 | ~~**屋内テンプレ見直し**~~ ✅ | Devonport のみ `interior`、他15名所は `outdoor` |

### D. コンテンツ完成

| ID | 内容 | メモ |
|----|------|------|
| D1 | **2e ラスボス＋エンディング**（作業中） | 16名所制覇後・拠点フクロウから挑戦。**なかま最低1人**必須（`canMeetLastBoss`）。開発用 `l` キー |
| D2 | **魚の入手ルート** | ✅ **案1 決定**: 陸の魔物ドロップ（主に《魔物カワセミ》→魚。スタンリー方面の浜付近） |
| D3 | **フェリー/海戦コード整理** | ✅ 撤回済（2026-06） |

### R. リリース前

| ID | 内容 |
|----|------|
| R1 | 開発用 **`b` キー**（即ダミーバトル）を外す |
| R2 | 主要動線の最終動作確認 |

### ボツ・保留

- ~~Spirit of Tasmania / フェリー出航 / 海戦~~ … **採用しない**

---

## 既知メモ / 注意

- **テスト用 `b` キー**: フィールドでランダムな魔物と即バトル開始（開発用）。**リリース前に外す**こと。
- 必殺技は `DEFAULT_SKILLS`（⚡ で編集）、Possiの幻覚は `halluUntil`（renderField末尾の演出）。
- 単一HTMLなので、**全文を作り直さず該当箇所だけ差分編集**するのが安全（巻き込み事故防止）。
- 変更したら**実ブラウザ（Live Server）で動作確認**してから完了とする。

---

## バージョン管理（推奨）

まだgit管理していなければ:

```bash
cd "PAX TASMANA!"
git init
printf "node_modules/\n.DS_Store\n" > .gitignore
git add Quest/pax-tasmania-quest.html Quest/game-assets Quest/README.md Quest/AGENTS.md .cursorrules
git commit -m "Pax Tasmana: フェーズ2c＋仕上げ修正までの状態"
```
