# MCP Todo List サンプル

Azure Container Apps と .NET を使用したModel Context Protocol (MCP) サーバーのサンプルアプリケーションです。

## 概要

このサンプルでは、Todo リストを管理する MCP サーバーを構築・実行する方法を説明します。VS Code の MCP エージェント機能と MCP Inspector の両方を使用して、6つの異なる実行パターンをテストできます。

### サポートするテストパターン

1. **VS Code エージェント + ローカル MCP サーバー**
2. **VS Code エージェント + ローカル MCP サーバー（Docker コンテナ）**
3. **VS Code エージェント + Azure リモート MCP サーバー**
4. **MCP Inspector + ローカル MCP サーバー**
5. **MCP Inspector + ローカル MCP サーバー（Docker コンテナ）**
6. **MCP Inspector + Azure リモート MCP サーバー**

### 提供するツール

- `get_todo_items`: Todo アイテム一覧の取得
- `add_todo_item`: 新しい Todo アイテムの追加
- `complete_todo_item`: Todo アイテムを完了状態にする
- `update_todo_item`: 既存の Todo アイテムの更新
- `delete_todo_item`: Todo アイテムの削除

## 📚 ドキュメント利用ガイド

### このREADMEを読む場合
- 🚀 **初回セットアップを素早く行いたい**
- 📖 **基本的な使用方法を理解したい**
- 🔍 **どのテストパターンが自分に適しているか知りたい**

### [詳細セットアップガイド](./MCP-Todo-List-Setup-Guide.md)を読む場合
- 🐛 **問題が発生してトラブルシューティングが必要**
- 🧪 **実際のテスト結果例を見たい**
- 🔧 **各パターンの独立した詳細手順が欲しい**
- 🔍 **MCP Inspector の詳細な使用方法を知りたい**
- 📋 **日本語での具体的な操作例を確認したい**
- ☁️ **Azure 環境での詳細な設定とデバッグ情報が必要**

> 💡 **ヒント**: まずはこのREADMEでクイックスタートを試し、問題が発生した場合や詳細な情報が必要な場合は[詳細セットアップガイド](./MCP-Todo-List-Setup-Guide.md)をご参照ください。

## 前提条件

### 必須ソフトウェア

- [.NET 9.0 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Visual Studio Code](https://code.visualstudio.com/)
- [C# Dev Kit 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)

### パターン別追加要件

| パターン | 追加要件 |
|---------|---------|
| Docker コンテナ | [Docker Desktop](https://docs.docker.com/get-started/get-docker/) |
| Azure リモート | [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd) |
| MCP Inspector | [Node.js](https://nodejs.org/) |

## クイックスタート

### 基本的な使い方

1. **リポジトリルートに移動**
   ```bash
   REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
   cd $REPOSITORY_ROOT/todo-list
   ```

2. **MCP 設定をコピー**
   ```bash
   mkdir -p $REPOSITORY_ROOT/.vscode
   cp $REPOSITORY_ROOT/todo-list/.vscode/mcp.json $REPOSITORY_ROOT/.vscode/mcp.json
   ```

3. **サーバーを起動**
   ```bash
   dotnet run --project ./src/McpTodoList.ContainerApp
   ```

4. **VS Code で接続**
   - `F1` または `Ctrl+Shift+P` でコマンドパレットを開く
   - `MCP: List Servers` を検索・選択
   - `mcp-todo-list-aca-local` を選択し、`Start Server` をクリック

5. **テスト用プロンプト**
   ```
   - Todo リストを表示してください
   - 「11時にミーティング」を追加してください
   - 1番目の Todo アイテムを完了にしてください
   - 2番目の Todo アイテムを削除してください
   ```

## 詳細なセットアップとテスト手順

### パターン 1: VS Code エージェント + ローカル MCP サーバー

#### サーバーの起動
```bash
# プロジェクトディレクトリに移動
REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
cd $REPOSITORY_ROOT/todo-list

# MCP サーバーを起動
dotnet run --project ./src/McpTodoList.ContainerApp
```

#### 成功時の出力例
```
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://0.0.0.0:5242
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
```

#### VS Code での接続とテスト

1. **MCP設定のコピー**
   ```bash
   mkdir -p $REPOSITORY_ROOT/.vscode
   cp $REPOSITORY_ROOT/todo-list/.vscode/mcp.json $REPOSITORY_ROOT/.vscode/mcp.json
   ```

2. **VS Codeでの操作**
   - コマンドパレット（`F1`）→ `MCP: List Servers`
   - `mcp-todo-list-aca-local` を選択
   - `Start Server` をクリック

3. **接続確認**
   VS Code の出力パネル（MCP セクション）で以下のログを確認：
   ```
   [MCP] Connected to server mcp-todo-list-aca-local
   [MCP] Initialized server with capabilities: tools
   [MCP] Available tools: get_todo_items, add_todo_item, complete_todo_item, update_todo_item, delete_todo_item
   ```

4. **実際のテスト例**
   ```
   ユーザー: Todo リストを表示してください
   
   アシスタント: Todo リストを確認いたします。
   
   現在のTodo リストは空です。新しいアイテムを追加してください。
   ```

### パターン 2: VS Code エージェント + ローカル MCP サーバー（Docker コンテナ）

#### コンテナの構築と実行
```bash
# プロジェクトディレクトリに移動
REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
cd $REPOSITORY_ROOT/todo-list

# Docker イメージをビルド
docker build -t mcp-todo-list:latest .

# コンテナを実行
docker run -d -p 8080:8080 --name mcp-todo-list mcp-todo-list:latest
```

#### VS Code での接続
- コマンドパレット → `MCP: List Servers`
- `mcp-todo-list-aca-container` を選択
- `Start Server` をクリック

#### コンテナ停止
```bash
docker stop mcp-todo-list
docker rm mcp-todo-list
```

### パターン 3: VS Code エージェント + Azure リモート MCP サーバー

#### Azure へのデプロイ
```bash
# Azure にログイン
azd auth login

# プロジェクトを初期化してデプロイ
azd up
```

#### デプロイ確認
```bash
# FQDN を取得
azd env get-value AZURE_RESOURCE_MCP_TODO_LIST_FQDN
```

#### VS Code での接続
- コマンドパレット → `MCP: List Servers`
- `mcp-todo-list-aca-remote` を選択
- `Start Server` をクリック
- Azure Container Apps の FQDN を入力

### パターン 4: MCP Inspector + ローカル MCP サーバー

#### 準備
1. **MCP サーバーを起動**（パターン 1 と同じ手順）

2. **MCP Inspector を起動**
   ```bash
   npx @modelcontextprotocol/inspector node build/index.js
   ```

#### Inspector での接続
1. **ブラウザでアクセス**
   表示された URL（例：http://localhost:6274）をブラウザで開く

2. **接続設定**
   - Transport type: `SSE`
   - URL: `http://0.0.0.0:5242/sse`
   - `Connect` をクリック

3. **接続確認**
   "Connected" ステータスが表示されることを確認

#### ツールのテスト
1. **List Tools** をクリック
2. 5つのツールが表示されることを確認：
   - `get_todo_items`
   - `add_todo_item`
   - `complete_todo_item`
   - `update_todo_item`
   - `delete_todo_item`

3. **実際のテスト例**
   ```json
   // add_todo_item テスト
   {
     "todoItemText": "テスト用タスク"
   }
   
   // 実行結果
   {
     "id": 1,
     "text": "テスト用タスク",
     "isCompleted": false
   }
   ```

### パターン 5: MCP Inspector + ローカル MCP サーバー（Docker コンテナ）

パターン 4 と同じ手順ですが、接続URLが異なります：
- URL: `http://0.0.0.0:8080/sse`

### パターン 6: MCP Inspector + Azure リモート MCP サーバー

パターン 4 と同じ手順ですが、接続URLが異なります：
- URL: `https://<acaapp-server-fqdn>/sse`

（`<acaapp-server-fqdn>` は `azd env get-value AZURE_RESOURCE_MCP_TODO_LIST_FQDN` で取得した値）

## トラブルシューティング

> 🔍 **より詳細なトラブルシューティング情報**: このセクションでは基本的な問題解決方法を記載しています。実際のテスト結果例、Inspector の詳細な使用方法、環境別の詳細な設定については、[詳細セットアップガイド](./MCP-Todo-List-Setup-Guide.md)をご確認ください。

### 共通の問題と解決方法

#### 1. ポート競合エラー
**症状**: `Address already in use` エラー
```bash
# 使用中のポートを確認
netstat -tulpn | grep :5242

# プロセスを停止
kill -9 <PID>
```

#### 2. Docker 関連の問題
**症状**: Docker コンテナが起動しない
```bash
# 既存のコンテナを確認・停止
docker ps -a
docker stop mcp-todo-list
docker rm mcp-todo-list

# イメージを再ビルド
docker build -t mcp-todo-list:latest . --no-cache
```

#### 3. Azure デプロイエラー
**症状**: `azd up` 失敗
```bash
# ログイン状態を確認
azd auth login

# サブスクリプションを確認
az account show

# リソースをクリーンアップ
azd down --force --purge
```

#### 4. MCP Inspector 接続エラー
**症状**: "Failed to connect" エラー

**解決方法**:
- MCP サーバーが起動していることを確認
- URL とポート番号が正しいことを確認
- ブラウザのシークレットモード／プライベートブラウジングを試す
- 異なるブラウザを試す

#### 5. VS Code MCP 接続問題
**症状**: サーバーリストに表示されない

**解決方法**:
1. `.vscode/mcp.json` ファイルが存在することを確認
2. VS Code を再起動
3. C# Dev Kit 拡張機能が有効になっていることを確認

### ログの確認方法

#### VS Code MCP ログ
1. VS Code で **表示** → **出力** を選択
2. 出力チャンネルで **MCP** を選択
3. 接続状況とエラーメッセージを確認

期待されるログ例：
```
[MCP] Connected to server mcp-todo-list-aca-local
[MCP] Initialized server with capabilities: tools
[MCP] Available tools: get_todo_items, add_todo_item, complete_todo_item, update_todo_item, delete_todo_item
```

#### MCP Inspector ログ
ブラウザの開発者ツール（F12）で：
1. **Console** タブでエラーメッセージを確認
2. **Network** タブで HTTP リクエスト/レスポンスを確認

#### サーバーログ
ローカル実行時のコンソール出力でサーバーの状態を確認：
```
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://0.0.0.0:5242
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
```

## 成功基準

各パターンが正常に動作している場合の確認ポイント：

### VS Code エージェント
- [ ] サーバーリストに表示される
- [ ] 接続時に出力パネルにログが表示される
- [ ] 日本語プロンプトに応答する
- [ ] Todo 操作が正常に実行される

### MCP Inspector
- [ ] ブラウザで "Connected" ステータスが表示される
- [ ] 5つのツールがリストに表示される
- [ ] 各ツールの実行で JSON レスポンスが返される
- [ ] Todo データの操作が反映される

## その他の情報

### 設定ファイル
- `.vscode/mcp.json`: VS Code MCP サーバー設定
- `azure.yaml`: Azure Developer CLI 設定
- `infra/`: Azure リソース定義（Bicep テンプレート）

### 詳細ガイドの利用方法

**[詳細セットアップガイド](./MCP-Todo-List-Setup-Guide.md)** では以下の詳細情報を提供しています：

- 📋 **全6パターンの独立した詳細手順**: 各パターンごとに完全に独立したステップバイステップの手順
- 🧪 **実際のテスト結果例**: 実際に実行した際の成功/失敗例と期待される出力
- 🔍 **MCP Inspector の詳細使用方法**: 接続ログの確認方法、UI操作、デバッグテクニック
- 🐛 **環境別トラブルシューティング**: ローカル、コンテナ、Azure環境それぞれの問題と解決方法
- 💬 **日本語での操作例**: VS Code Chat と Inspector の両方での具体的な操作例
- ⚙️ **詳細な設定とベストプラクティス**: FQDN設定、接続確認方法、最適化のヒント

**推奨利用シナリオ**:
1. このREADMEでクイックスタートを実行
2. 問題が発生または詳細情報が必要になったら詳細ガイドを参照
3. 本格的な開発・テストでは詳細ガイドの手順を活用

### 関連リンク
- [Model Context Protocol 仕様](https://spec.modelcontextprotocol.io/)
- [Azure Container Apps ドキュメント](https://learn.microsoft.com/azure/container-apps/)
- [.NET 9.0 ドキュメント](https://learn.microsoft.com/dotnet/core/whats-new/dotnet-9/overview)

## ライセンス

このプロジェクトは MIT ライセンスの下で提供されています。詳細は [LICENSE](../LICENSE) ファイルをご確認ください。
