本文主要介绍「开场白-正文」、「开场白-后置状态栏」、「面板输出规范」、「美化代码」如何填写、避雷速查以及设计的底层机制。

创作者阅读完提示词拼接逻辑，以及相关示例即可快速上手，想更细致地了解完整prompt以及背后的工程结构，可以阅读进阶部分

## 快速上手

### 提示词拼接的核心逻辑

下图展示了在lumeow中填写的内容如何被拼接成prompt，核心机制如下：

- 你填的「开场白-正文」和「开场白-后置状态栏」组成 `<output_showcase_example>` ，它会作为AI 每轮回复的 **模板范例** ，沿用你的 HTML 结构来写每一轮回复。
- 完整的 `output_showcase_example` = 「开场白-正文」 + \<state> + 「开场白-后置状态栏」+ \</state> ，其中\<state>\</state>标签由平台自动拼接，创作者无需标记
- 你填的「面板输出规范」告诉 AI 在使用模板时， **数值该如何变化，内容如何变化**
- 平台已内置 `<response_structure>` 和 `<state_rules>` 从大的方向上规范模型输出结构和生成state的要求。 **你填的规范不要和它们冲突** （具体可见进阶部分）。
![](https://docs.lumiao.ai/~gitbook/image?url=https%3A%2F%2F3288285137-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fa2A0U0i1uBCgXm5lQQc1%252Fuploads%252F0KKmnx23muXCTUZPoMqq%252Fimage.png%3Falt%3Dmedia%26token%3Df3ca40de-6ae8-41eb-a518-798d23c1e090&width=768&dpr=3&quality=100&sign=7c6daffb&sv=2)

### 速查表

框

填什么

不填什么

开场白-正文

场景（时间、地点，也可自定义）的HTML/XML 骨架(div class = "xxx" 部分) + 正文

1. inline-css代码(xxx style=yyyy, 比如div style="color:#FFD700;margin-top:6px;font-size:11px;") ，style代码应放置于美化代码中
2. \<state> 标签本签和相关内容

开场白-后置状态栏

状态栏的 HTML/XML 骨架(div class = "xxx" 部分)

1. inline-css代码(xxx style=yyyy, 比如div style="color:#FFD700;margin-top:6px;font-size:11px;")，style代码应放置于美化代码中
2. 外层 \<state> 标签
3. 没有任何标签包裹的纯文本

面板输出规范

自然语言描述"数值在什么情况下如何变化"

与平台已有规则冲突的指令

美化代码

所有 CSS 样式

业务逻辑

✅ 你可以使用下面的prompt完成拆分：

```
你是一位经验丰富的网页开发者，擅长重构，完成以下任务：
1.抽取css并合并：将多个html中的css和美化相关内容都抽取出来，放入到一个<style></style>标签中，注意如果当前代码中有没有写在<style></style>中的美化部分，要放入（比如将link ref改为import并放入<style>中等）
2.分别抽取body部分：抽取出的body数量应该和输入的html数量一致

输出格式示例（注意只参考格式，不要参考内容！！）：
1.美化代码
<style>...</style>
2. 开场白中前置的时间和地点的代码（只参考格式，不参考内容）：
<div class=...>
        <div class=...></div>
        <div class=...></div>
        <div class=...></div>
...
</div>

3. 后置状态栏代码（只参考格式，不参考内容）
<div class=...>
        <div class=...></div>
        <div class=...></div>
        <div class=...></div>
...
</div>

或者：
2. 状态栏代码
<div class=...>
        <div class=...></div>
        <div class=...></div>
        <div class=...></div>
...
</div>

这是我需要你处理的代码：

***这里直接替换为你的代码***
```

#### 最高频错误

⚠️ **状态栏内容放入到「开场白-正文」中** 正文是 AI 模仿剧情写作的样板，状态栏是独立结构。混写会让 AI 把状态信息写进正文，造成重复。

⚠️ **样式写在 HTML 里** 后置状态栏写 `<div style="color:red;...">` ，AI 每轮原样输出，浪费大量 token。把样式挪到美化代码框，用 class 选择器控制。

⚠️ **后置状态栏里又写了** `<state>` **标签** 外层 `<state>` 由系统在 `<output_showcase_example>` 里自动包裹，你只需写内部结构。重复写会导致嵌套异常。

⚠️ **面板规范里写"以 XX 结尾"** 平台 `<response_structure>` 已强制要求每轮以 `</state>` 结尾，你再写"以【完】结尾"会冲突，AI 表现不稳定。

⚠️ **面板规范里写"数值无变化时可省略"** 平台 `<state_rules>` 已强制要求 state 保留完整结构，无变化也要全量输出。写省略规则会被覆盖。

### 拼接示例

（从霍格沃茨模拟器中截取，已获得@+1+2老师的授权）

**开场白-正文：**

```
<!-- 组件A: 时空路标 --><div class="hw-component hw-waypoint">   ...</div>
几周前收到录取通知书时的震惊与狂喜，如今已沉淀为一种更加具体、更加令人心跳加速的现实感。你正站在一条你从未想象过的街道上。
...
直接走向\`奥利凡德魔杖店\`。 你已经等不及了。你觉得，<emphasize>只有当魔杖选择你的那一刻，这一切才算真正开始。</emphasize>
```

**开场白-后置状态栏:**

```
<!-- 组件A: 观察日志 --><div class="hw-observation-log">    <details>        <!-- 1. 折叠状态：魔法档案夹封面 -->        
...  </details></div>
```

**拼接后的完整结构：**

```
<output_showcase_example>
<!-- 组件A: 时空路标 --><div class="hw-component hw-waypoint">   ...</div>
几周前收到录取通知书时的震惊与狂喜，如今已沉淀为一种更加具体、更加令人心跳加速的现实感。你正站在一条你从未想象过的街道上。
...
直接走向\`奥利凡德魔杖店\`。 你已经等不及了。你觉得，<emphasize>只有当魔杖选择你的那一刻，这一切才算真正开始。</emphasize>
<state>
<!-- 组件A: 观察日志 --><div class="hw-observation-log">    <details>        <!-- 1. 折叠状态：魔法档案夹封面 --> 
...  </details></div>
</state>
</output_showcase_example>
```

## 状态栏拼接的完整prompt

你填写的【面板输出规范】、【开场白-正文】、【开场白-后置状态栏】都会在下面作为静态变量被替换掉，拼接到上行的prompt中：

```
<response_structure>
<p>整条回复 = 【正文（Main Body）】 + <state> 面板，顺序固定，不可颠倒。</p>
[Main Body: 以 <output_showcase_example> 为模板——
- **HTML 结构**：沿用示例的标签、class 和嵌套方式，禁止自定义和新增类名。
- **文本内容**：保持与 <output_showcase_example> 完全相同的视角，不要照搬示例文本。]
<state>[复杂的单/多角色状态面板。必须位于正文之后，且不得穿插进 <message> 或 <emphasize> 内部。]</state>
</response_structure>
<state_rules>
\`<state>\` 块包含由创作者定义的结构化 HTML 模板。在每次回复中生成 \`<state>\` 块时：
1. **保留完整结构。** 模板中存在的每一个 \`<div>\`、每一个嵌套元素、每一个字段都必须出现在你的输出中。不要省略、合并或简化结构的任何部分——即使某个字段的值与上一回合相比没有变化。
2. **动态更新数值。** 每个字段都必须反映当前回复时刻的**当前故事状态**。
3. **嵌套项目是强制性的。** 如果模板包含嵌套的子项目（例如，包含聊天群组的手机屏幕），则必须输出每个子项目。嵌套深度不能作为省略的理由。
4. **必须遵循模板中嵌入的行为指令**（例如，关于 {char} 在特定上下文中如何表现的指令），如同它们是系统提示词的一部分。
</state_rules>
<authors_response_rules>
{{面板输出规范}}
</authors_response_rules>
<output_showcase_example>
{{开场白-正文}}<state>{{开场白-后置状态栏}}</state>
</output_showcase_example>
```

## 底层机制：为什么要拆成多个框

### 1\. 节省token：上下文管理即LLM的一切

简单来说，因为后端要能精确识别 state 的边界（靠 `<state>` 标签），才能在拼接时把它单独拎出来处理。 模型每轮都要全量输出状态栏，当前一轮 state 的代码量可达 **5000 token** 。如果按朴素方式把每轮回复（正文 + state）原样塞进上下文，10 轮对话就有 5 万 token 全是 state，上下文很快爆炸。 **关键观察** ： **state 是全量快照，不是增量日志。**

- 每一轮的 `<state>` 输出的都是"当前世界的完整切片"——所有角色、所有数值、所有面板，一次性全列出
- 理解第 10 轮的 state， **不需要看第 1~9 轮的 state** ，只看第 10 轮的就够了
- 也就是说，历史轮次的 state 对模型而言是 **冗余信息**

**Lumeow 的处理** ：后端在拼接上下文时， **只保留最新一轮的** `<state>` **块** ，历史轮次的 state 全部剥离，只保留正文。这样 state 占用的上下文从 `5000 × N 轮` 压缩到 `5000 × 1` 。 **这就是为什么创作者需要把"开场白-正文"和"开场白-后置状态栏"拆成两个框填** ——后端要能精确识别 state 的边界（靠 `<state>` 标签），才能在拼接时把它单独拎出来处理。如果让创作者混在一起写，后端就没法稳定地剥离。

![](https://docs.lumiao.ai/~gitbook/image?url=https%3A%2F%2F3288285137-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fa2A0U0i1uBCgXm5lQQc1%252Fuploads%252F7WMEDhrS1BVy48bPpbbY%252Fimage.png%3Falt%3Dmedia%26token%3Dfe134da1-7ac8-447b-b300-d0305572cf6c&width=768&dpr=3&quality=100&sign=f67230f9&sv=2)

左图：拼接了所有的state，有大量冗余；右图：只拼接一轮state，节省创作者和玩家的token，提供更聪明的模型体验

### 2\. 出错时兜底

LLM 不是确定性程序。在实际运行中，模型输出的 `<state>` 块偶尔会出现问题——标签拼写错误、嵌套结构混乱、多余的转义字符导致 XML 解析失败，或者干脆在一轮超长回复中漏掉了整个状态块。无论哪种情况，系统都面临同一个局面： **这一轮拿不到有效状态。**

Lumeow 的处理方式是：当系统检测到最新一轮没有有效的 `<state>` 块时， **自动向上查找最近一次成功解析的状态快照** ，将其注入下一轮 prompt。

![](https://docs.lumiao.ai/~gitbook/image?url=https%3A%2F%2F3288285137-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fa2A0U0i1uBCgXm5lQQc1%252Fuploads%252FZgOsyGmsdXjvsacsKH8b%252Fimage.png%3Falt%3Dmedia%26token%3Ded8fcd64-d74c-4979-b80b-ceda320b2eca&width=768&dpr=3&quality=100&sign=9475dc5b&sv=2)

最后更新于