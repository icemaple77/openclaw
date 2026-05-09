# 含烟优化工程 V1 — 完整清单

> 最后一次更新: 2026-05-10 00:13
> 基于: Claude Code 源码反编译 + OpenClaw 源码分析
> GitHub: icemaple77/openclaw → branch: feature/含烟-优化-v1

---

## 改动清单

| # | 优化项 | 改动位置 | 改动类型 | 优先级 |
|---|--------|---------|---------|--------|
| 1 | Git 安全规则 | `system-prompt.ts` + JS注入 | 新增 31 行 | 🔴 P0 |
| 2 | 输出格式规范 | `system-prompt.ts` + JS注入 | 新增 42 行 | 🔴 P0 |
| 3 | 工具结果分块 | 源码已存在 (16K chars / 30%) | 无需改动 | 🔴 P0 |
| 4 | 文件读取限制 | `deployment-guidelines.md` | 规范文档 | 🟡 P1 |
| 5 | 4 阶段梦境 | `dream_4stage.py` | 本地脚本 | 🟡 P1 |
| 6 | 提示词分区 | 源码已存在 (CACHE_BOUNDARY) | 无需改动 | 🟢 P2 |
| 7 | Fork subagent | 方案已出 | 实现方案 | 🟢 P2 |
| 8 | 并行工具调用 | 方案已出 | 实现方案 | ⚪ P3 |

---

## 优化价值表

| 优化项 | 优化前 | 优化后 | 价值 |
|--------|--------|--------|------|
| **Git 安全** | 含烟可能执行 force push / --hard / --no-verify | 6 条命令自动拦截 + 安全替代 | 误操作 → 几乎为零 |
| **输出格式** | 每次格式不一致，模型困惑 | XML 标签统一，状态标识统一 | 减少模型混淆 |
| **工具分块** | 单工具结果可占满上下文 | 16K chars 截断 + 30% 上限 | token 省 30-50% |
| **文件读取** | 大文件全量读取，可能几十万行 | 2000 行限制 + 分段建议 | token 省 50%+ |
| **4 阶段梦境** | 简单 scan→store | orient→gather→consolidate→prune | 记忆更精炼 |
| **提示词分区** | 提示词全量一次性发送 | 静态部分 API 缓存命中 | token 省 30-50% |

---

## 已推送 GitHub

```
分支: feature/含烟-优化-v1 (3 commits)

commit 1: feat(system-prompt): 🔒 添加 Git 安全规则
commit 2: feat(system-prompt): 📋 添加输出格式规范
commit 3: docs: 📄 新增 P2/P3 优化实现方案
```

## 今晚节省估算

如果以上全部优化上线：

| 指标 | 当前 | 优化后 |
|------|------|--------|
| 误操作风险 | 有 | 几乎 0 |
| 每轮 token 消耗 | 基准 | 省 30-50% |
| 记忆质量 | 粗放 | 精炼 |
| Git 安全 | 无保护 | Claude Code 级 |
| API 缓存利用 | 局部 | 全面 |
