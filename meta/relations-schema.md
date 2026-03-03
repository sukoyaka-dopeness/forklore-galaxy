# relations.json — Schema Reference

`meta/relations.json` is the connective tissue of Forklore Galaxy.
It does not contain story. It contains the relationships between stories.

This document is the authoritative reference for the schema.

---

## Location

```
meta/relations.json
```

One file per repository instance. All nodes in the instance are recorded here.

---

## Structure

`relations.json` is a single JSON object. Each key is a **node ID**. Each value is a **metadata object** for that node.

```json
{
  "NODE_ID": {
    ...metadata...
  },
  "NODE_ID": {
    ...metadata...
  }
}
```

---

## Node ID format

```
TIMESTAMP-HASH-title
```

| Part | Format | Example |
|------|--------|---------|
| TIMESTAMP | `YYYY-MMDDTHHMMSSz` | `2026-0303T023800Z` |
| HASH | 8 alphanumeric characters | `00000000` |
| title | lowercase, hyphen-separated | `genesis` |

Example: `2026-0303T023800Z-00000000-genesis`

**Note:** The node ID does not include the `.md` extension.

**Special case:** The genesis node of each instance uses `00000000` as its hash. This is intentional — it marks the origin.

---

## Keys

### `precedes`
**Type:** array of node IDs | `[]`

Nodes that come **after** this node in the narrative. This node leads to them.

```json
"precedes": ["2026-0303T025900Z-00000001-and-the-word-branched"]
```

---

### `follows`
**Type:** array of node IDs | `[]`

Nodes that come **before** this node in the narrative. This node comes after them.

```json
"follows": ["2026-0303T023800Z-00000000-genesis"]
```

---

### `bridge_to`
**Type:** array of bridge objects | `[]`

Wormhole connections to nodes in other narrative threads or other repositories. Unlike `precedes`/`follows`, these are not sequential — they are jumps.

**Default direction: bidirectional (`directed: false`)**

```json
"bridge_to": [
  {
    "target": "2026-0303T130000Z-c3d4e5f6-other-node",
    "directed": false
  },
  {
    "target": "https://github.com/other/repo::node-id",
    "directed": true
  }
]
```

| Field | Value | Meaning |
|-------|-------|---------|
| `directed: false` | default | A ↔ B (bidirectional wormhole) |
| `directed: true` | explicit | A → B (one-way wormhole) |

---

### `characters`
**Type:** array of strings | `[]`

Character identifiers appearing in this node. Free-form strings. No controlled vocabulary.

```json
"characters": ["warrior_A", "old-woman"]
```

---

### `location`
**Type:** string | `null`

The setting of this node. Free-form string.

```json
"location": "battlefield"
```

---

### `tags`
**Type:** array of strings | `[]`

Descriptive tags for this node. Free-form. Used for future search and filtering.

```json
"tags": ["genesis", "origin", "japanese"]
```

---

### `cycle_annotation`
**Type:** string | `null`

Automatically or manually added when a cycle (loop) is detected in the narrative graph. A cycle is not an error — it is a valid mini-loop story structure.

```json
"cycle_annotation": "ループを検知しました。この分岐は循環しています。"
```

---

### `created_by`
**Type:** string

Who created this node.

```json
"created_by": "human"
```

Suggested values: `"human"` / `"ai"` / `"human+ai"` / GitHub username

---

### `degree`
**Type:** integer

The number of structural connections this node has (count of all `precedes` + `follows` + `bridge_to` entries). A structural metric, not a popularity metric.

```json
"degree": 1
```

---

### `heat`
**Type:** integer

A reader activity metric. How much attention this node has received. Separate from `degree` by design — a node can be structurally central but cold, or isolated but hot.

```json
"heat": 0
```

Initial value is always `0`.

---

### `visibility`
**Type:** string

Controls whether the node appears in the galaxy map and navigation.

| Value | Meaning |
|-------|---------|
| `"public"` | Default. Visible everywhere. |
| `"low"` | Deprioritized in display. Still exists. Still in Git history. |

Nodes are **never deleted**. `visibility` is the only mechanism for withdrawal.

```json
"visibility": "public"
```

---

### `status`
**Type:** string

The narrative status of the node.

| Value | Meaning |
|-------|---------|
| `"active"` | Default. Normal node. |
| `"floating"` | Isolated node — not connected to any other node. Equivalent to an external reference in display treatment. |

```json
"status": "active"
```

---

## Complete example

```json
{
  "2026-0303T023800Z-00000000-genesis": {
    "precedes": ["2026-0303T025900Z-00000001-and-the-word-branched"],
    "follows": [],
    "characters": [],
    "location": null,
    "bridge_to": [],
    "tags": ["genesis", "origin"],
    "cycle_annotation": null,
    "created_by": "human",
    "degree": 1,
    "heat": 0,
    "visibility": "public",
    "status": "active"
  }
}
```

---

## Validation rules

- `precedes`, `follows`, `bridge_to` must be arrays (never `null`)
- All node IDs referenced in `precedes`, `follows`, and `bridge_to` should exist in the file — or be valid external references (`repo_url::node_id`)
- A node may reference itself (`precedes` or `follows` pointing to its own ID) — this is a valid self-loop (mini-loop story)
- Cycles involving multiple nodes are valid and should be annotated with `cycle_annotation`
- Orphan nodes (no connections) are valid — set `status: "floating"`

---

## Cross-repository references

To link to a node in another Forklore Galaxy instance:

```json
"bridge_to": [
  {
    "target": "https://github.com/other-user/their-forklore::2026-0401T120000Z-ab1cd2ef-their-node",
    "directed": false
  }
]
```

Format: `REPO_URL::NODE_ID`

---

<details>
<summary>🇯🇵 日本語で読む</summary>

# relations.json — スキーマリファレンス

`meta/relations.json` はForklore Galaxyの結合組織です。
物語そのものではなく、物語同士の**関係**を記述します。

---

## 場所

```
meta/relations.json
```

リポジトリインスタンスごとに1ファイル。インスタンス内のすべてのノードがここに記録されます。

---

## 構造

`relations.json` は単一のJSONオブジェクトです。各キーが**ノードID**、各値がそのノードの**メタデータオブジェクト**です。

---

## ノードIDの形式

```
TIMESTAMP-HASH-title
```

**特別ケース：** 各インスタンスのgenesisノードはハッシュに `00000000` を使います。これは意図的なものです——起源を示す印です。

---

## キー一覧

### `precedes`
この後に続くノードのID配列。このノードはそれらに繋がります。

### `follows`
この前にあるノードのID配列。このノードはそれらの後に来ます。

### `bridge_to`
ワームホール接続。順序関係ではなく、ジャンプです。**デフォルトは双方向（`directed: false`）。**

### `characters`
このノードに登場するキャラクター識別子の配列。自由記述。

### `location`
このノードの舞台。自由記述の文字列。

### `tags`
説明タグの配列。将来の検索・フィルタリングに使用。

### `cycle_annotation`
ループ（循環）が検知されたときに付与される注釈。ループはエラーではなく、有効なミニループストーリー構造です。

### `created_by`
このノードを作成したのが誰か。推奨値：`"human"` / `"ai"` / `"human+ai"` / GitHubユーザー名

### `degree`
構造的接続数。`precedes` + `follows` + `bridge_to` の合計。人気指標ではなく構造指標。

### `heat`
読者活動指標。注目度を示します。`degree` とは意図的に分離されています。初期値は常に `0`。

### `visibility`
銀河マップとナビゲーションでの表示制御。`"public"`（デフォルト）または `"low"`（表示抑制）。ノードは**絶対に削除されません**。`visibility` が唯一の取り下げ手段です。

### `status`
ノードのナラティブ状態。`"active"`（デフォルト）または `"floating"`（孤立ノード）。

---

## バリデーションルール

- `precedes`、`follows`、`bridge_to` は配列である必要があります（`null` 不可）
- 参照するノードIDはファイル内に存在するか、有効な外部参照（`repo_url::node_id`）である必要があります
- 自己参照（自分自身を指す `precedes`/`follows`）は有効です——ミニループストーリーとして許容されます
- 複数ノードにまたがる循環も有効です。`cycle_annotation` で注釈を付けてください
- 接続のないノード（孤立ノード）は有効です。`status: "floating"` を設定してください

---

## クロスリポジトリ参照

別のForklore Galaxyインスタンスのノードへリンクする形式：

```
REPO_URL::NODE_ID
```

</details>

---

*This schema was finalized across Sessions 3–5 of the founding relay.*
