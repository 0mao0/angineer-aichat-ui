# Changelog

## 0.2.4

- fix: 回答落定后思考卡片不再消失——`useAIChat` 写落定消息时，`thinking_trace` 从「只取 transport 返回值」改为「transport 返回值优先，缺省回退本轮流式 `onThinking` 快照」，两者都空则字段缺省、不落空数组（与 `sop_flows` 兜底同型）。只在 SSE status 帧推进思考步骤、done 帧不带轨迹的宿主（如 ABP 专业问答 `ask/stream`），回答结束后思考步骤（意图判断/处理链路/检索）仍保留在消息内；自带 `thinking_trace` 的宿主行为不变
- test: `useAIChat` 新增三例——宿主仅流式上报时落定仍保留步骤、transport 自带轨迹优先于流式快照、整轮无步骤不落空数组

## 0.2.3

- feat: 回答图表化 v1（angineer 块协议）——markdown 渲染拦截 angineer-chart/angineer-map 代码块（占位/回退/普通代码块隔离）；chartBlock 以 echarts 按需渲染 bar/line/pie、暗色双选择器、脱离文档实例回收；BaseChat debounce 接线 + 块容器样式双主题；量级悬殊 ≥5 倍自动挂双 y 轴（大小两方向都判 + 防除零）；补 TitleComponent（echarts 按需模式下图标题此前不渲染）
- feat: SOP 流程聊天可视化——SSE sop_flow 快照通道 + SopFlowPanel 步骤卡（可折叠）；「流程图」抽屉（定义画布 + 本次执行着色）；步骤行展示「参数=值 → 结果」
- fix(citations): Kxx 引用标记容忍括号内空格（渲染、过滤、守卫三层同修）；引用点击丢文库归属——多库会话点圈码/内联链接不再回退首库（404）
- feat: hero 引导问题支持分行形态（长题面截断 + 悬停看全文）；流程图抽屉宽度收窄
- chore: 许可统一 MIT


## 0.2.2

- breaking: 多库选择去上限——删除导出常量 `MAX_LIBRARY_SELECTION`，勾选不再截断、提示文案去掉「≤5」（业主口径「选了 N 个组 M 个库就该 M 个库去检索」，服务端 `ANGINEER_MAX_CHAT_LIBRARIES` 截断同批移除，>20 库只记 warning）；消费方此前 import 该常量会编译报错
- breaking: 类型 `BaseChatModelOption` 移除，模型选项改由新导出 `BaseChatModelGroup`（`{ key, label, levels: [{ label, value }] }`）表达；另新增导出 `BaseChatLibrarySection`（`{ key, label, libraries: [{ id, name }] }`）
- feat: 模型 × 思考等级两个下拉——`BaseChat` 新增 `model-groups` prop（`levels[0]`=该模型默认档、`level.value`=发送给后端的配置名；空列表则整个模型区不渲染），换模型落到该模型默认档，组内只有一档（该模型无变体）时档位下拉自动隐藏；`AIChat` 自动按底层 `model` 归并 `transport.fetchModels()` 结果生成分组，无需宿主准备。档位后缀约定 `<端点>-<系列>-<档位>`：「无后缀别名 / `-off` / `-ctk`」是思考关闭的三种参数写法 → 统一显示「关」且只出一条（取值优先级 无后缀别名 > `-off` > `-ctk`），`-xhigh` 显示「思考」（底层原值在悬停提示里）；未登记的后缀按原样单列，配置不会从选择器静默消失；付费模型沉底排序沿用
- feat: 知识库「组 → 库」两级勾选——`BaseChat` 新增 `library-sections` prop（`AIChat` 透传），组行整勾/组内库行单勾都写回 `libraryValues`（保序追加/移除、不重排已有选择；不在分组清单内的 id 原样保留）；默认按组折叠、整行为展开热区（点复选框方块只切勾选，点名字/箭头/空白才展开收起），组行右侧显示「已选/总数」；不传 `library-sections` 沿用平铺选项，宿主未接两级结构时逐位不变
- fix: 模型选中值悬空——`syncSelectedModel` 改为「现值仍属于某个分组才保留」，`defaultModel` 也不在分组内时回退第一组首档；`fetchModels` 失败改为空分组（不再回退「默认模型」假项，模型区据此不渲染）
- fix: 两级下拉点行体不再夺走选择器焦点（组头行 `@mousedown.prevent`）——否则 select 失焦被 antd 判成「点到外面」、点组名即关下拉（仅真实点击可复现，合成事件测不出）
- style: 底栏三个下拉去填充底色只留描边（套盒两层视觉太丑）、宽度自适应内容（`min-width` 归零，「关」一个字不再占 84px；上限仍留，防超长模型名/库名撑破底栏）；多选态 antd 自带 chip 底色与内圈边框一并去掉，避免与选择器外框叠成双层；思考等级下拉上限再收一档（120px，窄屏 96px）
- docs: README 补「模型 × 思考等级」「多库勾选」两节（props 契约与分组数据结构）

## 0.2.1

- feat: 思考链路展示细化——① 每步前置时间 tag（浅蓝胶囊，首步 `0.3s` 无前缀、其余 `+1.5s`，取步骤 `duration_ms`；不足 50ms 不显示，消灭 `+0.0s` 噪声）；②「并行预检」两行 tag（置于序号列与时间 tag 同一竖线，第二行给出预检方真实耗时 `reusedMs`），要求调用-返回配对完成才显示，修流式期间 `tool_start` 先到闪 tag、`tool_end` 到达又被顶掉；③ 多行便签渲染（note 详情按换行拆「首行标题 + 逐行正文」）；④ 预检索合步（`attach:'pair'` 的便签并入上一对工具步为附注行，步数 8→7）与标题改写（`injected` 步显示「预检索：knowledge_search」）；⑤ 折叠头总耗时改锚在 `wallStartMs`（挂轨迹首步的 run 起点墙钟，分类等待计入），各步 tag 之和才与总账对上；⑥ 命中引用按文档归并（同文档多处引用合为一枚 `×N` chip、按引用次数排序、>8 篇折叠 `+N`、标签用可读规范名）；⑦ 两列布局对齐（序号+时间标签进 66px 左栏，正文/调用结果/命中引用统一到同一条文字列）
- feat: 输入区交互重构——编辑器随内容自动长高（上限 45vh 后内部滚动），底部操作行从悬浮覆盖改正常流（修「长文本撑破输入框、操作行浮到框中段、文字与组件重合」）；边框/焦点环从编辑器挪到新 `.input-box` 整盒（`:focus-within` 亮环）；描边三级链＝宿主钩子 > 包内档位兜底 > 浅色默认（暗色值落私有变量，避免压掉 hover/焦点环）；移除输入框 resizer（auto-grow 后它只剩「空闲基础高度」一个作用，且与 45vh 上限规则打架）
- feat: 新对话入口从顶栏迁到对话态输入框上方浮层——`BaseChat` 新增 `showNewChat` / `newChatLabel` props 与 `newChat` 事件（`AIChat` 透传，`ChatTopBar` 删按钮），与「回底部」同右对齐叠放；hero 空态不出现
- feat: hero 空态引导问题 props 化——`suggestedQuestions` 贯穿 `AIChat` → `BaseChat`，点击 chip 直发（与手动发送同 emit 链路、同恢复跟随）；不传或空数组整块不渲染，`hero-below` 插槽保留共存
- feat: 流式自动跟随可被用户上滑打断 + 「回底部」按钮——仅在 `scrollHeight` 未变时才判用户上滑（流式 markdown 重排瞬间收缩 `scrollHeight` 会令浏览器截断 `scrollTop`，酷似上滑；首版实测第一次发问即误解除），另挂 `@wheel` 快路径作明确回看意图；48px 阈值内或重新发送即恢复跟随
- feat: 多库勾选问答（阶段三）——主选择器多选 ≤5，新导出常量 `MAX_LIBRARY_SELECTION`（与服务端 `ANGINEER_MAX_CHAT_LIBRARIES` 兜底截断同值）；请求载荷集合化（消息类型新增 `library_ids`，非空时 `library_id` 兼容取首项）；本地缓存改单桶；引用溯源按库定位（消息类型新增 `library_id`，缺省=宿主按主库回退）；多选收口显示「N 个知识库」；库名登录后补加载、空库警示横幅随最近一次请求、库下拉面板加宽防库名截断
- feat: 输入区左下角 `@` 与知识库选择器位置对调（库选择器在前、`@` 提及按钮在后），纯模板兄弟节点换序
- fix: 公式渲染归一化未转义 `%`——模型照抄规范原文 `R_{1\%}` 的裸文本形态（`1%`/`2%` 记法）被 KaTeX 当注释符解析、整条公式渲染报错；`renderFormula` 剥定界后统一转义（负向回查不重复转义已写对的 `\%`），对任意模型的裸文本公式形态免疫
- fix: 浅色可读性三连——① 问答气泡加 1px 描边（宿主按主题给色，浅色可见细线／深色 transparent）；②「回答中」气泡与落下后同底色（原流式淡蓝与落下不同色）；③ 内联角标改双主题统一深灰圆（hover 仍主色蓝），角标 token 三件套只留 `:root` 一处防漂移
- fix: `index.less` 暗色段选择器补 `html.dark`（宿主只挂类名、`data-theme` 从未落地，此前该段对宿主整段失效）
- fix: `loadMessages` 收口剔除服务端历史里的 agent 内部消息（首轮直达注入的 `tool_calls` 围栏/JSON assistant 不再渲染成 JSON 代码块气泡）；删除前端合成的「汇总证据并生成最终回答」标记行（分组层加历史兼容过滤清旧数据）；「回到最下方」改名「回底部」（正文、悬浮提示、注释同步）
- fix: `@` 引用 hover 预览卡贴紧锚点并优先向上展开——改 `bottom` 锚定，内容不足满高时不再留 `maxHeight − 实高` 的空隙
- test: 新增 markdown 公式渲染 4 例与多库 `useAIChat` 用例（`node:module` 注册 css 空模块 loader，解 katex ESM import css 在 `tsx --test` 下的 `ERR_UNKNOWN_FILE_EXTENSION`）

## 0.2.0

- feat: 等待体验三件套——① 中间答案折叠留痕：新回调 `onInterimAnswer`，turn_start/边界规则改写/run_end 权威覆盖发生真实替换时，被顶替的正文快照收进置灰折叠块（流式区+最终气泡两处可展开回看），不再直接蒸发；② 分段进度：新回调 `onStage`（run_start→classify / tool_start→search / 首 delta→generate），`progressStage` + `elapsedSeconds`，等待文案从全程「思考中...」改为「意图理解…→检索规范库…（实时秒数）→生成回答…」；③ 流式重渲节流：onDelta 50ms 合帧写 currentStreamContent（改前每 delta 全量重解析 markdown，长答案 O(n²) 越打越卡）
- feat: `onError` 回调——宿主可据 403 `login_required` 等业务码弹登录浮层；未挂回调时错误气泡展示干净业务文案（不带「出错了」前缀）
- feat: `BaseChatMessage` / `AIChatMessage` 新增 `interim_answers` 字段（历史气泡回看被替换的中间答案）
- fix: 发给后端的 `session_id` 改发裸 id（剥掉池 key 的 `scene:` 前缀）——此前落库在 `docs:chat-x` 而宿主记录层打裸 id，快照补丁 400 unknown msg_seq 静默丢失、会话列表同会话双 id（服务端池 key 本就含 scene，行为不变；存量带前缀行保留仍可用）
- fix: 输入区 @ 按钮与答案角标去硬编码深色（light 模式白字隐形/浅底近黑圆喧宾夺主，v0.0.51 深色时代遗留），三轮迭代定案中性灰双主题：新 token 三件套 `--chat-citation-circle-{bg,border,text}`（light 浅灰圆面+中灰数字、dark 深灰圆+浅灰数字），hover 主色+白字不变
- fix: 中断时先冲刷 delta 合帧缓冲——50ms 合帧节流引入的回归：手动停止/插队时最后 ≤50ms 的 delta 压在缓冲里没落盘，截断答案丢尾；另将 run 计时器 unref（run 悬置时 interval 不再压住 Node 事件循环，测试/SSR 进程可自然退出）
- test: useAIChat 用例断言裸 session_id 与跟进式改写相关契约

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
