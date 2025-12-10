# UMC Cloud Variable API (Public)

Cloudflare Workers 上で動作する Cloud Variable API です。クラス定義と JSON スキーマを登録し、任意の JSON をバリデーション付きで保存・検索できます。OpenAPI ドキュメントを同梱し、Pages で公開可能です。

## 主な機能

- クラス定義の登録 (`/class/create`) とスキーマバリデーション
- アイテムの作成・更新・削除・検索 (`/users/:userId/class/:className/items/...`)
- 検索結果の任意テンプレート整形（`template` / `responseShape`）
- UMC 側 D1 を参照したセッション/IP チェック
- OpenAPI (`openapi.yaml`) と Swagger UI (`pages-openapi/index.html`)

## クイックスタート

前提: Node.js 18+, npm, Wrangler 4.x。

```bash
npm install
npm run db:local   # ローカル D1 (Miniflare) にスキーマ適用
npm run dev        # wrangler dev
```

## エンドポイント概要

| メソッド | パス | 用途 |
| --- | --- | --- |
| GET  | `/health` | ヘルスチェック |
| POST | `/register/resonite` | Resonite ID を登録 |
| POST | `/class/create` | クラス定義の作成（スキーマ保存） |
| GET  | `/users/:userId/class/:className` | クラス定義の取得 |
| GET  | `/users/:userId/class/:className/items` | アイテム一覧 |
| POST | `/users/:userId/class/:className/items/create` | アイテム作成 |
| POST | `/users/:userId/class/:className/items/update` | アイテム更新 |
| POST | `/users/:userId/class/:className/items/delete` | アイテム削除 |
| POST | `/users/:userId/class/:className/items/search` | アイテム検索（複数条件 AND） |
| GET  | `/openapi` `/docs` `/openapi.yaml` | OpenAPI リダイレクト |

### リクエスト例

**クラス作成**
```json
{
  "userId": 123,
  "className": "Inventory",
  "schema": {
    "items": { "type": "array", "items": { "type": "object", "properties": { "name": { "type": "string", "required": true }, "count": { "type": "number", "required": true } } } }
  },
  "canRead": 1,
  "canWrite": 0,
  "canDelete": 0
}
```

**アイテム作成**
```json
{
  "name": "Apple",
  "count": 3
}
```

**検索 + テンプレート整形**
```json
{
  "conditions": [
    { "key": "name", "op": "contains", "value": "App" }
  ],
  "template": "{id}:{name} ({count})"
}
```

`template` は `{path.to.value}` プレースホルダーを任意に配置できます（英数字/`_`/`.` のみ）。値が無い場合は空文字列を埋めます。`responseShape` も後方互換で利用可能です。

## 環境変数とバインディング

- D1: `DB` (UMC-CV), `UMC_DB` (UMC 本体)
- Vars: `OPENAPI_URL`, `OPENAPI_DOCS_URL`, `OPENAPI_YAML_URL`
- `wrangler.worker.toml` / `wrangler.pages.toml` で設定

## ローカル開発 & テスト

- 開発: `npm run dev`
- テスト: `npm test`
- OpenAPI を Pages 用に同期: `npm run docs:sync`（`openapi.yaml` を `pages-openapi/openapi.yaml` にコピー）

## デプロイ

- Worker: `wrangler deploy --config wrangler.worker.toml`
- Pages (Swagger UI): `wrangler pages deploy pages-openapi --project-name <project>`

## ライセンス

TBD（公開リポジトリで適切なライセンスを設定してください）。

## 注意

この README は公開用テンプレートです。秘密情報（鍵・DB ID など）は含めないでください。*** End Patch"**"}​
