# 合成年夜饭 - 开发设计文档

> **文档用途**：此文档记录项目设计决策和上下文，方便在新的 AI 对话中快速同步理解，减少误解。

## 项目快速概览

**游戏类型**：类似"合成大西瓜"的合成类网页游戏
**核心玩法**：点击投放食物，相同食物碰撞合成更大的年夜饭菜品
**技术栈**：纯 HTML/CSS/JavaScript（单文件，无框架依赖）
**文件位置**：`year-end-dinner.html`
**在线地址**：https://2003crimson.github.io/year-end-dinner.html
**仓库地址**：https://github.com/2003crimson/2003crimson.github.io

**当前状态**（2026-02-06）：
- ✅ 基础游戏功能完成
- ✅ 颠勺道具功能完成（含确认对话框、冷却期机制）
- ✅ UI 经过 3 轮优化
- ✅ 移动端响应式适配完成
- ✅ 设备自适应物理引擎（桌面端更快，移动端适中）
- ✅ 防止快速点击堆积
- 🔄 持续迭代中

---

## 核心游戏机制

### 合成链
```
饺子🥟 → 汤圆🥙 → 春卷🥙 → 清蒸鲈鱼🐟 → 东坡肉🍖 → 八宝饭🥘 → 白切鸡🍗 → 大闸蟹🦀 → 清蒸龙虾🦞 → 鲍鱼捞饭🍚 → 佛跳墙🍲
```

**图片资源**：所有食物使用自定义图片（存储在 `pic/` 目录）而非 emoji 显示

### 游戏结束判定
- 当有 3 个或更多食物在顶部区域（超过虚线）且速度很小时
- 总食物数量需大于 3 个（防止刚开始就结束）

### 物理引擎参数
```javascript
gravity = 0.3        // 重力加速度
friction = 0.98      // 摩擦力
bounce = 0.3         // 弹性系数
containerWidth = 400  // 游戏区域宽度
containerHeight = 550 // 游戏区域高度
```

---

## 颠勺道具功能设计

### 功能概述
一个紧急道具，每局游戏可使用 1 次，将所有食物向上抛起帮助调整布局。

### 设计决策记录

#### 1. 位置演进
- **初始设计**：放在头部得分下方
- **问题**：玩家容易误触，不知道是什么功能
- **改进**：移到游戏说明下方，改为灰色系
- **最终方案**：采用游戏主题色（红色），与整体设计统一

#### 2. UI 简化历程
- **v1**: "🥄 颠勺 1"
- **v2**: "💫 紧急道具: 颠勺 (剩余1次)" - 文字太多
- **v3**: "🥄 1" - 极简设计，图标+徽章

#### 3. 确认机制
**问题**：玩家不知道功能，误点击后浪费道具

**解决方案**：
- 点击时弹出确认对话框
- 说明功能："将所有食物向上抛起，帮助调整布局"
- 警告提示："每局游戏仅可使用 1 次"
- 提供取消/使用按钮

**关键代码**：
```javascript
function showConfirmDialog() {
    confirmDialog.style.display = 'block';
}

function useShakeTool() {
    if (shakeCount <= 0 || isGameOver) return;
    showConfirmDialog(); // 先显示对话框，不直接执行
}
```

#### 4. 冷却期机制
**问题**：颠勺时食物被抛到顶部，会触发游戏结束判定

**解决方案**：
- 颠勺后设置 2 秒冷却期
- 冷却期间暂停游戏结束检测
- 食物有足够时间落回稳定位置

**关键代码**：
```javascript
let isShaking = false;

function confirmUseShake() {
    isShaking = true;  // 开始冷却
    setTimeout(() => {
        isShaking = false;  // 2秒后结束冷却
    }, 2000);

    // 执行颠勺效果...
}

function checkGameOver() {
    if (isShaking) return;  // 冷却期间跳过检测
    // 正常检测逻辑...
}
```

#### 5. 对话框交互细节
**问题**：确认对话框打开时，点击游戏区域会意外投放食物

**解决方案**：
```javascript
gameArea.addEventListener('click', (e) => {
    if (confirmDialog.style.display === 'block') return; // 对话框打开时不投放
    // 正常投放逻辑...
});
```

---

## 代码结构

### 主要变量
```javascript
let foods = [];           // 食物数组
let score = 0;            // 得分
let nextFoodIndex = 0;    // 下一个食物索引
let isGameOver = false;   // 游戏结束标志
let shakeCount = 1;       // 颠勺剩余次数
let isShaking = false;    // 颠勺冷却期标志
```

### 食物配置
```javascript
const FOODS = [
    { emoji: '🥟', image: 'pic/jiaozi.png', name: '饺子', size: 40, score: 1 },
    { emoji: '🥙', image: 'pic/tangyuan.png', name: '汤圆', size: 50, score: 3 },
    { emoji: '🥙', image: 'pic/chunjuan.png', name: '春卷', size: 60, score: 6 },
    { emoji: '🐟', image: 'pic/luyu.png', name: '清蒸鲈鱼', size: 70, score: 10 },
    { emoji: '🍖', image: 'pic/hongshaorou.png', name: '东坡肉', size: 80, score: 15 },
    { emoji: '🥘', image: 'pic/babao.png', name: '八宝饭', size: 90, score: 21 },
    { emoji: '🍗', image: 'pic/ji.png', name: '白切鸡', size: 100, score: 28 },
    { emoji: '🦀', image: 'pic/pangxie.png', name: '大闸蟹', size: 110, score: 36 },
    { emoji: '🦞', image: 'pic/longxia.png', name: '清蒸龙虾', size: 120, score: 45 },
    { emoji: '🍚', image: 'pic/baozhi.png', name: '鲍鱼捞饭', size: 130, score: 55 },
    { emoji: '🍲', image: 'pic/fotiaoqiang.png', name: '佛跳墙', size: 145, score: 66 }
];
```

### 核心函数
- `init()` - 初始化游戏
- `dropFood(x)` - 在指定位置投放食物
- `updatePhysics()` - 物理引擎更新
- `checkCollisions()` - 碰撞检测
- `mergeFoods(index1, index2)` - 合并食物
- `checkGameOver()` - 游戏结束判定
- `useShakeTool()` - 显示颠勺确认对话框
- `confirmUseShake()` - 执行颠勺功能

### 颠勺相关函数
- `showConfirmDialog()` - 显示确认对话框
- `closeConfirmDialog()` - 关闭确认对话框
- `confirmUseShake()` - 确认使用颠勺
- `updateShakeButton()` - 更新按钮状态

---

## UI 设计规范

### 颜色系统
- **主题色**: `#d63031` (红色)
- **强调色**: `#ff6b6b` (浅红)
- **文字色**: `#2d3436` (深灰)
- **辅助色**: `#636e72` (灰色)

### 组件样式
- **圆角**: 统一使用 8px - 20px
- **阴影**: `0 2px 4px rgba(0, 0, 0, 0.15)`
- **过渡**: `0.3s` 平滑过渡
- **按钮**: hover 时放大 1.05 倍

### 颠勺按钮
```css
background: rgba(214, 48, 49, 0.9);
border: 2px solid #d63031;
border-radius: 20px;  /* 胶囊形状 */
padding: 6px 12px;
```

---

## 部署

### GitHub Pages 自动部署
- 仓库：`2003crimson.github.io` (用户主页仓库)
- 推送到 main 分支后自动触发部署
- 部署时间：1-3 分钟
- 文件路径：`year-end-dinner.html`

### 更新流程
1. 修改代码
2. `git add year-end-dinner.html`
3. `git commit -m "描述信息"`
4. `git push`

---

## 未来改进方向

### 可能的功能扩展
1. 多种道具类型（如：消除某个食物、冻结时间等）
2. 难度选择（简单/普通/困难）
3. 最高分记录（localStorage）
4. 音效系统
5. 更多合成链（节日主题）
6. 成就系统

### 技术优化
1. 性能优化（大量食物时的帧率）
2. 移动端适配优化
3. 触摸事件支持
4. 离线缓存（PWA）

---

## 已知问题

### 已解决
- ✅ 颠勺时误判游戏结束 → 添加 2 秒冷却期
- ✅ 玩家不知道颠勺功能 → 添加确认对话框说明
- ✅ 颠勺按钮位置太显眼易误触 → 移到下方并简化文案
- ✅ 对话框打开时点击背景投放食物 → 添加状态检查
- ✅ 点击对话框按钮时触发食物掉落 → 阻止事件冒泡（event.stopPropagation）
- ✅ 移动端食物不会落到底 → 物理引擎改为动态获取容器尺寸
- ✅ 移动端底部和边缘食物被截断 → 物理引擎预留 8px 边距空间，避免边框和圆角裁剪
- ✅ 桌面端下落速度太慢 → 根据设备类型使用不同的物理参数
- ✅ 快速点击导致虚线上方堆积 → 添加投放区域检查

### 待优化
- ⚠️ 食物过多时可能影响性能
- ⚠️ 没有最高分记录功能
- ⚠️ 可以添加更多移动端手势支持

---

## 开发历史

### 2026-02-06
- 将游戏从 emoji 显示升级为自定义图片显示
- 更新合成链：饺子→汤圆→春卷→清蒸鲈鱼→东坡肉→八宝饭→白切鸡→大闸蟹→清蒸龙虾→鲍鱼捞饭→佛跳墙
- 添加 11 张食物图片到 `pic/` 目录
- 更新 `createFood` 函数使用 `<img>` 标签而非 emoji 文本
- 更新"下一个"预览和底部食物列表显示图片
- 创建基础游戏
- 添加颠勺道具功能
- UI 迭代优化（3 个版本）
- 添加确认对话框
- 修复冷却期误判问题
- 修复对话框交互问题（点击按钮触发食物掉落）
- 添加移动端响应式设计
  - 小屏幕适配（max-width: 420px）
  - 游戏区域高度调整为 70vh（最大 600px）
  - 颠勺按钮放大增强可见性
  - 小屏幕高度时隐藏食物列表
  - 确认对话框响应式适配
- 修复移动端物理引擎问题
  - 将固定的容器尺寸改为动态获取（offsetWidth/offsetHeight）
  - 确保食物正确响应不同屏幕尺寸的边界
  - 预留 8px 边距空间，防止边框和圆角（border-radius: 10px）裁剪食物
- 添加设备自适应物理引擎
  - 移动端（≤420px）：gravity 0.3, 初始速度 2（适中）
  - 桌面端（>420px）：gravity 0.7, 初始速度 4（更快）
  - 根据屏幕宽度动态选择物理参数
- 添加投放区域检查
  - 检查虚线上方区域（y < 80）是否有食物
  - 防止快速点击导致上方堆积和游戏失败

---

## 联系方式

如有问题或建议，请访问：https://github.com/2003crimson/2003crimson.github.io

---

## 给 AI 助手的重要提示

### 在新对话开始时请先阅读此部分

#### 项目特点
1. **单文件架构**：所有代码都在 `year-end-dinner.html` 一个文件中，方便部署但需要注意代码组织
2. **无框架依赖**：纯原生 JS，不要引入任何外部库
3. **玩家反馈驱动**：颠勺功能经过了多次基于玩家反馈的迭代（详见上方"颠勺道具功能设计"）

#### 设计原则
1. **简单优先**：避免过度设计，保持代码简单易懂
2. **用户体验优先**：功能要直观，避免让玩家困惑
3. **渐进式改进**：先实现基础功能，再根据反馈优化

#### 容易误解的点
⚠️ **颠勺按钮的设计是有意为之**：
- 不是"太简单"，而是经过 3 轮优化的结果
- 确认对话框是必须的，不是多余的步骤
- 冷却期机制是必要的，否则会误判游戏结束

⚠️ **UI 设计风格**：
- 采用红色主题色（#d63031），与中国新年主题一致
- 圆角、阴影等要保持统一风格
- 不要随意改变颜色方案

⚠️ **部署方式**：
- 仓库是 `2003crimson.github.io`（用户主页仓库）
- 直接推送 main 分支即可自动部署到 GitHub Pages
- 不需要 GitHub Actions workflow

#### 常见任务参考
- **添加新功能**：参考颠勺功能的实现流程（先基础实现 → 优化 UI → 添加确认 → 修复边界问题）
- **UI 调整**：保持现有的设计系统和颜色方案
- **Bug 修复**：注意检查是否会触发游戏结束误判

#### 更新此文档
每次重要的功能改动或设计决策后，请更新此文档，特别是：
- 当前状态
- 已知问题
- 开发历史

这样可以避免未来的对话重复讨论已经解决的问题。
