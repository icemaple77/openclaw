# OpenClaw 优化实现方案
## Fork Subagent / 提示词分区 / 并行工具调用

---

## 1. Fork Subagent

### 目标
让 subagent 共享父会话的 system prompt 缓存，减少 API 调用浪费。

### 改动文件
`src/agents/acp-spawn.ts`

### 实现思路

```
当前流程:
  sessions_spawn → 创建新 session → 独立 system prompt → 独立 API 调用 (无缓存共享)

Fork 流程:
  sessions_spawn → 创建子 session → 继承父 session 的:
    1. system prompt hash (缓存 key)
    2. 关键上下文文件 (AGENTS.md / SOUL.md / USER.md)
    3. 工具定义
    不需要传递:
    4. 对话历史 (不是 fork)
  → 子 session 复用父 session 的 API 缓存
```

### 需要修改的函数
- `createChildSession()` - 在创建 session 时传递缓存 key
- 新增 `forkContext` 参数，包含父 session 的 system prompt hash

### 风险
低 - 不影响现有 subagent 行为，新增 fork 选项

---

## 2. 提示词分区 (Prompt Caching)

### 目标
将 system prompt 分为静态/动态分区，静态分区复用 API 缓存

### 改动文件
`src/agents/system-prompt.ts`
`src/agents/pi-embedded-runner/`

### 实现思路

```
当前: [全部 System Prompt] → [用户消息] → API 调用 (无缓存)

分区后: [静态 Prompt (缓存)] [动态 Prompt] → [用户消息] → API 调用 (静态部分命中缓存)

静态 (缓存，不常变):
  - 工具定义
  - 安全规则
  - 行为准则
  - 人格设定

动态 (每轮变):
  - 当前时间
  - 情绪状态
  - 工具结果
```

### 技术方案
利用 `anthropic-cache-control-payload.ts` 中的缓存控制逻辑，给静态部分添加 `ephemeral` 缓存标记

### 风险
中 - 需要理解 API 缓存机制的实现细节

---

## 3. 并行工具调用

### 目标
同一轮中并行调用多个独立工具，减少等待时间

### 改动文件
`src/agents/pi-embedded-runner/run.ts` (agent 核心循环)

### 实现思路

```
当前: 工具1 → 等待 → 工具2 → 等待 → (串行)
并行: 工具1 → 并行 → 工具2 → (同时执行)
       工具3 →       工具4 →
条件: 工具之间无数据依赖
```

### 检测依赖
工具之间如果输入输出不存在引用关系，则可以并行：
- `Bash(read file A)` + `Bash(read file B)` → 无依赖 → 可并行
- `Bash(git log)` + `Bash(git diff HEAD)` → 有依赖 → 串行

### 风险
高 - 需要改 agent 核心执行循环，可能影响所有工具调用行为

---

## 实施优先级

```
1. Fork Subagent     ← 低风险，高收益 (节省 40-60% token)
2. 提示词分区        ← 中风险，高收益 (节省 30-50% token)
3. 并行工具调用      ← 高风险，中收益 (快 2-3 倍)
```

## 分支策略

每个优化在 `feature/含烟-优化-v1` 分支上独立 commit。
公子可以在 GitHub 上逐 commit review，选择合入或回退。
