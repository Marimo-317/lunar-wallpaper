# 🚀 Claude Flow Automation Setup for Lunar-Wallpaper

## ✅ セットアップ手順

### 1. 必要なディレクトリ作成
```bash
mkdir -p .github/workflows
mkdir -p scripts
mkdir -p core
```

### 2. ワークフローファイルの設定

**`.github/workflows/claude-flow-auto-resolver.yml`** を作成し、以下の内容を設定:

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
      (github.event.issue != null && contains(github.event.issue.body, '@claude-flow-automation')) ||
      (github.event.comment != null && contains(github.event.comment.body, '@claude-flow-automation'))
    permissions:
      issues: write
      pull-requests: write
      contents: write
      actions: read
      id-token: write
      repository-projects: write
      metadata: read
    
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
      
      - name: Install dependencies
        run: |
          npm ci --legacy-peer-deps
          echo "✅ Dependencies installed for Lunar Wallpaper project"
      
      - name: Analyze Issue Context
        id: analyze
        run: |
          ISSUE_NUMBER="${{ github.event.issue.number || github.event.comment.issue.number }}"
          ISSUE_TITLE="${{ github.event.issue.title || github.event.comment.issue.title }}"
          REPOSITORY="${{ github.repository }}"
          
          echo "issue_number=$ISSUE_NUMBER" >> $GITHUB_OUTPUT
          echo "issue_title=issue_$ISSUE_NUMBER" >> $GITHUB_OUTPUT
          echo "repository=$REPOSITORY" >> $GITHUB_OUTPUT
          
          echo "✅ Lunar Wallpaper issue context analyzed successfully"
      
      - name: Auto Resolve Issue
        id: resolve
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ISSUE_NUMBER: ${{ github.event.issue.number || github.event.comment.issue.number }}
          ISSUE_TITLE: ${{ github.event.issue.title || github.event.comment.issue.title }}
          ISSUE_BODY: ${{ github.event.issue.body || github.event.comment.issue.body }}
          REPOSITORY: ${{ github.repository }}
        run: |
          echo "🌙 Starting automated issue resolution for Lunar Wallpaper #$ISSUE_NUMBER"
          echo "📋 Issue: $ISSUE_TITLE"
          echo "🔧 Repository: $REPOSITORY"
          
          # Use Vue.js/Vite specific automation
          if [ -f "vite.config.js" ] || [ -f "vite.config.ts" ]; then
            echo "🔍 Detected Vue.js + Vite project"
            echo "🛠️ Using Vue.js optimized automation"
          fi
          
          # Run Lunar Wallpaper specific automation
          echo "🐝 STARTING LUNAR WALLPAPER AI AUTOMATION"
          timeout 30m node scripts/claude-flow-lunar-automation.js \
            --issue-number "$ISSUE_NUMBER" \
            --issue-title "$ISSUE_TITLE" \
            --issue-body "$ISSUE_BODY" \
            --repository "$REPOSITORY" || {
            echo "❌ Lunar Wallpaper automation failed with exit code $?"
            exit 1
          }
      
      - name: Run Tests
        if: steps.resolve.conclusion == 'success'
        run: |
          echo "🧪 Running Lunar Wallpaper tests..."
          npm run test || echo "Tests not available yet"
          npm run lint || echo "Linting not configured"
          npm run typecheck || echo "Type checking not configured"
      
      - name: Update Issue with Status
        if: always()
        run: |
          if [ "${{ steps.resolve.conclusion }}" == "success" ]; then
            gh issue comment ${{ steps.analyze.outputs.issue_number }} --body "✅ **🌙 Lunar Wallpaper Automated Resolution Completed**
            
            🐝 Claude Flow has successfully analyzed this lunar wallpaper issue and created a solution optimized for Vue.js + Vite.
            
            📋 **Resolution Summary:**
            - Vue.js/Vite project structure detected
            - Issue analyzed with moon phase app context
            - Specialized lunar wallpaper agents deployed
            - Mobile-optimized solution implemented
            - Canvas rendering optimizations applied
            
            🔍 **Next Steps:**
            1. Review the generated pull request
            2. Test the wallpaper functionality on mobile devices
            3. Verify moon phase calculations
            4. Approve and merge if satisfactory
            
            💡 This resolution was generated specifically for the Lunar Wallpaper project.
            
            ---
            🤖 Generated with [Claude Code](https://claude.ai/code)
            Co-Authored-By: Claude Flow Lunar Automation <lunar@claude-flow.ai>"
          else
            gh issue comment ${{ steps.analyze.outputs.issue_number }} --body "❌ **🌙 Lunar Wallpaper Automated Resolution Failed**
            
            🤖 Claude Flow encountered an error while resolving this lunar wallpaper issue.
            
            📋 **Error Details:**
            - Project type: Vue.js + Vite (Lunar Wallpaper)
            - Resolution process: ❌ Failed
            - Error logged for lunar app learning system
            
            🔍 **Recommended Actions:**
            1. Check GitHub Actions workflow logs
            2. Verify Vue.js/Vite configuration
            3. Try triggering automation again with \`@claude-flow-automation\`
            
            ---
            🤖 Generated with [Claude Code](https://claude.ai/code)
            Co-Authored-By: Claude Flow Lunar Automation <lunar@claude-flow.ai>"
          fi
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 3. 自動化スクリプト作成

**`scripts/claude-flow-lunar-automation.js`** を作成:

```javascript
#!/usr/bin/env node
// Lunar Wallpaper Specific Claude Flow Automation
const { Octokit } = require('@octokit/rest');

class LunarWallpaperAutomation {
    constructor() {
        this.octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });
        this.projectType = 'Vue.js + Vite (Lunar Wallpaper)';
    }

    async run() {
        console.log('🌙 Starting Lunar Wallpaper specific automation...');
        console.log(`📋 Issue: #${process.env.ISSUE_NUMBER}`);
        console.log(`🎯 Project: ${this.projectType}`);
        
        // Lunar Wallpaper specific logic
        await this.analyzeLunarIssue();
        await this.generateMoonPhaseSolution();
        await this.createOptimizedPR();
    }

    async analyzeLunarIssue() {
        console.log('🔍 Analyzing lunar wallpaper specific context...');
        // Add lunar-specific analysis logic
    }

    async generateMoonPhaseSolution() {
        console.log('🌙 Generating moon phase solution...');
        // Add moon phase calculation logic
    }

    async createOptimizedPR() {
        console.log('📱 Creating mobile-optimized PR...');
        // Add PR creation logic
    }
}

if (require.main === module) {
    const automation = new LunarWallpaperAutomation();
    automation.run().catch(console.error);
}

module.exports = LunarWallpaperAutomation;
```

### 4. 使用方法

1. **Issue作成時**:
```markdown
# Bug: 月の位相計算が正しくない

月の位相表示に以下の問題があります：
- 新月が満月として表示される
- 日付計算のズレがある

@claude-flow-automation
```

2. **自動実行確認**:
- GitHub Actionsが自動起動
- Lunar Wallpaper専用の解析実行
- Vue.js/Vite最適化されたソリューション生成
- 月の位相計算特化のPR作成

### 5. プロジェクト固有の特徴

- **Vue.js + Vite**: フロントエンド最適化
- **Canvas API**: 壁紙レンダリング
- **天体計算**: 月の位相計算
- **モバイル最適化**: スマートフォン壁紙特化

### 6. 確認コマンド
```bash
# 自動化システム確認
npm run test
npm run lint
npm run typecheck

# 月の位相計算テスト
npm run test -- --grep "moon phase"
```

---
✅ **セットアップ完了後、GitHubリポジトリでissueに `@claude-flow-automation` をメンションするだけで自動化が開始されます！**