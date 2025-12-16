# UMC Cloud Variable API

UMC に登録済みのユーザー向け Cloud Variable API です。UMC 本体で発行された `userId` を使って、クラス定義を登録し、そのスキーマに沿った JSON データを保存・検索できます。利用には UMC への登録とログイン（IP 一致によるセッション確認）が必要です。

## エンドポイント

| メソッド | パス | 説明 |
| --- | --- | --- |
| GET  | `https://cv.umcapp.org/health` | ヘルスチェック |
| GET  | `https://cv.umcapp.org/docs` | OpenAPI HTML へリダイレクト |
| GET  | `https://cv.umcapp.org/openapi` | OpenAPI HTML へのリダイレクト |
| GET  | `https://cv.umcapp.org/openapi.yaml` | OpenAPI YAML へのリダイレクト |
| GET  | `https://cv.umcapp.org/playground` | WebUI (Playground) |
| POST | `https://cv.umcapp.org/register/resonite` | Resonite ID を UMC-CV に登録 |
| POST | `https://cv.umcapp.org/class/create` | クラス定義の作成（スキーマ登録） |
| POST | `https://cv.umcapp.org/users/:userId/class/:className/delete` | クラス定義の削除（紐づくアイテムも削除） |
| GET  | `https://cv.umcapp.org/users/:userId/classes` | クラス定義の一覧取得 |
| GET  | `https://cv.umcapp.org/users/:userId/class/:className` | クラス定義の取得 |
| GET  | `https://cv.umcapp.org/users/:userId/class/:className/items` | アイテム一覧取得 |
| GET  | `https://cv.umcapp.org/users/:userId/class/:className/items/:itemId` | アイテム単体取得 |
| POST | `https://cv.umcapp.org/users/:userId/class/:className/items/create` | アイテム作成 |
| POST | `https://cv.umcapp.org/users/:userId/class/:className/items/update` | アイテム更新 |
| POST | `https://cv.umcapp.org/users/:userId/class/:className/items/delete` | アイテム削除 |
| POST | `https://cv.umcapp.org/users/:userId/class/:className/items/search` | アイテム検索（複数条件 AND） |

`userId` は UMC 本体のユーザー ID を指定してください。`can_read = 0` のクラスを読む/書く場合は、UMC 側でログイン中であること（IP 一致）が必須です。クラス削除は紐づくアイテムもまとめて削除します。

## リクエスト例

### クラス作成
`POST /class/create`
```json
{
  "userId": 123,
  "className": "Inventory",
  "schema": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string", "required": true },
          "count": { "type": "number", "required": true }
        }
      }
    }
  },
  "canRead": 1,
  "canWrite": 0,
  "canDelete": 0
}
```

### アイテム作成
`POST /users/:userId/class/:className/items/create`
```json
{
  "name": "Apple",
  "count": 3
}
```

### 検索 + 任意フォーマットで返す
`POST /users/:userId/class/:className/items/search`
```json
{
  "conditions": [
    { "key": "name", "op": "contains", "value": "App" }
  ],
  "template": "{id}:{name} ({count})"
}
```

`template` には `{path.to.value}` プレースホルダーを自由に配置できます（英数字/`_`/`.` のみ）。値が無い場合は空文字列を埋めます。

### レスポンス整形（簡易オプション）
- `{"type": "value", "path": "foo.bar"}` → `data.foo.bar` だけを配列で返す
- `{"type": "keyValue", "keyPath": "id", "valuePath": "name"}` → `[{ "key": <id>, "value": <name> }, ...]` を返す

## 認証・前提
- すべての `/users/:userId/...` 系エンドポイントで UMC 登録ユーザーであることが前提です。
- UMC 側でログイン（IP 一致）している必要があります。

## OpenAPI
- 詳細なパラメータ・スキーマは OpenAPI で確認してください。ブラウザで HTML ドキュメントを開く場合は [`https://cv.umcapp.org/docs`](https://cv.umcapp.org/docs) または [`https://cv.umcapp.org/openapi`](https://cv.umcapp.org/openapi) を参照してください。YAML は [`https://cv.umcapp.org/openapi.yaml`](https://cv.umcapp.org/openapi.yaml) から取得できます。
- ブラウザだけで試す場合は [`https://cv.umcapp.org/playground`](https://cv.umcapp.org/playground) を開いてください。エンドポイントとボディを切り替えながら送信できます。
