### 任务生命周期

1. **领取任务**：原子操作，从 `data/dev-tasks.json` 获取任务  

2. **创建工作区**：
   - `git worktree add -b task/xxx ../voice-notes-worktrees/task-xxx`
   - 创建隔离的 `data/` 目录（实验数据库）
   - Symlink 共享文件：`dev-tasks.json`, `api-key.json`（⚠️ `PROGRESS.md` 禁止 symlink）
   - Symlink `node_modules/` 加速启动
   - 分配专属端口  

3. **实现功能**：Claude Code 在隔离环境中工作  

4. **提交代码**：`git commit` 在任务分支  

5. **Merge + 测试**：
   - `git fetch origin && git merge origin/main`
   - `npm test`  

6. **自动合并到 main**：
   - `git fetch origin main`
   - `git rebase origin/main`，如果失败，按照下面的“冲突处理”来 resolve rebase conflict
   - 如果成功，则：
     - `git merge main task-xxx`
     - `git push origin main`
     - 继续执行下一步
   - 如果这一步有任何失败，则退回到步骤 5  

7. **标记完成**：更新 `dev-tasks.json`（必须在清理之前，防止进程被杀时任务状态丢失）  

8. **清理**：
   - `git worktree remove` + 删除本地分支
   - 删除远程 task 分支
   - 重启 dev server  

9. **经验沉淀**：在 `PROGRESS.md` 记录经验教训（可选，如果被杀也不影响任务状态）

## 多实例并行开发（Git Worktree）

### 架构说明

支持多个 Claude Code 实例并行工作，每个实例在独立的 `git worktree` 中执行任务。

---

### 并行开发工作流

```
┌──────────────────────────────────────────────┐
│                并行开发工作流                │
└──────────────────────────────────────────────┘

   ┌────────────────────┐   ┌────────────────────┐   ┌────────────────────┐
   │      Worker 1      │   │      Worker 2      │   │      Worker 3      │
   │     port:5200      │   │     port:5201      │   │     port:5202      │
   │      worktree      │   │      worktree      │   │      worktree      │
   └────────────────────┘   └────────────────────┘   └────────────────────┘
            │                         │                         │
        ┌────────┐               ┌────────┐               ┌────────┐
        │ data/  │               │ data/  │               │ data/  │
        └────────┘               └────────┘               └────────┘

                           （隔离的实验数据）
```

---

### 共享文件（symlink）

- `dev-tasks.json`（任务队列）
- `dev-task.lock`（文件锁）
- `api-key.json`（API 密钥）

---

### ⚠️ 禁止 symlink

- `PROGRESS.md`（直接用 `git -C` 编辑主仓库文件）

### 冲突处理

#### Rebase 失败时的处理流程

1. 如果是 `"unstaged changes"` 错误，先 `commit` 或 `stash` 当前改动  
2. 如果有 merge conflicts：
   - 查看冲突文件：`git status`
   - 读取冲突文件内容，理解双方改动意图
   - 手动解决冲突（保留正确的代码）
   - `git add <resolved-files>`
   - `git rebase --continue`
3. 重复直到 rebase 完成  

---

#### 测试失败时的处理流程

1. 运行测试：`npm test`
2. 如果失败，分析错误信息
3. 修复代码中的 bug
4. 重新运行测试，直到全部通过
5. 提交修复：`git commit -m "fix: ..."`

---

**不要放弃**：遇到 rebase 或测试失败时，必须解决问题后才能继续，不能直接标记任务失败。

---

## 经验教训沉淀

每次遇到问题或完成重要改动后，要在 [`PROGRESS.md`](./PROGRESS.md) 中记录：

- 遇到了什么问题
- 如何解决的
- 以后如何避免
- **必须附上 git commit ID**

---

**同样的问题不要犯两次！**