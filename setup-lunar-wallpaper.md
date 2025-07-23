# lunar-wallpaperリポジトリでClaude Flow自動化を設定する手順

## 🚀 セットアップ方法

### 1. GitHub Secretsの設定

lunar-wallpaperリポジトリに以下のSecretを設定してください：

1. **リポジトリページにアクセス**: https://github.com/Marimo-317/lunar-wallpaper
2. **Settings → Secrets and variables → Actions** に移動
3. **New repository secret** をクリック
4. 以下のSecretを追加：
   - **Name**: `CLAUDE_CODE_OAUTH_TOKEN`
   - **Value**: claude_flow_windowsで使用している同じトークン

### 2. ワークフローファイルの追加

以下の手順でワークフローファイルを追加します：

#### オプション1: GitHub Web UIから直接追加
1. https://github.com/Marimo-317/lunar-wallpaper にアクセス
2. **Create new file** をクリック
3. ファイル名: `.github/workflows/claude-flow-auto-resolver.yml`
4. 以下の内容をコピー＆ペースト：

```yaml
name: Claude Flow Auto Issue Resolver
on:
  issues:
    types: [opened, edited, labeled, reopened]
  issue_comment:
    types: [created]
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  auto-resolve:
    runs-on: ubuntu-latest
    if: |
      (github.event.issue != null || github.event.comment != null) &&
      (contains(github.event.issue.body || github.event.comment.body, '@claude-flow-automation') ||
       contains(github.event.issue.body || github.event.comment.body, '@claude'))
    permissions:
      issues: write
      pull-requests: write
      contents: write
      actions: read
      id-token: write
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install minimal dependencies
        run: |
          # Check if package.json exists
          if [ -f "package.json" ]; then
            npm ci --legacy-peer-deps || npm install --legacy-peer-deps
          else
            # Initialize npm project if needed
            npm init -y
          fi
          npm install -g @anthropic-ai/claude-code@latest
      
      - name: Claude Code Integration
        uses: anthropics/claude-code-action@v0.0.33
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          issue-number: ${{ github.event.issue.number || github.event.pull_request.number }}
          
      - name: Auto-resolve with Claude
        if: contains(github.event.issue.body || github.event.comment.body, '@claude-flow-automation')
        run: |
          echo "🤖 Claude Flow Automation Triggered"
          
          # Create a simple automation script inline
          cat > auto-resolve.js << 'EOF'
          const { Octokit } = require('@octokit/rest');
          
          async function autoResolve() {
              const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });
              const issue = {
                  number: process.env.ISSUE_NUMBER,
                  title: process.env.ISSUE_TITLE || 'Issue',
                  body: process.env.ISSUE_BODY || ''
              };
              
              console.log(`📊 Analyzing issue #${issue.number}: ${issue.title}`);
              
              // Post initial comment
              await octokit.rest.issues.createComment({
                  owner: process.env.GITHUB_REPOSITORY_OWNER,
                  repo: process.env.GITHUB_REPOSITORY_NAME,
                  issue_number: parseInt(issue.number),
                  body: `🤖 **Claude Flow Automation Activated**
          
          I'm analyzing this issue and will attempt to create a solution automatically.
          
          **Status:** 🔄 Processing...`
              });
              
              // Use Claude Code to analyze and solve
              console.log('🧠 Using Claude Code to analyze and implement solution...');
              
              // Simple implementation approach
              const branchName = `fix/issue-${issue.number}`;
              console.log(`📝 Working on branch: ${branchName}`);
              
              // The actual solution will be implemented by Claude Code action
              console.log('✅ Claude Code integration will handle the implementation');
          }
          
          autoResolve().catch(console.error);
          EOF
          
          # Install required packages
          npm install @octokit/rest
          
          # Run the automation
          node auto-resolve.js
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ISSUE_NUMBER: ${{ github.event.issue.number }}
          ISSUE_TITLE: ${{ github.event.issue.title }}
          ISSUE_BODY: ${{ github.event.issue.body }}
          GITHUB_REPOSITORY_OWNER: ${{ github.repository_owner }}
          GITHUB_REPOSITORY_NAME: ${{ github.event.repository.name }}
      
      - name: Update Issue Status
        if: always()
        run: |
          if [ "${{ job.status }}" == "success" ]; then
            echo "✅ Automation completed successfully"
          else
            echo "⚠️ Automation encountered issues"
          fi
```

5. **Commit new file** をクリック

#### オプション2: GitHub CLIから追加
```bash
# ワークフローファイルをbase64エンコード
base64 -w 0 lunar-wallpaper-workflow.yml > workflow-base64.txt

# APIを使用してファイルを作成
gh api repos/Marimo-317/lunar-wallpaper/contents/.github/workflows/claude-flow-auto-resolver.yml \
  --method PUT \
  --field message="Add Claude Flow automation workflow" \
  --field content="$(cat workflow-base64.txt)" \
  --field branch="master"
```

### 3. 使用方法

設定が完了したら、lunar-wallpaperリポジトリで以下のように使用できます：

1. **新しいIssueを作成**
2. Issue本文に `@claude-flow-automation` または `@claude` とメンション
3. 自動化システムが起動し、問題を分析して解決を試みます

### 4. 動作確認

- **GitHub Actions**: https://github.com/Marimo-317/lunar-wallpaper/actions でワークフローの実行状況を確認
- **Secrets設定**: Settings → Secrets → Actions で `CLAUDE_CODE_OAUTH_TOKEN` が設定されていることを確認

## 🔧 トラブルシューティング

### ワークフローが動作しない場合
1. GitHub Secretsが正しく設定されているか確認
2. ワークフローファイルが正しいパスに配置されているか確認
3. GitHub Actionsが有効になっているか確認

### 権限エラーが発生する場合
1. リポジトリの Settings → Actions → General で権限設定を確認
2. "Workflow permissions" で "Read and write permissions" を選択

## 📝 注意事項

- このワークフローは簡略版で、基本的なClaude Code統合のみを含んでいます
- より高度な機能（データベース、マルチエージェントなど）が必要な場合は、claude_flow_windowsリポジトリの完全版を参考にしてください
- lunar-wallpaperリポジトリに`package.json`がない場合、自動的に初期化されます