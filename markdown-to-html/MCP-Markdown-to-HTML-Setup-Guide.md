# MCP Markdown-to-HTML Setup Guide

## 📋 概要

このガイドでは、MCP Markdown-to-HTMLプロジェクトの段階的なセットアップ、テスト、検証方法について詳しく説明します。

## 📋 前提条件

> **注意**: 詳細な前提条件については[メインREADME.mdのPrerequisitesセクション](https://github.com/microsoft/mcp-dotnet-samples/tree/main/markdown-to-html#prerequisites)を参照してください。

### 必要なツール確認

Phase 1の「4. Run ASP.NET Core MCP server (SSE) remotely」を実行する場合は、以下のツールがインストールされている必要があります：

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
- [Node.js](https://nodejs.org/) (MCP Inspector パターン6-10で使用)

## 🏗️ プロジェクト構造と実行フロー

### 全体フロー

```
🏗️ Phase 1: サーバー準備 → 🔗 Phase 2: クライアント接続 → 💻 Phase 3: 実際の使用・テスト
```

### Phase 1: サーバー準備

**MCPサーバーを起動する**ための準備段階です：

#### 1. STDIO Containerビルド
```bash
docker build -f Dockerfile.stdio -t mcp-md2html-stdio:latest .
```
- **目的**: STDIOタイプのMCPサーバーをDockerイメージとしてビルド
- **結果**: `mcp-md2html-stdio:latest` イメージが作成される

#### 2. SSE Localサーバー起動
```bash
cd /workspaces/mcp-dotnet-samples/markdown-to-html
dotnet run --project ./src/McpMarkdownToHtml.ContainerApp
```
- **目的**: SSEタイプのMCPサーバーをローカル環境で直接実行
- **結果**: `http://localhost:5280/sse` でSSEサーバーが起動

#### 3. SSE Containerサーバー起動
```bash
docker build -f Dockerfile.sse -t mcp-md2html-sse:latest .
docker run -d -p 8080:8080 --name mcp-md2html-sse mcp-md2html-sse:latest
```
- **目的**: SSEタイプのMCPサーバーをDockerコンテナで実行
- **結果**: `http://localhost:8080/sse` でSSEサーバーが起動

#### 4. SSE Remoteサーバーデプロイ
```bash
azd auth login
azd up
```
- **目的**: SSEタイプのMCPサーバーをAzure Container Appsにデプロイ
- **結果**: `https://<fqdn>/sse` でリモートSSEサーバーが起動

### Phase 2: クライアント接続

**Phase 1で準備したサーバーに接続する**ためのクライアント設定です。

#### 接続パターン一覧

| パターン | クライアント | サーバータイプ | 実行場所 | 説明 |
|---------|-------------|-------------|----------|------|
| **1** | VS Code | STDIO | Local | 直接dotnet実行、最速 |
| **2** | VS Code | STDIO | Container | Dockerコンテナ経由 |
| **3** | VS Code | SSE | Local | ローカルWebサーバー |
| **4** | VS Code | SSE | Container | コンテナWebサーバー |
| **5** | VS Code | SSE | Remote | Azure Container Apps |
| **6** | MCP Inspector | STDIO | Local | デバッグ・検証用 |
| **7** | MCP Inspector | STDIO | Container | コンテナデバッグ |
| **8** | MCP Inspector | SSE | Local | SSEデバッグ |
| **9** | MCP Inspector | SSE | Container | コンテナSSEデバッグ |
| **10** | MCP Inspector | SSE | Remote | リモートSSEデバッグ |

## 🎯 Phase 2: 実践ガイド

### 共通操作手順

#### MCP設定切り替え
```bash
cd /workspaces/mcp-dotnet-samples/markdown-to-html
mkdir -p /workspaces/mcp-dotnet-samples/.vscode

# 各パターンに応じて以下のいずれかを実行
cp .vscode/mcp.stdio.local.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json      # パターン1
cp .vscode/mcp.stdio.container.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json  # パターン2
cp .vscode/mcp.sse.local.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json        # パターン3
cp .vscode/mcp.sse.container.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json    # パターン4
cp .vscode/mcp.sse.remote.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json       # パターン5
```

#### VS Code MCPサーバー起動（パターン1-5）
**F1** → **"MCP: List Servers"** → 対象サーバーを選択 → **Start Server**

#### MCP Inspector起動（パターン6-10）
```bash
npx @modelcontextprotocol/inspector
```

#### 共通テスト手順
1. **MCPサーバー起動確認**
2. **VS Code Chat または MCP Inspector でテスト実行**
3. **出力確認**

#### 共通成功確認方法
- ✅ **VS Code Chat/Inspector**: MCPツール使用でHTML変換が実行される
- ✅ **出力確認**: 
  - STDIO: `method 'tools/call' request handler called/completed.`
  - SSE: `Discovered 1 tools`, `接続状態: 実行中`
- ✅ **期待される結果**: Markdig による高品質HTML変換（見出しID、段落間隔など）

#### SSE接続の正常動作について
```
[info] 接続状態: エラー Error reading SSE stream: TypeError: terminated
```
上記メッセージは**正常な動作**です。SSEはツール実行完了後に接続を適切に終了し、次回使用時に自動再接続されます。

---

### パターン1: VS Code + STDIO Local
**特徴**: 直接dotnet実行、最速起動  
**サーバー名**: `mcp-md2html-stdio-local`

**テストマークダウン**:
```
# パターン1テスト (STDIO Local)
これは**直接dotnet実行**によるMCPサーバーのテストです。
- ローカル1
- ローカル2
```

### パターン2: VS Code + STDIO Container
**特徴**: Dockerコンテナ経由、隔離性あり  
**サーバー名**: `mcp-md2html-stdio-container`

**事前確認**:
```bash
docker images | grep mcp-md2html-stdio
# 期待される結果: mcp-md2html-stdio latest <image_id> ...
```

**テストマークダウン**:
```
# パターン2テスト (STDIO Container)
これは**Dockerコンテナ**で動作するMCPサーバーのテストです。
- コンテナ1
- コンテナ2
```

### パターン3: VS Code + SSE Local
**特徴**: ローカルSSEサーバー、Webベース通信  
**サーバー名**: `mcp-md2html-sse-local`

**事前確認**:
```bash
curl -I http://localhost:5280/sse
# 期待される結果: HTTP/1.1 405 Method Not Allowed (正常)
```

**テストマークダウン**:
```
# パターン3テスト (SSE Local)
これは**SSE (Server-Sent Events)**で動作するMCPサーバーのテストです。
- SSE接続1
- SSE接続2
```

### パターン4: VS Code + SSE Container
**特徴**: DockerコンテナのSSEサーバー、本番環境類似  
**サーバー名**: `mcp-md2html-sse-container`

**事前確認**:
```bash
docker ps | grep mcp-md2html-sse     # コンテナが Up 状態
curl -I http://localhost:8080/sse    # HTTP/1.1 405 Method Not Allowed (正常)
```

**テストマークダウン**:
```
# パターン4テスト (SSE Container)
これは**Dockerコンテナ内のSSE**で動作するMCPサーバーのテストです。
- コンテナSSE1
- コンテナSSE2
```

### パターン5: VS Code + SSE Remote
**特徴**: Azure Container Apps、クラウド本番環境  
**サーバー名**: `mcp-md2html-sse-remote`

**事前確認**:
```bash
cd /workspaces/mcp-dotnet-samples/markdown-to-html
FQDN=$(azd env get-value AZURE_RESOURCE_MCP_MD2HTML_FQDN)
echo "Azure FQDN: https://$FQDN/sse"
curl -I "https://$FQDN/sse"    # HTTP/1.1 405 Method Not Allowed (正常)
```

**テストマークダウン**:
```
# パターン5テスト (SSE Remote)
これは**Azure Container Apps**で動作するMCPサーバーのテストです。
- リモートSSE1
- リモートSSE2
```

### パターン6: MCP Inspector + STDIO Local
**特徴**: デバッグ・検証用、最速テスト

**Inspector設定**:
- **Transport Type**: `STDIO`
- **Command**: `dotnet`
- **Arguments**: `run --project /workspaces/mcp-dotnet-samples/markdown-to-html/src/McpMarkdownToHtml.ConsoleApp`

**テストマークダウン**:
```
# パターン6テスト (Inspector STDIO Local)
これは**MCP Inspector + STDIO**の組み合わせテストです。
- インスペクター1
- インスペクター2
```

**動作テスト手順**:
1. **List Tools**ボタンをクリック
2. `convert_markdown_to_html`ツールを選択
3. **The markdown text**に上記マークダウンを入力
4. **Run Tool**ボタンをクリック

### パターン7: MCP Inspector + STDIO Container
**特徴**: コンテナ経由デバッグ

**事前確認**:
```bash
docker images | grep mcp-md2html-stdio  # イメージが存在することを確認
```

**Inspector設定**:
- **Transport Type**: `STDIO`
- **Command**: `docker`
- **Arguments**: `run -i --rm mcp-md2html-stdio:latest`

**テストマークダウン**:
```
# パターン7テスト (Inspector STDIO Container)
これは**MCP Inspector + Docker STDIO**の組み合わせテストです。
- コンテナインスペクター1
- コンテナインスペクター2
```

**動作テスト手順**:
1. **List Tools**ボタンをクリック
2. `convert_markdown_to_html`ツールを選択
3. **The markdown text**に上記マークダウンを入力
4. **Run Tool**ボタンをクリック

### パターン8: MCP Inspector + SSE Local
**特徴**: SSEデバッグ

**事前確認**:
```bash
curl -I http://localhost:5280/sse  # 405 Method Not Allowed が返ることを確認
```

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `http://localhost:5280/sse`
- **重要**: MCP Inspector v0.3.0 を使用 (`npx @modelcontextprotocol/inspector@0.3.0`)

**テストマークダウン**:
```
# パターン8テスト (Inspector SSE Local)
これは**MCP Inspector + ローカルSSE**の組み合わせテストです。
- インスペクターSSE1
- インスペクターSSE2
```

**動作テスト手順**:
1. **List Tools**ボタンをクリック
2. `convert_markdown_to_html`ツールを選択
3. **The markdown text**に上記マークダウンを入力
4. **Run Tool**ボタンをクリック

### パターン9: MCP Inspector + SSE Container
**特徴**: コンテナSSEデバッグ

**事前確認**:
```bash
docker ps | grep mcp-md2html-sse     # コンテナが Up 状態
curl -I http://localhost:8080/sse    # 405 Method Not Allowed が返ることを確認
```

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `http://localhost:8080/sse`

**テストマークダウン**:
```
# パターン9テスト (Inspector SSE Container)
これは**MCP Inspector + コンテナSSE**の組み合わせテストです。
- コンテナインスペクターSSE1
- コンテナインスペクターSSE2
```

**動作テスト手順**:
1. **List Tools**ボタンをクリック
2. `convert_markdown_to_html`ツールを選択
3. **The markdown text**に上記マークダウンを入力
4. **Run Tool**ボタンをクリック

### パターン10: MCP Inspector + SSE Remote
**特徴**: リモートSSEデバッグ

**事前確認**:
```bash
cd /workspaces/mcp-dotnet-samples/markdown-to-html
FQDN=$(azd env get-value AZURE_RESOURCE_MCP_MD2HTML_FQDN)
echo "Azure FQDN: https://$FQDN/sse"
curl -I "https://$FQDN/sse"  # 405 Method Not Allowed が返ることを確認
```

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `https://<FQDN>/sse` （上記で取得したFQDN）

**テストマークダウン**:
```
# パターン10テスト (Inspector SSE Remote)
これは**MCP Inspector + Azure SSE**の組み合わせテストです。
- リモートインスペクター1
- リモートインスペクター2
```

**動作テスト手順**:
1. **List Tools**ボタンをクリック
2. `convert_markdown_to_html`ツールを選択
3. **The markdown text**に上記マークダウンを入力
4. **Run Tool**ボタンをクリック

## 📊 パターン選択ガイド

### 用途別推奨パターン

| 用途 | 推奨パターン | 理由 |
|------|-------------|------|
| **クイック動作確認** | パターン6 (Inspector + STDIO Local) | 最速、ビルド不要 |
| **開発時のIDE統合** | パターン3 (VS Code + SSE Local) | IDE内でシームレス |
| **本番環境テスト** | パターン10 (Inspector + SSE Remote) | 実際のクラウド環境 |
| **コンテナ動作確認** | パターン7,9 (Inspector + Container) | 本番環境に近い状態 |
| **デバッグ・検証** | パターン6-10 (Inspector系) | UI上で詳細確認可能 |

### Phase 1完了状況確認

```bash
# Phase 1完了状況の一括確認
echo "=== Phase 1 Status Check ==="

# 1. STDIO Container
echo "1. STDIO Container:"
docker images | grep mcp-md2html-stdio && echo "✅ Ready" || echo "❌ Not ready"

# 2. SSE Local
echo "2. SSE Local Server:"
curl -s -I http://localhost:5280/sse | head -1 | grep -q "405" && echo "✅ Running" || echo "❌ Not running"

# 3. SSE Container
echo "3. SSE Container:"
docker ps | grep -q mcp-md2html-sse && echo "✅ Running" || echo "❌ Not running"

# 4. SSE Remote
echo "4. SSE Remote (Azure):"
cd /workspaces/mcp-dotnet-samples/markdown-to-html
if FQDN=$(azd env get-value AZURE_RESOURCE_MCP_MD2HTML_FQDN 2>/dev/null); then
    curl -s -I "https://$FQDN/sse" | head -1 | grep -q "405" && echo "✅ Deployed: $FQDN" || echo "⚠️ Deployed but not responding: $FQDN"
else
    echo "❌ Not deployed"
fi
```

## 🚀 クイックスタート

### 最も簡単な開始方法

1. **MCP Inspector + ローカルSTDIO** (ビルド不要)
```bash
cd /workspaces/mcp-dotnet-samples/markdown-to-html
npx @modelcontextprotocol/inspector
```

2. **VS Code + ローカルSSE**
```bash
# ターミナル1
cd /workspaces/mcp-dotnet-samples/markdown-to-html
dotnet run --project ./src/McpMarkdownToHtml.ContainerApp

# ターミナル2
cp .vscode/mcp.sse.local.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json
# VS Codeでチャット開始
```

## 🔧 トラブルシューティング

### よくある問題と解決方法

#### VS Code変数展開が機能しない
- **症状**: プロンプトが表示されない、または変数が空になる
- **解決**: 設定ファイルで直接パスを指定

#### "Port already in use"
- **原因**: 既に同じポートでサーバーが実行中
- **解決**: `lsof -i :PORT` でプロセスを確認し、必要に応じて停止

#### "dotnet run failed"
- **原因**: プロジェクトのビルドエラー
- **解決**: `dotnet build` で事前にビルドを確認

#### MCP Inspector接続エラー
- **原因**: プロキシ認証またはサーバー起動失敗
- **解決**: ブラウザコンソールでエラー詳細を確認

この構造により、**段階的な開発・テスト・デプロイ**のワークフローが実現され、各環境での動作確認が可能になります。
