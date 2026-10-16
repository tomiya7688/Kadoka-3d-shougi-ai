# AI開発用コンテキスト

## Project

- `Kadoka 3D Shougi AI` / 日本語正本 / C++20を予定。
- 将棋ったー版の9×9×9三次元将棋を、AI抜きでも動くCoreから構築する。
- 本家: https://github.com/tomiya7688/Kadoka-shougi-ai

## まず読む

1. 該当GitHub Issueのスコープ・受入条件。
2. `doc/specifications/3D_RULES.md`（3D駒の移動と未決事項）。
3. `doc/specifications/POSITION_AND_MOVE.md`（座標とフォーマット）。
4. `doc/architecture/ARCHITECTURE.md`（責務境界）。
5. 必要なときだけ `doc/roadmap/UPSTREAM_PARITY.md` と `doc/references/REFERENCES.md`。

## 守るべき設計

- Coreが正本の合法性と局面を保持し、AIは着手候補を返すだけ。
- 3D王は26近傍。飛車の動きは平面状、角の遮蔽は特殊。**6方向の飛車・通常の直線角へ置き換えない**。
- 3D盤は729マス。2DのSquare/Move/SFEN/USIを無検証で流用しない。
- 合法手の一覧、王手フラグ、学習ターゲット等を通常PlayerObservationに必須としない。
- 違法手は正本盤面・手番・plyを変更せず、結果を明示する。
- 学習・解析・重いI/OはRuntimeの外に置く。
- ルールの未確定箇所は明記してfixtureに落とす。外部サービスへの依存をCIで必須にしない。

## 実装時の報告事項

- 対象Issueと変更ファイル、ルール根拠。
- 追加した3D fixture、通したテスト、未検証部分。
- 本家との差分、3D独自実装の理由。
- ルール上の推測や未解決の曖昧さは明確に記録する。

現在は初期ドキュメント段階。ソース、CI、実動作はまだ存在しない。
