# Skill: Publish Blog

## 何时触发

当 Tomz 使用下列表达时，默认直接按对应站点处理，不重复询问目标站点：

- “发博客” / “发布博客” → 主站 `tomz.io`
- “发 Mira 博客” / “发布 Mira 博客” → Mira 旧站 `mira.tomz.io`

如果用户明确指定其他站点、仓库或路径，以本次明确指令为准。

## 默认映射

### 普通博客

- GitHub：`dangjingtao/tomz-io`
- 站点：`https://tomz.io`
- 内容根目录：`src/pages/blogs/`
- 部署：提交到 `main` 后由现有 GitHub Actions / Cloudflare Pages 流程处理

### Mira 博客

- GitHub：`dangjingtao/uichat-mira-docs`
- 站点：`https://mira.tomz.io`
- 内容根目录：`src/pages/blogs/`
- 该站点视为 Mira 旧站；不要因为主站迁移而默认把 Mira 博客发到 `tomz-io`

## 执行步骤

1. 读取目标仓库最新的同类文章和相关配置，确认 frontmatter、目录和命名习惯。
2. 根据用户提供的正文创建或更新 Markdown；尽量保留用户原意和写作风格，不擅自扩写事实。
3. 检查站内链接、图片、canonical / site URL 等，避免把另一站的域名或仓库依赖误带进来。
4. 提交到目标仓库默认分支。
5. 如可访问 Actions，则检查构建 / 部署结果；失败时直接定位日志，不把正常 fallback 当成成功。
6. 向用户报告：发布到哪个站、改了什么、commit SHA，以及部署是否已验证。

## 迁移例外

“把旧站某篇博客迁到新站”属于迁移，不改变上面的发布默认值。迁移时先比较两边版本，只迁真正新增或变更的内容，避免无意义覆盖。

## 安全边界

- 不把任何 API key、token、密码写入 Markdown、commit、日志或本仓库。
- 只记录 Secret / 环境变量的名称，不记录值。
- 如果目标文章涉及非公开私人信息，发布前必须以用户本次明确提供或确认的内容为准。
