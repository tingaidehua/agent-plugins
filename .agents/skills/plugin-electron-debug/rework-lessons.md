---
# 为何 IMT / Harness 联调反复返工（经验）
---

## 一句话

门禁量错了对象、修错了层、又没让运行时加载到新代码 → 对话里「修好了」，用户截图仍坏。

## 返工链条（按真实发生顺序）

### 1. 假 allPass

- 只量了 aside **子节点** EN/ZH 矩形，没量 **`.source-file` 路径条** 与 aside 同列重叠。
- 用户红框里是 `lox/Lox.java` + `(print (eval (read)))` 叠在译文上，不是 sibling 叠层。
- **教训：** 右栏 = aside **加** absolute 芯片；门禁表要有 `chipAside` / `chipChip`。

### 2. 修了 sibling，没修「列竞争」

- 双语 aside 长高后，站点 `refreshAsides` + 绝对定位 `.source-file` 抢同一 `left`。
- 再用 **cumulative `translateY` nudge** 与 pack 双记账 → 全列 `nudge=272` 灾难。
- **教训：** 一种定位手段（pack **或** nudge）；pack 只消 **真实碰撞**，禁止整列压实。

### 3. FAB 切换「每次不一样」

- 原文模式 **硬卸 163 个 DOM** 再挂回 → 重 paint、重 pack，指纹漂移。
- `requestAnimationFrame` pack 与连点重入交错。
- FAB ensure **不重绑** → 页面上跑的是旧 click handler。
- **教训：** 原/双/译用 CSS 显隐（软原文）；FAB 必须 remove+create rebind；切换加锁 + 取消过期 rAF。

### 4. 译文把侧栏顶飞

- clip 源节点后 aside 高度塌缩，再 pack 会按塌缩盒重写 `style.top` → top:267 / -29。
- **教训：** 在 **双语高度** 下定位并 **冻结** aside top；translation 态禁止再 resolve aside。

### 5. 运行时没吃到新代码

- junction 仍指向已删 `dsh-cdp` → 无 FAB。
- CDP 热注入绿了，但 Electron watcher 仍旧 bundle → 用户刷新又坏。
- **教训：** 分清「当前标签热修」vs「Harness 进程持久加载」；改 Node 入口必重启 Electron。

### 6. 对照原文不彻底

- 应用 `strict-compare`：同一 scroll、三态截图、aside idx 序、芯片间距。
- 像素用 `Read` 看 `.probe/*.png`，不要只信 JSON 计数。

## Immersive Translate 对齐原则（少返工）

| 原则 | 做法 |
|------|------|
| 双语 | 译文 **sibling 插在原文后**，文档流，禁止 absolute 叠在原文上 |
| 三态 | `imt-state` + `dsh-view` 成对；translation 用 clip/overlay，不是拆 DOM 乱插 |
| 原文 | 优先 CSS 隐藏 wrapper，记录留在 `__dshImtRecords` |
| 绝对边栏 | 只处理 overlap；芯片与 aside 分治 |

## 以后开口「好了」前

1. `strict-compare.mjs` 或等价门禁退出码 0  
2. FAB ×N 指纹每态唯一  
3. 用户同视口截图（或 CDP png）肉眼过右栏路径条  
4. 热修后说明是否已重启 Harness / 是否需刷新页  
