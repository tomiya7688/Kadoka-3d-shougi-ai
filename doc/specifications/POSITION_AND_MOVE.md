# 3D局面・指し手・データ形式（設計案）

> Status: Draft。未実装。これは実装の検討材料であり、現時点でワイヤ互換性を保証する規格ではない。

## 座標とメモリ表現

- 盤は x,y,z 各1..9の729マス。`x=筋`, `y=段`, `z=層`。
- 初期段階の内部盤は `std::array<Piece, 729>` を推奨。局面境界は `Square3D{x,y,z}` を受けるようにし、不正座標を拒否する。
- 保存・演算用の `SquareId` は `std::uint16_t`。1バイトでは729マスを識別できない。
- 0始まりの添字を `index=(z-1)*81+(y-1)*9+(x-1)` とする案。逆変換と境界チェックを単体テストで保証。
- `Color` は先手（black）、後手（white）。前方 `forward_y = black ? -1 : +1`。z方向を向きに応じて反転しない。
- 全種類の駒と成駒を表現し、持ち駒は両手番分を盤から独立して保持する。

## C++型の概形

```cpp
struct Square3D { std::uint8_t x, y, z; }; // 1..9
using Board3D = std::array<Piece, 729>;

struct Move3D {
  std::optional<Square3D> from; // absent => drop
  Square3D to;
  PieceType drop_piece;       // None unless drop
  bool promote;
};
```

本家 `engine/include/kadoka/types.hpp` の構造を参考にする。ただしその `Board=std::array<Piece,81>` と `Square{file,rank}` は2D仕様なのでそのまま移植しない。型は実装Issueで確定する。

## 局面操作

1. `Position3D::startpos()` は確認済みの**完全な初期配置fixture**から生成する。勝手な左右／層対称配置は作らない。
2. `Position3D::at(square)`, `side_to_move`, `hands`, `ply` を保持し、合法手のときだけ`after_move(move)`を返す。
3. 本家と同じく最初はimmutable/copy-basedでもよい。make/unmake、Zobrist hash、効率的占有集合は性能計測後に追加する。
4. 擬似合法手（駒形状・障害物・打ち／成り）と、合法手（王手回避・打ち歩詰め等）を別APIとする。
5. 王の詰みと、一般の移動可能マス数は同義ではない。3Dの特殊な飛車角遮蔽を `is_square_attacked` にも共通適用する。

## シリアライズ方針

- 通常のSFEN/USIは2Dの盤・段数・移動記号を前提とするため、勝手な3D拡張を標準SFEN/USIと呼ばない。
- まず `Position3D` と `Move3D` の**明示的な構造化JSON（暫定）**を定義し、テスト用fixtureとround-tripを用意する。最終的な層区切りのテキスト表現は別途比較する。
- 盤／持ち駒／手番／ply／バージョンを欠かさず記録する。入力エラーは型付きの失敗として扱う。
- 公開契約確定後は本家の `PlayerObservation/PlayerAction/ActionResult` と同じ責務を持つ3D版schemaをversion付きで定義する。2D形式との互換を偽装しない。

## 暫定fixture例（説明用。確定schemaではない）

```json
{
  "schema": "kadoka.3d.position.draft",
  "board_size": [9, 9, 9],
  "pieces": [{"x": 5, "y": 5, "z": 5, "piece": "king", "color": "black"}],
  "hands": {"black": {}, "white": {}},
  "side_to_move": "black",
  "ply": 0
}
```

## 必須テスト

- 添字 0 は (1,1,1)、添字728 は (9,9,9)。全729マスが互いに別の添字になる。
- 盤の各面・辺・頂点で座標検証し、範囲外が例外/エラーで拒否される。
- JSON往復で盤、持ち駒、手番、成り、着手数を保存する。
- 不正な盤サイズ・重複square・不明な駒種・不正なdrop/promoteを拒否する。
- 違法手の後も盤と手番は同一、合法手のみ局面が進む。

関連: [3Dルール](3D_RULES.md) / [アーキテクチャ](../architecture/ARCHITECTURE.md)。
