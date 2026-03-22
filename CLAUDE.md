# What we are building

Placeholder here.

---

# Public Server

You can access the public server using:

```bash
ssh newserver
```

This server is publicly accessible and will host the page.

---

# Your Job

Continue working until **all features are confirmed working and all tests pass**.

Use your **bash and Chrome MCP** to verify that all functions work.

Keep working until **all functions are confirmed working**.

---

---

# Deploy SOP (部署标准流程)

**关键原则：在 worktree 中修改代码后，必须在 main repo 中 pull 最新代码再构建，不要在 worktree 中构建。**

完整部署流程：

```bash
# 1. 在 worktree 中提交并推送到 main
git add <files>
git commit -m "feat/fix: ..."
git fetch origin main && git rebase origin/main
git push origin <worktree-branch>:main

# 2. 在 main repo 中同步并构建（关键！）
cd /home/mhliu/podcast-transcript-forum
git pull origin main
cd client && rm -rf dist && npx vite build

# 3. 上传构建产物到服务器
rsync -avz --delete /home/mhliu/podcast-transcript-forum/client/dist/ newserver:/home/prod/podcast-forum/client/dist/

# 4. 同步服务端代码（如果改了 server/）
ssh newserver "cd /home/prod/podcast-forum && git pull origin main"

# 5. 重启服务器
ssh newserver "kill $(ssh newserver 'ss -tlnp | grep 4010 | grep -oP "pid=\d+" | grep -oP "\d+"') 2>/dev/null"
ssh newserver "cd /home/prod/podcast-forum && nohup node server/src/index.js > server.log 2>&1 & echo PID=\$!"

# 6. 验证
sleep 2 && ssh newserver "curl -s -o /dev/null -w '%{http_code}' http://localhost:4010/"
```

**常见踩坑**：
- 服务器 Node v20 无法运行 `vite build`（需 Node ≥ 22），所以必须在本地构建
- `express.static` 对 index.html 已配置 `no-cache`，部署后用户刷新即可获取最新版本
- 不要混淆 worktree 的构建产物和 main repo 的构建产物

---

# Submit Code

All code should be committed on a **task branch**:

```bash
git commit
```

---

# Merge + Test

```bash
git fetch origin && git merge origin/main
npm test
```

---

# Auto Commit and Merge to Main

After each feature is completed and all tests pass, you must **commit and then merge to `main`**.

Before each commit, update related documentation if new features are added.

Workflow:

1. Sync main branch

```bash
git fetch origin main
```

2. Rebase your task branch

```bash
git rebase origin/main
```

3. If rebase fails, follow the **Conflict Resolution** section below.

4. If rebase succeeds:

```bash
git merge main task-xxx
git push origin main
```

5. Continue with the next task.

6. If **any step fails**, return to **Step 5** in the workflow.

---

# Mark Task Completion

Update `dev-tasks.json` **before cleanup** to prevent losing task status if the process is killed.

---

# Cleanup

After task completion:

- Remove the worktree:

```bash
git worktree remove
```

- Delete the local branch
- Delete the remote task branch
- Restart the development server

---

# Knowledge Capture (Optional)

Record lessons learned in `PROGRESS.md`.

This is optional because task status is already recorded in `dev-tasks.json`, so even if the process is killed, the task state is preserved.

---

# Multi-Instance Parallel Development (Git Worktree)

## Architecture Overview

Multiple Claude Code instances can run **in parallel**, with each instance working in an **independent `git worktree`**.

---

## Parallel Development Workflow

```
┌──────────────────────────────────────────────┐
│              Parallel Development Workflow   │
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

                     (isolated experimental data)
```

---

# ⚠️ Symlink Is Forbidden

Do **not create symbolic links** for:

```
PROGRESS.md
```

Always edit the main repository file directly using:

```bash
git -C
```

---

# Conflict Resolution

## Handling Rebase Failures

1. If the error is `"unstaged changes"`:

Commit or stash the current modifications first.

```bash
git commit
```

or

```bash
git stash
```

---

2. If there are **merge conflicts**:

Check conflicting files:

```bash
git status
```

Open the conflicting files and understand the changes from both sides.

Manually resolve the conflicts.

Stage the resolved files:

```bash
git add <resolved-files>
```

Continue the rebase:

```bash
git rebase --continue
```

Repeat until the rebase completes.

---

# Handling Test Failures

1. Run tests:

```bash
npm test
```

2. If tests fail:

- Analyze the error messages
- Fix the bugs in the code

3. Run tests again until **all tests pass**.

4. Commit the fix:

```bash
git commit -m "fix: ..."
```

---

# Never Give Up

If a **rebase or test fails**, you **must resolve the issue before continuing**.

Do **not mark the task as failed**.

---

# Knowledge Recording

Whenever you encounter a problem or complete an important change, record it in:

```
PROGRESS.md
```

Include:

- What problem occurred
- How it was solved
- How to avoid it in the future
- **The corresponding Git commit ID**

---

# Important Rule

**Do not make the same mistake twice.**

And remember:

> **After every new feature, update the README and documentation.**
