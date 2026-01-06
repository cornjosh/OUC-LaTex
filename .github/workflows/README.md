# GitHub Actions 使用说明

本项目已配置 GitHub Actions 自动化工作流，用于自动编译 LaTeX 文档并发布 PDF。

## 功能特性

### 1. 自动编译 LaTeX 为 PDF

每次推送代码到仓库时，GitHub Actions 会自动：
- 使用 XeLaTeX 编译器编译 `main.tex` 文件
- 生成 PDF 文档
- 将编译好的 PDF 作为 Artifact 上传，保留 90 天

### 2. 在线预览

编译完成后，可以通过以下方式预览 PDF：

1. **下载 Artifact 预览**：
   - 进入 GitHub 仓库的 **Actions** 标签页
   - 点击对应的工作流运行记录
   - 在页面底部的 **Artifacts** 区域下载 `compiled-pdf`
   - 下载后解压即可查看 PDF

2. **在线查看**：
   - 下载 Artifact 后，可以使用浏览器或 PDF 阅读器打开
   - 或者直接在 GitHub Release 中查看（如果创建了 Release）

### 3. 发布到 Releases

当推送一个以 `v` 开头的 tag 时（例如 `v1.0.0`），GitHub Actions 会自动：
- 编译 LaTeX 文档
- 创建一个新的 GitHub Release
- 将 PDF 附加到 Release 中
- 在 Release 描述中提供下载链接和预览说明

## 触发方式

工作流可以通过以下三种方式触发：

### 1. 推送代码（自动触发）

```bash
git add .
git commit -m "更新论文内容"
git push
```

推送后，GitHub Actions 会自动开始编译。

### 2. 手动触发

1. 进入 GitHub 仓库的 **Actions** 标签页
2. 点击左侧的 **LaTeX Compilation and Release** 工作流
3. 点击右侧的 **Run workflow** 按钮
4. 选择要运行的分支
5. 点击 **Run workflow** 确认

### 3. 推送 Tag 创建 Release

```bash
# 创建并推送 tag
git tag -a v1.0.0 -m "第一版论文"
git push origin v1.0.0
```

这将触发编译并创建一个新的 Release。

## 工作流程详解

### Build Job（构建任务）

在每次推送或手动触发时执行：

1. **检出代码**：从仓库获取最新代码
2. **编译 LaTeX**：使用 XeLaTeX 编译 `main.tex`
3. **重命名 PDF**：添加时间戳到文件名
4. **上传 Artifact**：将 PDF 上传为可下载的 Artifact
5. **生成摘要**：在工作流运行页面显示下载链接

### Release Job（发布任务）

仅在推送 tag 时执行（tag 需以 `v` 开头）：

1. **获取 Tag 名称**：提取 tag 名称（如 `v1.0.0`）
2. **下载 PDF**：从 build job 下载已编译的 PDF artifact
3. **重命名 PDF**：使用 tag 名称重命名文件（如 `OUC-Thesis-v1.0.0.pdf`）
4. **创建 Release**：在 GitHub Releases 中创建新版本
5. **上传 PDF**：将 PDF 作为 Release Asset 上传
6. **生成摘要**：显示 Release 链接和 PDF 下载链接

## 常见问题

### Q: 编译失败怎么办？

A: 
1. 进入 Actions 标签页查看失败的工作流
2. 点击失败的任务查看详细日志
3. 检查 LaTeX 语法错误或缺少的包
4. 修复后重新推送代码

### Q: 如何查看编译日志？

A:
1. 进入 GitHub 仓库的 **Actions** 标签页
2. 点击对应的工作流运行记录
3. 点击 **build** 或 **release** 任务
4. 展开 **Compile LaTeX document** 步骤查看详细日志

### Q: 为什么 Release 没有创建？

A: 确保：
1. 推送的 tag 以 `v` 开头（如 `v1.0.0`、`v2.1.3`）
2. 编译成功完成
3. 仓库有写入权限

### Q: 如何更改 PDF 文件名？

A: 编辑 `.github/workflows/latex-compile.yml` 文件，修改 `Rename PDF` 步骤中的文件名格式。

## 技术细节

- **编译器**：XeLaTeX（与 Overleaf 配置一致）
- **编译参数**：`-interaction=nonstopmode -file-line-error`
- **编译次数**：使用 `latexmk` 自动多次编译以生成正确的目录和引用
- **Artifact 保留期**：90 天
- **所需权限**：`contents: write`（用于创建 Release）

## 相关链接

- [GitHub Actions 文档](https://docs.github.com/cn/actions)
- [xu-cheng/latex-action](https://github.com/xu-cheng/latex-action) - LaTeX 编译 Action
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release) - Release 创建 Action
