# MCP YouTube Subtitles Extractor Setup Guide

## 📋 概要

このガイドでは、MCP YouTube Subtitles Extractorプロジェクトの段階的なセットアップ、テスト、検証方法について詳しく説明します。YouTube動画から字幕を抽出し、要約や分析を行うMCPサーバーの構築・実行方法を学習できます。

## 📋 前提条件

> **注意**: 詳細な前提条件については[メインREADME.mdの前提条件セクション](https://github.com/microsoft/mcp-dotnet-samples/tree/main/youtube-subtitles-extractor#前提条件)を参照してください。

### 必要なツール確認

```bash
# .NET SDK の確認
dotnet --version

# Docker の確認 (Container Apps版)
docker --version

# Azure Functions Core Tools の確認 (Functions版)
func --version

# Node.js の確認 (MCP Inspector用)
node --version
```

**期待される結果:**
```
.NET 9.0.x
Docker version 24.x.x
Azure Functions Core Tools 4.x.x
Node.js v18.x.x (MCP Inspector用)
```

**インストールが必要な場合:**
- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Docker Desktop](https://docs.docker.com/get-started/get-docker/)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local?pivots=programming-language-csharp#install-the-azure-functions-core-tools)
- [Node.js](https://nodejs.org/) (MCP Inspector パターンで使用)

## 🏗️ プロジェクト構造と実行フロー

### MCPツール機能

MCP YouTube Subtitles Extractorサーバーは以下の2つのツールを提供します：

| ツール名 | 機能 | 説明 |
|---------|------|------|
| `get_available_languages` | 言語一覧取得 | 指定されたYouTube動画で利用可能な字幕言語の一覧を取得 |
| `get_subtitle` | 字幕取得 | 指定された言語でYouTube動画の字幕を取得 |

### サンプル動画とテスト用URL

このガイドでは以下のテスト用YouTube動画を使用します：
```
https://youtu.be/XwnEtZxaokg?si=V39ta45iMni_Uc_m
```

### 全体フロー

```
🏗️ Phase 1: サーバー準備 → 🔗 Phase 2: クライアント接続 → 💻 Phase 3: 字幕抽出・要約テスト
```

### Phase 1: サーバー準備

**MCPサーバーを起動する**ための準備段階です：

#### Container Apps版パターン

##### 1. ローカル実行
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/containerapp
dotnet run --project ./src/McpYouTubeSubtitlesExtractor.ContainerApp
```
- **目的**: Container Apps版のMCPサーバーをローカル環境で直接実行
- **結果**: `http://localhost:5202/sse` でSSEサーバーが起動
- **特徴**: 開発・テスト環境での素早い検証

##### 2. コンテナ実行
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/containerapp
docker build -t mcp-youtube-subtitles-extractor:latest .
GUID=$(uuidgen)  # アクセスキー生成
docker run -d -p 8080:8080 -e Mcp__ApiKey=$GUID --name mcp-youtube-subtitles-extractor mcp-youtube-subtitles-extractor:latest
```
- **目的**: Container Apps版のMCPサーバーをDockerコンテナで実行
- **結果**: `http://localhost:8080/sse?code=$GUID` でSSEサーバーが起動
- **特徴**: 本番環境に近いコンテナ環境での実行

#### Functions版パターン

##### 3. Functions ローカル実行
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/functionapp/src/McpYouTubeSubtitlesExtractor.FunctionApp
func start
```
- **目的**: Azure Functions版のMCPサーバーをローカル環境で実行
- **結果**: `http://localhost:7071/runtime/webhooks/mcp/sse` でSSEサーバーが起動
- **特徴**: サーバーレス環境でのテスト

### Phase 2: クライアント接続

**Phase 1で準備したサーバーに接続する**ためのクライアント設定です。

#### 接続パターン一覧

| パターン | クライアント | サーバータイプ | 実行場所 | 説明 |
|---------|-------------|-------------|----------|------|
| **1** | VS Code | Container Apps | Local | ローカル開発サーバー |
| **2** | VS Code | Container Apps | Container | コンテナ実行サーバー |
| **3** | VS Code | Functions | Local | Functions ローカル実行 |
| **4** | MCP Inspector | Container Apps | Local | デバッグ・検証用 |
| **5** | MCP Inspector | Container Apps | Container | コンテナデバッグ |
| **6** | MCP Inspector | Functions | Local | Functions デバッグ |

## 🎯 Phase 2: 実践ガイド

### 共通操作手順

#### MCP設定切り替え
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor
mkdir -p /workspaces/mcp-dotnet-samples/.vscode

# Container Apps版の設定をコピー
cp containerapp/.vscode/mcp.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json

# または Functions版の設定をコピー
cp functionapp/.vscode/mcp.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json
```

#### VS Code MCPサーバー起動（パターン1-3）
**F1** → **"MCP: List Servers"** → 対象サーバーを選択 → **Start Server**

#### MCP Inspector起動（パターン4-6）
```bash
# 推奨: バージョン0.3.0を使用
npx @modelcontextprotocol/inspector@0.3.0
```

---

### パターン1: VS Code + Container Apps Local
**特徴**: Container Apps版のローカル開発、最速起動  
**サーバー名**: `mcp-youtube-subtitles-extractor-local`

**事前確認**:
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/containerapp
dotnet run --project ./src/McpYouTubeSubtitlesExtractor.ContainerApp

# 別ターミナルで接続確認
curl -I http://localhost:5202/sse
# 期待される結果: HTTP/1.1 405 Method Not Allowed (正常)
```

**テストコマンド例**:
```
- このYouTube動画を5つの要点でまとめてください: https://youtu.be/XwnEtZxaokg
- この動画で利用可能な字幕言語を教えてください: https://youtu.be/XwnEtZxaokg
- この動画の英語字幕を取得してください: https://youtu.be/XwnEtZxaokg
```

**動作テスト手順**:
1. **MCP設定のコピー**:
   ```bash
   cp containerapp/.vscode/mcp.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json
   ```
2. **VS Code Chat**を開く
3. 上記テストコマンドを順次実行
4. 各操作の結果を確認

**実行結果サンプル**:
```
👤 リクエスト: このYouTube動画を5つの要点でまとめてください: https://youtu.be/XwnEtZxaokg

🤖 レスポンス: まず利用可能な言語を確認します...
get_available_languages を実行中...

利用可能な言語: en (英語), ja (日本語), ko (韓国語)

get_subtitle を実行中 (言語: en)...

この動画の要点は以下の通りです：
• [要点1の内容]
• [要点2の内容]
• [要点3の内容]
• [要点4の内容]  
• [要点5の内容]
```

**成功サインの確認**:
- ✅ MCPサーバーが正常に起動（ポート5202で稼働）
- ✅ 2つのツールが発見されたことをログで確認
- ✅ YouTube動画の字幕取得と要約が正常動作
- ✅ VS Code Chatでの自然言語操作が成功

### パターン2: VS Code + Container Apps Container
**特徴**: Dockerコンテナ実行、本番環境類似  
**サーバー名**: `mcp-youtube-subtitles-extractor-container`

**事前確認**:
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/containerapp
docker build -t mcp-youtube-subtitles-extractor:latest .
GUID=$(uuidgen)
echo "Generated GUID: $GUID"
docker run -d -p 8080:8080 -e Mcp__ApiKey=$GUID --name mcp-youtube-subtitles-extractor mcp-youtube-subtitles-extractor:latest

# 接続確認
curl -I "http://localhost:8080/sse?code=$GUID"
# 期待される結果: HTTP/1.1 405 Method Not Allowed (正常)
```

**テストコマンド例**:
```
- この動画の字幕を日本語で取得してください: https://youtu.be/XwnEtZxaokg
- 動画の内容を3つのキーポイントで説明してください: https://youtu.be/XwnEtZxaokg
```

**動作テスト手順**:
1. **VS Code Chat**を開く
2. **F1** → **"MCP: List Servers"** → **"mcp-youtube-subtitles-extractor-container"** を選択
3. **Start Server** をクリック
4. **アクセスキーの入力**: 上記で生成したGUIDを入力
5. 上記テストコマンドを実行

**実行結果サンプル**:
```
👤 リクエスト: この動画の字幕を日本語で取得してください: https://youtu.be/XwnEtZxaokg

🤖 レスポンス: まず利用可能な言語を確認します...
get_available_languages を実行中...

利用可能な言語: en, ja, ko

get_subtitle を実行中 (言語: ja)...

日本語字幕を取得しました：
[字幕の内容が表示される]
```

**成功サインの確認**:
- ✅ Dockerコンテナが正常に起動（ポート8080で稼働）
- ✅ アクセスキーによる認証が成功
- ✅ 2つのツールが発見されたことをログで確認
- ✅ YouTube動画の字幕取得が正常動作
- ✅ コンテナ環境での分離実行が確認できる

### パターン3: VS Code + Functions Local
**特徴**: Azure Functions版のローカル実行  
**サーバー名**: `mcp-youtube-subtitles-extractor-function-local`

**事前確認**:
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/functionapp/src/McpYouTubeSubtitlesExtractor.FunctionApp
func start

# 別ターミナルで接続確認
curl -I http://localhost:7071/runtime/webhooks/mcp/sse
# 期待される結果: HTTP/1.1 405 Method Not Allowed (正常)
```

**テストコマンド例**:
```
- この技術プレゼンテーション動画の要点を教えてください: https://youtu.be/XwnEtZxaokg
- 動画で使用されている言語を確認してください: https://youtu.be/XwnEtZxaokg
```

**動作テスト手順**:
1. **MCP設定のコピー**:
   ```bash
   cp functionapp/.vscode/mcp.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json
   ```
2. **VS Code Chat**を開く
3. **F1** → **"MCP: List Servers"** → **"mcp-youtube-subtitles-extractor-function-local"** を選択
4. **Start Server** をクリック
5. 上記テストコマンドを実行

**実行結果サンプル**:
```
👤 リクエスト: この技術プレゼンテーション動画の要点を教えてください: https://youtu.be/XwnEtZxaokg

🤖 レスポンス: 動画の字幕を分析します...
get_available_languages を実行中...

get_subtitle を実行中 (言語: en)...

技術プレゼンテーションの要点：
1. [技術的なポイント1]
2. [実装に関するポイント2]
3. [ベストプラクティス3]
```

**成功サインの確認**:
- ✅ Azure Functions Core Toolsが正常に起動（ポート7071で稼働）
- ✅ Functions ランタイムでのMCP動作確認
- ✅ 2つのツールが発見されたことをログで確認
- ✅ YouTube動画の字幕取得と分析が正常動作

### パターン4: MCP Inspector + Container Apps Local
**特徴**: Container Apps版のローカルデバッグ・検証

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `http://0.0.0.0:5202/sse`
- **重要**: MCP Inspector v0.3.0 を使用

**テスト手順**:
1. **Container Apps ローカルサーバーが起動していることを確認**
2. **List Tools**ボタンをクリック
3. 各ツールを順次テスト:

   **get_available_languages**:
   - videoUrl: `"https://youtu.be/XwnEtZxaokg"`

   **get_subtitle**:
   - videoUrl: `"https://youtu.be/XwnEtZxaokg"`
   - languageCode: `"en"`

**実行結果サンプル**:
```
get_available_languages:
{
  "languages": [
    {"code": "en", "name": "English"},
    {"code": "ja", "name": "Japanese"},
    {"code": "ko", "name": "Korean"}
  ]
}

get_subtitle:
{
  "subtitle": "Welcome to this presentation about...",
  "language": "en",
  "duration": "00:10:30"
}
```

**成功サインの確認**:
- ✅ MCP Inspector のツール一覧に2つのツールが表示
- ✅ 各ツールの個別実行が正常動作
- ✅ YouTube動画の言語一覧取得が成功
- ✅ 指定言語での字幕取得が成功
- ✅ UIでツールのパラメータと結果が確認可能

### パターン5: MCP Inspector + Container Apps Container
**特徴**: Container Apps版のコンテナデバッグ

**事前確認**:
```bash
docker ps | grep mcp-youtube-subtitles-extractor     # コンテナが Up 状態
GUID=$(docker exec mcp-youtube-subtitles-extractor printenv Mcp__ApiKey)  # アクセスキー取得
echo "Access Key: $GUID"
curl -I "http://localhost:8080/sse?code=$GUID"      # 405 Method Not Allowed が返ることを確認
```

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `http://0.0.0.0:8080/sse?code=<GUID>`

**テスト手順**:
1. **List Tools**ボタンをクリック
2. 各ツールを順次テスト（パターン4と同様）

**実行結果サンプル**:
```
get_available_languages:
{
  "languages": [
    {"code": "en", "name": "English"},
    {"code": "ja", "name": "Japanese"}
  ]
}

get_subtitle (言語: ja):
{
  "subtitle": "このプレゼンテーションへようこそ...",
  "language": "ja",
  "duration": "00:10:30"
}
```

**成功サインの確認**:
- ✅ MCP Inspector がコンテナポート8080に正常接続
- ✅ アクセスキー認証が成功
- ✅ コンテナ内MCPサーバーの2つのツールが表示
- ✅ YouTube字幕取得機能が正常動作
- ✅ コンテナ環境でのセキュアな動作確認

### パターン6: MCP Inspector + Functions Local
**特徴**: Functions版のローカルデバッグ

**事前確認**:
```bash
# Functions が起動していることを確認
curl -I http://localhost:7071/runtime/webhooks/mcp/sse  # 405 Method Not Allowed が返ることを確認
```

**Inspector設定**:
- **Transport Type**: `SSE`
- **URL**: `http://0.0.0.0:7071/runtime/webhooks/mcp/sse`

**テスト手順**:
1. **List Tools**ボタンをクリック
2. 各ツールを順次テスト（パターン4と同様）

**実行結果サンプル**:
```
get_available_languages:
{
  "languages": [
    {"code": "en", "name": "English"},
    {"code": "ja", "name": "Japanese"},
    {"code": "ko", "name": "Korean"}
  ]
}

get_subtitle (言語: ko):
{
  "subtitle": "이 프레젠테이션에 오신 것을 환영합니다...",
  "language": "ko",
  "duration": "00:10:30"
}
```

**成功サインの確認**:
- ✅ MCP Inspector がFunctionsランタイムに正常接続
- ✅ Functions特有のエンドポイント（`/runtime/webhooks/mcp/sse`）で接続成功
- ✅ サーバーレス環境での2つのツールが表示
- ✅ YouTube字幕取得機能が正常動作
- ✅ Functions環境での動作確認

## 📊 パターン選択ガイド

### 用途別推奨パターン

| 用途 | 推奨パターン | 理由 |
|------|-------------|------|
| **クイック動作確認** | パターン4 (Inspector + Container Apps Local) | 最速、ビルド不要、UI確認可能 |
| **開発時のIDE統合** | パターン1 (VS Code + Container Apps Local) | IDE内でシームレス、自然言語操作 |
| **コンテナ動作確認** | パターン5 (Inspector + Container Apps Container) | 本番環境に近い状態、UI確認 |
| **Functions環境テスト** | パターン6 (Inspector + Functions Local) | サーバーレス環境、詳細確認可能 |
| **セキュリティテスト** | パターン2,5 (Container Apps + アクセスキー) | 認証機能の確認 |
| **自然言語操作** | パターン1-3 (VS Code系) | 会話形式でのYouTube字幕操作 |

### Phase 1完了状況確認

```bash
# Phase 1完了状況の一括確認
echo "=== YouTube Subtitles Extractor MCP Server Status Check ==="

# 1. Container Apps Local
echo "1. Container Apps Local Server:"
curl -s -I http://localhost:5202/sse | head -1 | grep -q "405" && echo "✅ Running" || echo "❌ Not running"

# 2. Container Apps Container
echo "2. Container Apps Container:"
docker ps | grep -q mcp-youtube-subtitles-extractor && echo "✅ Running" || echo "❌ Not running"

# 3. Functions Local
echo "3. Functions Local:"
curl -s -I http://localhost:7071/runtime/webhooks/mcp/sse | head -1 | grep -q "405" && echo "✅ Running" || echo "❌ Not running"
```

## 🚀 クイックスタート

### 最も簡単な開始方法

1. **MCP Inspector + Container Apps Local** (最速デバッグ)
```bash
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/containerapp
dotnet run --project ./src/McpYouTubeSubtitlesExtractor.ContainerApp

# 別ターミナル
npx @modelcontextprotocol/inspector@0.3.0
```

2. **VS Code + Container Apps Local** (自然言語操作)
```bash
# ターミナル1
cd /workspaces/mcp-dotnet-samples/youtube-subtitles-extractor/containerapp
dotnet run --project ./src/McpYouTubeSubtitlesExtractor.ContainerApp

# ターミナル2
cp containerapp/.vscode/mcp.json /workspaces/mcp-dotnet-samples/.vscode/mcp.json
# VS Codeでチャット開始
```

## 📝 YouTube字幕操作の学習例

### 基本的なワークフロー

1. **利用可能言語の確認**
   ```
   この動画で利用可能な字幕言語を教えてください: https://youtu.be/XwnEtZxaokg
   ```

2. **特定言語での字幕取得**
   ```
   この動画の日本語字幕を取得してください: https://youtu.be/XwnEtZxaokg
   ```

3. **動画の要約作成**
   ```
   この動画を5つの要点でまとめてください: https://youtu.be/XwnEtZxaokg
   ```

4. **技術的内容の分析**
   ```
   この技術プレゼンテーションの主なトピックを教えてください: https://youtu.be/XwnEtZxaokg
   ```

### 高度な操作例

- **多言語字幕の比較分析**
- **長時間動画の章分けと要約**
- **専門用語の抽出と解説**
- **プレゼンテーション資料との対比**

## 🔧 トラブルシューティング

### よくある問題と解決方法

#### "YouTube動画にアクセスできません" エラー
- **原因**: 動画が非公開、地域制限、削除済み
- **解決**: 公開動画のURLを使用、テスト用動画で確認

#### "字幕が利用できません" エラー
- **原因**: 字幕が存在しない、または指定言語が利用不可
- **解決**: `get_available_languages`で利用可能言語を事前確認

#### Container Apps のアクセスキーエラー
- **原因**: GUIDの生成・設定ミス
- **解決**: 
  ```bash
  # 新しいGUIDを生成
  GUID=$(uuidgen)
  echo "New GUID: $GUID"
  
  # コンテナを再起動
  docker stop mcp-youtube-subtitles-extractor
  docker rm mcp-youtube-subtitles-extractor
  docker run -d -p 8080:8080 -e Mcp__ApiKey=$GUID --name mcp-youtube-subtitles-extractor mcp-youtube-subtitles-extractor:latest
  ```

#### Functions Core Tools 起動エラー
- **原因**: ポート競合、Functions Core Tools未インストール
- **解決**: 
  ```bash
  # ポート確認
  netstat -tulpn | grep :7071
  
  # Functions Core Tools 再インストール
  npm install -g azure-functions-core-tools@4 --unsafe-perm true
  ```

#### Inspector 接続エラー（YouTube特有）
- **原因**: Inspector バージョン、ライブラリ制限
- **解決**: 
  - `npx @modelcontextprotocol/inspector@0.3.0` を使用
  - リモートデプロイではなくローカル実行を確認

### 制限事項の理解

#### YouTube字幕抽出ライブラリの制限
- **リモート実行不可**: Azure上のリモートサーバーでは動作しません
- **対応動画**: 公開YouTube動画のみ
- **対応字幕**: 自動生成またはアップロード者提供の字幕のみ

## 🎓 学習ポイント

### MCP YouTube Subtitles Extractorで学べること

1. **外部API連携**: YouTubeからのデータ取得とMCPツール統合
2. **多言語対応**: 複数言語での字幕処理
3. **コンテンツ分析**: 字幕データを使った動画分析・要約
4. **セキュリティ実装**: アクセスキーによる認証機能
5. **環境の違い**: Container Apps vs Functions の実装比較

### 実世界での応用例

- **教育プラットフォーム**: 講義動画の自動要約・翻訳
- **コンテンツ分析**: マーケティング動画の効果測定
- **アクセシビリティ向上**: 多言語字幕提供システム
- **研究・調査**: 動画コンテンツの定性分析支援

このYouTube字幕抽出機能を通じて、MCPプロトコルでの**外部サービス連携**と**マルチメディア処理**の開発方法を学習できます。
