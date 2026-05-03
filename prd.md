
## 详细产品需求文档：2048 数字合体（uni-app x 版）

| 文档版本 | 修改日期 | 修改人 | 修改内容 |
|---------|----------|--------|----------|
| V1.0 | 2025-01-XX | AI助手 | 初版创建 |

### 1. 项目概述

#### 1.1 项目标识
- **项目名称**：2048 数字合体
- **技术栈**：uni-app x + Vue 3 + TypeScript + Composition API
- **目标平台**：iOS 9.0+， Android 5.0+


#### 1.2 项目目标
实现一款符合原始 2048 游戏机制、触控流畅、状态持久化的单机益智游戏，并具备扩展潜力（本期仅实现核心功能与暗黑主题切换）。

#### 1.3 全局约束
- 禁止使用第三方游戏引擎（如 Cocos），仅依赖 uni-app x 原生能力。
- 所有UI组件必须基于 Vue 单文件组件，不可使用 HTML 原生拖拽 API。
- 必须通过 TypeScript 严格模式编译（`strict: true`）。

---

### 2. 功能性需求

#### 2.1 棋盘系统

| 需求ID | 需求描述 | 验收标准 |
|--------|----------|----------|
| CHESS-01 | 棋盘为 4x4 网格，每个格子可存储数值 0（空）或 2ⁿ（n≥1） | 渲染后每格初始为2或4，其余为0 |
| CHESS-02 | 游戏开始时棋盘随机生成两个非零方块（数值2概率0.9，4概率0.1） | 实际生成100次，统计符合概率分布 |
| CHESS-03 | 每次有效移动后，在随机空白格生成一个新方块（同样遵循2/4概率） | 移动后存在空白格时必定生成一个非零方块 |
| CHESS-04 | 禁止在无效移动后生成新方块 | 滑动但无任何移动/合并时棋盘不变且无新方块生成 |

#### 2.2 移动与合并规则（核心算法）

**输入**：当前棋盘 `board[4][4]`，滑动方向 `direction ∈ {up, down, left, right}`  
**输出**：新棋盘、本次移动增加的分数、棋盘是否变化

**处理规则**（以向左滑动为例）：
1. 对每一行单独处理，提取非零数字序列；
2. 从左向右合并：若相邻两数相等，则将左边数×2，右边数置0，并将合并产生数值累加到分数；
3. 合并后再次提取非零数字，右边补0至长度4；
4. 将处理后的行写回棋盘对应行。

**关键机制**：
- 每个方块在每个移动方向中**最多合并一次**（防止链式合并：如 [2,2,2,2] 向左滑动 → [4,4,0,0]，而不是 [8,0,0,0]）。
- 合并顺序决定最终结果：靠滑动方向边缘的方块优先合并（例如右滑时最右侧的先合并）。

**伪代码**：
```typescript
function moveLeft(row: number[]): { newRow: number[], score: number } {
  let filtered = row.filter(v => v !== 0)
  let score = 0
  for (let i = 0; i < filtered.length - 1; i++) {
    if (filtered[i] === filtered[i+1]) {
      filtered[i] *= 2
      score += filtered[i]
      filtered[i+1] = 0
      i++ // 跳过下一个元素，防止重复合并
    }
  }
  filtered = filtered.filter(v => v !== 0)
  while (filtered.length < 4) filtered.push(0)
  return { newRow: filtered, score }
}
```

#### 2.3 胜负判定

| 状态 | 触发条件 | 行为 |
|------|----------|------|
| 胜利 | 任意格子数值 ≥ 2048 | 弹出胜利对话框，用户可点击“继续游戏”或“重新开始” |
| 失败 | 棋盘无空格 且 任意相邻格子（上下左右）均不相等 | 弹出失败对话框，仅提供“重新开始”选项 |
| 进行中 | 非胜利/失败状态 | 正常接受输入 |

#### 2.4 分数系统

- **当前分数**：每次合并增加的分数累加（合并出的新数值直接加入总分）。
- **最高分**：仅当当前分数 > 历史最高分时更新，并持久化存储。最高分不因游戏重置而清零，除非用户主动重置数据。

#### 2.5 主题切换（暗黑模式）

- 支持手动切换“浅色模式”和“深色模式”。
- 默认跟随系统主题（通过 `matchMedia` 检测）。
- 主题切换时所有UI颜色平滑过渡，过渡时间 0.2s。
- 棋盘颜色适配方案：
  | 数值 | 浅色模式背景色 | 深色模式背景色 |
  |------|----------------|----------------|
  | 2 | #EEE4DA | #3D3D3D |
  | 4 | #EDE0C8 | #4D4D4D |
  | 8 | #F2B179 | #9E4A1B |
  | 16 | #F59563 | #B85D1A |
  | ... | 等 | 保持色调饱和度降低20% |

---

### 3. 非功能性需求

#### 3.1 性能要求
- 滑动操作响应时间 < 50ms（从触摸结束到UI更新完成）。
- 内存占用峰值 < 50MB（避免频繁创建历史对象）。
- 帧率保持 ≥ 55 fps（动画过渡期间）。

#### 3.2 兼容性要求
- 支持 iPhone SE 到 iPhone 15 Pro Max 的全尺寸屏幕适配。
- 支持 Android 折叠屏（展开态和折叠态均正常显示）。

#### 3.3 可靠性要求
- 应用闪退后重启时，应恢复到最近一次有效游戏状态（需增加自动保存机制）。
- 存储读写失败时应有降级方案（内存保留本次数据）。

#### 3.4 易用性要求
- 滑动操作的触发阈值 < 30px，防止误触。
- 游戏结束后滑动无效，且需给出震动反馈（可选）。

---

### 4. 数据设计

#### 4.1 本地存储结构

| Key | 类型 | 说明 |
|-----|------|------|
| `bestScore` | number | 历史最高分数 |
| `themePreference` | 'light' \| 'dark' \| 'auto' | 主题设置 |
| `gameState` | object | 自动保存的游戏状态（棋盘、分数、历史栈等） |

#### 4.2 游戏状态数据结构
```typescript
interface GameSnapshot {
  board: number[][];          // 4x4 棋盘
  score: number;              // 当前分数
  bestScore: number;          // 最高分（冗余存储便于恢复）
  status: 'playing' | 'win' | 'lose';
  timestamp: number;          // 保存时间戳
}
```

#### 4.3 方向枚举
```typescript
enum Direction {
  Up = 'up',
  Down = 'down',
  Left = 'left',
  Right = 'right'
}
```

---

### 5. 接口定义（供开发使用）

#### 5.1 移动算法模块 `gameLogic.ts`
```typescript
/**
 * 执行移动并返回新棋盘和增加的分数
 * @param board 当前棋盘（4x4）
 * @param direction 移动方向
 * @returns { newBoard: number[][], scoreGained: number, isChanged: boolean }
 */
export function move(
  board: number[][], 
  direction: Direction
): { newBoard: number[][]; scoreGained: number; isChanged: boolean }

/**
 * 在随机空格生成新方块
 * @param board 棋盘（会被直接修改）
 * @returns boolean 是否成功生成（无空格时返回false）
 */
export function addRandomTile(board: number[][]): boolean

/**
 * 检查游戏是否失败
 * @param board 棋盘
 * @returns boolean 是否无法移动
 */
export function isGameOver(board: number[][]): boolean

/**
 * 检查是否胜利（存在 >=2048 的方块）
 */
export function isGameWin(board: number[][]): boolean
```

#### 5.2 存储模块 `storage.ts`
```typescript
export function saveBestScore(score: number): void
export function loadBestScore(): number
export function saveTheme(theme: 'light' | 'dark' | 'auto'): void
export function loadTheme(): 'light' | 'dark' | 'auto'
export function autoSaveGame(snapshot: GameSnapshot): void
export function loadAutoSave(): GameSnapshot | null
export function clearAutoSave(): void
```

---

### 6. UI 组件规格

#### 6.1 主页布局（自上而下）
1. **标题区**：显示“2048”Logo，右侧显示主题切换图标（月亮/太阳）。
2. **分数面板**：左侧当前分数，右侧最高分，字体加粗。
3. **棋盘区**：4x4 网格，每个格子圆角 12px，带有轻微阴影。
4. **操作区**：“重新开始”按钮 + “撤销”按钮（二期实现）。
5. **状态提示区**：显示“游戏胜利”或“游戏失败”横幅（可关闭）。

#### 6.2 格子数值与颜色对应（浅色）
| 数值 | 背景色 | 文字色 |
|------|--------|--------|
| 2 | #EEE4DA | #776E65 |
| 4 | #EDE0C8 | #776E65 |
| 8 | #F2B179 | #F9F6F2 |
| 16 | #F59563 | #F9F6F2 |
| 32 | #F67C5F | #F9F6F2 |
| 64 | #F65E3B | #F9F6F2 |
| 128 | #EDCF72 | #F9F6F2 |
| 256 | #EDCC61 | #F9F6F2 |
| 512 | #EDC850 | #F9F6F2 |
| 1024 | #EDC53F | #F9F6F2 |
| 2048+ | #EDC22E | #F9F6F2 |

#### 6.3 暗黑模式适配表（替代色）
- 背景：#1E1E1E
- 卡片底色：#2D2D2D
- 2号格：#3A3A3A，文字：#E0E0E0
- 4号格：#4A4A4A，文字：#E0E0E0
- 8及以上：保持色调，但降低亮度（使用 `filter: brightness(0.8)` 或手动调整色值）

---

### 7. 异常与边界处理

| 场景 | 处理方式 |
|------|----------|
| 存储空间不足 | 仅记录内存中的最高分，并弹出一次性提示 |
| 触摸滑动时中断（如来电） | 游戏状态不变，恢复后可继续 |
| 棋盘满但仍有可合并项 | 正常进行合并，合并后腾出空位 |
| 快速连续滑动（抖动） | 防抖处理：200ms内仅执行最后一次移动 |
| 首次安装无存储数据 | 最高分默认为0，主题默认跟随系统 |

---

### 8. 测试用例（部分关键项）

| 编号 | 操作 | 预期结果 |
|------|------|----------|
| TC-01 | 新游戏，连续左滑两次 | 第二次左滑有合并效果，分数增加 |
| TC-02 | 满屏2，向左滑 | 生成4个4，分数增加16，出现新方块2或4 |
| TC-03 | 游戏胜利后继续移动 | 游戏状态保持胜利，可继续累加分数 |
| TC-04 | 暗黑模式开启后重启App | 主题保持暗黑，棋盘颜色正确 |
| TC-05 | 滑动距离过短（<15px） | 不触发移动，无新方块生成 |

---


### 功能一：撤销功能

#### 1. 功能描述
允许用户回退到上一步操作（最多可撤销 N 步，建议 5 步），用于纠正误操作。

#### 2. UI 交互
- 在棋盘下方或右上角增加“撤销”按钮（图标+文字）。
- 按钮状态：无历史步骤时置灰/不可点击；有历史步骤时高亮可点。
- 撤销时播放简单动画（如棋盘闪烁一次）。

#### 3. 技术实现

**3.1 数据结构设计**
```typescript
// 历史记录条目
interface HistoryStep {
  board: number[][];     // 上一步的棋盘状态
  score: number;         // 上一步的分数
}

// 在游戏状态中增加
const historyStack = ref<HistoryStep[]>([]);  // 最多存储 5 条
```

**3.2 记录时机**
- 每次**有效移动**（棋盘发生变化）后，将移动**前**的状态推入栈。
- 初始状态（新游戏/重新开始）清空历史栈。
- 撤销后，将当前状态推入“重做栈”（可选，若需重做功能）。

**3.3 撤销逻辑**
```typescript
function undo() {
  if (historyStack.value.length === 0) return;
  
  const lastStep = historyStack.value.pop();  // 取出上一步
  board.value = lastStep.board;
  score.value = lastStep.score;
  
  // 更新最高分（注意：撤销后分数降低不应影响已保存的最高分）
  // 游戏状态恢复为 playing（如果之前胜利/失败）
  gameStatus.value = 'playing';
  
  // 可选：清空重做栈（若实现重做）
}
```

**3.4 边界处理**
- 撤销后如果再次移动，新移动会覆盖此后的历史（清空重做栈）。
- 游戏胜利/失败后，撤销按钮应禁用或提示“游戏中不可撤销”。
- 撤销不应影响最高分记录（最高分保留历史峰值）。

#### 4. 验收标准
- [ ] 有效移动后，撤销按钮可点，点击后棋盘和分数回退到上一步。
- [ ] 连续撤销最多 5 步，第 6 步按钮置灰。
- [ ] 撤销后重新移动，旧历史被覆盖（无法再次撤销到更早状态）。
- [ ] 新游戏/重新开始清空历史栈。
- [ ] 游戏结束后撤销无效（按钮禁用或点击无效果）。

---

### 功能二：最佳通关时间统计

#### 1. 功能描述
记录用户从新游戏开始到**首次合成 2048** 所花费的时间，并存储最短通关时间。

#### 2. UI 交互
- 在分数旁增加“最佳时间”显示（格式：mm:ss，如 “01:23”）。
- 当用户合成 2048 时，弹出提示并判断是否刷新最佳记录。
- 若刷新最佳，显示祝贺动画（如“新纪录！”横幅）。

#### 3. 技术实现

**3.1 时间追踪**
```typescript
let startTime: number | null = null;        // 开始时间戳（毫秒）
let timerInterval: any = null;              // 定时器句柄
const currentTime = ref(0);                 // 当前耗时（秒）
const bestTime = ref<number | null>(null);  // 最佳耗时（秒）
```

**3.2 计时触发**
- **新游戏/重新开始**：重置 `currentTime = 0`，记录 `startTime = Date.now()`，启动定时器（每秒更新一次 `currentTime`）。
- **游戏暂停**：不需要（2048 通常无暂停需求，可简化）。
- **游戏胜利**：停止计时，计算耗时 `elapsed = (Date.now() - startTime) / 1000`，与 `bestTime` 比较并存储。
- **游戏未胜利但用户重新开始**：停止旧计时，重置并启动新计时。

**3.3 存储结构**
```typescript
// 使用 uni.setStorageSync 存储
interface TimeRecord {
  bestTime: number | null;   // 最短通关秒数
  hasEverWon: boolean;       // 是否曾经通关过（用于判断首次胜利）
}
```

**3.4 边界处理**
- 如果用户快速重新开始，确保旧定时器被清除，避免内存泄漏。
- 游戏胜利后继续玩（2048 之上继续合并），时间不再累加，也不重复记录胜利时间。
- 最高分重置时，最佳通关时间**不应清空**（除非用户选择“重置所有数据”）。

#### 4. 验收标准
- [ ] 新游戏开始后，当前时间实时增长显示。
- [ ] 合成 2048 时停止计时，若优于历史最佳则更新并存储。
- [ ] 重新开始重置当前计时并重新计时。
- [ ] 多次通关只记录首次胜利时间（或每次胜利都更新最快的一次）。
- [ ] 应用关闭后重新打开，最佳时间正确加载。

---

### 功能三：暗黑模式

#### 1. 功能描述
支持浅色/深色主题切换，可跟随系统或手动设置。

#### 2. UI 交互
- 右上角增加主题切换图标（月亮/太阳），点击切换。
- 提供跟随系统选项（通过 ActionSheet 或开关）。
- 切换时颜色平滑过渡（0.3s 渐变动画）。

#### 3. 技术实现

**3.1 主题定义（CSS 变量）**
```scss
// light 主题
:root {
  --bg-primary: #faf8ef;
  --bg-board: #bbada0;
  --tile-2: #eee4da;
  --tile-4: #ede0c8;
  // ... 其他数值颜色（可做亮度适配）
  --text-dark: #776e65;
  --text-light: #f9f6f2;
}

// dark 主题
[data-theme="dark"] {
  --bg-primary: #2d2d2d;
  --bg-board: #4a4a4a;
  --tile-2: #3c3c3c;
  --tile-4: #4a4a4a;
  --text-dark: #e0e0e0;
  // 可维持数值背景亮度，文字反色
}
```

**3.2 实现方式**
- **方法一（推荐）**：通过 `data-theme` 属性切换，在根元素上动态设置。
- **方法二**：通过 Vue 的响应式类绑定（性能较低，不推荐大面积重绘）。

**3.3 跟随系统**
```typescript
// 监听系统主题变化（uni-app x 支持）
const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
mediaQuery.addEventListener('change', (e) => {
  if (followSystem) {
    setTheme(e.matches ? 'dark' : 'light');
  }
});
```

**3.4 保存用户偏好**
```typescript
// 存储用户设置
const themeMode = ref<'light' | 'dark' | 'auto'>('auto');
// 读取存储时应用相应主题
```

#### 4. 验收标准
- [ ] 手动切换浅色/深色模式，棋盘和方块颜色正确变化。
- [ ] 跟随系统模式：切换系统主题后游戏主题自动变化。
- [ ] 主题偏好本地保存，下次打开应用时沿用上次选择。
- [ ] 所有文字对比度满足无障碍要求（WCAG AA 标准）。

---

### 功能四：不同棋盘尺寸（5x5、6x6）

#### 1. 功能描述
支持切换 4x4、5x5、6x6 三种棋盘尺寸，不同尺寸下的目标数值需调整（保持难度系数相当）。

#### 2. UI 交互
- 在设置菜单或顶部工具栏增加尺寸选择按钮（图标或下拉）。
- 切换尺寸时弹出确认框：“切换尺寸将重置当前游戏，是否继续？”
- 不同尺寸显示不同的胜利目标：
  - 4x4 → 2048
  - 5x5 → 4096 或 2048（建议 4096 平衡难度）
  - 6x6 → 8192 或 4096

#### 3. 技术实现

**3.1 棋盘适配**
```typescript
type BoardSize = 4 | 5 | 6;
const boardSize = ref<BoardSize>(4);
const board = ref<number[][]>([]);

// 初始化棋盘函数
function initBoard(size: BoardSize): number[][] {
  const newBoard = Array(size).fill(0).map(() => Array(size).fill(0));
  // 生成两个初始方块
  addRandomTile(newBoard);
  addRandomTile(newBoard);
  return newBoard;
}
```

**3.2 移动算法泛化**
```typescript
// gameLogic.ts 中的函数需支持动态尺寸
function move(board: number[][], direction: Direction): {
  newBoard: number[][],
  scoreGained: number,
  changed: boolean
} {
  const size = board.length;  // 动态获取尺寸
  // 原有逻辑基于 size 循环即可（无需硬编码 4）
}
```

**3.3 胜利目标调整**
```typescript
function getWinTarget(size: BoardSize): number {
  const targetMap = { 4: 2048, 5: 4096, 6: 8192 };
  return targetMap[size];
}

// 在胜利判断中
if (hasTileValue(board.value, getWinTarget(boardSize.value))) {
  gameStatus.value = 'win';
}
```

**3.4 UI 布局自适应**
- 棋盘容器宽度固定（如 350px），格子尺寸根据 `boardSize` 动态计算：
  ```css
  .grid {
    display: grid;
    grid-template-columns: repeat(v-bind(boardSize), 1fr);
    gap: 8px;
  }
  .tile {
    aspect-ratio: 1 / 1;  /* 保持正方形 */
    font-size: calc(24px - (boardSize - 4) * 2px); /* 尺寸越大字越小 */
  }
  ```

**3.5 历史与存储隔离**
- 不同尺寸的分数、最高分、最佳时间应**分开存储**（key 带尺寸后缀）。
  ```typescript
  const storageKey = `bestScore_${boardSize.value}`;
  const timeKey = `bestTime_${boardSize.value}`;
  ```

#### 4. 
- [ ] 可通过 UI 切换 4x4、5x5、6x6，切换后游戏重置并正确初始化。
- [ ] 移动合并算法在所有尺寸下运行正确（边界测试：5x5 边缘合并）。
- [ ] 不同尺寸的胜利目标正确判断（5x5 需达到 4096 胜利）。
- [ ] 不同尺寸的分数记录、最高分、最佳时间互不干扰。
- [ ] 5x5 和 6x6 棋盘在高分下的视觉适配（不超出屏幕，文字清晰）。
- [ ] 撤销功能在所有尺寸下正常工作。



| 优先级 | 功能 | 预估工时 | 风险点 |
|--------|------|----------|--------|
| P0 | 撤销功能 | 2 小时 | 历史栈深度控制、性能影响小 |
| P1 | 最佳通关时间 | 3 小时 | 定时器清理、多尺寸存储隔离 |
| P2 | 暗黑模式 | 4 小时 | 颜色适配工作量、跟随系统兼容 |
| P3 | 不同棋盘尺寸 | 6 小时 | 算法泛化、UI 适配、测试覆盖大 |
