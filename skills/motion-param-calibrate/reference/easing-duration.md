# 缓动与时长参数表

> 这张表回答两个问题：这段动画该多快、该用什么曲线。数值是起点，不是圣旨——但改动要有理由，理由不能是「我觉得」。
>
> 曲线取值来自公开规范（easings.net 的 quint/quart 族、Material Design 3 的 standard 与 emphasized 曲线、iOS 抽屉曲线），不是拍脑袋调的。

## 一、先定角色，再查数值

动画的时长由**角色**决定，不由元素大小决定。先归到下面五类，再查表。

| 角色 | 典型场景 | 时长 | 曲线 |
|---|---|---|---|
| 即时反馈 | 按钮按压、开关拨动、勾选、图标形变 | 100–160ms | 强 ease-out |
| 小范围出现 | Tooltip、气泡卡、下拉菜单、Toast | 120–220ms | 强 ease-out |
| 中等容器进出 | Modal、Dialog、Drawer、底部抽屉 | 200–320ms | 强 ease-out（进）/ ease-in（出） |
| 页面与结构切换 | 路由转场、Tab 内容替换、视图切换 | 250–400ms | 进 ease-out，出 ease-in |
| 叙事与演示 | 滚动显现、图表生长、大图揭示 | 400–700ms | ease-out，首段略慢 |

**进用 ease-out，出用 ease-in。** 这条不是习惯，是感知原因：入场时用户刚触发，想看结果，所以要"立刻动、慢慢停"；退场时用户已经决定离开，快走掉才不碍事。

## 二、曲线具体取值

浏览器内置的 `ease`、`ease-out` 太弱，减速感不足，看起来"温"而不是"果断"。用下面四条。

| 名称 | 值 | 用在哪 |
|---|---|---|
| 强 ease-out | `cubic-bezier(0.22, 1, 0.36, 1)` | 入场、出现、按下后回弹 |
| 强 ease-in-out | `cubic-bezier(0.76, 0, 0.24, 1)` | 屏内两点间移动、形态变化 |
| 标准 | `cubic-bezier(0.2, 0, 0, 1)` | 通用默认，Material 3 standard |
| 强调减速 | `cubic-bezier(0.05, 0.7, 0.1, 1)` | 需要更明显"冲进来"的大面板 |
| 抽屉曲线 | `cubic-bezier(0.32, 0.72, 0, 1)` | 底部抽屉、侧边面板，iOS 手感 |

```css
:root {
  --ease-out-strong: cubic-bezier(0.22, 1, 0.36, 1);
  --ease-in-out-strong: cubic-bezier(0.76, 0, 0.24, 1);
  --ease-standard: cubic-bezier(0.2, 0, 0, 1);
  --ease-emphasized: cubic-bezier(0.05, 0.7, 0.1, 1);
  --ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
}
```

**用 linear 的只有三种情况**：匀速位移（跑马灯）、进度指示、无限旋转的转圈。任何有起点终点的动画都不要 linear——现实里没有东西匀速启停。

**UI 里禁用 ease-in 入场。** 它起步慢，用户正盯着看的那一瞬偏偏没反应，200ms 的 ease-in 体感比 200ms 的 ease-out 慢。

## 三、距离修正

同样角色，位移越远，时长越长。固定时长在长距离上会显得"嗖一下"，在短距离上会显得"磨蹭"。

| 位移距离 | 时长修正 |
|---|---|
| 小于 10px（微位移） | 取下限，100–140ms |
| 10–100px（组件内位移） | 表内中值，180–250ms |
| 100–400px（面板级位移） | 取上限，250–350ms |
| 超过 400px 或全屏 | 350–500ms，用强调减速 |

**面积也影响时长。** 大面积元素（全屏背景、整页内容）移动同样的距离，需要更长的时间才不显得突兀；小元素则相反。

## 四、例外条款：什么情况可以超过 300ms

常规 UI 应控制在 300ms 以内。超过就要说明理由，合法理由只有下面四种：

1. **计数与数值变化。** 数字从 0 滚到 100 万，2 秒是合理的——用户要看清过程，这不是位移，判据是**可读性**不是距离。同理适用：进度增长、分数跳动。
2. **循环与呼吸。** 骨架屏明暗呼吸、背景渐变流动，这些参数是**周期**不是时长，1.5–8 秒都正常。判据是"不引起注意"，不是"快"。
3. **叙事与营销页。** 滚动显现、大图揭示、品牌开场，允许 400–700ms，因为此时用户在"看"而不是"操作"。但同一页里不能有超过两个这类长动效。
4. **超远距离位移。** 全屏面板从屏外滑入，距离本身决定了时长下限，350–500ms 是物理需要。

**这四种之外，超过 300ms 就是错。** 原书自己定了"超过 1 秒会显得拖沓"，正文里却给了数字滚动 2 秒、渐变流动 8 秒——不是作者笔误，是两类参数的判据不同。原书没写出这个区分，这里补上。

## 五、弹簧参数

需要"活着"的感觉、可中断的运动、拖拽带惯性时，用弹簧而不是固定时长。

| 预设 | 刚度 | 阻尼 | 质量 | 用在哪 |
|---|---|---|---|---|
| 轻快 | 400 | 35 | 1 | 按钮、开关、小图标 |
| 标准 | 300 | 30 | 1 | 弹层、卡片、抽屉回位 |
| 柔和 | 150 | 25 | 1 | 大面板、全屏元素 |

```js
{ type: "spring", stiffness: 300, damping: 30, mass: 1 }
```

规则：
- **常规 UI 的回弹不超过 0.2。** 超过 0.35 开始显得俏皮，0.5 以上只留给庆祝类交互。
- **阻尼低于临界值会持续摆动。** 阻尼约等于 `2 * sqrt(stiffness * mass)` 时最快停稳且不过冲。
- **按感知时长判断快慢。** 弹簧可能真实计算了 1.2 秒，但用户 0.4 秒就认为结束了——调参时看体感，不看数字。
- 全站固定三组预设，不临时新造。每次都现调的团队，最终会有十几种手感。

## 六、减弱动态配方

这是原书完全缺失的一块。一层入场、视差、粒子不加降级，对前庭敏感人群是真实的身体不适，不是审美偏好。

### 原则：减弱不是归零

官方措辞是 reduce，不是 remove。正确做法是**保留理解所需的过渡，去掉运动与位移**。

| 原效果 | 降级为 |
|---|---|
| 位移入场（上浮、滑入） | 纯淡入，200ms |
| 缩放入场 | 淡入，不改尺寸 |
| 视差、多层位移 | 静止，各层固定位置 |
| 循环动画（呼吸、粒子、流动） | 静态首帧，或极慢的透明度变化 |
| 自动轮播 | 停止自动播放，保留手动切换 |
| 3D 翻转、纵深推进 | 直接切换，无过渡 |
| 拖拽惯性 | 直接跟手，松手不回弹 |

### 精细版（推荐，按元素写）

```css
@media (prefers-reduced-motion: reduce) {
  .reveal { transform: none; opacity: 0; transition: opacity 200ms ease; }
  .reveal.is-visible { opacity: 1; }
  .parallax-layer { transform: none !important; }
  .ambient-loop { animation: none; }
}
```

```jsx
const reduce = useReducedMotion();
const offset = reduce ? 0 : 24;
```

### 兜底版（不许精细控制时用它）

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

兜底版会把必要的透明度反馈也杀掉，只在紧急修补或存量项目上用。新项目走精细版。

### 同时要处理的其他偏好

```css
@media (prefers-reduced-transparency: reduce) {
  .glass { backdrop-filter: none; background: rgba(255,255,255,0.96); }
}

@media (prefers-contrast: more) {
  .card { border-color: currentColor; border-width: 2px; }
}
```

## 七、门控与暂停

### 悬停只在真有悬停的设备上生效

触屏会模拟 :hover，导致"点一下没反应、点两下才行"，还会让悬停唤出的浮层没有退出路径。

```css
@media (hover: hover) and (pointer: fine) {
  .card:hover { transform: translateY(-4px); }
}
```

### 主题切换时禁用过渡

```css
.theme-switching *, .theme-switching *::before, .theme-switching *::after {
  transition: none !important;
}
```

切换主题时如果所有颜色都做 300ms 过渡，会出现一片拖影和中间色闪烁。

### 不可见就停

```js
const io = new IntersectionObserver(([e]) => {
  e.isIntersecting ? animation.play() : animation.pause();
});
```

页面隐藏时同理，监听 `visibilitychange`。循环动画在后台继续跑，是在偷电。

## 八、交付前自检

- [ ] 每个动效都能归到「角色」表里的一类；归不进去说明它可能不该存在
- [ ] 入场用 ease-out，退场用 ease-in，没有例外
- [ ] 距离超过 100px 的位移做过时长修正
- [ ] 超过 300ms 的播放有三个条件之一，能当场说出口
- [ ] `prefers-reduced-motion` 有实际实现，不只是提了一句
- [ ] 所有 `:hover` 动效都在 hover 门控里
- [ ] 循环动画离开视口或切后台时暂停
- [ ] 转换主题时禁用了过渡
