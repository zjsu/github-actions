# github-actions

用于集中维护 Quarto 站点部署相关的 GitHub reusable workflow。

当前提供的公共 workflow：

- [`.github/workflows/quarto-site.yml`](/home/hjw/Dev/github.com/zjsu/github-actions/.github/workflows/quarto-site.yml)

## 适用场景

适合多个 Quarto 项目共用同一套部署逻辑，只把这些差异留给调用方配置：

- 触发条件和 `paths`
- 发布目录和发布分支
- 是否安装中文字体
- Quarto 版本
- 是否安装 Chrome
- commit message

公共 workflow 固定处理的流程是：

```text
checkout -> 安装字体 -> 安装 Quarto -> 安装 chrome-headless-shell -> render -> 发布到分支
```

## 使用方式

在业务仓库中通过 `workflow_call` 调用：

```yaml
name: Deploy Quarto Site

on:
  push:
    branches: [main, master]
    paths:
      - "**.qmd"
      - "_quarto.yml"
      - "slides/**"
      - ".github/workflows/deploy-site.yml"
  workflow_dispatch:

permissions:
  contents: write

concurrency:
  group: deploy-site-${{ github.ref }}
  cancel-in-progress: true

jobs:
  deploy:
    uses: zjsu/github-actions/.github/workflows/quarto-site.yml@v1
    with:
      commit_message: "deploy: ${{ github.event.head_commit.message }}"
```

## 可选参数

`quarto-site.yml` 当前支持这些输入参数：

- `quarto_version`，默认 `release`
- `install_chinese_fonts`，默认 `true`
- `install_chrome`，默认 `true`
- `render_path`，默认 `.`
- `publish_dir`，默认 `./_site`
- `publish_branch`，默认 `site`
- `force_orphan`，默认 `true`
- `commit_message`，默认 `deploy: update site`
- `timeout_minutes`，默认 `10`

## 约定与建议

- 调用方显式声明 `permissions.contents: write`
- 调用时优先使用带 tag 的版本，例如 `@v1`，不要直接引用 `@main`
- `on.push.paths`、`branches` 等触发条件保留在业务仓库，不放进公共 workflow
