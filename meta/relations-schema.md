# relations.json — Schema Reference

`meta/relations.json` is the connective tissue of Forklore Galaxy.
It does not contain story. It contains the relationships between stories.

This document is the authoritative reference for the schema.

**Schema Version: 1.1**
*(v1.0: Sessions 3–5 · v1.1: Session 6 — added `language`, `summary`, `created_at`; clarified `degree` update rule and `bridge_to` bidirectional responsibility)*

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

**Bidirectional responsibility rule:**
When `directed: false`, both sides should record the connection in their own `bridge_to`. The author who creates the bridge is responsible for adding the entry to their own `relations.json`. For cross-repository bridges, it is recommended (but not required) to notify the target repository's maintainer via a GitHub Issue, so they can add the reciprocal entry on their side.

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

### `language`
**Type:** string | `null`

The language of the node's body text. Use [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) two-letter codes.
```json
"language": "ja"
```

| Value | Meaning |
|-------|---------|
| `"ja"` | Japanese |
| `"en"` | English |
| `"fr"` | French |
| `null` | Unknown or unspecified |

This field is used in Phase 2 for translation button rendering and language-based filtering on the galaxy map. Node language is entirely the author's choice — there is no restriction.

---

### `summary`
**Type:** string | `null`

A 1–2 sentence description of the node's content. Free-form. Used in Phase 2 for hover cards, OGP metadata, and onboarding display.
```json
"summary": "The first word is spoken. The galaxy stirs."
```

This field is optional. If omitted, the UI will display the first line of the node's `.md` file as a fallback.

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

### `created_at`
**Type:** string (ISO 8601) | `null`

The creation timestamp of the node in UTC. In most cases this is redundant with the TIMESTAMP in the node ID — but recording it explicitly here enables future sorting, filtering, and display without parsing filenames.
```json
"created_at": "2026-03-03T02:38:00Z"
```

If omitted, the timestamp in the node ID is used as the canonical creation time.

---

### `degree`
**Type:** integer

The number of structural connections this node has (count of all `precedes` + `follows` + `bridge_to` entries). A structural metric, not a popularity metric.
```json
"degree": 1
```

> ⚠️ **Manual update required:** `degree` is not computed automatically. Whenever you add or remove entries in `precedes`, `follows`, or `bridge_to`, you must recalculate and update `degree` manually. Phase 2 tooling will automate this.

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
    "language": "ja",
    "summary": "The first word is spoken. The galaxy stirs.",
    "cycle_annotation": null,
    "created_by": "human",
    "created_at": "2026-03-03T02:38:00Z",
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
- `degree` must be updated manually whenever `precedes`, `follows`, or `bridge_to` changes

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

**スキーマバージョン: 1.1**
*(v1.0: セッション3〜5 · v1.1: セッション6 — `language`・`summary`・`created_at` を追加。`degree` 手動更新ルールと `bridge_to` 双方向責任を明記)*

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

**双方向の責任ルール：**
`directed: false` の場合、A・B 両側がそれぞれ自分の `relations.json` に `bridge_to` エントリを記録するのが原則です。ブリッジを作成した執筆者が自分側のエントリを追加する責任を持ちます。クロスリポジトリの場合は、相手リポジトリのメンテナーにGitHub Issue等で通知し、相手側のエントリ追加を促すことを推奨します（必須ではありません）。

### `characters`
このノードに登場するキャラクター識別子の配列。自由記述。

### `location`
このノードの舞台。自由記述の文字列。

### `tags`
説明タグの配列。将来の検索・フィルタリングに使用。

### `language`
ノード本文の言語。[ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) の2文字コードを使用。不明・未指定の場合は `null`。Phase 2の翻訳ボタン表示と銀河マップの言語フィルタリングに使用します。ノードの言語は執筆者の完全な自由です。

### `summary`
ノード内容の1〜2文の説明。自由記述。Phase 2のホバーカード・OGPメタデータ・オンボーディング表示に使用。省略時はUIが `.md` ファイルの先頭行をフォールバックとして使用します。

### `cycle_annotation`
ループ（循環）が検知されたときに付与される注釈。ループはエラーではなく、有効なミニループストーリー構造です。

### `created_by`
このノードを作成したのが誰か。推奨値：`"human"` / `"ai"` / `"human+ai"` / GitHubユーザー名

### `created_at`
ノードの作成タイムスタンプ（UTC・ISO 8601形式）。ノードIDのTIMESTAMPと通常は一致しますが、明示的に記録しておくことでファイル名のパースなしにソート・フィルタ・表示が可能になります。省略時はノードIDのタイムスタンプが正規の作成日時として使用されます。

### `degree`
構造的接続数。`precedes` + `follows` + `bridge_to` の合計。人気指標ではなく構造指標。

> ⚠️ **手動更新が必要：** `degree` は自動計算されません。`precedes`・`follows`・`bridge_to` を変更した際は、必ず `degree` を手動で再計算・更新してください。Phase 2のツールでこの作業は自動化される予定です。

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
- `precedes`・`follows`・`bridge_to` を変更した際は `degree` を手動で更新してください

---

## クロスリポジトリ参照

別のForklore Galaxyインスタンスのノードへリンクする形式：
```
REPO_URL::NODE_ID
```

</details>

---

*Schema v1.0 finalized across Sessions 3–5 of the founding relay.*
*Schema v1.1 updated in Session 6.*
