# Bazel Wiki Ingest Workflow

## 📋 新工作流：两级目录 + 零Token追踪

### 目录结构

```
raw/
├── inbox/          ← 📥 新资料（待处理）
│   └── [未 ingest 的源文件]
├── processed/      ← ✅ 已完成的资料
│   └── [已 ingest 的源文件]
├── articles/       ← 网页、博客文章
├── books/          ← 书籍章节
├── videos/         ← 视频转录
├── experiments/    ← 实验代码
└── assets/         ← 图片、图表
```

---

## 🔄 工作流步骤

### Step 1: 添加新资料
将新的源文档放入 `raw/inbox/`：
- 从官方文档复制粘贴
- 从网页剪裁保存
- 添加视频转录

### Step 2: 告诉我处理
```bash
# 在对话中说：
"处理 raw/inbox/ 中的所有文件"
```

### Step 3: 我 ingest 全部
- 我自动扫描 `raw/inbox/` 中的**所有**文件
- 为每个文件创建/更新 wiki 页面
- 更新 index.md 和 log.md

### Step 4: 移动到 processed
完成后，我会告诉你，然后执行：
```bash
mv raw/inbox/* raw/processed/
```

或者你手动在 Obsidian 侧边栏拖拽。

---

## ✨ 优势

| 方面 | 优势 |
|------|------|
| **直观性** | 一眼看 `raw/inbox/` 就知道还有什么要处理 |
| **Token 开销** | **零额外开销** — 不需要维护处理状态列表 |
| **原子操作** | 处理完 → move → done，简单明快 |
| **可扩展** | 用户随时丢新文件，我自动处理 |
| **易于重做** | 需要重新处理某个源？从 processed/ 移回 inbox/ |
| **重复幂等** | 同一个文件 ingest 多次不会出问题 |

---

## 📊 当前状态

**raw/inbox/ (待处理) — 13 个源文件：**
1. General Rules.md ⭐ (最高优先)
2. Python Rules.md ⭐
3. C++ Rules.md ⭐
4. Remote Execution Overview.md
5. Adapting Bazel Rules for Remote Execution.md
6. Shell Rules.md
7. Protocol Buffer Rules.md
8. Platforms and Toolchains Rules.md
9. Extra Actions Rules.md
10. Objective-C Rules.md
11. Bazel registries.md
12. Calling Bazel from scripts.md
13. Clientserver implementation.md

**raw/processed/ (已完成) — 17 个源文件：**
- BUILD Style Guide, Recommended Rules, Sharing Variables, ...
- Commands and Options, Repository Rules, Bazel modules, ...
- Module extensions, Write bazelrc configuration, ...

---

## 🎯 推荐下一步

**立即 ingest top 3（填补最大 gap）：**
```
处理这三个文件：General Rules, Python Rules, C++ Rules
```

这会填补：
- ✅ Rules & rule anatomy (70% open → 30% open)
- ✅ Language-specific guides (Python & C++)

---

## 💡 Tips

1. **批量处理** — 一次 ingest 多个文件效率更高
2. **优先级顺序** — 按上面列表顺序处理，gaps 填充最快
3. **质量检查** — 每次 ingest 后检查 [[wiki/log.md]] 看结果
4. **反复迭代** — 如果 ingest 结果不满意，文件移回 inbox/ 重新处理

---

**Last Updated:** 2026-07-19
**Workflow Version:** 2.0 (两级目录系统)
