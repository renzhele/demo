# Graphite 代码审查学习路径

你写的 graphit 多半是指 [Graphite](https://graphite.dev)（graphite.dev）：它仍然用 Git + GitHub，但在「分支/PR 的组织方式」和「合并与评审体验」上和「普通开分支 + 开一个巨大 PR」有明显差别。下面是一份操作型学习路径：按顺序做，每一步都能体会到和普通 GitHub 不一样的地方。

---

## 1. 先建立「对比心智模型」（约 5 分钟）


| 普通 Git + GitHub                | Graphite 想解决的问题                         |
| ------------------------------ | --------------------------------------- |
| 一个功能 = 一个很大的 PR，评审慢、CI 慢、难拆    | 堆叠 PR（stack）：一串小 PR，每个可独立评审/测试          |
| 改底层 PR 后要手搓 rebase / 改一堆上游分支   | `gt modify` + 自动 restack：改中间一层，上面整串跟着对齐 |
| 多个 PR 合并顺序乱容易冲突                | 按栈顺序合并 / merge queue（团队启用时）更可控          |
| Review 分散在各个 repo 的 GitHub 通知里 | Graphite Inbox + 过滤：跨 repo 的评审队列（团队场景）  |


你要学的「独特价值」核心就三件事：**栈式 PR**、`**gt` 对 rebase/restack 的封装**、**Graphite 网页上的栈视图与合并方式**。AI Review、Inbox、Insights 是加分项。

---

## 2. 环境准备（一次到位）

1. 安装 CLI：`gt`（按 CLI Quick Start 官方文档安装）。
2. 在你自己的测试仓库里执行 `gt init`，选好 trunk（一般是 `main`）。
3. 浏览器登录 Graphite 并完成对 GitHub 的授权（组织仓库需要管理员装 App，个人 Hobby 可先玩个人仓库）。

---

## 练习说明：角色与分工

文档里**不单独写「committer」**，因为这个词容易和两种含义混在一起：


| 含义                        | 说明                                                                                          |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| Git 提交元数据里的 **Committer** | 执行 `git commit` 时记录在对象里的身份，通常与 **Author** 相同；练习 A–D 里一般都是你自己。                               |
| 团队口语里的「有合并权的人」            | 能在 GitHub 上把 PR 并进默认分支的人（常叫 **maintainer**、**合并者**，或沿用「committer」）；对应下面练习 7 里「点 Merge」的那一方。 |


**练习 3–6（A–D）与自检：** 按**单人、单账号**设计。你同时扮演**写改代码的人**（作者/开发者）和**用 `gt` 维护栈、推到远程的人**；不必再拆「谁是开发者、谁是 committer」。

**练习 7（评审视角）：** 刻意换成**评审者**：读栈、评论、批准或合并。一人练习时，可以用**另一个 GitHub 账号** fork/协作，或请同事当你的评审者；若你本人也有合并权限，也可以自己 Merge，此时**作者与合并者是同一人**——这在个人仓库里很常见。

**练习 D** 里「别人已经往 `main` 合了东西」：不必虚构角色名，把它理解成**除你当前这条栈以外的任意方式更新了 trunk**（第二个本地 clone、队友、机器人、网页上直接 Merge 均可）。

---

## 3. 练习 A：单 PR 也要用 `gt`（体会「提交/修改」差异）

**目标：** 感受 `gt create` / `gt submit` / `gt modify` 和普通 `git commit` + `git push` 的差别。

```bash
gt checkout main
```

改一个小文件，然后：

```bash
gt create --all --message "chore: tiny change for graphite practice"
gt submit
```

再改同一条 PR 的内容：

```bash
gt modify --all
gt submit
```

**你要刻意对比的点：** 在 GitHub 上你仍然看到「一个 PR」，但本地你用的是 amend + 再 submit 的心智模型；后面叠栈时这套会反复用到。

---

## 4. 练习 B：刻意做「两层栈」（核心价值）

**目标：** 这是和普通「开一个分支到底」最大的不同——第二个 PR 基于第一个 PR 的 tip，而不是基于 `main`。

在练习 A 的分支上（或 `gt checkout` 选中有 PR 的分支），再改代码：

```bash
gt create --all --message "feat: second slice on top of first PR"
gt submit --stack
```

本地看栈：

```bash
gt log short
# 或 gt ls
```

打开 Graphite 看 PR 页面（或 `gt pr`）：应能看到 PR 之间的上下层关系。

**成功标准：** GitHub 上可能是两个 PR，但 Graphite 里它们显示为一条链；评审的人可以按「从下往上」读 diff（底层先合、上层再合）。

---

## 5. 练习 C：改「栈中间」的一层（体会 restack）

**目标：** 普通做法很痛；Graphite 的价值是改底层，上层自动重新对齐。

`gt checkout` 切到栈底那个分支（第一个 PR）。改文件，然后：

```bash
gt modify --all
gt submit --stack
```

看上层 PR 的 diff：应反映底层变更（无需你手动对每个上层分支 rebase）。

**可选：** 用「多一条 commit」的方式再练一次：

```bash
gt modify -cam "chore: address review on base PR"
gt submit --stack
```

---

## 6. 练习 D：`gt sync`（和 main 漂移对抗）

**目标：** 体会「整条栈一起跟 trunk 对齐」的自动化路径。

1. 在 GitHub 上往 `main` 合入别的东西（或本地另一个 clone 推上去），让你的栈落后于 `main`。
2. 回到你的仓库：

```bash
gt sync
```

若提示冲突，按 CLI 指引在对应分支解决后 `gt restack`。

**对比普通 Git：** 你要自己记「哪些分支要 rebase、顺序怎样」；`gt sync` 把「拉 trunk + 重排栈」收成一条命令。

---

## 7. 以「评审者」身份用 Graphite（体现产品差异）

建议你固定做这几件事（和只在 GitHub Files 里点评论不同）：

- 从栈底开始审：先批准/要求修改最下面 PR，再往上走；这样评论和依赖关系一致。
- 用栈视图理解「为什么拆成多个 PR」：每个 PR 是否真的是可独立理解的切片（若只是机械切碎但没有叙事，价值会打折）。
- 合并时试「只合栈的一部分」（若团队允许）：在 Graphite UI 里从某一层的 Merge 操作，观察上层 PR 如何相对 `main` 变化——这是纯 GitHub 单页 PR 不太直观的地方。
- 若团队开了 merge queue，再观察：按顺序进队、冲突时行为——这是「多 PR 协作」规模化时的差异点。

---

## 8. AI Review：怎么「用出差异」而不是当玩具

把它当**一稿预审**，而不是终审：

- 在每个小 PR 上跑 AI 建议，优先处理：接口契约、命名、明显漏测、跨文件一致性问题。
- 栈场景里更有价值：让 AI 看「本层相对上一层」是否引入了重复抽象或破坏了下层约定。
- **避免：** 整栈合成一个巨大 diff 再让 AI 看——那又退回到「巨型 PR」问题了。

---

## 9. Inbox / 规则（多人时才有感觉）

若你是多仓库、多 PR 的评审者：

- 在 Graphite 里建 filter（例如：我是 reviewer、某 team、某 label）。
- 把「每天先清空 Inbox」当成工作流，对比 GitHub 通知瀑布流。

个人单仓库时，这部分体感会弱一些。

---

## 10. 自检：你是否真的「会用 Graphite」了

能不假思索回答这几个问题，就说明独特功能已经内化：

1. 什么时候该 `gt submit` 和 `gt submit --stack`？
2. 改栈底后，为什么上层 PR 的 diff 会变？是谁在帮你做对齐？
3. `gt sync` 失败后，下一步通常是 checkout 哪类分支、`gt restack` 解决什么？
4. 合并栈时，从「中间某层」合并与从栈顶合并，对剩余 PR 分别意味着什么？

