# 用 DeepSeek Harness（DSH）做 Git：组员上手指南

这份文档不教你背 git 命令，而是教你**怎么用 DSH 把活干完**。
你在对话框里说人话，DSH 替你跑命令、读报错、改文件；你负责**看懂它在干什么**和**批准危险操作**。

- 练手仓库：<https://github.com/Camarovl/git-test>
- 📌 **练手时会走一次"合并到 main"，正式项目平时不要合并** —— 规则见 [附录 C](#附录-c正式开工的时候应该怎么做)。

---

## 〇、为什么统一用 DSH

DSH（DeepSeek Harness）是一个**能直接在你电脑上干活的 AI 编程助手**：你说要什么结果，它去跑命令、改文件、读报错，遇到危险操作才找你确认。

| | 不用 DSH | 用 DSH |
|---|---|---|
| git 报错了 | 截图去搜，对着英文猜半天 | 整段红字粘给它，直接问怎么办 |
| 想建分支 / 提交 | 记一堆命令，怕敲错 | 说人话，它替你执行 |
| 环境没配好 | 花一下午装东西 | 让它体检并修 |

### 🎬 入门视频（50 分钟，全组统一教材）

**【全网最全】DeepSeek Harness超详细教程！50分钟吃透DSH，让SuperAgent给你干活！**
👉 <https://www.bilibili.com/video/BV1REx46NEc4/>

看完做到三件事就行：① 装上 DSH Desktop；② 会新建会话、打字提需求；③ 知道审批弹窗要看理由再点。
时间紧就先看前 20 分钟（安装 + 界面 + 第一次对话），剩下的做完练手任务再补。

---

## 一、开工前准备（只做一次，约 20 分钟）

### ① 装 Git for Windows

下载地址：<https://git-scm.com/download/win>（64-bit 安装包，约 60 MB）

安装时**一路 Next 即可**，只有两处注意：

| 界面 | 选什么 | 说明 |
|---|---|---|
| Choosing Default Editor | **Use Visual Studio Code / Notepad**（看你习惯） | 选 Vim 的话新手改提交信息会卡住 |
| Adjusting your name and email | 先随便填 | 下一步会用正确身份覆盖掉 |

⚠️ 别勾选 "Enable file system caching" 之外的奇怪选项，默认值最稳。

装完验证 —— 打开 **PowerShell**（Win + R 输入 `powershell`）：

```powershell
git --version
```

出现 `git version 2.xx.x.windows.x` 就成功了。**报"不是内部或外部命令"** = 没装好或没进 PATH，重装一次。

### ② 装 Python 3.11

下载地址：<https://www.python.org/downloads/release/python-3119/>（拉到页面底部，选 **Windows installer (64-bit)**）

🔴 **安装第一屏必须勾选 "Add python.exe to PATH"**，不勾后面全项目都会踩坑。
其余一路 Next。

验证：

```powershell
python --version      # 期望输出 Python 3.11.x
pip --version
```

### ③ 注册 GitHub 账号

<https://github.com/signup> —— 用你自己的邮箱注册。**用户名随你起**，唯一要求是别人看到 `test/<你的名字>` 这种分支时能认出是你，比如叫 `zhangsan`、`lisi99` 都行。

注册后去 **Settings → Emails** 确认那个邮箱是 **Verified** 状态。
Git 提交记录靠**邮箱**认人，邮箱没在 GitHub 上验证过，Contributors 页面就不会显示你。

### ④ 配 git 身份（每人必做）

打开 PowerShell 逐行执行，换成自己的信息：

```bash
git config --global user.name  "你的名字或拼音"
git config --global user.email "<你在 GitHub 注册用的同一个邮箱>"
git config --global init.defaultBranch main     # 与 GitHub 默认分支保持一致
git config --global --list                       # 检查是否都写进去了
```

🔴 **这里的邮箱必须和你 GitHub 账号里 Verified 的那个一模一样**，一个字母都不能差。

| | 结果 |
|---|---|
| 两边邮箱一致 | ✅ 提交算你的，Contributors 有你头像，答辩能证明你做了啥 |
| 邮箱不一致 / 没验证 | ❌ 提交会显示成灰色无名用户，GitHub 不认这是你干的活 |

常见翻车：注册 GitHub 用的是 QQ 邮箱，配 git 时随手填了学校邮箱或 163 邮箱 —— 提交照样能推上去，但功劳不算你的。

不确定自己 GitHub 用的是哪个邮箱？登录后看 <https://github.com/settings/emails>。

### ⑤ 测试能不能连上 GitHub

```powershell
curl.exe -I https://github.com --max-time 15
```

看到 `HTTP/1.1 200` 或 `301` = 通；看到 `Recv failure` / `Could not connect` = 不通，**先回头做第一节第⑥步**。

### ⑥ 让命令行也能走代理（第⑤步不通才需要）

浏览器能打开 GitHub ≠ 命令行能 push —— 浏览器走代理，git 默认不走。
先看你代理软件的端口（Clash 常见 `7890`，v2rayN 常见 `10808` / `10809`），然后：

```bash
git config --global http.proxy  http://127.0.0.1:<你的端口>
git config --global https.proxy http://127.0.0.1:<你的端口>

# 撤销配置
git config --global --unset http.proxy
git config --global --unset https.proxy
```

或者更省事：**打开代理软件的 TUN / 系统代理模式**，让所有程序都走。

> 🔴 代理软件必须**常驻运行**才能 push。答辩前一晚发现某人代理没开，是最常见的事故。

### 全部装好后，交给组长这个清单

- [ ] `git --version` 有版本号
- [ ] `python --version` 显示 3.11.x
- [ ] GitHub 已注册且邮箱 Verified
- [ ] `git config --global --list` 里能看到自己的 name / email，且**邮箱与 GitHub 那个完全一致**
- [ ] `curl.exe -I https://github.com` 返回 200/301

---

## 二、核心心智模型

| 角色 | 干什么 |
|---|---|
| **你** | 说清楚要什么结果 + 审批请求 + 检查结果 |
| **DSH** | 跑 `git` 命令、读输出、编辑文件、诊断报错 |
| **审批弹窗** | 「写工作目录之外」「联网」「更高权限」的操作都会问你，看清理由再点 |

⚠️ DSH **不会等你想清楚** —— 你说"帮我加个登录页"，它就直接开始改文件了。
所以组内约定：**提问只让它给方案，确定要改了才让它执行；做好了才 push。**（第六节第 6、7 条）

DSH 默认只能写**当前会话的工作目录**（如 `C:\Users\你的用户名\Desktop\dsh`）。clone 到别处要走一次审批，这是防止 AI 乱改你电脑上的东西。

---

## 三、第一次让 DSH 帮你体检（懒人版）

上面第一节你也可以**不手动装**，直接让 DSH 代劳。打开 DSH 新建会话，对它说：

```text
我还没装 git 和 python，帮我检查本机有没有装、版本多少，缺的告诉我该下载什么
```

它会跑 `git --version`、`python --version` 之类去查，并给出下一步。接着说：

```text
帮我检查本地 git 配置，看看有没有配 user.name / user.email，以及能不能连上 github
```

它会自己跑 `git config --list --show-origin`、测网络连通性，把结论告诉你。最后：

```text
把我的 git 用户名设成 <你的名字>，邮箱 <你的邮箱>
```

> ⚠️ 但**下载安装包这一步 DSH 做不了**（它不能替你点安装向导），所以图形化安装还是得你自己动手，见第一节。
> 推荐做法：**照着第一节手动装，装完再用 DSH 验证。**

如果体检后仍然推不上去，多半是代理问题 —— 处理办法见 [第一节第⑥步](#一开工前准备只做一次约-20-分钟)。

---

## 四、日常五件事（每件事一句话）

**1. 克隆仓库**

```text
帮我把 https://github.com/Camarovl/git-test.git 克隆到桌面
```

目标在工作目录外会弹审批。嫌麻烦就说「克隆到当前工作目录里」，全程无弹窗。

**2. 建自己的分支**

```text
在当前仓库基于 main 建一个叫 test/zhangsan 的分支并切过去
```

**3. 加东西 / 改东西**

```text
在根目录新建 hello_zhangsan.py，打印一句 hello world 和我的名字
```

⚠️ 粘给 DSH 的内容会被提交到公开仓库，别塞密钥、同学手机号、老师未公开的题目。

**4. commit + push**

```text
看一下当前改动，写一条合适的 commit message，推到远端我的分支
```

它依次执行 `git status` → `git diff` → `git add` → `git commit` → `git push -u origin <分支>`。
push 成功后终端会给出创建 Pull Request 的链接。

**5. 拉别人的更新**

```text
帮我 pull 最新代码，有冲突就列出来给我看，先别自己解决
```

最后半句很重要：**冲突怎么取舍是你和组员的事，不要让 AI 替你猜**。

---

## 五、出问题时，直接把报错丢给它

不要截图去搜，粘贴过来问：

```text
这个报错什么意思，先告诉我原因和方案，别改文件：
fatal: could not read Username for 'https://github.com': terminal prompts disabled
```

同理适用于：`non-fast-forward`、`CONFLICT (content)`、`nothing to commit, working tree clean`、
`Please tell me who you are`、`Recv failure`、`Failed to connect to github.com:443`。

---

## 六、组内约定（务必遵守）

1. **DSH 只能在你批准后才能碰工作目录以外的文件。** 审批弹窗要读完理由再点，别形成肌肉记忆一路同意。
2. **不要让 DSH 自动解决 merge 冲突。** 让它列出冲突文件和双方内容，人来决定。
3. **绝不提交**：`.env`、API key、原始数据集、模型权重、虚拟环境。让 DSH 帮你写 `.gitignore`。
4. **不要用 DSH 做这些**：`git push --force`、`git reset --hard`、删远端分支、改写已推送的历史。
   真要撤销，用 `git revert <commit>` —— 生成一条新提交，不毁别人的历史。
5. **AI 生成的提交也要署你自己的名。** 邮箱必须是 GitHub 账号里验证过的那个，否则 Contributors 看不到你，答辩讲不出自己做了啥。

### ⭐ 6. 问归问，改归改 —— 默认只让 DSH 出方案，不动代码

组内**最重要的一条**。Agent 手很快，你说一句它就能改十个文件；问一次改一次，工作区堆满半成品，仓库历史也跟着变脏。

| 你想干什么 | 该怎么说 |
|---|---|
| 咨询、看思路 | 「**先别改任何文件**，列出方案和优缺点」 |
| 真的动手 | 「按方案二执行」 |
| 先看再说 | 「读代码告诉我问题在哪，**不要修改**」 |
| 中途反悔 | 「停，撤销你刚才的改动」（未 commit 时即 `git restore .`） |

话术模板，复制着用：

```text
第一步：只分析，给方案，不改代码
第二步：我选定方案后你再执行
第三步：执行完先跑一遍验证，把结果给我看
第四步：确认没问题我才让你 commit
```

好处很直接：动手前你能把关（避免 AI 理解偏了白干半天），改动范围可控，commit 也干净。

### ⭐ 7. 做好了再上传，不要动不动就 push

`push` 是给别人看的，不是给自己备份的。半成品推上去，队友拉下来一跑就报错，浪费全组时间；
一天十几条 "临时保存 / 试试 / 改错了" 的提交，`git log` 也没法看。

正确节奏：

```text
本地随便折腾 → 想存就用 git stash 或本地 commit（不 push）
             → 功能真的跑通、测试过了
             → 整理成有意义的提交
             → 这时才 push
```

配套做法：

- **攒了一堆碎提交？** 说「把这个分支今天几个小提交整理成一条，还没 push，可以放心改写」。
  （即 rebase / `git reset --soft`，⚠️ **只能用于尚未 push 的提交**）
- **只是想防丢？** `git stash` 或本地 commit，没 push 就没人看得见。
- **必须中途推（比如换电脑）？** 提 **Draft PR**，标题写 `WIP:`，队友一眼就知道没做完。

一句话：**commit 可以频繁，push 要有分寸。**

---

## 七、推荐分工用法

| 谁 | 让 DSH 干什么 |
|---|---|
| 组员 A（组长 / 文档） | 起草 README、需求文档，整理会议纪要进仓库 |
| 组员 B（代码） | 写功能代码、跑测试、修 lint |
| 组员 E（仓库规范） | 管 `.gitignore`、PR 模板，检查有没有敏感文件混进去 |
| 遇到冲突的人 | 让 DSH 解释双方差异，再由两人线下对齐 |

协作规矩很简单：**一人一个分支，一个小目录，少碰同一个文件**，冲突自然少。

---

## 附录 A：git 四个动作（背下来就够用）

| 动作 | 大白话 | 命令 |
|---|---|---|
| **拉** | 把别人最新的改动拿到我电脑上 | `git pull` |
| **存** | 把我这次的改动存档 | `git add .` → `git commit -m "做了什么"` |
| **推** | 把我的存档传到网上给大家看 | `git push` |
| **看** | 现在有哪些改动还没存 | `git status` |

剩下都可以让 DSH 做，你只要看得懂它在敲什么。

---

## 附录 B：你的第一次任务 —— 加入分支，交一个 `hello world.py`

共 5 步，做完就算通关。**不需要你记命令**，照着把话发给 DSH 就行。

### 第 1 步 · 把仓库拿到自己电脑上

> 前提：已按 [〇 节](#〇为什么统一用-dsh) 看完视频、装好 DSH。没装先回去补。

```text
帮我 clone https://github.com/Camarovl/git-test.git，放到当前工作目录里
```

它会建出一个 `git-test` 文件夹，以后所有操作都在里面做。

### 第 2 步 · 加入分支（每人一条自己的分支）

**分支就是平行宇宙**：你在 `test/你的名字` 上怎么改都不影响别人的 `main`。
为什么不能直接改 `main`？五个人同时往主干推，一定会打起来（冲突）。

```text
在 git-test 仓库里，基于 main 新建一个叫 test/zhangsan 的分支并切换过去
```

（`zhangsan` 换成你自己的拼音缩写。）它实际执行的是：

```bash
git switch -c test/zhangsan     # -c = create，新建并立刻切过去
git branch                      # 确认带 * 号的是你的分支
```

⚠️ **分支名只能用大小写字母、数字、`-`、`_`、`/`，不要空格和中文**，否则后面各种报错。

### 第 3 步 · 写 `hello world.py`

```text
在当前分支根目录建一个 hello_zhangsan.py，内容是打印一句 hello world 和我的名字
```

```python
# hello_zhangsan.py
print("Hello, world!")
print("我是张三，这是我第一次用 git 提交的代码")
```

文件名带上自己的名字，五个人就不会撞车。顺手让它跑一遍：

```text
用 python 跑一遍这个文件，把输出给我看
```

看到 `Hello, world!` 就对了。**别提交跑不通的代码**——这是给正式项目养习惯。

### 第 4 步 · commit（存档）+ push（上传）

```text
看一下当前改动，提交这条改动，然后推到我的分支 origin/test/zhangsan
```

```bash
git status                              # 看看有哪些改动
git add hello_zhangsan.py               # 挑选要存档的文件
git commit -m "feat: zhangsan 提交 hello world"   # 存一个档
git push -u origin test/zhangsan        # 传到网上（第一次推要 -u）
```

两个新手最容易懵的点：

- **`add` 和 `commit` 是两步**。`add` 是把东西摆到桌上，`commit` 才是拍照存档。
- **`commit` 只存在你自己电脑上，`push` 之后队友才看得见。** 光 commit 不 push，等于没交作业。

### 第 5 步 · 提 Pull Request（**这次练手才需要合并，平时不要**）

push 成功后终端会吐出一行链接：

```text
remote:      https://github.com/Camarovl/git-test/pull/new/test/zhangsan
```

浏览器打开 → **Create pull request** → 标题 `test: zhangsan 首次提交` → 拉一位组员当 Reviewer → 他点 **Merge** → 完成 🎉

回本地收尾：

```text
切回 main，拉最新代码，把我那条已经合并的分支删掉
```

> ⚠️ **合并是本次练手的终点，不是日常动作。** 之所以要合，是为了让你亲眼看到"我的改动进了主干"。
> 正式项目里 `main` 是交付用的干净主干，不能想合就合 —— 见下面附录 C。

---

## 附录 C：正式开工的时候应该怎么做

练手时我们把「提 PR → 立刻 Merge」当流程终点，真实项目里规则完全不同：
**`main` 只装随时能演示的版本，合并是有门槛的。**

### 一句话原则

> **自己的分支自己推，什么时候进 `main` 由组长说了算。**
> 你平时只做一件事：把改动推到**自己的分支**上。

### 每天的实际循环（不合并）

```text
早上开工：「切到我的分支 feature/login，pull 一下 main 的最新改动同步进来」
白天干活：每做完一小块 →「看一下改动，commit 一条，推到我的分支」
晚上收工：「确认我本地没有未提交的改动」（git status 干净才算下班）
```

对应命令（认识即可）：

```bash
git switch feature/login
git pull origin main            # 同步主干进展，避免最后一次性冲突爆掉
git add <文件> && git commit -m "feat: ..."
git push                        # 只推自己的分支
```

**为什么要每天同步 main**：你和别人各憋两周不合，最后合并时冲突多到没法收拾。
小步同步 = 每次只处理一点点冲突，这是团队协作里最值钱的习惯。

### 什么时候才允许合进 main

三条全满足才提合并请求：

1. 这块功能**在自己电脑上跑通了**，不是"应该没问题"；
2. **另一位组员看过并同意**（review），不是自己点头；
3. 属于里程碑收口点 —— 通常**每周一次**：W1 需求冻结、W2 主干打通、W3 上线、W4 定版。

### 谁有权点 Merge

**只有组长 A**，其他四人只负责「提出 PR + 请人 review」。
理由很实在：各自决定往主干合，一旦某次合进跑不起来的东西，全组第二天 clone 到的就是废代码，还很难查是谁哪次合进去的。

`main` 坏了怎么办：

```bash
git log --oneline -10           # 找到可疑提交
git revert <那个 sha>            # 生成一条撤销提交，不改历史
git push
```

**不要用 `git push --force` 修 main** —— 会把别人的提交一起抹掉，这是全组最容易翻车的操作。

### 分支命名建议

| 场景 | 分支名 | 例子 |
|---|---|---|
| 练手 / 试工具 | `test/<缩写>` | `test/zhangsan` |
| 做某个功能模块 | `feature/<名字>` | `feature/login` |
| 修 bug | `fix/<名字>` | `fix/csv-encoding` |
| 写文档 | `docs/<名字>` | `docs/readme-cn` |

一人可以同时有若干条分支，但**同一时间只在一条上干活**，切之前先确认 `git status` 干净或已 `git stash`。

### PR 描述模板（照着填，别留空）

```markdown
## 改了什么
- 新增 app.py 的登录页路由

## 怎么验证
- 本地 `streamlit run app.py`，用 demo 账号能登进去

## 影响范围
- 只动了 pages/login.py，没碰数据处理模块

## 遗留问题
- 密码还是明文，W2 换成哈希
```

### 正式项目的节奏对照表

| 时间 | 每个人做什么 | 是否合并 main |
|---|---|---|
| 每天 | 在自己的 feature 分支上 commit + push，同步一次 main | ❌ |
| 每周结束 | 功能跑通 + 找人 review → 提 PR | ✅ 组长 A 审完合并 |
| W4 答辩前 | 冻结代码，只允许 `fix/` 分支进 main | ✅ 定版合并 |

> ⚠️ 「每天 commit + push」指**当天有可交付成果时推一次**，不是每改十行推一把。
> 判断标准：这条提交别人拉下来能不能正常跑？不能就别推（第六节第 7 条）。

> 记住差别：**练手的目标是"合进去看看"，正式项目的目标是"main 永远是好的"。**

---

## 附录 D：任务提交要求（组长用）

每位组员交两样东西，缺一项视为未完成练手：

1. **DSH 入门视频看完**：<https://www.bilibili.com/video/BV1REx46NEc4/>
2. **`hello_<你的名字>.py` 已合并进 `main`** —— GitHub 上 Contributors 能看到你头像。

### 检查清单（交作业前对一遍）

- [ ] DSH Desktop 装好，能新建会话并跑通一次对话
- [ ] 我的分支叫 `test/<我的拼音缩写>`，不是直接改 `main`
- [ ] 文件名带了自己的名字，如 `hello_zhangsan.py`
- [ ] 代码在我电脑上真的跑通过
- [ ] `git status` 里没有多余文件（尤其别把 `.env`、数据集、`__pycache__/` 混进去）
- [ ] commit message 说得清这次干了什么，不是 `111` / `update`
- [ ] PR 已发出并且有人 review 合并（**仅本次练手，正式项目见附录 C**）
- [ ] 合并后本地 `git pull` 过，旧分支已删

### 日常习惯自查（第六节第 6、7 条）

- [ ] 问问题时说了「先别改文件，只列方案」
- [ ] 只有确认要动手了才让 agent 执行
- [ ] 碎改动留在本地 commit / `git stash`，没往远端推
- [ ] push 之前功能真的跑通过
- [ ] 没在 `git log` 里留下 "试试"、"临时保存" 这种提交

### 卡住了怎么办

| 现象 | 处理 |
|---|---|
| push 提示要用户名密码 | GitHub 不收密码。说「push 认证失败了，帮我配 Personal Access Token」 |
| `failed to push some refs` | 别人刚合了东西。先说「帮我 pull 一下再重新推」 |
| 冒出 `<<<<<<<` 怪符号 | 这就是冲突。说「列出冲突文件和双方内容，我自己决定」，**别让 AI 替你选** |
| 提交了不该提交的文件 | 说「把这个文件从上一次提交里撤掉，用安全的方式」（即 `git revert`） |
| `Failed to connect to github.com:443` | 多为 DNS 解析到不可达节点或代理没开，重试几次即可，不是你操作错了 |
| 完全看不懂它在说什么 | 整段红字原样粘给 DSH，问「这是什么意思，该怎么办」—— **是问方案，不是让它直接改** |

> 通关标准：**GitHub 上能看到你那个 `hello_zhangsan.py` 被合并进了 `main`**。
> 但请记住这次是你自己点 Merge 练手；正式开工后往 `main` 合并只有组长 A 能做，
> 平时你只把改动推到自己的分支上 —— 完整规则见附录 C。
