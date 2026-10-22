# FrontEnd

五个前端交互动效的 skill，给 AI 用。

解决的问题很具体：你想让某个东西动一下，但说不清怎么动。「高级一点」「有点科技感」「照着 XX 网站那样」——这类话丢给 AI，每次做出来都不一样，因为你没说清它要往哪动、动多久、动完停在哪。这五个 skill 就是把这层模糊翻译成可执行规格的工具链。

拆解自《Vibe Coding 前端交互效果术语手册》（才聚智能，17 页速查手册）的 68 条术语与其方法部分。

---

## 五个 skill

| skill | 干什么 | 什么时候用 |
|---|---|---|
| [`interaction-brief`](skills/interaction-brief/) | 把模糊意图写成五格规格：触发时机 / 对象 / 变化 / 时长缓动 / **收尾状态** | 效果还没做出来，要写提示词 |
| [`motion-term-lookup`](skills/motion-term-lookup/) | 说不出名字时定位术语。68 条全表挂在 `reference/terminology.md` | 「那种滚到才出现的效果叫什么」 |
| [`motion-param-calibrate`](skills/motion-param-calibrate/) | 按动画角色分派时长与缓动，不是随便填个数 | 做出来了，但节奏不对、发飘、像 PPT |
| [`motion-audit`](skills/motion-audit/) | 四组体检：注意力预算 / 渲染成本 / 输入设备 / 状态完整性 | 上线前挑硬伤 |
| [`loading-state-pick`](skills/loading-state-pick/) | 按等待时长 × 结构可预期性 × 有无真实进度选加载形态 | 决定加载中显示什么 |

## 路由

这五个 skill 的边界是互相咬合的，选错的代价是拿到的答案不对路：

```
说不出是什么      → motion-term-lookup
还没做，要规格    → interaction-brief
做完了，节奏不对  → motion-param-calibrate
做完了，哪儿怪    → motion-audit
加载中显示什么    → loading-state-pick
```

最容易混的两处：

- **查词 vs 选型**。`motion-term-lookup` 只回答「这东西叫什么」。问「我这种情况该用哪个」，加载类归 `loading-state-pick`，其余由 `interaction-brief` 在补规格时一并选。
- **视觉设计 vs 动效**。排版、留白、层级、配色的问题，动效一项都查不出来——而且拿动效去补这些窟窿，恰好是「廉价感」的成因之一。`motion-audit` 遇到这类问题会直接说「这不是动效的活」。

## 用法

每个 skill 是一个目录，`SKILL.md` 带 YAML frontmatter（`name` / `description` / 触发信号 / 边界）。

装到本地技能库：

```bash
cp -R skills/* ~/.workbuddy/skills/
```

放进 `~/.claude/skills/` 或任何读 `SKILL.md` 约定的 agent 框架同理。

每个目录下的 `test-prompts.json` 是一组路由测试用例，分 `should_trigger` / `should_not_trigger` / `edge` 三类。改完 skill 要跑一遍，尤其别把「必须介入但结论是拒绝」的场景写进「不要使用」列表里——agent 会照字面执行，该拦的事一件不拦。

## 这本书的硬伤

拆解时发现的问题，已写进各 skill 的边界段，不藏：

- **零提 `prefers-reduced-motion`。** 一本教人大量加入场、视差、粒子和光斑跟随的手册，完全不提无障碍降级。这是合规问题，不是审美问题。
- **把「高级感」归因于动效。** 实际来源是排版、留白、层级、字重；动效只是放大器。用动效去补排版留下的问题，会产出它自己反对的「廉价感」。
- **12 类里没有一类涉及应用态。** 乐观更新、FLIP、并发中断、四态设计全部缺席。遇到这类问题，各 skill 会明说超出范围。
- **自己的硬边界被自己违反两次。** 「超过 1 秒会显得拖沓」在正文里被数字滚动（2 秒）和渐变流动（周期 8 秒）破例。不是笔误——计数和循环不是「移动」，判据不同。这个区分原书没写，补进了 `motion-param-calibrate` 的例外条款。

## 关于引用

`skills/motion-term-lookup/reference/terminology.md` 收录了 68 条术语的话术原文，逐字保留（含原文在中文与数字/拉丁字母之间的半角空格），因为改一个字就可能改掉参数语义。

蒸馏的中间产物——源文本、候选池、被否的条目、盲测记录——没有放进这个仓库。源文本与全书逐字引用涉及原书版权，公开不合适。需要审计痕迹的话，本地留存。
