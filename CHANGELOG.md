# Changelog

## 0.1.9

- perf: package.json 声明 `sideEffects` 仅样式文件（`**/*.css` / `**/*.less` / `**/styles/**`）——组件模块可被消费方 bundler tree-shake，此前未声明时打包器保守保留整个库及其重依赖（katex / pdfjs / xlsx 等），monorepo 同批实测消费方关键路径 −895KB
- ci: package.json 无 BOM 断言（发版改版本号时容易带入 BOM，vendored 引用会解析失败）

## 0.1.8

- feat: 生成期间可继续输入与发送——发送即入队（上限 10 条），当前回答结束后按序自动发出；每条可「编辑」（取回输入框、连 @ 引用一并带回，改完再发）、「插队」（打断当前生成并立即处理该条，原会话上下文完整沿用）、「删除」；生成中按「停止」= 当前回答停止 + 队列暂停保留，手动再发即恢复推进。队列内核在 `useAIChat`（不含 UI 依赖），组件层面新增 `queuedMessages` prop 与 `removeQueued`/`promoteQueued` 事件
- feat: 知识库下拉从工具条中央移到输入框左侧（`@` 之后），并在「对话起步」（存在非 system 消息）后锁定，锁定态 hover 提示换库请点新建对话；空会话仍可自由换库
- feat: 工具条自适应——两个下拉可收缩（库 96–160px / 模型 100–180px，≤480px 再收缩一档），`@` 与发送按钮永不参与收缩，被压缩的是下拉文字（已 ellipsis）
- fix: 去除 `package.json` 的 UTF-8 BOM（0.1.7 带入）——registry 安装时 pnpm 容忍，但以 `file:`/目录方式引用本包（vendored）会以 `Unexpected token '' … is not valid JSON` 直接解析失败
- refactor: 删除输入区失效的图片上传入口整条链路（-117 行）——该按钮只把图片读成 DataURL 做本地预览，不上传也不随消息发送，宿主侧早已硬编码禁用；连带移除公开 prop `allowImageUpload`（自始至终未实现过任何能力，传与不传行为一致）
- style: 待发送托盘重做——原底色 `--bg-tertiary` 在浅色主题下与输入框 `--bg-secondary` 数值相同（≈#fafafa），整块只剩一根几乎不可见的边框；改 primary 淡染 + 同色描边，圆角与输入框统一 12px，新增标题行「待发送 N 条」，序号改圆形徽标，动作区独立成组并换掉 antd text 按钮的裸文字观感，全部颜色仍走 `--chat-queue-*` 双回退钩子

## 0.1.7

- feat: npm registry 正式上架（@angineer/aichat-ui）

## 0.1.6

- feat: 新增 dark 主题 token 段（`[data-theme="dark"]`）——色调类给 antd 暗色系对应值，其余链宿主语义变量（`--bg-secondary` 等）自动跟随；导入 `./style` 即获得完整 dark 切换，不导入仍为 light 默认（无回归）
- docs: README 主题表补 dark 列

## 0.1.5

- fix: 修复主题 token 文件自引用失效（`--chat-x: var(--chat-x, …)` 为 CSS 循环引用，按规范整体无效）；组件使用处全部补齐 fallback，不导入 `./style` 也能正常渲染，`./style` 降级为可选的集中覆盖层
- fix: `--bg-secondary` 使用处补 fallback
- chore: 新增 `typecheck`（vue-tsc）与 `test` 脚本；README 收编主仓库（安装/导出/主题契约/token 表）

## 0.1.4

- feat: 同步 monorepo 0.2.16 改动——流式引用 tag 实时渲染（正文 [Kx]/[Tx] 标记在流式期间即可显示引用框）与引用 id 归一化（剥掉 target:/table:/formula:/figure:/chunk: 前缀）

## 0.1.3

- feat: AI 对话知识库切换响应式（切库后聊天作用域实时生效，修复挂载时快照问题）

## 0.1.2

- feat: 检索结果项支持 KaTeX 公式渲染（新增 searchSnippet 工具与测试）

## 0.1.1

- 同步 AnGIneer monorepo 最新代码：
  - 引用完整映射：citations 与 items 合并、按 marker 去重；
  - 证据数字圆圈与思考过程展示优化；
  - thinking 工具与测试补充。
