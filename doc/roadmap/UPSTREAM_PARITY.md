# 本家追随ロードマップ

> 基準日: 2026-10-16。本家 `tomiya7688/Kadoka-shougi-ai` のmain（確認時 `4e74c6648c3a5c64633eb1451c766ca60329ba9f`）と現存するソース／ドキュメントを調査した初期棚卸し。状況は更新され得る。

## 「本家に追いつく」の意味

- **実装済み基盤への追随**: 本家mainに存在する機能を、3D版のルールに合うよう移植し、3D用テストを通す。
- **計画された機能への追随**: 本家のIssues/READMEには存在するが、mainの実装としては未確認のAI・UI・学習機能を3Dでも段階的に実装する。
- 文書・Issueが存在するだけで実装済みとは数えない。3Dのルール検証・動作するAPI・自動テストの3点が揃って初めて到達とする。

## 本家の状態と3D版の差分

| 分野 | 本家mainで確認できるもの | 3D版の対応方針 | 分類 |
| --- | --- | --- | --- |
| ビルド・CI | C++20 / CMake / Linux・Windows CI / CTest、検証ツール | まず最小C++20 CIとルールテストの土台 | 基盤追随 |
| Core | Position、81マス、SFEN、MoveGen、Legal、Attack、Repetition、Impasse、Terminal | 729マス、3D駒の移動・遮蔽、新局面表現・終局判定 | 基盤追随（大幅再実装） |
| Runtime | Player API、Turn Runner、Headless、clock、outcome、GameHistory、プロセスBackend | 3D Player API、違法手非反映、時計、headless、3D履歴 | 基盤追随 |
| AI Package | JSON観測パーサ、内部盤面変換・画面認識Adapter | 3D局面の正確な変換とAdapter | 基盤追随 |
| 学習データ入力 | Training Parser、Core History Parser、再帰Importer、回帰テスト | 3D版の履歴とschema変換 | 基盤追随 |
| AIメタデータ | 共通schemaの文書と例、検証ツール | game/variant等の3D識別と互換管理 | 基盤追随 |
| USI/CLI | CLI雛形。USI完成はREADME上もマイルストーン | 3D固有のCLI/transport。2D USIとは区別 | 計画追随 |
| AIファミリー | READMEと複数の設計・Issue。mainに `engines/` 実装フォルダは未確認 | Evaluateから段階導入。未実装を実装済み扱いしない | 計画追随 |
| Game UI / 3D表示 | 本家にもUI関連Issueが存在。mainではGame UI実装未確認 | 9層の視認性を検証し入力可能にする | 計画追随 |
| モデル作成 / 学習 / Model Hub / リーグ | 本家で多数の機能Issueが進行中 | Core・Runtime確立後に同じ設計原則で展開 | 計画追随 |

## フェーズと到達条件

### M0 — ルールと基礎設計

- 将棋ったーの公開仕様と実対局の挙動を照合し、初期局面fixture・成り・打ちの未決部分を解消。
- 3Dの座標、盤、Move、判定契約を明文化。参考資料と一次資料の根拠を記録。
- C++20／CMake／CTest／Linux・Windows CIの骨格。

### M1 — 3DゲームCore（最優先）

- 729マス局面と入出力のround-trip、手番・持ち駒・成り。
- 王／歩／香／桂／銀／金の動き、飛車の平面・直方体遮蔽、角の45度長方形方式をそれぞれ独立検証。
- pseudo-legal → legal（王手、自玉王手回避、打ち歩詰め）→ 遷移 → 終局、AIなしで対局完走。

### M2 — 本家Runtime相当

- 3D Player Observation/Action/Result、違法手再操作、Headless Match、時計。
- 3D版の `BoardState` / `GameAux` 相当のJSONL、入出力検証、CLI。
- 停止・投了・千日手等、採用済みの3Dルールに応じたゲーム結果区別。

### M3 — 最小AI・データ基盤

- 再現可能なランダム／基礎Evaluateエンジンを同じPlayer境界で動かす。
- 3D AI Package観測パーサ、reference converter、最小training parser。
- 合法手数・ノード数・メモリ・1手遅延のベンチマークで3D探索爆発を計測。

### M4 — 本家の計画機能を段階導入

- Evaluate Learning、Hyper Fast、Mine Learning、Tree AI、Kadoka Best、Super Star、Obake等の3D版。
- Model Registry、Training、Dataset、リーグ、セキュリティ、モデル共有。
- 3D Game UI とAI作成UI、学習・評価UI、E2Eリリーステスト。

## 優先順位原則

1. **ルール確定と検証がAIより先**。飛車角の遮蔽を仮置きしたまま探索結果を信頼しない。
2. **CoreとAIを分離**。UIや学習でルールを二重実装しない。
3. **追随対象を定期再確認**。本家mainが更新されたら、機能表に実装済み/計画中の変化を追記する。
4. **小さなIssueと受入テスト**。各Issueに目標・範囲・依存・完了条件・非対象を記載。
5. **ライセンスを分離**。本家の技術コードはMIT、キャラクター名・外観は本家別ライセンス。

関連: [Architecture](../architecture/ARCHITECTURE.md) / [Rule Reference](../specifications/3D_RULES.md) / [参考資料](../references/REFERENCES.md)。
