# 参考資料・出典

> 2026-10-16確認。公開資料の事実、設計上の提案、実際の対局での検証を区別する。リンク先のルール・実装・利用条件が変更される可能性がある。

## ルールに関する一次資料

1. [将棋ったー：三次元将棋のルール](https://shogitter.com/rule/三次元将棋) — 9×9×9、筋・段・層、初期配置の概要、二歩の定義、勝利条件。**本プロジェクトのルール基準**。
2. [将棋ったー：三次元将棋の駒の動き](https://shogitter.com/rule/三次元将棋/koma) — 先後の前方、王26近傍、金18近傍、飛車・角の移動面および遮蔽の説明。**移動の一次資料**。
3. [Sen Miuate：高次元将棋における飛車角の一般化](https://note.com/senmiuate/n/nef16dc36e702) — 2026-08-02公開。n次元の飛車角をどう一般化するかという考察。**数理的参考資料であり、将棋ったー実装の代替規則ではない**。

## 本家 Kadoka Shougi AI の設計・実装参照

- [Repository](https://github.com/tomiya7688/Kadoka-shougi-ai) / [README](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/README.md) — 原則、現在の実装状況、AIファミリー計画。
- [Architecture](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/architecture/ARCHITECTURE.md) — Core/Runtime/AI Package/学習の責務分離。
- [C++ types.hpp](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/engine/include/kadoka/types.hpp) / [position.hpp](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/engine/include/kadoka/position.hpp) — Square、Move、Board、Handsの参考。2Dの81マス型は3D向けに変更が必要。
- [Move Generation](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/specifications/MOVE_GENERATION.md) / [Legal Move Generation](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/specifications/LEGAL_MOVE_GENERATION.md) — 擬似合法手と王手回避の責務境界。
- [Engine Runtime](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/specifications/ENGINE_RUNTIME.md) / [Player JSON](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/specifications/PLAYER_JSON_AND_GAME_HISTORY.md) — AIは指し手を提案、Coreが正本を検証。通常観測に合法手一覧を含めない。
- [Game History Record](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/specifications/GAME_HISTORY_RECORD.md) — BoardState / GameAux 2系列の対局履歴。
- [Headless Match Runtime](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/specifications/HEADLESS_MATCH_RUNTIME.md) — 違法手の再試行、時計、終局、replay、安全停止。
- [Training Parser](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/training/TRAINING_PARSER_V1.md) — 履歴を共通中間Recordに変換するパーサ。
- [AI Model Metadata](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/specifications/AI_MODEL_METADATA.md) — 系列共通のAIメタデータ。3D版ではgame識別を別にする。
- [本家 Issues](https://github.com/tomiya7688/Kadoka-shougi-ai/issues) — 学習、AI、UI、配布の**計画中**機能の詳細。

## 利用・移植時の注意

- 本家の `LICENSE` はMIT。技術的な再利用時はMIT表記を維持すること。
- 本家の公式キャラクター名・キャラクターの見た目には別建ての `CHARACTER_LICENSE.md` がある。キャラクター関連素材を流用する前に確認する。
- 将棋ったーの説明・棋譜・画像・ソースを無断で再配布可能とは解釈しない。テストfixtureは独自に記述し、他サイトのコードや素材の転載は利用条件を確認する。
- 参考資料が主張することと、このリポジトリが検証できたことは異なる。検証済みケースを `tests/fixtures/` へ追加し、参照URL・確認日をテストの説明へ残す。

## 今後ほしい参考データ

- 公開対局に由来しない、権利を確認できる初期局面と合法手fixture。
- 各種遮蔽・王手・成り・持ち駒打ちの盤面例。
- 3D盤の描画・入力方式（9層表示、断面、回転）に関するUI実験。
- 空盤候補数／複雑な盤面でのノード数／走り駒判定時間のベンチマーク。
