# PocketJS 学习清单

这份清单按「先跑起来，再读得懂」的顺序排好。每项是一格可勾选的 Checklist，后面内联**预计耗时**与**前置依赖**。路径都指向仓库里真实存在的文件。

## 〇、总览（约 15 min）

- [ ] **`README.md`**：读完一遍，记住三句话——组件编译成**一棵原生树**、QuickJS guest 驱动 Rust core、**没有 DOM / CSS 引擎 / WebView**。｜预计 10 min ｜依赖：无
- [ ] **三种框架同一棵树**：能说出 Solid `solid-js` · Vue Vapor `vue` · Octane `octane` 都编译到同一棵树，区别只在应用代码层。｜预计 5 min ｜依赖：上一项

## 一、先跑起来（约 40 min）

- [ ] **装工具**：装好 [Bun](https://bun.sh/) 与 [Rust](https://rustup.rs/)；浏览器 dev host 还需要 wasm target。｜预计 20–30 min ｜依赖：无
- [ ] **`bun install`**：确认没有报错。｜预计 5 min ｜依赖：Bun 已装好
- [ ] **加 wasm target**：运行 `rustup target add wasm32-unknown-unknown`，让 core 能编译进 WASM。｜预计 5 min ｜依赖：Rust 已装好
- [ ] **跑 dev host**：运行 `bun run dev`——构建 WASM + Hero app，在浏览器里渲染（见 `site/content/docs/getting-started.md`）。｜预计 15 min ｜依赖：上面三项全部完成
- [ ] **读 manifest**：打开 `apps/hero/pocket.json`，看懂一个应用用**同一份 manifest**声明 viewport 与所需 API。｜预计 10 min ｜依赖：`bun run dev` 已跑通

## 二、写并挂载第一个组件（约 75 min）

- [ ] **读 `mount()`**：`framework/src/index.ts:308`——它接收一个返回 JSX 的函数，是 guest 里启动 UI 的入口。｜预计 15 min ｜依赖：第一节完成
- [ ] **三种写法一棵树**：对比 `apps/hero/app.tsx`（Solid）、`app.vue-vapor.tsx`、`app.octane.tsx`，确认渲染出同一棵树。｜预计 10 min ｜依赖：上一项
- [ ] **分清两类 import**：框架原语从 `solid-js` / `vue` / `octane` 直接导入；运行时、宿主组件、生命周期、输入与动画 API 从 `@pocketjs/framework/*` 导入（见 CLAUDE.md「API ownership」）。｜预计 10 min ｜依赖：上一项
- [ ] **写个小组件**：用 `Text` / `View` 等宿主组件（`framework/src/components.ts`）写一个会响应状态的小组件，确认 `createSignal` 这类原语来自框架本身。｜预计 30 min ｜依赖：import 规则已分清

## 三、读懂渲染管线：从组件到像素（约 1 h）

- [ ] **读 Rust core**：`engine/core/`（`no_std`）——它拥有 flexbox 布局、烘焙的样式表与 drawlist。｜预计 45 min ｜依赖：第二节完成
- [ ] **找到边界**：guest→core→backend；确认**没有 vdom diff、没有 CSS cascade / specificity、没有 reflow**。｜预计 20 min ｜依赖：上一项
- [ ] **看一个 backend**：`engine/backends/` 下任一个（如 web），理解同一份 drawlist 如何提交到不同硬件层。｜预计 20 min ｜依赖：边界已找到
- [ ] **对号入座**：对照 `README.md`「From component to pixel」图，说出 PocketJS 用**1 线程、1 进程**完成一帧。｜预计 10 min ｜依赖：上一项

## 四、样式与动画是构建期烘焙的（约 50 min）

- [ ] **样式表**：读 `framework/src/styles.ts` 与 `styles.generated.ts`——class 字面量在构建期变成一张烘焙样式表，运行时只做查表。｜预计 20 min ｜依赖：第三节完成
- [ ] **Tailwind 子集**：明白接受的词汇是一个**固定的 Tailwind 子集**（见 `site/content/docs/tailwind.md`），不是完整 CSS。｜预计 10 min ｜依赖：上一项
- [ ] **动画也烘焙**：读 `framework/src/animation.ts` / `anim.ts`——关键帧时间线与弹簧曲线同样烘焙进样式表，由 Rust core 用自己的时钟推进，**可以没有每帧 JavaScript**。｜预计 20 min ｜依赖：Tailwind 子集已懂

## 五、目标准入与打包（约 40 min）

- [ ] **平台清单**：读 `contracts/spec/platforms.ts`（权威 host/target 清单）里的某一条，知道每项记录了「验证过什么、怎么验证」。｜预计 15 min ｜依赖：无
- [ ] **核对 target**：运行 `pocket check --target psp`，看它如何用 manifest 的 viewport + 所需 API 核对一个 target profile。｜预计 10 min ｜依赖：Bun CLI 可用；第一节完成
- [ ] **`.pocket` 格式**：读 `docs/PLATFORM.md`——它是**可检视、可按目标裁剪**的打包格式，不是每个平台一份目录。｜预计 10 min ｜依赖：上一项

## 六、帧契约与确定性（约 1 h）

- [ ] **时间即帧计数器**：读懂一次 `frame(buttons)` 是一个不可打断的事务（README「The frame contract」）。｜预计 20 min ｜依赖：无
- [ ] **效果落在帧边界**：读 `docs/DETERMINISM.md`——效果落在**帧边界**上，`after()` 用**帧数**而非墙钟衡量截止。｜预计 30 min ｜依赖：上一项
- [ ] **跑一次测试**：找一次（`tests/clock.test.ts`、`tests/sim.test.ts`）跑通，体会 tape 可以逐字节重放。｜预计 10 min ｜依赖：`bun install` 完成

## 七、框架源码走查（进阶，约 1 h）

- [ ] **生命周期**：读 `framework/src/lifecycle.ts`——它如何接线到那棵树。｜预计 20 min ｜依赖：第三节完成
- [ ] **输入与焦点**：读 `framework/src/input.ts` / `input-api.ts`——输入与焦点如何在 core 一侧落地。｜预计 20 min ｜依赖：上一项
- [ ] **三个 renderer**：对比 `renderer-solid.ts` · `renderer-vue-vapor.ts` · `renderer-octane.ts`，确认它们收敛到同一 native tree（`native-tree.ts`）。｜预计 25 min ｜依赖：第二节 import 规则已清楚

## 八、Pocket Vapor：AOT 编译器（独立分支，约 1 h）

- [ ] **读 vapor README**：它把严格的 Vue Vapor 子集**提前编译成目标原生 C**（`.gba` / `.gb` / `.nes` / ESP32 / PlayDate），设备上没有 JS engine、GC 或 allocator。｜预计 30 min ｜依赖：无
- [ ] **独立编译器**：确认它是**另一套编译器、自己的 target/board contract**，不是低内存模式。｜预计 10 min ｜依赖：上一项
- [ ] **输入约定**：读 `vapor/DESIGN.md §5`——Pocket Vapor app 只通过硬件中立的 `RelativeAxis` / `onAxisDelta`（`vapor/host/input.ts:38`）取增量输入，应用代码里不出现设备 SDK 概念、也不把摇杆动作编码成假按钮。｜预计 20 min ｜依赖：上一项

## 九、把某个 target 真正点亮（选做其一）

- [ ] **PSP**：跑 `pocket build --target psp`，产出 EBOOT；看它如何加载 pinned Rust / `cargo-psp` 工具链。｜预计 45–60 min ｜依赖：第一、五节完成
- [ ] **PS Vita**：按 `hosts/vita/README.md` 装好 VitaSDK + 指定 Rust nightly，产出一个 VPK。｜预计 60+ min ｜依赖：PSP 行已完成；需额外 SDK
- [ ] **浏览器 / Playground**：什么都不装也能跑（`https://pocketjs.dev/playground/`），对照本地 dev host 的输出。｜预计 10 min ｜依赖：无

## 十、读完再回头：它到底在做什么（约 1 h）

- [ ] **对号入座三层**：重读 `site/content/docs/architecture.md` 与 `concepts.md`，把「guest / core / backend」三层对上。｜预计 30 min ｜依赖：第三节完成
- [ ] **看一篇博客**：如 `Shipping OpenStrike`——一个**完整应用**如何在真机上跑（OpenStrike 每帧 **2.2 ms JS**）。｜预计 15 min ｜依赖：无
- [ ] **讲给别人听**：为什么同一 bundle 能在 PSP、Vita、iPhone 上各用各自的 native renderer，而应用代码不变。｜预计 20 min ｜依赖：全清单完成
