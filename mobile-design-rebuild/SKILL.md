---
name: mobile-design-rebuild
description: 当用户提供一张或多张移动端设计图，并要求 Codex 对 iOS、Android 或 H5 的指定页面进行高保真还原、重构或重新设计时使用；尤其适用于需要先产出设计规格文档、再确认开发、生成缺失图片与图标、并做多端视觉验证的任务。
---

# 移动端设计还原

## 核心原则

把设计图先转成可确认、可执行、可验证的规格包，再进入开发。不要在规格确认前生成素材或修改业务代码。

必须遵守：

- 支持任意移动端设计图，不固定为 iPhone 15 Pro Max；先识别设计图尺寸、比例、平台倾向和目标端。
- 支持 iOS、Android、H5，可单端也可多端；尺寸、单位、字体、间距和验证流程必须按目标端分别计算。
- HTML 是给用户确认的视觉审阅面；JSON/Markdown 是后续开发的执行约束。
- 设计图中的图片、插图、图标、纹理、装饰元素只能作为参考，不允许直接裁成最终 App/网页资产。
- 项目里存在完全相同的原始素材时才可复用；只是相似、不完全一致、语义接近的素材必须拒绝并重新生图。
- 缺失图片和图标必须在用户确认规格后通过 `$imagegen` / 生图能力生成，再裁剪、抠图、压缩、导入项目。
- 子线程生成或处理的素材必须保存到主线程可读取的规格目录，并回传绝对路径、候选清单和审阅预览；不能只在子线程聊天里展示图片。
- 严禁用 SVG、代码绘图、系统图标、IconFont、文本字符、渐变拼接、Canvas/Shape/Path 等方式模拟设计图里的图片或图标。
- 默认采用“主线程统筹 + 子线程执行”的工作流；如果当前环境没有可用子线程工具，必须记录原因并由主线程完成，不能假装已分发。

## 阶段 0：输入契约

开始分析前，先记录输入契约。信息已足够时直接记录并继续；缺少阻塞信息时只问最少问题。

记录：

- 模式：严格还原、基于设计方向重新设计、还是混合。
- 目标端：iOS、Android、H5，或多端组合；用户未指定时先根据项目和设计图推断，并标记置信度。
- 目标页面：页面名、路由、Tab、入口、已有文件或新增页面。
- 范围：单页、多页、组件体系、还是共享模块。
- 页面状态：新增、重构已有页面、还是待代码定位确认。
- 设备目标：设计图原始尺寸、目标机型/视口、横竖屏、状态栏/导航栏/底部安全区要求。
- 业务边界：只改 UI、保留现有行为、还是新增交互/数据逻辑。
- 审批规则：用户确认规格包前，不生成最终素材，不改代码。

## 阶段 1：设计图判断与基准换算

先判断输入是否像移动端设计图或移动端截图：

- 记录每张图的像素尺寸、方向、宽高比、状态栏、导航栏、Tab/底栏、WebView/H5 特征、Android/iOS 系统特征。
- 判断平台倾向：`ios`、`android`、`h5`、`mixed`、`unknown`，并给出证据和置信度。
- 不能因为设计图接近某个机型就固定使用该机型；用户指定目标端或项目实际端优先。
- 如果图是 `1290 x 2796`，可记录为接近 iPhone Pro Max 3x 画布；仍需按目标端重新换算。
- 如果图不是标准设计稿尺寸，按原图像素建立 `sourcePx` 坐标系，再为每个目标端生成独立换算。

换算基准：

- `sourceWidthPx` / `sourceHeightPx`：设计图原始像素。
- `targetDesignWidth` / `targetDesignHeight`：目标端基准宽高。
- `scaleX = sourceWidthPx / targetDesignWidth`。
- `scaleY = sourceHeightPx / targetDesignHeight`。
- 常规垂直/水平尺寸优先使用对应方向 scale；图标、圆角、正方形元素用平均 scale 或主宽度 scale，并记录选择。

## 阶段 2：确定修改端与页面定位

在代码中定位目标页面，不要直接猜文件。

- iOS：查找 SwiftUI View、UIKit Controller、Storyboard、路由、Tab、NavigationLink、`Assets.xcassets`、可见文案。
- Android：查找 Compose Screen、XML layout、Fragment/Activity、Navigation route、resource name、`res/drawable`、可见文案。
- H5：查找 React/Vue/Svelte/HTML 页面、路由、组件、样式文件、资源目录、可见文案。
- 多端项目要分别列出每端候选文件和证据。
- 如果多个候选页面都合理，列出证据并让用户确认。
- 该阶段只定位和记录，不改代码。

## 阶段 3：组件与结构提取

把所有设计图作为一组扫描：

- 重复元素只录入一次组件注册表，记录变体和出现页面。
- 提取导航栏、搜索区、卡片、列表行、按钮、输入框、标签、Tab、弹窗、空状态、固定底栏、图标、图片框、背景、装饰元素。
- 区分共享组件、页面专属组件、平台专属组件。
- 为每个页面写实现导向的结构树：根容器、滚动区、头部、内容组、列表、浮层、固定底部、边距、安全区、复用组件。
- 结构树必须能指导代码布局，不能只写视觉描述。

## 阶段 4：细节科学计算

不要凭视觉感觉估算字体、颜色、间距。先测量，再换算，再记录置信度。

### 多端单位规则

| 目标端 | 布局单位 | 字体单位 | 计算方式 |
| --- | --- | --- | --- |
| iOS | pt | pt | `pt = px / scale`，字体用测量文本高度、层级和字重校正 |
| Android | dp | sp | `dp = px / densityScale`，字体用 `sp` 记录，并注明是否跟随系统字体缩放 |
| H5 | CSS px / rem | CSS px / rem | `cssPx = px / viewportScale`，`rem = cssPx / rootFontSize` |

必须记录：

- 字体角色：导航标题、页面标题、分区标题、卡片标题、正文、辅助文本、按钮、Tab、输入框、角标。
- 每个字体角色的 size、weight、line-height、letter-spacing、color、confidence。
- 字重必须用明确值：iOS `.regular/.medium/.semibold/.bold`，Android `FontWeight.Normal/Medium/SemiBold/Bold`，H5 `400/500/600/700`。
- 颜色从稳定区域采样 3-5 个非边缘像素，记录 Hex/RGB、使用位置和置信度。
- 间距、边距、卡片尺寸、圆角、阴影、边框、分割线、图标尺寸、图片框尺寸、固定底栏高度。
- 不能确定的值要标注不确定原因和验证方式，不要写成确定值。

## 阶段 5：规格包输出

在项目本地创建规格目录：

```text
work/mobile-design-rebuild/<screen-slug>/
  design-spec.html
  input-contract.json
  platform-targets.json
  design-tokens.common.json
  design-tokens.ios.json
  design-tokens.android.json
  design-tokens.h5.json
  component-registry.json
  asset-manifest.json
  implementation-plan.md
  spec-qa.md
  references/
  generated-assets/
```

如果项目不适合使用 `work/`，选择最安全的项目本地临时规格目录，并报告绝对路径。

### `design-spec.html`

这是用户确认用的视觉看板，不是把所有技术规格堆在网页里的长文档。它的目标是让用户快速判断：页面找得对不对、设计理解对不对、哪些内容会严格还原、哪些素材要生图、哪些问题需要确认，以及是否可以进入开发。

HTML 必须采用“主看板 + 技术附录”结构：

1. **首屏决策区**
   - 任务名称、目标页面、目标端、设计图数量、新增/重构判断、当前状态。
   - 明确的审批状态：`等待确认`、`需补充信息`、`已批准开发`、`已冻结规格`。
   - 3-7 条本次开发会做什么和不会做什么的摘要。
   - 一个清晰的确认结论区：用户确认后才允许生图和改代码。

2. **设计图总览**
   - 原始设计图大图预览，标注原始像素尺寸、方向、平台倾向和目标端换算基准。
   - 多图任务要能并排比较，标明每张图对应的页面、状态或变体。
   - 参考裁剪图必须标注 `仅参考，不是最终资产`。

3. **页面结构确认**
   - 用可视化分区方式展示页面组成，例如状态栏、导航栏、主内容、卡片、列表、底部栏、弹窗、浮层。
   - 每个区域写清楚后续对应的布局实现思路，但不要把完整结构树直接塞在主视觉区。
   - 对新增页面、重构页面、复用组件分别标记。

4. **还原重点**
   - 列出 5-10 个必须严格还原的关键点，例如背景、主图、按钮、列表图标、字体层级、间距、安全区、固定底部。
   - 每个关键点必须说明判断依据、测量值摘要和验收方式。
   - 不要把所有 token 都展示在主区；只展示会影响用户视觉判断的关键值。

5. **素材缺口看板**
   - 按 `完全复用`、`相似但拒绝`、`缺失需生图`、`参考图`、`已生成候选` 分类展示。
   - 缺失图片、插图、图标、纹理、装饰元素必须显示 `assetId`、参考位置、为什么不能复用、预期生成方式。
   - 生图候选出现后，展示候选图、处理后图片、contact sheet、主线程选择结果。

6. **需要用户确认的问题**
   - 只列真正阻塞开发或会影响视觉结果的问题。
   - 每个问题必须给出推荐选项、影响范围和默认处理方式。
   - 不要把普通测量置信度或内部实现细节混入这里。

7. **开发执行摘要**
   - 会修改哪些端、哪些页面/路由/文件范围。
   - 是否需要子线程、生图共享目录、素材交付方式、验证设备/视口。
   - 明确进入开发后的验收方式：Simulator、Emulator 或 Browser/Playwright 截图对比。

8. **技术附录**
   - 折叠或放在页面后部，包含完整 token 表、组件表、结构树、素材 manifest 摘要、测量置信度和开放细节。
   - 技术附录是给 AI 和开发执行用的，不应干扰用户首屏判断。

### 机器可读文件

- `input-contract.json`：用户输入、目标端、目标页面、审批规则。
- `platform-targets.json`：每端设备/视口、单位、缩放基准、安全区、验证方式。
- `design-tokens.common.json`：跨端共用颜色、层级、组件语义。
- `design-tokens.ios.json`：iOS pt、字体、颜色、圆角、阴影、Asset 命名。
- `design-tokens.android.json`：Android dp/sp、Material/Compose/XML 对应规则、资源命名。
- `design-tokens.h5.json`：CSS px/rem、CSS 变量、响应式断点、资源路径。
- `component-registry.json`：组件 id、角色、变体、测量、出现页面、实现提示。
- `asset-manifest.json`：每个图片/图标/纹理/装饰元素的状态、参考、生成要求、候选路径、处理后路径、导入路径。
- `implementation-plan.md`：文件范围、端侧实现计划、子线程任务拆分、验证清单。
- `spec-qa.md`：规格自检结果。

### 规格自检

请求用户确认前，`spec-qa.md` 必须确认：

- 每张设计图都进入 HTML。
- 每个目标端都有换算基准。
- 每个页面都有结构树。
- 每个重复组件都进入组件注册表。
- 每个可见图片、图标、纹理、装饰元素都进入素材清单。
- 每个字体角色都有 size、weight、color、line-height、confidence。
- 重要颜色都有采样值。
- 相似但不完全一致的项目素材被标记为 `rejected-similar`。
- 设计图裁剪只作为参考，不作为最终资产。
- 没有接受系统图标、SVG、代码绘图作为自定义图标或图片替代品。
- 每个 `missing-generate` 素材都有计划中的共享交付目录和 `assetId`。

## 规格确认 Gate

完成规格包后：

- 给用户 `design-spec.html` 的绝对路径。
- 简要说明看板里的确认结论：目标端、页面定位、还原重点、缺失素材、阻塞问题、是否建议批准开发。
- 停止执行，不生图，不改代码。
- 用户要求调整时，先更新规格包。
- 用户明确确认后，冻结该版本作为开发合同。
- 后续任何实现偏差都要更新规格或写入批准偏差。

## 阶段 6：素材与图标处理

只在规格确认后处理最终素材。

硬性规则：

- 设计图中的视觉元素要作为生图参考，而不是直接裁图导入项目。
- 每个缺失图片、插图、图标、纹理、装饰元素都要用 `$imagegen` / 生图能力生成候选。
- 生成多张候选后，由主线程按设计图相似度选择；不要让子线程自行决定最终视觉接受。
- 对选中候选进行裁剪、抠图、透明背景处理、尺寸归一、压缩和命名。
- 只有完全相同的项目原始素材才可复用；相似图、相似图标、相似系统符号都必须拒绝。
- 列表图标、设置图标、隐私/协议/删除/FAQ 等小图标也是资产，缺失时必须生图。
- 禁止使用 SF Symbols、Material Icons、IconFont、Emoji、SVG、SwiftUI Shape/Path/Canvas、Compose Canvas、CSS shape 等近似替代。

### 子线程素材交付协议

主线程分发生图任务前，必须为每个 `missing-generate` 素材指定共享交付目录：

```text
work/mobile-design-rebuild/<screen-slug>/generated-assets/<asset-id>/
  brief.md
  candidates/
    candidate-01.png
    candidate-02.png
  processed/
    candidate-01-trimmed.png
    candidate-01-transparent.png
  contact-sheet.png
  asset-review.html
  handoff.json
```

子线程必须遵守：

- 只能把最终可审核候选保存到上述共享目录或主线程指定的项目本地目录。
- 不能只在聊天消息里展示图片，不能只返回临时 URL，不能只描述“已生成”。
- 每张候选图和处理后图片都要有稳定文件名、绝对路径、尺寸、格式和用途说明。
- 必须生成 `contact-sheet.png` 或 `asset-review.html`，让主线程能一次性视觉对比候选。
- 必须写入 `handoff.json`，记录 `assetId`、任务 id、提示词、参考来源、候选图绝对路径、处理图绝对路径、预览文件路径、失败候选、处理步骤和开放问题。
- 必须更新或返回可合并到 `asset-manifest.json` 的候选记录。
- 交付前必须自检文件存在且能被读取；不能把不可访问路径交给主线程。

主线程选择候选前必须验证：

- `handoff.json` 存在且字段完整。
- `candidatePaths`、`processedPaths`、`contactSheetPath`、`reviewHtmlPath` 中的路径能在主会话文件系统中访问。
- 图片能打开，尺寸和透明背景符合素材要求。
- `asset-manifest.json` 已记录候选、处理图、主线程选择状态和最终导入状态。

如果主线程拿不到真实图片文件，必须把该生图任务标记为 `blocked` 或重新分发，不能根据子线程文字描述做视觉选择。

导入位置按目标端记录：

- iOS：`Assets.xcassets`，代码使用 `Image("assetName")` / `UIImage(named:)`。
- Android：`res/drawable`、`res/mipmap` 或项目现有资源体系。
- H5：`src/assets`、`public/assets` 或项目现有静态资源目录。

## 阶段 7：子线程工作流

默认分工：

- 主线程负责规格、视觉判断、候选选择、代码集成、最终验证和最终报告。
- 子线程负责生图候选、视觉相似度初筛、生成图裁剪抠图、共享目录落盘、局部组件实现、局部页面实现、定向修复。

使用规则：

- 当前环境存在可用多代理/子线程工具时，优先把独立任务分发给子线程。
- 如果创建子线程需要用户额外授权，先请求授权；没有授权时主线程继续执行并记录原因。
- 每个子线程必须有互不重叠的写入范围。
- 生图子线程不能修改代码或项目设置。
- 代码子线程不能决定最终图片候选。
- 生图子线程必须按“子线程素材交付协议”交付文件；没有真实文件路径的回报视为未完成。
- 子线程不能回滚其他线程或用户的改动。

子线程回报格式：

```text
任务：
写入范围：
修改文件：
生成/处理素材：
候选图绝对路径：
处理后图片绝对路径：
contact sheet / 审阅 HTML：
handoff.json：
实现的规格项：
规格偏差：
验证方式：
需要主线程处理的问题：
```

## 阶段 8：多端实现规则

- 保持项目现有架构、状态管理、路由、依赖注入、命名、格式化和设计系统。
- 按已确认规格实现，不把测量值散落成无来源 magic number。
- 优先使用项目现有 token/theme；没有时新增小范围 token 层，并与规格文件命名对应。
- iOS 遵守 SwiftUI/UIKit 的安全区、动态字体策略、Asset 体系和模拟器验证。
- Android 遵守 Compose/XML、dp/sp、资源密度、主题和 Emulator 验证。
- H5 遵守现有框架、CSS 变量、响应式视口、图片加载和浏览器验证。
- 如果实现中发现规格不可执行，先更新规格或记录批准偏差，再继续。

## 阶段 9：多端验证

按目标端验证，不相关的端不用跑。

- iOS：使用 iOS Simulator，构建运行、进入目标页面、截图并与规格/设计图对比。
- Android：使用 Android Emulator，构建运行、进入目标页面、截图并与规格/设计图对比。
- H5：使用浏览器或 Playwright，打开目标页面，按目标视口截图并与规格/设计图对比。

在规格目录创建或更新 `verification-diff.md`：

```text
平台 | 区域 | 规格/设计图 | 实际结果 | 状态 | 处理
ios/android/h5 | 标题字体 | 20pt/20sp/20px semibold #111111 | matches | pass | none
ios/android/h5 | 列表图标 | generated PNG asset | still uses system icon | fix | replace asset
```

状态只能使用：`pass`、`fix`、`approved-deviation`、`blocked`。存在重要 `fix` 项时，不能宣称完成。

## 完成检查

最终答复前，对照：

- 用户原始需求。
- 已确认的 `design-spec.html`。
- `input-contract.json` 和 `platform-targets.json`。
- 各端 `design-tokens.*.json`。
- `component-registry.json`。
- `asset-manifest.json`。
- `generated-assets/` 中的候选图、处理图、`handoff.json` 和审阅预览。
- `implementation-plan.md`。
- `spec-qa.md`。
- `verification-diff.md` 和截图证据。

最终报告必须说明：

- 修改了哪些端、哪些文件。
- 生成/导入了哪些素材。
- 子线程生成素材的共享目录、候选数量、选中候选和被拒候选。
- 哪些素材是完全复用，哪些相似素材被拒绝。
- 是否替换了系统图标、SVG 或代码绘图替代品。
- 每端验证设备/视口和结果。
- 剩余偏差、阻塞项或用户批准的偏差。

## 常见失败模式

- 没有先产出规格包就开始改代码。
- 把设计图裁剪当成最终资产。
- 用项目里的相似素材替代不完全一致的设计素材。
- 用 SF Symbols、Material Icons、IconFont 或 SVG 近似还原图标。
- 只写 HTML，不写 JSON/Markdown 机器约束。
- 固定按 iPhone 15 Pro Max 计算，忽略 Android/H5 目标端。
- 多端任务只算一套尺寸。
- 凭感觉估算字体、颜色、间距，不记录测量和置信度。
- 规格确认后偷偷偏离规格。
- 说需要生图但没有实际调用生图能力。
- 子线程只在聊天里展示生图结果，没有把图片落盘到主线程可访问目录。
- 子线程没有返回候选图绝对路径、`handoff.json` 或 contact sheet，主线程却继续审核。
- 让子线程决定最终视觉验收。
- 构建成功后没有用 Simulator/Emulator/Browser 做视觉对比。
