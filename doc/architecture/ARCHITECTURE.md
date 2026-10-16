# アーキテクチャ設計（初期版）

> Status: Design Draft / 2026-10-16。実装に先行する設計であり、API確定・機能実装を表すものではない。

## 目的と境界

本家 [Kadoka Shougi AI](https://github.com/tomiya7688/Kadoka-shougi-ai) と同様、**AIなしでも3D将棋を完全に進行できるゲームCore** を先に作る。AIは局面を観測し、指し手・投了等を提案するだけ。合法性・正本局面・終局処理の最終権限はCoreが持つ。

参照ルールは [将棋ったー 三次元将棋](https://shogitter.com/rule/三次元将棋)。729マス、王将の26近傍、面内に広がる飛車、特殊な角の45度長方形方式は2D将棋と異なるため、既存の2D利きアルゴリズムを無条件で流用しない。

## 予定ディレクトリ・責務

| 領域 | 役割 | 禁止する逆依存 |
| --- | --- | --- |
| engine/ | 盤面、持ち駒、着手、擬似合法手・合法手、王手、局面遷移、終局、ハッシュ | AI/Runtime/UI/学習に依存しない |
| runtime/ | PlayerObservation、PlayerAction、合法性チェック付き1手実行、時計、Headless Match、履歴 | 具体的AI・学習器・GUIに依存しない |
| ai_package/ | 観測のパース、AI内部盤面への変換、AIの実装に必要な独立したルール再構成 | 正本局面の変更を担わない |
| engines/ | 各AIファミリー、探索、評価、学習済みモデルの推論 | Coreの権威を上書きしない |
| protocol/ | CLI、将来の3D対局プロトコル、外部AI I/O | ルールの重複実装をしない |
| tools/ | 記録の解析、学習前処理、リーグ、ベンチマーク、データ生成 | Runtimeの対局ホットパスへ逆依存しない |
| tests/ | 駒の利き、局面、王手、対局、API、性能回帰 | 実装と同時に整備 |
| doc/ | 正本仕様、設計、参考資料 | 未決事項を確定事項として扱わない |

依存方向（概念）:

    protocol / UI / headless tools
                ↓
             runtime  ←  engines / AI adapters
                ↓
          authoritative engine/core

AI Packageや学習基盤はRuntime/Coreの公開契約を利用してよいが、その逆は不可。これは本家の [Architecture](https://github.com/tomiya7688/Kadoka-shougi-ai/blob/main/doc/architecture/ARCHITECTURE.md) と整合する。

## Coreと3D表現

- 座標は `(x,y,z)`、各1..9。zは上下の層で、先手・後手とも共通。盤面は729マス。
- `Square3D`, `Piece`, `Move3D`, `Position3D`, `Hands`, `Color` などを責務別に設ける。具体のバイナリ配置・APIは設計段階で固定しない。
- 3Dの移動候補算出・遮蔽・捕獲・合法手は分離する。特に飛車は単なる6方向スライダーではなく、共有座標のある面へ動く。
- 違法手は正本局面を変更せず、合法手のみ手番・着手数を進める。
- 探索内部では高速なmake/unmakeやハッシュを許すが、Coreの意味論を変更しない。
- 3Dの盤面フォーマット・棋譜・着手表現は独自のversioned contractとして定義し、既存SFEN/USIとの混同を避ける。

詳細: [ルール](../specifications/3D_RULES.md) / [局面と指し手](../specifications/POSITION_AND_MOVE.md)。

## Public Player API（本家互換の設計思想）

    可視の局面情報 / 可視の時計
                ↓
       Player / AI Adapter
                ↓
       Move / Resign
                ↓
      Coreで検証・適用
                ↓
     Accepted / Illegal と結果

局面の合法手一覧、王手フラグ、探索候補、policy/value、評価特徴量、学習データ用IDは通常のPlayerObservationの必須項目に含めない。AIが必要なら独自に計算する。AIの返答が違法なら盤面を維持して再操作を許可する。Source of truthはCore。

## 対局履歴・学習への接続

本家の `BoardState JSONL` と `GameAux JSONL` の分割、および `game_id + ply` の対応付けを参考にする。3D局面のシリアライズ契約を確定するまでは本家v1と互換を名乗らず、別schema/versionを用意する。学習前処理・モデル評価はRuntimeの外側に置く。

## AIファミリーの継承方針（予定）

Evaluate / Evaluate Learning / Hyper Fast / Mine Learning / Tree AI / Kadoka Best / Super Star / Obake Kadoka・Maru の **役割・設計思想** を本家から引き継ぐ。現時点でこれらが3D版に実装されているわけではない。3Dならではの評価（王の3D逃げ道、面・立体の利き、玉周囲の密度等）は共通特徴量ではなく各AIの評価設計として検証する。

## 開発順序・品質ゲート

1. 参照サイトと一致するルール・局面の実例を固定する。
2. 729マス状態・局面入出力のround-trip・駒移動・遮蔽を単体テスト。
3. 王手・合法手・打ち・成り・局面遷移・終局を正しく実装。
4. AI抜きでHeadless対局可能、Runtimeの不正応答耐性、時計、棋譜が揃う。
5. AIファミリーと学習に入る。CPU単体CIで完走する最小検証を維持。

**未決**: 将棋ったー版の初期局面完全列挙、成り範囲・強制成り、打ち歩詰め、千日手・入玉、公式実装における王手条件など。外部サイトに書かれていない挙動は実盤面の再現試験とともにIssueで解決する。

## 設計の更新ルール

日本語を仕様の正本とする。API変更時は仕様・回帰テスト・互換性の説明を同じ変更に含める。現行実装と将来計画を混ぜない。親本家の更新を追う際は [追随ロードマップ](../roadmap/UPSTREAM_PARITY.md) で差分を記録する。
