# MCPサーバー: Markdown to HTML

これは、MarkdownテキストをHTMLに変換するMCPサーバーです。

## 前提条件

- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Visual Studio Code](https://code.visualstudio.com/) と以下の拡張機能
  - [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) 拡張機能
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)
- [Docker Desktop](https://docs.docker.com/get-started/get-docker/)

## はじめに

- [ASP.NET Core MCPサーバー（STDIO）をローカルでコンテナにビルド](#aspnet-core-mcpサーバーstdioをローカルでコンテナにビルド)
- [ASP.NET Core MCPサーバー（SSE）をローカルで実行](#aspnet-core-mcpサーバーsseをローカルで実行)
- [ASP.NET Core MCPサーバー（SSE）をローカルのコンテナで実行](#aspnet-core-mcpサーバーsseをローカルのコンテナで実行)
- [ASP.NET Core MCPサーバー（SSE）をリモートで実行](#aspnet-core-mcpサーバーsseをリモートで実行)
- [MCPサーバーをMCPホスト/クライアントに接続](#mcpサーバーをmcpホストクライアントに接続)
  - [VS Code + Agent Mode + ローカルMCPサーバー（STDIO）](#vs-code--agent-mode--ローカルmcpサーバーstdio)
  - [VS Code + Agent Mode + ローカルMCPサーバー（STDIO）コンテナ](#vs-code--agent-mode--ローカルmcpサーバーstdioコンテナ)
  - [VS Code + Agent Mode + ローカルMCPサーバー（SSE）](#vs-code--agent-mode--ローカルmcpサーバーsse)
  - [VS Code + Agent Mode + ローカルMCPサーバー（SSE）コンテナ](#vs-code--agent-mode--ローカルmcpサーバーsseコンテナ)
  - [VS Code + Agent Mode + リモートMCPサーバー（SSE）](#vs-code--agent-mode--リモートmcpサーバーsse)
  - [MCP Inspector + ローカルMCPサーバー（STDIO）](#mcp-inspector--ローカルmcpサーバーstdio)
  - [MCP Inspector + ローカルMCPサーバー（STDIO）コンテナ](#mcp-inspector--ローカルmcpサーバーstdioコンテナ)
  - [MCP Inspector + ローカルMCPサーバー（SSE）](#mcp-inspector--ローカルmcpサーバーsse)
  - [MCP Inspector + ローカルMCPサーバー（SSE）コンテナ](#mcp-inspector--ローカルmcpサーバーsseコンテナ)
  - [MCP Inspector + リモートMCPサーバー（SSE）](#mcp-inspector--リモートmcpサーバーsse)

### ASP.NET Core MCPサーバー（STDIO）をローカルでコンテナにビルド

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. MCPサーバーアプリをコンテナイメージとしてビルドします。

    ```bash
    cd $REPOSITORY_ROOT/markdown-to-html
    docker build -f Dockerfile.stdio -t mcp-md2html-stdio:latest .
    ```

### ASP.NET Core MCPサーバー（SSE）をローカルで実行

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. MCPサーバーアプリを実行します。

    ```bash
    cd $REPOSITORY_ROOT/markdown-to-html
    dotnet run --project ./src/McpMarkdownToHtml.ContainerApp
    ```

   > **注意**: [Microsoft Tech Community](https://techcommunity.microsoft.com/)向けにMarkdownテキストを変換する場合、以下のパラメータが役立ちます。
   >
   > - `--tech-community`/`-tc`: Microsoft Tech Community向けにMarkdownテキストをHTMLに変換することを示すスイッチ。
   > - `--extra-paragraph`/`-p`: `--tags`引数で定義されたHTML要素間に段落を追加するかどうかを示すスイッチ。
   > - `--tags`: 段落を追加するHTMLタグのコンマ区切りリスト。デフォルト値は `p,blockquote,h1,h2,h3,h4,h5,h6,ol,ul,dl`
   >
   > これらのパラメータを使用して、以下のようにMCPサーバーを実行できます：
   >
   > ```bash
   > dotnet run --project ./src/McpMarkdownToHtml.ContainerApp -- -tc -p --tags "p,h1,h2,h3,ol,ul,dl"
   > ```

### ASP.NET Core MCPサーバー（SSE）をローカルのコンテナで実行

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. MCPサーバーアプリをコンテナイメージとしてビルドします。

    ```bash
    cd $REPOSITORY_ROOT/markdown-to-html
    docker build -f Dockerfile.sse -t mcp-md2html-sse:latest .
    ```

3. MCPサーバーアプリをコンテナで実行します。

    ```bash
    docker run -d -p 8080:8080 --name mcp-md2html-sse mcp-md2html-sse:latest
    ```

   > **注意**: [Microsoft Tech Community](https://techcommunity.microsoft.com/)向けにMarkdownテキストを変換する場合、以下のパラメータが役立ちます。
   >
   > - `--tech-community`/`-tc`: Microsoft Tech Community向けにMarkdownテキストをHTMLに変換することを示すスイッチ。
   > - `--extra-paragraph`/`-p`: `--tags`引数で定義されたHTML要素間に段落を追加するかどうかを示すスイッチ。
   > - `--tags`: 段落を追加するHTMLタグのコンマ区切りリスト。デフォルト値は `p,blockquote,h1,h2,h3,h4,h5,h6,ol,ul,dl`
   >
   > これらのパラメータを使用して、以下のようにMCPサーバーを実行できます：
   >
   > ```bash
   > docker run -d -p 8080:8080 --name mcp-md2html-sse mcp-md2html-sse:latest -tc -p --tags "p,h1,h2,h3,ol,ul,dl"
   > ```

### ASP.NET Core MCPサーバー（SSE）をリモートで実行

1. Azureにログインします。

    ```bash
    # Azure Developer CLIでログイン
    azd auth login
    ```

2. MCPサーバーアプリをAzureにデプロイします。

    ```bash
    azd up
    ```

   プロビジョニングとデプロイの際に、サブスクリプションID、場所、環境名の入力が求められます。

3. デプロイが完了したら、以下のコマンドを実行して情報を取得します：

   - Azure Container Apps FQDN:

     ```bash
     azd env get-value AZURE_RESOURCE_MCP_MD2HTML_FQDN
     ```

### MCPサーバーをMCPホスト/クライアントに接続

#### VS Code + Agent Mode + ローカルMCPサーバー（STDIO）

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. `mcp.json`をリポジトリのルートにコピーします。

    ```bash
    mkdir -p $REPOSITORY_ROOT/.vscode
    cp $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.stdio.local.json \
       $REPOSITORY_ROOT/.vscode/mcp.json
    ```

    ```powershell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/.vscode -Force
    Copy-Item -Path $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.stdio.local.json `
              -Destination $REPOSITORY_ROOT/.vscode/mcp.json -Force
    ```

3. `F1`キーまたはWindowsでは`Ctrl`+`Shift`+`P`、Mac OSでは`Cmd`+`Shift`+`P`でコマンドパレットを開き、`MCP: List Servers`を検索します。
4. `mcp-md2html-stdio-local`を選択し、`Start Server`をクリックします。
5. プロンプトが表示されたら、`McpMarkdownToHtml.ConsoleApp`プロジェクトの絶対ディレクトリを入力します。
6. 以下のようなプロンプトを入力します：

    ```text
    ハイライトされたMarkdownテキストをHTMLに変換し、converted.htmlに保存してください
    ```

7. 結果を確認します。

#### VS Code + Agent Mode + ローカルMCPサーバー（STDIO）コンテナ

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. `mcp.json`をリポジトリのルートにコピーします。

    ```bash
    mkdir -p $REPOSITORY_ROOT/.vscode
    cp $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.stdio.container.json \
       $REPOSITORY_ROOT/.vscode/mcp.json
    ```

    ```powershell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/.vscode -Force
    Copy-Item -Path $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.stdio.container.json `
              -Destination $REPOSITORY_ROOT/.vscode/mcp.json -Force
    ```

3. `F1`キーまたはWindowsでは`Ctrl`+`Shift`+`P`、Mac OSでは`Cmd`+`Shift`+`P`でコマンドパレットを開き、`MCP: List Servers`を検索します。
4. `mcp-md2html-stdio-container`を選択し、`Start Server`をクリックします。
5. 以下のようなプロンプトを入力します：

    ```text
    ハイライトされたMarkdownテキストをHTMLに変換し、converted.htmlに保存してください
    ```

6. 結果を確認します。

#### VS Code + Agent Mode + ローカルMCPサーバー（SSE）

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. `mcp.json`をリポジトリのルートにコピーします。

    ```bash
    mkdir -p $REPOSITORY_ROOT/.vscode
    cp $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.sse.local.json \
       $REPOSITORY_ROOT/.vscode/mcp.json
    ```

    ```powershell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/.vscode -Force
    Copy-Item -Path $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.sse.local.json `
              -Destination $REPOSITORY_ROOT/.vscode/mcp.json -Force
    ```

3. `F1`キーまたはWindowsでは`Ctrl`+`Shift`+`P`、Mac OSでは`Cmd`+`Shift`+`P`でコマンドパレットを開き、`MCP: List Servers`を検索します。
4. `mcp-md2html-sse-local`を選択し、`Start Server`をクリックします。
5. 以下のようなプロンプトを入力します：

    ```text
    ハイライトされたMarkdownテキストをHTMLに変換し、converted.htmlに保存してください
    ```

6. 結果を確認します。

#### VS Code + Agent Mode + ローカルMCPサーバー（SSE）コンテナ

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. `mcp.json`をリポジトリのルートにコピーします。

    ```bash
    mkdir -p $REPOSITORY_ROOT/.vscode
    cp $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.sse.container.json \
       $REPOSITORY_ROOT/.vscode/mcp.json
    ```

    ```powershell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/.vscode -Force
    Copy-Item -Path $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.sse.container.json `
              -Destination $REPOSITORY_ROOT/.vscode/mcp.json -Force
    ```

3. `F1`キーまたはWindowsでは`Ctrl`+`Shift`+`P`、Mac OSでは`Cmd`+`Shift`+`P`でコマンドパレットを開き、`MCP: List Servers`を検索します。
4. `mcp-md2html-sse-container`を選択し、`Start Server`をクリックします。
5. 以下のようなプロンプトを入力します：

    ```text
    ハイライトされたMarkdownテキストをHTMLに変換し、converted.htmlに保存してください
    ```

6. 結果を確認します。

#### VS Code + Agent Mode + リモートMCPサーバー（SSE）

1. リポジトリのルートを取得します。

    ```bash
    # bash/zsh
    REPOSITORY_ROOT=$(git rev-parse --show-toplevel)
    ```

    ```powershell
    # PowerShell
    $REPOSITORY_ROOT = git rev-parse --show-toplevel
    ```

2. `mcp.json`をリポジトリのルートにコピーします。

    ```bash
    mkdir -p $REPOSITORY_ROOT/.vscode
    cp $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.sse.remote.json \
       $REPOSITORY_ROOT/.vscode/mcp.json
    ```

    ```powershell
    New-Item -Type Directory -Path $REPOSITORY_ROOT/.vscode -Force
    Copy-Item -Path $REPOSITORY_ROOT/markdown-to-html/.vscode/mcp.sse.remote.json `
              -Destination $REPOSITORY_ROOT/.vscode/mcp.json -Force
    ```

3. `F1`キーまたはWindowsでは`Ctrl`+`Shift`+`P`、Mac OSでは`Cmd`+`Shift`+`P`でコマンドパレットを開き、`MCP: List Servers`を検索します。
4. `mcp-md2html-sse-remote`を選択し、`Start Server`をクリックします。
5. Azure Container Apps FQDNを入力します。
6. 以下のようなプロンプトを入力します：

    ```text
    ハイライトされたMarkdownテキストをHTMLに変換し、converted.htmlに保存してください
    ```

7. 結果を確認します。

#### MCP Inspector + ローカルMCPサーバー（STDIO）

1. MCP Inspectorを実行します。

    ```bash
    npx @modelcontextprotocol/inspector node build/index.js
    ```

2. Webブラウザを開いて、アプリで表示されるURL（例：http://localhost:6274）からMCP Inspector Webアプリにアクセスします。
3. トランスポートタイプを`STDIO`に設定します。
4. コマンドを`dotnet`に設定します。
5. コンソールアプリプロジェクトを指す引数を設定し、**Connect**をクリックします：

    ```text
    run --project {{absolute/path/to/markdown-to-html}}/src/McpMarkdownToHtml.ConsoleApp
    ```

   > **注意**:
   >
   > 1. [Microsoft Tech Community](https://techcommunity.microsoft.com/)向けにMarkdownテキストを変換する場合、以下のパラメータが役立ちます。
   >
   >    - `--tech-community`/`-tc`: Microsoft Tech Community向けにMarkdownテキストをHTMLに変換することを示すスイッチ。
   >    - `--extra-paragraph`/`-p`: `--tags`引数で定義されたHTML要素間に段落を追加するかどうかを示すスイッチ。
   >    - `--tags`: 段落を追加するHTMLタグのコンマ区切りリスト。デフォルト値は `p,blockquote,h1,h2,h3,h4,h5,h6,ol,ul,dl`
   >
   >    これらのパラメータを使用すると、引数の値は以下のようになります：
   >
   >     ```bash
   >     run --project {{absolute/path/to/markdown-to-html}}/src/McpMarkdownToHtml.ConsoleApp -- -tc -p --tags "p,h1,h2,h3,ol,ul,dl"
   >     ```
   >
   > 2. プロジェクトパスは絶対パスでなければなりません。

6. **List Tools**をクリックします。
7. ツールをクリックして、適切な値で**Run Tool**を実行します。

#### MCP Inspector + ローカルMCPサーバー（STDIO）コンテナ

1. MCP Inspectorを実行します。

    ```bash
    npx @modelcontextprotocol/inspector node build/index.js
    ```

2. Webブラウザを開いて、アプリで表示されるURL（例：http://localhost:6274）からMCP Inspector Webアプリにアクセスします。
3. トランスポートタイプを`STDIO`に設定します。
4. コマンドを`docker`に設定します。
5. コンソールアプリプロジェクトを指す引数を設定し、**Connect**をクリックします：

    ```text
    run -i --rm mcp-md2html-stdio:latest
    ```

6. **List Tools**をクリックします。
7. ツールをクリックして、適切な値で**Run Tool**を実行します。

#### MCP Inspector + ローカルMCPサーバー（SSE）

1. MCP Inspectorを実行します。

    ```bash
    npx @modelcontextprotocol/inspector node build/index.js
    ```

2. Webブラウザを開いて、アプリで表示されるURL（例：http://localhost:6274）からMCP Inspector Webアプリにアクセスします。
3. トランスポートタイプを`SSE`に設定します。
4. 実行中のFunctionアプリのSSEエンドポイントのURLを設定し、**Connect**をクリックします：

    ```text
    http://0.0.0.0:5280/sse
    ```

5. **List Tools**をクリックします。
6. ツールをクリックして、適切な値で**Run Tool**を実行します。

#### MCP Inspector + ローカルMCPサーバー（SSE）コンテナ

1. MCP Inspectorを実行します。

    ```bash
    npx @modelcontextprotocol/inspector node build/index.js
    ```

2. Webブラウザを開いて、アプリで表示されるURL（例：http://localhost:6274）からMCP Inspector Webアプリにアクセスします。
3. トランスポートタイプを`SSE`に設定します。
4. 実行中のFunctionアプリのSSEエンドポイントのURLを設定し、**Connect**をクリックします：

    ```text
    http://0.0.0.0:8080/sse
    ```

5. **List Tools**をクリックします。
6. ツールをクリックして、適切な値で**Run Tool**を実行します。

#### MCP Inspector + リモートMCPサーバー（SSE）

1. MCP Inspectorを実行します。

    ```bash
    npx @modelcontextprotocol/inspector node build/index.js
    ```

2. Webブラウザを開いて、アプリで表示されるURL（例：http://0.0.0.0:6274）からMCP Inspector Webアプリにアクセスします。
3. トランスポートタイプを`SSE`に設定します。
4. 実行中のFunctionアプリのSSEエンドポイントのURLを設定し、**Connect**をクリックします：

    ```text
    https://<acaapp-server-fqdn>/sse
    ```

5. **List Tools**をクリックします。
6. ツールをクリックして、適切な値で**Run Tool**を実行します。
