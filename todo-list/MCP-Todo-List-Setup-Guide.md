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
- Show me the list to do
- Add "meeting at 11am"
- Add "buy groceries"
- Complete the to-do item #1
- Delete the to-do item #2
- Show me the current to-do list
```

**動作テスト手順**:
1. **VS Code Chat**を開く
2. 上記テストコマンドを順次実行
3. 各操作の結果を確認

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
- Show me the list to do
- Add "docker container test"
- Add "container deployment check"
- Complete the to-do item #1
- Update the to-do item #2 with "updated container test"
- Show me the current to-do list
```

**動作テスト手順**:
1. **VS Code Chat**を開く
2. 上記テストコマンドを順次実行
3. 各操作の結果を確認

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
- Show me the list to do
- Add "Azure cloud test"
- Add "remote deployment verification"
- Complete the to-do item #1
- Delete the to-do item #2
- Show me the current to-do list
```

**動作テスト手順**:
1. Azure FQDNを入力
2. **VS Code Chat**を開く
3. 上記テストコマンドを順次実行
4. 各操作の結果を確認

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

**テスト手順**: パターン4と同じ手順で、異なるアイテムテキストを使用
- todoItemText: `"Inspector container test"`

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

**テスト手順**: パターン4と同じ手順で、異なるアイテムテキストを使用
- todoItemText: `"Inspector remote test"`

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
   Show me the list to do
   ```

2. **タスク追加**
   ```
   Add "prepare presentation for tomorrow"
   Add "buy birthday gift for mom"
   Add "finish quarterly report"
   ```

3. **リスト再確認**
   ```
   Show me the current to-do list
   ```

4. **タスク完了**
   ```
   Complete the to-do item #1
   ```

5. **タスク更新**
   ```
   Update the to-do item #2 with "buy birthday gift and card for mom"
   ```

6. **不要なタスク削除**
   ```
   Delete the to-do item #3
   ```

7. **最終確認**
   ```
   Show me the final to-do list
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
