# GitHub Actions 使用说明

本项目已配置 GitHub Actions 自动化工作流，用于自动编译 LaTeX 文档并发布 PDF。

## 功能特性

### 1. 自动编译 LaTeX 为 PDF

每次推送代码到仓库时（除了 `pdf` 分支），GitHub Actions 会自动：
- 使用 XeLaTeX 编译器（通过 latexmk）编译 `main.tex` 文件
- 生成带时间戳的 PDF 文档（格式：`thesis_YYYYMMDD_HHMMSS.pdf`）
- 将 PDF 提交到专门的 `pdf` 分支用于在线预览
- 自动创建 Release 发布

### 2. PDF 分支存储

所有编译的 PDF 都存储在 `pdf` 分支中：
- 每个 PDF 文件以时间戳命名，方便追溯
- 可以通过 GitHub 直接在线预览
- 不会污染主分支的提交历史

### 3. 自动 Release 发布

**每次推送或手动触发都会创建 Release**（不再需要 tag）：
- Release 名称格式：`build-YYYYMMDD_HHMMSS`
- 包含详细的构建信息（时间、提交哈希、分支、触发者等）
- 自动生成最近提交的变更摘要
- 提供多种下载和预览方式

### 4. 在线预览

编译完成后，可以通过以下方式**直接在浏览器中预览 PDF**（无需下载）：

1. **从 Release 页面预览**：
   - 进入 GitHub 仓库的 **Releases** 标签页
   - 点击最新的 Release
   - 在 Release 说明中点击 **"在线预览 PDF"** 链接

2. **从 PDF 分支预览**：
   - 访问 `https://github.com/你的用户名/OUC-LaTex/blob/pdf/thesis_YYYYMMDD_HHMMSS.pdf`
   - GitHub 会自动渲染 PDF 供在线查看

3. **下载后预览**：
   - 从 Release Assets 下载 PDF
   - 从 PDF 分支直接下载

## 触发方式

工作流可以通过以下两种方式触发：

### 1. 推送代码（自动触发）

```bash
git add .
git commit -m "更新论文内容"
git push
```

推送后，GitHub Actions 会自动：
1. 编译 LaTeX 文档
2. 将 PDF 推送到 `pdf` 分支
3. 创建新的 Release

### 2. 手动触发

1. 进入 GitHub 仓库的 **Actions** 标签页
2. 点击左侧的 **LaTeX Compilation and Release** 工作流
3. 点击右侧的 **Run workflow** 按钮
4. 选择要运行的分支
5. 点击 **Run workflow** 确认

## Release 信息说明

每个自动创建的 Release 包含以下信息：

### 构建信息
- **构建时间**：精确到秒的编译时间
- **提交哈希**：触发构建的完整提交 SHA
- **分支**：触发构建的分支名称
- **触发者**：触发构建的 GitHub 用户
- **触发方式**：push 或 workflow_dispatch

### 变更摘要
- 最近 5 次提交的简要说明
- 每条提交包含提交信息和作者

### 预览和下载
- **在线预览链接**：直接在 GitHub 上查看 PDF
- **Release 下载**：从 Release Assets 下载
- **PDF 分支下载**：从 pdf 分支直接下载

## 工作流程详解

### Build-and-Release Job（构建并发布任务）

在每次推送或手动触发时执行：

1. **检出代码**：从仓库获取完整历史（用于生成提交摘要）
2. **生成时间戳和 Release 信息**：创建唯一的构建标识
3. **编译 LaTeX**：使用 latexmk + XeLaTeX 编译
4. **重命名 PDF**：添加时间戳到文件名
5. **配置 Git**：设置 GitHub Actions bot 身份
6. **提交到 PDF 分支**：
   - 检查 `pdf` 分支是否存在
   - 如果不存在，创建新的孤立分支
   - 将 PDF 文件添加到 `pdf` 分支
   - 推送更改
7. **生成变更摘要**：收集最近的提交信息
8. **创建 Release**：
   - 使用时间戳创建 Release tag
   - 生成包含详细信息的 Release 说明
   - 附加 PDF 文件
9. **输出摘要**：在工作流运行页面显示预览和下载链接

## 常见问题

### Q: 为什么每次推送都创建 Release？

A: 这是为了方便查看每次构建的结果。每个 Release 都包含：
- 对应时间点的 PDF 快照
- 详细的构建信息
- 在线预览链接
- 变更历史

### Q: PDF 分支是什么？

A: `pdf` 分支是一个专门用于存储编译后 PDF 文件的孤立分支：
- 不包含源代码，只有 PDF 文件
- 每个 PDF 以时间戳命名
- 可以直接通过 GitHub 在线预览
- 不会污染主分支的提交历史

### Q: 如何查看历史版本的 PDF？

A: 有两种方式：
1. **通过 Releases**：访问 Releases 页面，查看任何一个历史 Release
2. **通过 PDF 分支**：访问 `pdf` 分支，可以看到所有历史 PDF 文件

### Q: 编译失败怎么办？

A: 
1. 进入 Actions 标签页查看失败的工作流
2. 点击失败的任务查看详细日志
3. 检查 LaTeX 语法错误或缺少的包
4. 修复后重新推送代码

### Q: 如何删除旧的 Release？

A: 
1. 进入 Releases 页面
2. 找到要删除的 Release
3. 点击右侧的删除按钮
4. 注意：删除 Release 不会删除 `pdf` 分支中的 PDF 文件

### Q: PDF 分支会无限增长吗？

A: 是的，每次构建都会在 `pdf` 分支添加一个新文件。如果需要清理：
1. 可以手动删除 `pdf` 分支中的旧文件
2. 或者定期清理整个 `pdf` 分支（工作流会在下次运行时重新创建）

## 技术细节

- **编译器**：latexmk + XeLaTeX
- **编译参数**：`-interaction=nonstopmode -file-line-error -xelatex`
- **PDF 命名**：`thesis_YYYYMMDD_HHMMSS.pdf`
- **Release 命名**：`build-YYYYMMDD_HHMMSS`
- **所需权限**：`contents: write`（用于创建 Release 和推送到 PDF 分支）

## 相关链接

- [GitHub Actions 文档](https://docs.github.com/cn/actions)
- [xu-cheng/latex-action](https://github.com/xu-cheng/latex-action) - LaTeX 编译 Action
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release) - Release 创建 Action
