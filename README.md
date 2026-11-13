# 用 DeepSeek Harness（DSH）做 Git：组员上手指南

这份文档不是教你背 git 命令，而是教你**怎么用 DSH 把 git 的活干完**。
你只需要在对话框里说人话，DSH 会替你执行命令、读报错、改文件；你负责**看懂它在干什么**和**批准它做危险操作**。

- 本机版本：DSH Desktop **2.0.17**（开始菜单搜 "DSH Desktop" 启动，浏览器打开 `http://127.0.0.1:43120`）
- 练手仓库：<https://github.com/Camarovl/git-test>
- 想先补基础概念，看文末「附录 A：git 四个动作」即可，不用通读。

---

## 一、核心心智模型

| 角色 | 干什么 |
|---|---|
| **你** | 说清楚要什么结果 + 审批 DSH 的请求 + 检查结果 |
| **DSH** | 跑 `git` 命令、读输出、编辑文件、诊断报错 |
| **审批弹窗** | 凡是「写到工作目录之外」「联网」「需要更高权限」的操作都会问你，看清楚理由再点 |

DSH 的文件写入默认被限制在**当前会话的工作目录**里（比如 `C:\Users\你的用户名\Desktop\dsh`）。
所以要 clone 到别的地方，得走一次审批——这不是麻烦，是防止 AI 乱改你电脑上的东西。

---

## 二、第一次配置（只做一次）

打开 DSH，新建一个会话，直接对它说：

```text
帮我检查本地 git 配置，看看有没有配 user.name / user.email，以及能不能连上 github
```

它会自己跑 `git config --list --show-origin`、测网络、报告结论。然后说：

```text
把我的 git 用户名设成 <你的名字>，邮箱 <你的邮箱>
```

### ⚠️ 校园网 / 代理必看

本项目实测过的坑：**浏览器能打开 GitHub ≠ 命令行能 push**。浏览器走代理，git 默认不走。

让 DSH 帮你查和修：

```text
我的 git 推不上去，怀疑是代理问题。帮我查系统代理设置、git 的 http.proxy 配置，
然后按我代理软件的实际端口给 git 配上代理
```

它给出的修法一般是这两种之一：

```bash
git config --global http.proxy  http://127.0.0.1:<你的端口>   # Clash 常见 7890，v2rayN 常见 10809
git config --global https.proxy http://127.0.0.1:<你的端口>
# 或者：打开代理软件的 TUN / 系统代理模式，让所有程序都走
```

> 代理软件必须**常驻运行**才能 push。答辩前最后一晚发现某人的代理没开，是最常见的事故。

---

## 三、日常五件事（每件事一句话）

### 1. 克隆仓库

```text
帮我把 https://github.com/Camarovl/git-test.git 克隆到桌面
```

DSH 会在沙箱里试，若目标在工作目录外会弹审批；批准后它就 clone 好了。
嫌审批麻烦就说「克隆到当前工作目录里」，全程无弹窗。

### 2. 建自己的分支

```text
在当前仓库基于 main 建一个叫 test/zhangsan 的分支并切过去
```

它会跑 `git switch -c test/zhangsan`，并把结果告诉你。

### 3. 加东西 / 改东西

```text
在 members/ 下新建 zhangsan.md，写上姓名、学号、我今天做了什么
```

或者更省事：把你从微信/QQ 收到的文字直接粘给它，说「整理成 markdown 放进仓库」。
**注意**：粘给 DSH 的内容会被提交到公开仓库，别往里塞密钥、同学手机号、老师未公开的题目。

### 4. commit + push

```text
看一下当前改动，帮我写一条合适的 commit message，然后推到远端我的分支
```

它会依次执行 `git status` → `git diff` → `git add` → `git commit -m "..."` → `git push -u origin <分支>`。
push 涉及联网凭据，通常会有一次审批。

推送成功后终端里会有创建 Pull Request 的链接，复制去网页点 Merge 就行。

### 5. 拉别人的更新

```text
帮我 pull 最新代码，有冲突就列出来给我看，先别自己解决
```

最后半句很重要：**冲突怎么取舍是你和组员的事，不要让 AI 替你猜**。

---

## 四、出问题时，直接把报错丢给它

这是 DSH 最省时间的用法。不要自己截图去搜，粘贴过来问：

```text
这个报错什么意思，帮我修好：
fatal: could not read Username for 'https://github.com': terminal prompts disabled
```

它会判断成因（这里是 GitHub 不接受账号密码，要用 Personal Access Token 或 SSH key），
给出方案，并在你同意后执行。

同理适用于：`non-fast-forward`、`CONFLICT (content)`、`nothing to commit, working tree clean`、
`Please tell me who you are`、`Recv failure`。

---

## 五、安全边界（组内约定，务必遵守）

1. **DSH 只能在你批准后才能碰工作目录以外的文件。** 每次审批弹窗都要读完那句理由再点，别形成肌肉记忆一路点同意。
2. **不要让 DSH 自动解决 merge 冲突。** 让它列出冲突文件和双方内容，人来决定。
3. **绝不提交**：`.env`、API key、数据集原始 csv、模型权重、虚拟环境。让 DSH 帮你写 `.gitignore`：
   ```text
   这是个 Python + Streamlit 项目，帮我写一份 .gitignore，把密钥、缓存、数据、模型都排除掉
   ```
4. **不要用 DSH 做这些**：`git push --force`、`git reset --hard`、删远端分支、改写已推送的历史。
   真要撤销，让它用 `git revert <commit>` —— 生成一条新提交，不会毁掉别人的历史。
5. **AI 生成的提交也要署你自己的名。** 提交身份用第 2 节配好的 `user.name/email`，
   且邮箱必须是 GitHub 账号里验证过的那个，否则 Contributors 页面上看不到你，答辩讲不出自己做了啥。

---

## 六、推荐分工用法

| 谁 | 让 DSH 干什么 |
|---|---|
| 组员 A（文档） | 起草 README / 需求文档，整理会议纪要进仓库 |
| 组员 B（代码） | 写功能代码、跑测试、修 lint |
| 组员 E（仓库规范） | 管 `.gitignore`、PR 模板、检查有没有敏感文件混进去 |
| 遇到冲突的人 | 让 DSH 解释冲突双方的差异，再由两人线下对齐 |

多人多机协作时，规矩很简单：**一人一个分支，一个小目录，少碰同一个文件**，冲突自然少。

---

## 附录 A：git 四个动作（背下来就够用）

| 动作 | 大白话 | 命令 |
|---|---|---|
| **拉** | 把别人最新的改动拿到我电脑上 | `git pull` |
| **存** | 把我这次的改动存档 | `git add .` → `git commit -m "做了什么"` |
| **推** | 把我的存档传到网上给大家看 | `git push` |
| **看** | 现在有哪些改动还没存 | `git status` |

剩下都可以让 DSH 做，你只要看得懂它在敲什么。

## 附录 B：完整流程示例（从零到 PR）

```text
# 对 DSH 依次说这四句话
1) 把工作目录设为 C:\Users\<你>\Desktop\dsh，clone https://github.com/Camarovl/git-test.git
2) 建分支 test/zhangsan 并切换过去
3) 在 members/zhangsan.md 写下我的姓名、学号和今天完成的练习
4) 提交并推送到远端，把创建 PR 的链接给我
```

对应实际执行的命令（可以对照着看，理解 DSH 在做什么）：

```bash
git clone https://github.com/Camarovl/git-test.git && cd git-test
git switch -c test/zhangsan
# ... 新建 members/zhangsan.md ...
git add members/zhangsan.md
git commit -m "docs: zhangsan 完成 git 练手"
git push -u origin test/zhangsan
```

合并后收尾：

```bash
git switch main && git pull && git branch -d test/zhangsan
```

---

## 附录 C：你的第一次任务 —— 加进分支，交一个 `hello world.py`

整个任务只有 5 步，做完就算通关。**全程不需要你记命令**，照着把话发给 DSH 就行。

### 第 1 步 · 把仓库拿到自己电脑上

对 DSH 说：

```text
帮我 clone https://github.com/Camarovl/git-test.git，放到当前工作目录里
```

它会建出一个 `git-test` 文件夹。以后所有操作都在这个文件夹里做。

### 第 2 步 · 加入分支（每人一条自己的分支）

先搞懂一句话：**分支就是平行宇宙**。你在 `test/你的名字` 上怎么改，都不会影响别人的 `main`。
为什么不能直接在 `main` 上改？因为五个人同时往 `main` 推东西，一定会打起来（冲突）。

对 DSH 说：

```text
在 git-test 仓库里，基于 main 新建一个叫 test/zhangsan 的分支并切换过去
```

（把 `zhangsan` 换成你自己的拼音缩写，比如 `lisi`、`wangwu`。）

它实际执行的是这两条，扫一眼认识一下就好：

```bash
git switch -c test/zhangsan     # -c = create，新建并立刻切过去
git branch                      # 确认前面有 * 号的是你的分支
```

⚠️ **分支名只能有小写字母、数字、`-`、`_`、`/`，不要有空格和中文**，否则后面各种报错。

### 第 3 步 · 写 `hello world.py`

对 DSH 说：

```text
在当前分支根目录建一个 hello_zhangsan.py，内容是打印一句 hello world 和我的名字
```

文件长这样（文件名带上自己的名字，五个人就不会撞车）：

```python
# hello_zhangsan.py
print("Hello, world!")
print("我是张三，这是我第一次用 git 提交的代码")
```

顺手让它跑一下确认能运行：

```text
用 python 跑一遍这个文件，把输出给我看
```

看到 `Hello, world!` 就对了。**别提交跑不通的代码**——这是给正式项目养习惯。

### 第 4 步 · commit（存档）+ push（上传）

对 DSH 说：

```text
看一下当前改动，提交这条改动，然后推到我的分支 origin/test/zhangsan
```

它按顺序干这四件事：

```bash
git status                              # 看看有哪些改动
git add hello_zhangsan.py               # 挑选要存档的文件
git commit -m "feat: zhangsan 提交 hello world"   # 存一个档
git push -u origin test/zhangsan        # 传到网上（第一次推要 -u）
```

两个新手最容易懵的点：

- **`add` 和 `commit` 是两步**。`add` 是把东西摆到桌上，`commit` 才是拍照存档。没 `add` 的文件不会进 commit。
- **`commit` 只存在你自己电脑上，`push` 之后队友才看得见。** 光 commit 不 push，等于没交作业。

### 第 5 步 · 提 Pull Request，让组员帮你合并

push 成功后，终端会吐出一行类似这样的链接：

```text
remote:      https://github.com/Camarovl/git-test/pull/new/test/zhangsan
```

浏览器打开它 → 点 **Create pull request** → 标题写 `test: zhangsan 首次提交` → 拉一位组员进来当 Reviewer → 他点 **Merge pull request** → 完成 🎉

回到本地收尾，对 DSH 说：

```text
切回 main，拉最新代码，把我那条已经合并的分支删掉
```

```bash
git switch main
git pull                    # 现在你的 hello world 已经在 main 里了
git branch -d test/zhangsan
```

---

### 检查清单（交作业前对一遍）

- [ ] 我的分支叫 `test/<我的拼音缩写>`，不是直接改 `main`
- [ ] 文件名带了自己的名字，如 `hello_zhangsan.py`
- [ ] 代码在我电脑上真的跑通过
- [ ] `git status` 里没有多余文件（尤其别把 `.env`、数据集、`__pycache__/` 混进去）
- [ ] commit message 说得清这次干了什么，不是 `111` / `update`
- [ ] PR 已发出并且有人 review 合并
- [ ] 合并后本地 `git pull` 过，旧分支已删

### 卡住了怎么办

| 现象 | 处理 |
|---|---|
| push 提示要用户名密码 | GitHub 不收密码。对 DSH 说「push 认证失败了，帮我配 Personal Access Token」 |
| `failed to push some refs` | 别人刚合了东西。先说「帮我 pull 一下再重新推」 |
| 冒出 `<<<<<<<` 这种怪符号 | 这就是冲突。说「列出冲突文件和双方内容，我自己决定怎么改」，**别让 AI 替你选** |
| 提交了不该提交的文件 | 说「帮我把这个文件从上一次提交里撤掉，用安全的方式」（即 `git revert`） |
| 完全看不懂它在说什么 | 把终端整段红字原样粘给 DSH，问「这是什么意思，我该怎么办」 |

> 通关标准很简单：**GitHub 上能看到你那个 `hello_zhangsan.py` 被合并进了 `main`**。
> 做到这一步，你就已经具备做正式项目仓库的能力了。
