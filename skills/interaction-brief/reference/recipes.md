# 组件级动效配方

> 16 个高频组件的完整规格：触发条件、改哪个属性、曲线、时长、收尾状态、降级方案、最常见的错法。
>
> 曲线与时长取值见 `motion-param-calibrate` 的 `reference/easing-duration.md`。这里只写每个组件该怎么用。

---

## 1. 按钮按压

**触发**：`pointerdown` 按下即给反馈，不等 `click`——`click` 在抬起时才触发，那时手指已经离开，反馈来晚了。

```css
.btn { transition: transform 150ms var(--ease-out-strong); }
.btn:active { transform: scale(0.97); }
```

- **变化**：`scale 1 → 0.97`。幅度不超过 0.95，超过就像橡皮。
- **收尾**：松开回到 1，回位 150ms，不留残余形变。
- **降级**：减弱动态时只做背景色变化（`filter: brightness(0.95)`），不做缩放。
- **常见错**：只写 `:hover` 不写 `:active`，点下去毫无回应；用 `width` 做收缩而非 `transform`。

---

## 2. 开关

```css
.switch-thumb { transition: transform 200ms var(--ease-out-strong); }
.switch-track { transition: background-color 200ms var(--ease-out-strong); }
.switch[aria-checked="true"] .switch-thumb { transform: translateX(20px); }
```

- **变化**：thumb 位移 + track 底色同时开始、同时结束。
- **收尾**：停在端点，不回弹。开关是状态控件，不是玩具。
- **降级**：只改颜色，不移位。
- **常见错**：给 thumb 加弹簧回弹——每次切换都"啵"一下，一天点十次就会烦。

---

## 3. 下拉菜单 / 气泡卡

```js
// transform-origin 指向触发点，让面板"从按钮长出来"
panel.style.transformOrigin = `${triggerX}px ${triggerY}px`;
```

```css
.panel { transform: scale(0.96); opacity: 0; transition: transform 160ms var(--ease-out-strong), opacity 160ms; }
.panel[data-open] { transform: scale(1); opacity: 1; }
```

- **变化**：`scale 0.96 → 1` + 透明度。**不用 `scale(0)`**——现实中东西不会从无到有。
- **方向**：`transform-origin` 设在触发点的对应角（左下弹出则 `bottom left`）。
- **时长**：150–200ms。
- **收尾**：关闭时反向，120ms，比进入快 40ms。
- **降级**：只做透明度。
- **常见错**：从中心放大（看起来像从屏幕中间飞过来）；用 `height` 做展开。

---

## 4. Tooltip

- **触发**：悬停或聚焦 **400ms 后**才出现。立即出现会让鼠标划过时到处弹气泡。
- **变化**：透明度 + 竖直位移 4px。
- **时长**：进入 150ms，退出 100ms。
- **收尾**：鼠标移开立即消失，**没有延迟**——延迟消失会让用户以为卡住了。
- **降级**：只淡入。
- **必须**：`aria-describedby` 关联，读屏能用；确认 `Esc` 可关闭。
- **常见错**：只绑 hover 不绑 focus，键盘用户永远看不到。

---

## 5. Modal / Dialog

```css
.scrim { opacity: 0; transition: opacity 200ms var(--ease-out-strong); }
.dialog { transform: scale(0.96); opacity: 0; transition: transform 220ms var(--ease-out-strong), opacity 220ms; }
.dialog[open] { transform: scale(1); opacity: 1; }
```

- **顺序**：遮罩与面板同时开始；关闭时面板先走（150ms），遮罩后走（180ms），有"退出去"的层次感。
- **遮罩**：黑 40–60%，点击遮罩关闭。
- **收尾**：关闭后焦点**必须**回到触发按钮。
- **降级**：只淡入，不缩放。
- **必须有**：`role="dialog"`、`aria-modal`、焦点捕获、`Esc` 关闭。
- **常见错**：从 `scale(0)` 弹；遮罩层用 `visibility` 切换导致无法过渡；关闭后焦点丢到 `<body>`。

---

## 6. Drawer / Bottom Sheet

```css
.drawer { transform: translateX(-100%); transition: transform 300ms var(--ease-drawer); }
.drawer[open] { transform: translateX(0); }
```

- **属性**：只用 `transform`，不用 `left`。
- **曲线**：抽屉曲线 `cubic-bezier(0.32, 0.72, 0, 1)`，比标准 ease-out 更有"被吸住"的感觉。
- **时长**：侧边 260–300ms；底部面板 300–350ms。
- **收尾**：完全贴边，不留 1px 缝隙。
- **降级**：直接显隐，无滑动。
- **常见错**：用 `left: -280px → 0` 触发重排；抽屉内滚动与外层滚动同时存在时不做滚动锁定。

---

## 7. Toast

```css
.toast { transform: translateY(8px); opacity: 0; transition: transform 220ms var(--ease-out-strong), opacity 220ms; }
```

- **进入**：位移 + 淡入，220ms。
- **停留**：3 秒（错误提示延长到 5 秒，因为要读）。
- **退出**：180ms 淡出上移，比进入快。
- **收尾**：删除节点，不留在 DOM 里。
- **降级**：只淡入。
- **常见错**：用 `@keyframes` 做进出——连续触发两次时第二次会从 0 重播，用 transition 才能从中途改向；堆叠时新 Toast 应把旧的往上推，不是盖住。
- **必须**：`role="status"` 或 `aria-live="polite"`。

---

## 8. Accordion / 折叠面板

```css
.acc-body { display: grid; grid-template-rows: 0fr; transition: grid-template-rows 260ms var(--ease-out-strong); }
.acc-body[open] { grid-template-rows: 1fr; }
.acc-body > div { overflow: hidden; }
```

- **属性**：`grid-template-rows: 0fr → 1fr` 是现代做法，能拿真实高度且可过渡；`height: auto` 不可过渡，`max-height` 会拖慢。
- **时长**：250–300ms；高度超过 400px 时延长到 350ms。
- **收尾**：箭头同步旋转 180°，与展开同时结束。
- **降级**：直接显隐。
- **常见错**：用 `max-height: 9999px`，导致短内容也按最长时长走、看起来发飘。

---

## 9. 列表交错入场

```js
items.forEach((el, i) => {
  el.style.transitionDelay = `${Math.min(i * 40, 240)}ms`;
});
```

- **间隔**：40ms，**总延迟封顶 240ms**。20 条以上的列表必须封顶，否则最后一条要等 800ms 才出现。
- **变化**：`translateY(16px) → 0` + 淡入，单条 400ms。
- **降级**：全部直接显示或统一淡入，不交错。
- **常见错**：给 30 个列表项每个 +80ms，用户滑到第三屏还在等动画；滚动容器里用交错但没做视口检测，看不见的部分也在跑。

---

## 10. Tab 指示条

```js
indicator.style.transform = `translateX(${tab.offsetLeft}px)`;
indicator.style.width = `${tab.offsetWidth}px`;
```

- **做法**：指示条用绝对定位 + `transform`，切换时只改 `translateX` 和 `width`。
- **曲线**：250ms 强 ease-out。
- **内容**：新面板淡入 200ms，旧面板立即消失（不做位移，避免两套布局同时存在）。
- **收尾**：指示条与选中文字完全对齐，误差不超过 1px。
- **降级**：指示条瞬间跳位，内容淡入保留。
- **常见错**：每次切换重建指示条（无法过渡）；用 `left` 而不是 `translateX`。

---

## 11. 滚动显现

```js
const io = new IntersectionObserver((entries) => {
  entries.forEach(e => {
    if (e.isIntersecting) { e.target.classList.add('in'); io.unobserve(e.target); }
  });
}, { threshold: 0.2 });
```

- **触发阈值**：元素露出 20% 时触发，不是 100%。
- **变化**：`translateY(24px) → 0` + 淡入，500ms。
- **收尾**：**`unobserve` 保证只播一次**。没有 once 就会在阈值附近反复穿越反复重播。
- **降级**：直接显示，全页不做显现。
- **常见错**：监听 `scroll` 事件逐个算位置（掉帧）；阈值设 0 导致刚露一个像素就播。

---

## 12. 数字滚动

```js
const start = performance.now();
function tick(now) {
  const p = Math.min((now - start) / 1600, 1);
  const eased = 1 - Math.pow(1 - p, 3);
  el.textContent = Math.round(target * eased).toLocaleString();
  if (p < 1) requestAnimationFrame(tick);
}
```

- **时长**：1200–1800ms。这个属于例外条款里的"计数"，判据是可读性不是距离。
- **曲线**：`1 - (1-p)^3`，结尾减速，让数字"落定"。
- **必须**：`font-variant-numeric: tabular-nums`，否则位数变化时文字左右抖。
- **收尾**：落在精确值，不加千分位跳变。
- **降级**：直接显示终值。
- **常见错**：用 CSS 动画做不到真实数字；用 `textContent` 更新但没设等宽数字；千分位每帧重新格式化导致跳动。

---

## 13. 骨架屏

```css
.skeleton {
  background: linear-gradient(90deg, transparent 25%, rgba(255,255,255,.35) 50%, transparent 75%);
  background-size: 200% 100%;
  animation: shimmer 1500ms linear infinite;
}
@keyframes shimmer { from { background-position: 200% 0; } to { background-position: -200% 0; } }
```

- **周期**：1500ms，线性，方向从右到左（与阅读方向相反，更不刺眼）。
- **形状**：必须与真实内容同尺寸。文字行宽度要长短不一（100% / 80% / 60%），全等宽像文本框不像文字。
- **范围**：只骨架内容区，不骨架导航和已加载的外壳。
- **降级**：降低闪光对比度或改静态灰块。
- **常见错**：骨架形状与真实卡片不一致，数据到达时跳动；用白色闪光在浅色背景上根本看不见。

---

## 14. 拖拽排序（含 FLIP 补位）

```js
// FLIP：先记录旧位置，改 DOM，再反向位移回旧位置，最后过渡到 0
const first = items.map(el => el.getBoundingClientRect());
reorder();
items.forEach((el, i) => {
  const dx = first[i].left - el.getBoundingClientRect().left;
  const dy = first[i].top - el.getBoundingClientRect().top;
  el.animate([{ transform: `translate(${dx}px, ${dy}px)` }, { transform: 'none' }],
             { duration: 250, easing: 'cubic-bezier(0.22, 1, 0.36, 1)' });
});
```

- **拖动中**：被拖元素放大 1.02 倍 + 阴影加深，**其他元素平滑让位**（FLIP，250ms）。
- **阈值**：移动超过 8px 才进入拖拽态，未超过视为点击。
- **收尾**：落位后阴影收起，全程 250ms，不留残余 `transform`。
- **降级**：允许拖拽但瞬时落位，不补位动画。
- **常见错**：只移动被拖元素、其他元素瞬间跳位（这是"廉价感"最典型的一处）；拖动中修改 `top/left`。

---

## 15. 拖拽关闭（Swipe to Dismiss）

- **跟手**：位移实时跟随指针，不做缓动。
- **判定**：位移超过 30% **或**甩速超过 0.5px/ms 即关闭；两者满足其一。
- **收尾**：关闭时按当前速度继续飞出屏幕（继承速度），不是匀速飞完。
- **回弹**：未达阈值时以弹簧回位（刚度 300、阻尼 30），0.3 秒内停稳。
- **降级**：改为点击关闭按钮。
- **常见错**：回弹用固定时长 ease-out，手感像被吸回去；关闭后元素仍占位。

---

## 16. 页面转场

```css
@view-transition { navigation: auto; }
.hero-img { view-transition-name: hero; }
```

- **首选**：View Transitions API，浏览器自动处理共享元素。
- **降级**：手写则只做 200ms 交叉淡入，**不做位移**——真实应用里整页位移最容易掉帧。
- **时长**：300–400ms，超过 500ms 会让人觉得"每次跳页都在等"。
- **收尾**：新页滚动位置重置到顶部，除非是返回操作。
- **降级（无障碍）**：直接切换，无过渡。
- **常见错**：给每个页面加不同方向的滑入，结果方向逻辑混乱；转场期间允许点击导致重复跳转。

---

## 通用检查

每个组件交付前过一遍：

- [ ] 触发是 `pointerdown` 还是 `click`？该早的没早
- [ ] 只动了 `transform` / `opacity`（`grid-template-rows` 是折叠类的例外）
- [ ] 收尾状态明确：停在哪、会不会自动消失、会不会残留
- [ ] `prefers-reduced-motion` 有对应处理
- [ ] `:hover` 效果在 `@media (hover: hover)` 内
- [ ] 键盘可达，焦点有去处
- [ ] 连续触发两次不会错乱（transition 而非 keyframes）
