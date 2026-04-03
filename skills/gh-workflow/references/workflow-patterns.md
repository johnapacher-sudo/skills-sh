# Workflow 模板库

本文件包含各类 GitHub Action workflow 的标准模板。生成 workflow 时以此为基础，根据用户项目的实际情况调整参数。

## 目录

- [CI 测试](#ci-测试)
- [CD 部署](#cd-部署)
- [npm 发布](#npm-发布)
- [Docker 构建](#docker-构建)
- [定时任务](#定时任务)
- [PR 检查](#pr-检查)
- [Release](#release)
- [复合场景](#复合场景)

---

## CI 测试

适用场景：代码推送或 PR 时自动运行测试、lint、类型检查。

### Node.js 项目

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

### Python 项目

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest flake8

      - name: Lint
        run: flake8 . --count --show-source --statistics

      - name: Test
        run: pytest
```

### Go 项目

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: "1.22"
          cache: true

      - name: Vet
        run: go vet ./...

      - name: Test
        run: go test -v -race ./...

      - name: Build
        run: go build ./...
```

---

## CD 部署

适用场景：代码合并到 main 分支后自动部署到目标环境。

### 部署到 Vercel（前端项目）

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: "--prod"
```

**需要配置的 Secrets：**
- `VERCEL_TOKEN`：Vercel 访问令牌
- `VERCEL_ORG_ID`：组织 ID（在 vercel.com/account 查看）
- `VERCEL_PROJECT_ID`：项目 ID（在 .vercel/project.json 中查看）

### 部署到 AWS S3（静态站点）

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install and build
        run: |
          npm ci
          npm run build

      - name: Deploy to S3
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - name: Sync to S3
        run: aws s3 sync ./dist s3://${{ secrets.S3_BUCKET }} --delete

      - name: Invalidate CloudFront cache
        run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

**需要配置的 Secrets：**
- `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`：AWS 凭证
- `S3_BUCKET`：S3 桶名称
- `CLOUDFRONT_DISTRIBUTION_ID`：CloudFront 分发 ID

### SSH 部署到自有服务器

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /var/www/myapp
            git pull origin main
            npm ci --production
            npm run build
            pm2 restart myapp
```

**需要配置的 Secrets：**
- `SERVER_HOST`：服务器 IP 或域名
- `SERVER_USER`：SSH 用户名
- `SERVER_SSH_KEY`：SSH 私钥

---

## npm 发布

适用场景：推送版本 tag 时自动发布到 npm registry。

```yaml
name: Publish to npm

on:
  push:
    tags:
      - "v*"

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: https://registry.npmjs.org/
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Publish
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**需要配置的 Secrets：**
- `NPM_TOKEN`：npm 发布令牌（`npm token create` 生成，选择 Automation 类型）

**使用方式：**
```bash
npm version patch   # 或 minor / major
git push --follow-tags
```

### 发布 scoped 包（@scope/name）

如果包名是 scoped 格式，需要添加 `--access public`：

```yaml
      - name: Publish
        run: npm publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

或者在 `package.json` 中设置：
```json
{
  "publishConfig": {
    "access": "public"
  }
}
```

---

## Docker 构建

适用场景：推送 tag 时自动构建 Docker 镜像并推送到 Docker Hub 或 GHCR。

### 推送到 Docker Hub

```yaml
name: Docker Build & Push

on:
  push:
    tags:
      - "v*"

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKER_USERNAME }}/myapp
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**需要配置的 Secrets：**
- `DOCKER_USERNAME`：Docker Hub 用户名
- `DOCKER_PASSWORD`：Docker Hub 访问令牌（在 Docker Hub Account Settings → Security 创建）

### 推送到 GHCR（GitHub Container Registry）

```yaml
name: Docker Build & Push to GHCR

on:
  push:
    tags:
      - "v*"

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

GHCR 使用 `GITHUB_TOKEN` 自动认证，无需额外配置 Secrets。

---

## 定时任务

适用场景：周期性执行依赖检查、数据同步、清理等任务。

```yaml
name: Scheduled Task

on:
  schedule:
    # UTC 时间，注意不是本地时间
    # 每5分钟: '*/5 * * * *'
    # 每天凌晨2点: '0 2 * * *'
    # 每周一上午9点: '0 9 * * 1'
    # 每月1号: '0 0 1 * *'
    - cron: "0 2 * * *"
  workflow_dispatch: # 支持手动触发，方便调试

jobs:
  run:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run task
        run: node scripts/scheduled-task.js
```

**注意事项：**
- cron 使用 **UTC 时区**，北京时间需要减 8 小时（如北京时间 10:00 = UTC 02:00）
- 实际触发时间可能有 5-15 分钟延迟
- 建议同时启用 `workflow_dispatch` 以便手动调试
- 长期不活跃的仓库可能被 GitHub 停止定时任务

### 依赖安全检查示例

```yaml
name: Security Audit

on:
  schedule:
    - cron: "0 3 * * 1" # 每周一 UTC 3:00
  workflow_dispatch:

jobs:
  audit:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Check outdated packages
        run: npx npm-check-updates
```

---

## PR 检查

适用场景：PR 提交时自动运行代码质量检查。

```yaml
name: PR Check

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  check:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # 获取完整历史用于 lint diff

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run typecheck
        continue-on-error: true

      - name: Test with coverage
        run: npm test -- --coverage

      - name: Comment PR with coverage
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const coverage = fs.readFileSync('coverage/coverage-summary.json', 'utf8');
            const data = JSON.parse(coverage);
            const total = data.total.lines;
            const body = `## Test Coverage\n| Metric | Coverage |\n|--------|----------|\n| Lines | ${total.pct}% |\n| Statements | ${data.total.statements.pct}% |\n| Functions | ${data.total.functions.pct}% |\n| Branches | ${data.total.branches.pct}% |`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
```

---

## Release

适用场景：推送 tag 时自动创建 GitHub Release 并上传构建产物。

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install and build
        run: |
          npm ci
          npm run build

      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            dist/*.zip
            dist/*.tar.gz
          generate_release_notes: true
          draft: false
          prerelease: ${{ contains(github.ref, '-rc.') || contains(github.ref, '-beta.') }}
```

**使用方式：**
```bash
npm version patch   # 或 minor / major
git push --follow-tags
```

Release notes 会从 commits 自动生成。tag 名包含 `-rc.` 或 `-beta.` 时自动标记为预发布。

---

## 复合场景

### CI + npm 发布 + Release（完整 Node.js 发布流程）

```yaml
name: Release Pipeline

on:
  push:
    tags:
      - "v*"

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm test

  publish-npm:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: https://registry.npmjs.org/
          cache: npm
      - run: npm ci
      - run: npm run build
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

  github-release:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true

```

### Monorepo CI（基于 path 过滤）

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  changes:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      backend: ${{ steps.filter.outputs.backend }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            frontend:
              - 'frontend/**'
            backend:
              - 'backend/**'

  frontend-test:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    permissions:
      contents: read
    defaults:
      run:
        working-directory: frontend
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
          cache-dependency-path: frontend/package-lock.json
      - run: npm ci
      - run: npm test

  backend-test:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    permissions:
      contents: read
    defaults:
      run:
        working-directory: backend
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
          cache-dependency-path: backend/package-lock.json
      - run: npm ci
      - run: npm test
```
