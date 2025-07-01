# MCP Todo List Setup Guide

## 📋 概要

このガイドでは、MCP Todo Listプロジェクトの段階的なセットアップ、テスト、検証方法について詳しく説明します。MCP Todo Listは、To-doリストの管理機能を提供するMCPサーバーです。

## 📋 前提条件

> **注意**: 詳細な前提条件については[メインREADME.mdのPrerequisitesセクション](https://github.com/microsoft/mcp-dotnet-samples/tree/main/todo-list#prerequisites)を参照してください。

### 必要なツール確認

```bash
# Azure Developer CLI の確認
azd version

# Azure CLI の確認
az version

# .NET SDK の確認
dotnet --version

# Docker の確認
docker --version

# Node.js の確認 (MCP Inspector用)
node --version
```

**期待される結果:**
```
azd version 1.x.x
Azure CLI 2.x.x
.NET 9.0.x
Docker version 24.x.x
Node.js v18.x.x (MCP Inspector用)
```

**インストールが必要な場合:**
- [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Docker Desktop](https://docs.docker.com/get-started/get-docker/)
- [Node.js](https://nodejs.org/) (MCP Inspector パターンで使用)

## 🏗️ プロジェクト構造と実行フロー

### MCPツール機能

MCP Todo Listサーバーは以下の5つのツールを提供します：

| ツール名 | 機能 | 説明 |
|---------|------|------|
| `add_todo_item` | 追加 | 新しいTo-doアイテムをデータベースに追加 |
| `get_todo_items` | 一覧取得 | 全To-doアイテムの一覧を取得 |
| `update_todo_item` | 更新 | 既存のTo-doアイテムのテキストを更新 |
| `complete_todo_item` | 完了 | To-doアイテムを完了状態にマーク |
| `delete_todo_item` | 削除 | To-doアイテムをデータベースから削除 |

### 全体フロー

```
🏗️ Phase 1: サーバー準備 → 🔗 Phase 2: クライアント接続 → 💻 Phase 3: 実際の使用・テスト
```

### Phase 1: サーバー準備

**MCPサーバーを起動する**ための準備段階です：

#### 1. SSE Localサーバー起動
```bash
cd /workspaces/mcp-dotnet-samples/todo-list
dotnet run --project ./src/McpTodoList.ContainerApp
```
- **目的**: SSEタイプのMCPサーバーをローカル環境で直接実行
- **結果**: `http://localhost:5242/sse` でSSEサーバーが起動
- **データ**: In-memoryデータベースを使用

#### 2. SSE Containerサーバー起動
```bash
docker build -t mcp-todo-list:latest .
docker run -d -p 8080:8080 --name mcp-todo-list mcp-todo-list:latest
```
- **目的**: SSEタイプのMCPサーバーをDockerコンテナで実行
- **結果**: `http://localhost:8080/sse` でSSEサーバーが起動
- **特徴**: コンテナ環境での隔離実行

#### 3. SSE Remoteサーバーデプロイ
```bash
azd auth login
azd up
```
- **目的**: SSEタイプのMCPサーバーをAzure Container Appsにデプロイ
- **結果**: `https://<fqdn>/sse` でリモートSSEサーバーが起動
- **特徴**: 本番クラウド環境での実行

### Phase 2: クライアント接続

**Phase 1で準備したサーバーに接続する**ためのクライアント設定です。

#### 接続パターン一覧

| パターン | クライアント | サーバータイプ | 実行場所 | 説明 |
|---------|-------------|-------------|----------|------|
| **1** | VS Code | SSE | Local | ローカル開発サーバー |
| **2** | VS Code | SSE | Container | コンテナ実行サーバー |
| **3** | VS Code | SSE | Remote | Azure Container Apps |
| **4** | MCP Inspector | SSE | Local | デバッグ・検証用 |
| **5** | MCP Inspector | SSE | Container | コンテナデバッグ |
| **6** | MCP Inspector | SSE | Remote | リモートデバッグ |

## 🎯 Phase 2: 実践ガイド

### 共通操作手順

#### MCP設定切り替え
```bash
cd /workspaces/mcp-dotnet-samples/todo-list
mkdir -p /workspaces/mcp-dotnet-samples/.vscode

# 対象パターンの設定をコピー
cp .vscode/mcp.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json
```

#### VS Code MCPサーバー起動（パターン1-3）
**F1** → **"MCP: List Servers"** → 対象サーバーを選択 → **Start Server**

#### MCP Inspector起動（パターン4-6）
```bash
# 推奨: バージョン0.3.0を使用
npx @modelcontextprotocol/inspector@0.3.0
```

**🔍 接続ログの確認方法**:
MCP Inspector 0.3.0では、接続状況とログを以下の場所で確認できます：

1. **ターミナルログ**: `npx` コマンドを実行したターミナルでリアルタイム表示
   ```
   Starting MCP inspector...
   🔍 MCP Inspector is up and running at http://localhost:5173 🚀
   New SSE connection
   Query parameters: { transportType: 'sse', url: 'http://localhost:5242/sse' }
   Connected to SSE transport
   Connected MCP client to backing server transport
   ```

2. **ブラウザ開発者ツール**: 
   - **F12** → **Console** タブで詳細ログを確認
   - **Network** タブでSSE接続状況を監視

3. **MCP Inspector UI**:
   - ブラウザで `http://localhost:5173` にアクセス
   - 接続状況は画面上部に表示（Connected/Disconnected）
   - ツール一覧が表示されれば接続成功

#### 共通テスト手順
1. **MCPサーバー起動確認**
2. **VS Code Chat または MCP Inspector でテスト実行**
3. **To-doリスト操作テスト**

#### 共通成功確認方法
- ✅ **VS Code Chat/Inspector**: MCPツール使用でTo-do操作が実行される
- ✅ **出力確認**: ツール実行成功メッセージ
- ✅ **期待される結果**: To-doアイテムの追加・取得・更新・完了・削除が正常動作

---

### パターン1: VS Code + SSE Local
**特徴**: ローカル開発、最速起動  
**サーバー名**: `mcp-todo-list-aca-local`

**事前確認**:
```bash
curl -I http://localhost:5242/sse
# 期待される結果: HTTP/1.1 405 Method Not Allowed (正常)
```

**テストコマンド例**:
```
- todo リストを表示して
- "11時の会議" を追加して
- "食料品の買い物" を追加して
- todo アイテム #1 を完了して
- todo アイテム #2 を削除して
- 現在の todo リストを表示して
```

**動作テスト手順**:
1. **VS Code Chat**を開く
2. 上記テストコマンドを順次実行
3. 各操作の結果を確認

**実行結果サンプル**:
```
👤 リクエスト: todo リストを表示して
🤖 レスポンス: todo アイテムが見つかりません。

👤 リクエスト: "11時の会議" を追加して
🤖 レスポンス: Todo item added: '11時の会議' (ID: 1)

👤 リクエスト: "食料品の買い物" を追加して  
🤖 レスポンス: Todo item added: '食料品の買い物' (ID: 2)

👤 リクエスト: todo アイテム #1 を完了して
🤖 レスポンス: Todo item completed: '1'

👤 リクエスト: todo アイテム #2 を削除して
🤖 レスポンス: Todo item deleted: '2'

👤 リクエスト: 現在の todo リストを表示して
🤖 レスポンス: ID: 1, Text: 11時の会議, Completed: True
```

**成功サインの確認**:
- ✅ MCPサーバーが正常に起動（ポート5242で稼働）
- ✅ 5つのツールが発見されたことをログで確認
- ✅ to-doアイテムの追加/完了/削除が正常動作
- ✅ VS Code Chatでの自然言語操作が成功

**🔧 設定の最適化**:
初回実行時に405エラーが発生する場合は、`mcp.json`の設定を以下のように修正：
```json
"mcp-todo-list-aca-local": {
  "type": "sse",
  "url": "http://127.0.0.1:5242/sse"  // 0.0.0.0 → 127.0.0.1 に変更
}
```
この修正により、ローカルホストでの接続が安定します。

### パターン2: VS Code + SSE Container
**特徴**: Dockerコンテナ実行、本番環境類似  
**サーバー名**: `mcp-todo-list-aca-container`

**事前確認**:
```bash
docker ps | grep mcp-todo-list     # コンテナが Up 状態
curl -I http://localhost:8080/sse  # HTTP/1.1 405 Method Not Allowed (正常)
```

**テストコマンド例**:
```
- todo リストを表示して
- "Docker コンテナテスト" を追加して
- "コンテナデプロイ確認" を追加して
- todo アイテム #1 を完了して
- todo アイテム #2 を "更新されたコンテナテスト" に変更して
- 現在の todo リストを表示して
```

**動作テスト手順**:
1. **VS Code Chat**を開く
2. 上記テストコマンドを順次実行
3. 各操作の結果を確認

**実行結果サンプル**:
```
👤 リクエスト: todo リストを表示して
🤖 レスポンス: No todo items found.

👤 リクエスト: "Docker コンテナテスト" を追加して
🤖 レスポンス: Todo item added: 'Docker コンテナテスト' (ID: 1)

👤 リクエスト: "コンテナデプロイ確認" を追加して  
🤖 レスポンス: Todo item added: 'コンテナデプロイ確認' (ID: 2)

👤 リクエスト: todo アイテム #1 を完了して
🤖 レスポンス: Todo item completed: '1'

👤 リクエスト: todo アイテム #2 を "更新されたコンテナテスト" に変更して
🤖 レスポンス: Todo item updated: '2'

👤 リクエスト: 現在の todo リストを表示して
🤖 レスポンス: ID: 1, Text: Docker コンテナテスト, Completed: True
                ID: 2, Text: 更新されたコンテナテスト, Completed: False
```

**成功サインの確認**:
- ✅ Dockerコンテナが正常に起動（ポート8080で稼働）
- ✅ MCPサーバーがコンテナ内で正常動作
- ✅ 5つのツールが発見されたことをログで確認
- ✅ to-doアイテムの追加/完了/更新が正常動作
- ✅ VS Code Chatでの日本語操作が成功
- ✅ コンテナ環境での分離実行が確認できる

**🔧 設定とトラブルシューティング**:
コンテナ接続で問題が発生する場合の確認項目：
```bash
# コンテナ状態確認
docker ps | grep mcp-todo-list

# コンテナログ確認
docker logs mcp-todo-list

# ポート確認
curl -I http://localhost:8080/sse
```

**🐳 コンテナ環境の特徴**:
- 本番環境に近い隔離された実行環境
- ローカル開発環境の影響を受けない
- Dockerイメージの動作確認が可能

### パターン3: VS Code + SSE Remote
**特徴**: Azure Container Apps、クラウド本番環境  
**サーバー名**: `mcp-todo-list-aca-remote`

**事前確認**:
```bash
cd /workspaces/mcp-dotnet-samples/todo-list
FQDN=$(azd env get-value AZURE_RESOURCE_MCP_TODO_LIST_FQDN)
echo "Azure FQDN: https://$FQDN/sse"
curl -I "https://$FQDN/sse"    # HTTP/1.1 405 Method Not Allowed (正常)
```

**テストコマンド例**:
```
- todo リストを表示して
- "Azure クラウドテスト" を追加して
- "リモートデプロイ確認" を追加して
- todo アイテム #1 を完了して
- todo アイテム #2 を削除して
- 現在の todo リストを表示して
```

**動作テスト手順**:
1. **Azure FQDN を準備**:
   ```bash
   cd /workspaces/mcp-dotnet-samples/todo-list
   FQDN=$(azd env get-value AZURE_RESOURCE_MCP_TODO_LIST_FQDN)
   echo "FQDN: $FQDN"
   ```
2. **VS Code MCPサーバー起動**:
   - **F1** → **"MCP: List Servers"** → **"mcp-todo-list-aca-remote"** を選択 → **Start Server**
   - プロンプトで **"Azure Container Apps FQDN"** の入力を求められる
   - 上記で確認したFQDN（例：`myapp-12345.redpond-67890.eastus.azurecontainerapps.io`）**のみ**を入力
   - `https://` や `/sse` は**不要**（自動で付加されます）
3. **VS Code Chat**を開く
4. 上記テストコマンドを順次実行
5. 各操作の結果を確認

**実行結果サンプル**:
```
👤 リクエスト: todo リストを表示して
🤖 レスポンス: No todo items found.

👤 リクエスト: "Azure クラウドテスト" を追加して
🤖 レスポンス: Todo item added: 'Azure クラウドテスト' (ID: 1)

👤 リクエスト: "リモートデプロイ確認" を追加して  
🤖 レスポンス: Todo item added: 'リモートデプロイ確認' (ID: 2)

👤 リクエスト: todo アイテム #1 を完了して
🤖 レスポンス: Todo item completed: '1'

👤 リクエスト: todo アイテム #2 を削除して
🤖 レスポンス: Todo item deleted: '2'

👤 リクエスト: 現在の todo リストを表示して
🤖 レスポンス: ID: 1, Text: Azure クラウドテスト, Completed: True
```

**成功サインの確認**:
- ✅ Azure Container Appsが正常にデプロイ済み
- ✅ HTTPS接続でMCPサーバーにアクセス可能
- ✅ 5つのツールが発見されたことをログで確認
- ✅ to-doアイテムの追加/完了/削除が正常動作
- ✅ VS Code Chatでの日本語操作が成功
- ✅ クラウド本番環境での動作が確認できる

**🔧 Azure環境の確認**:
リモート接続で問題が発生する場合の確認項目：
```bash
# Azure リソース確認
azd env get-values

# FQDN確認
FQDN=$(azd env get-value AZURE_RESOURCE_MCP_TODO_LIST_FQDN)
echo "Azure FQDN: https://$FQDN/sse"

# 接続テスト
curl -I "https://$FQDN/sse"
```

**💡 FQDN入力のヒント**:
- VS Code MCPサーバー起動時は **FQDNのみ** を入力
- ❌ 間違い: `https://myapp-12345.redpond-67890.eastus.azurecontainerapps.io/sse`
- ✅ 正しい: `myapp-12345.redpond-67890.eastus.azurecontainerapps.io`
- プロトコル（https://）とパス（/sse）は自動で追加されます

**☁️ クラウド環境の特徴**:
- 本番環境での実際の動作確認
- インターネット経由でのアクセス
- Azure Container Appsの自動スケーリング機能

### パターン4: MCP Inspector + SSE Local
**特徴**: ローカルデバッグ・検証

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `http://localhost:5242/sse`
- **重要**: MCP Inspector v0.3.0 を使用

**テスト手順**:
1. **List Tools**ボタンをクリック
2. 各ツールを順次テスト:

   **add_todo_item**:
   - todoItemText: `"Inspector local test"`

   **get_todo_items**:
   - (パラメータなし)

   **update_todo_item**:
   - id: `1`
   - todoItemText: `"Updated inspector test"`

   **complete_todo_item**:
   - id: `1`

   **delete_todo_item**:
   - id: `1`

**実行結果サンプル**:
```
add_todo_item: Todo item added: '"Inspector local test"' (ID: 5)
get_todo_items: ID: 1, Text: , Completed: True
                ID: 3, Text: , Completed: True  
                ID: 5, Text: "Inspector local test", Completed: False
update_todo_item: Todo item updated: '1' with text: 'McpTodoList.ContainerApp.Models.TodoItem'
complete_todo_item: Todo item completed: '1'
delete_todo_item: Todo item deleted: '1'

最終状態:
ID: 3, Text: , Completed: True
ID: 5, Text: "Inspector local test", Completed: False
```

**成功サインの確認**:
- ✅ MCP Inspector のツール一覧に5つのツールが表示
- ✅ 各ツールの個別実行が正常動作
- ✅ CRUD操作（追加/取得/更新/完了/削除）が全て成功
- ✅ UIでツールのパラメータと結果が確認可能

**🔍 接続成功の確認ポイント**:
1. **ターミナル**: `Connected to SSE transport` メッセージ表示
2. **ブラウザUI**: 画面上部に **Connected** 状態表示
3. **ツール一覧**: `List Tools` クリックで5つのツールが表示
4. **接続URL**: `http://localhost:5242/sse` への接続成功

### パターン5: MCP Inspector + SSE Container
**特徴**: コンテナデバッグ

**事前確認**:
```bash
docker ps | grep mcp-todo-list     # コンテナが Up 状態
curl -I http://localhost:8080/sse  # 405 Method Not Allowed が返ることを確認
```

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `http://localhost:8080/sse`

**テスト手順**:
1. **List Tools**ボタンをクリック
2. 各ツールを順次テスト:

   **add_todo_item**:
   - todoItemText: `"Inspector container test"`

   **get_todo_items**:
   - (パラメータなし)

   **update_todo_item**:
   - id: `1`
   - todoItemText: `"Updated inspector container test"`

   **complete_todo_item**:
   - id: `1`

   **delete_todo_item**:
   - id: `1`

**実行結果サンプル**:
```
add_todo_item: Todo item added: 'Inspector container test' (ID: 1)
get_todo_items: ID: 1, Text: Inspector container test, Completed: False
update_todo_item: Todo item updated: '1' with text: 'McpTodoList.ContainerApp.Models.TodoItem'
complete_todo_item: Todo item completed: '1'
delete_todo_item: Todo item deleted: '1'

最終状態:
No todo items found.
```

**成功サインの確認**:
- ✅ MCP Inspector がコンテナポート8080に正常接続
- ✅ コンテナ内MCPサーバーの5つのツールが表示
- ✅ CRUD操作（追加/取得/更新/完了/削除）が全て成功
- ✅ コンテナ環境でのクリーンな動作確認
- ✅ 最終的に空の状態に戻る（完全なライフサイクル確認）

**🔍 接続成功の確認ポイント**:
1. **ターミナル**: `Connected to SSE transport` メッセージ表示
2. **ブラウザUI**: 画面上部に **Connected** 状態表示
3. **ツール一覧**: `List Tools` クリックで5つのツールが表示
4. **接続URL**: `http://localhost:8080/sse` への接続成功

### パターン6: MCP Inspector + SSE Remote
**特徴**: リモートデバッグ

**事前確認**:
```bash
cd /workspaces/mcp-dotnet-samples/todo-list
FQDN=$(azd env get-value AZURE_RESOURCE_MCP_TODO_LIST_FQDN)
echo "Azure FQDN: https://$FQDN/sse"
curl -I "https://$FQDN/sse"  # 405 Method Not Allowed が返ることを確認
```

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `https://<FQDN>/sse` （上記で取得したFQDN）

**テスト手順**:
1. **List Tools**ボタンをクリック
2. 各ツールを順次テスト:

   **add_todo_item**:
   - todoItemText: `"Inspector remote test"`

   **get_todo_items**:
   - (パラメータなし)

   **update_todo_item**:
   - id: `1`
   - todoItemText: `"Updated inspector remote test"`

   **complete_todo_item**:
   - id: `3` (新規追加されたアイテムのIDを使用)

   **delete_todo_item**:
   - id: `1`

**実行結果サンプル**:
```
add_todo_item: Todo item added: '"Inspector remote sse test"' (ID: 3)
get_todo_items: ID: 1, Text: , Completed: True
                ID: 3, Text: "Inspector remote sse test", Completed: False
update_todo_item: Todo item updated: '1' with text: 'McpTodoList.ContainerApp.Models.TodoItem'
complete_todo_item: Todo item completed: '3' (新規追加アイテムを完了)
delete_todo_item: Todo item deleted: '1'

最終状態:
ID: 3, Text: , Completed: True
```

**成功サインの確認**:
- ✅ MCP Inspector がAzure FQDNに正常接続（HTTPS）
- ✅ リモートMCPサーバーの5つのツールが表示
- ✅ CRUD操作（追加/取得/更新/完了/削除）が全て成功
- ✅ Azure Container Apps環境での安定動作
- ✅ 既存データとの共存・操作が正常動作
- ✅ クラウド本番環境での動作確認完了

**🔍 接続成功の確認ポイント**:
1. **ターミナル**: `Connected to SSE transport` メッセージ表示
2. **ブラウザUI**: 画面上部に **Connected** 状態表示
3. **ツール一覧**: `List Tools` クリックで5つのツールが表示
4. **接続URL**: `https://<FQDN>/sse` への接続成功（HTTPS通信）

## 📊 パターン選択ガイド

### 用途別推奨パターン

| 用途 | 推奨パターン | 理由 |
|------|-------------|------|
| **クイック動作確認** | パターン4 (Inspector + Local) | 最速、ビルド不要、UI確認可能 |
| **開発時のIDE統合** | パターン1 (VS Code + Local) | IDE内でシームレス、自然言語操作 |
| **本番環境テスト** | パターン6 (Inspector + Remote) | 実際のクラウド環境、詳細確認可能 |
| **コンテナ動作確認** | パターン5 (Inspector + Container) | 本番環境に近い状態、UI確認 |
| **デバッグ・検証** | パターン4-6 (Inspector系) | 各ツール個別テスト、詳細確認 |
| **自然言語テスト** | パターン1-3 (VS Code系) | 会話形式でのTo-do操作 |

### Phase 1完了状況確認

```bash
# Phase 1完了状況の一括確認
echo "=== Todo List MCP Server Status Check ==="

# 1. SSE Local
echo "1. SSE Local Server:"
curl -s -I http://localhost:5242/sse | head -1 | grep -q "405" && echo "✅ Running" || echo "❌ Not running"

# 2. SSE Container
echo "2. SSE Container:"
docker ps | grep -q mcp-todo-list && echo "✅ Running" || echo "❌ Not running"

# 3. SSE Remote
echo "3. SSE Remote (Azure):"
cd /workspaces/mcp-dotnet-samples/todo-list
if FQDN=$(azd env get-value AZURE_RESOURCE_MCP_TODO_LIST_FQDN 2>/dev/null); then
    curl -s -I "https://$FQDN/sse" | head -1 | grep -q "405" && echo "✅ Deployed: $FQDN" || echo "⚠️ Deployed but not responding: $FQDN"
else
    echo "❌ Not deployed"
fi
```

## 🚀 クイックスタート

### 最も簡単な開始方法

1. **MCP Inspector + ローカルSSE** (最速デバッグ)
```bash
cd /workspaces/mcp-dotnet-samples/todo-list
dotnet run --project ./src/McpTodoList.ContainerApp

# 別ターミナル
npx @modelcontextprotocol/inspector@0.3.0
```

2. **VS Code + ローカルSSE** (自然言語操作)
```bash
# ターミナル1
cd /workspaces/mcp-dotnet-samples/todo-list
dotnet run --project ./src/McpTodoList.ContainerApp

# ターミナル2
cp .vscode/mcp.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json
# VS Codeでチャット開始
```

## 📝 To-doリスト操作の学習例

### 基本的なワークフロー

1. **現在のリスト確認**
   ```
   todo リストを表示して
   ```

2. **タスク追加**
   ```
   "明日のプレゼン準備" を追加して
   "お母さんの誕生日プレゼントを買う" を追加して
   "四半期レポートを完成させる" を追加して
   ```

3. **リスト再確認**
   ```
   現在の todo リストを表示して
   ```

4. **タスク完了**
   ```
   todo アイテム #1 を完了して
   ```

5. **タスク更新**
   ```
   todo アイテム #2 を "お母さんの誕生日プレゼントとカードを買う" に変更して
   ```

6. **不要なタスク削除**
   ```
   todo アイテム #3 を削除して
   ```

7. **最終確認**
   ```
   最終的な todo リストを表示して
   ```

### 高度な操作例

- **複数タスクの一括管理**
- **タスクの優先順位付け（テキスト内で指定）**
- **期限付きタスクの追加**
- **カテゴリ別タスクの整理**

## 🔧 トラブルシューティング

### よくある問題と解決方法

#### "Port already in use" (5242/8080)
- **原因**: 既に同じポートでサーバーが実行中
- **解決**: `lsof -i :5242` でプロセスを確認し、必要に応じて停止

#### VS Code変数展開（FQDN入力）
- **症状**: Azure FQDNの入力プロンプトが表示されない
- **解決**: 手動で設定ファイルにFQDNを記載

#### Inspector接続エラー（SSE）
- **原因**: Inspector バージョン互換性問題
- **解決**: `npx @modelcontextprotocol/inspector@0.3.0` を使用

#### データ永続化
- **注意**: 現在のTo-doリストはIn-memoryデータベースのため、サーバー再起動でデータは消去されます
- **本番環境**: Azure SQL Database等の永続化ストレージとの連携が可能

## 🎓 学習ポイント

### MCP Todo Listで学べること

1. **CRUD操作の実装**: Create, Read, Update, Delete の基本操作
2. **状態管理**: To-doアイテムの完了状態管理
3. **エラーハンドリング**: 存在しないIDに対する適切なエラー応答
4. **自然言語インターフェース**: VS Code Chatでの直感的操作
5. **API設計**: RESTfulな操作のMCPツール実装

### 実世界での応用例

- **プロジェクト管理システム**との統合
- **チームタスク管理**の自動化
- **個人生産性ツール**の開発
- **ワークフロー管理**システムとの連携

このTo-doリスト機能を通じて、MCPプロトコルでの**状態を持つアプリケーション**の開発方法を学習できます。
