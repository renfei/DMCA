# AGENTS.md - RENFEI.NET DMCA 项目规范

项目根目录：`/Users/renfei/WorkSpace/DMCA/`
目的：归档 RENFEI.NET 收到的 DMCA 投诉，并维护 BitTorrent 追踪器的黑名单。
更多项目背景见 `README.md`。

## 接收到 DMCA 投诉后的标准流程

DMCA 投诉应同时落到三处，缺一不可：

1. **新建 `records/<date>-dmca-NNN.md`**
   - 日期格式 `YYYY-MM-DD`，从邮件落地的日期取；当日序号从 001 起
   - 文件内容为**邮件原文**（保留原文措辞、拼写、格式），不作润色
   - 一事一议，一次投诉一个文件；同一天多次投诉按 `-001 / -002 / -003` 递增
2. **追加 `BitTorrent-Tracker-Blacklisted-InfoHash.md`**
   - 新 hash 追加在代码块**末尾**（项目历史采用"时间倒序、新条目在最下"）
   - 保持 16 进制大写 / base32 大写字母数字 的统一形式，不在文件中混排其它状态
   - 同一 hash 已在列表中则**跳过追加**（避免重复），只在 DMCA-Records.md 里记状态为 `Already Blocked`
3. **更新 `DMCA-Records.md`**
   - 新行插入表格**顶部**（最新的在最上）
   - 列：Date Received / Sender / Type / Infringing URL or Info Hash / Status / Notice Record
   - Status 取值见下方"状态字典"
   - 同步把文档末尾 `Last updated:` 日期更新到当日

### 状态字典

| Status             | 触发条件                                                       |
|--------------------|----------------------------------------------------------------|
| `Blocked`          | 新 hash，已加入黑名单                                           |
| `Already Blocked`  | hash 已在黑名单，只做归档记录                                   |
| `Removed`          | 实体内容（非 tracker）已下线                                    |
| `Invalid`          | 邮件缺 info hash 等关键要素，无法处理                           |
| `Verification Needed` | 仓库层面无法确认，需要查实际运行环境                          |
| `Counter-Notice Received` | 收到合格反通知后恢复                                          |

## `records/` 文件的排版约定

为了让 GitHub 渲染出该有的版面，新建的 `records/*.md` 必须遵守：

- **结构化表格（多列表）**：使用真正的 GitHub Flavored Markdown 表格
  ```markdown
  | Title Name | Tracker | Info Hash |
  | --- | --- | --- |
  | ... | ... | ... |
  ```
  ❌ 不要使用 tab 分隔的"假表格"（会被 GitHub 当作纯文本挤成一行）
- **多行平铺段（如签名块、版权方块、署名块）**：每一行末尾追加 **两个空格**，GitHub 会渲染为 `<br>`
  ```markdown
  DIGITAL SIGNATURE FOR COPYRIGHT CLAIM:  
  Name  
  Company  
  ```
- **段落之间空一行**（标准 Markdown 段落分隔）
- 保留邮件原文措辞，但**不要复制不存在的 trailing 空格**——按本约定显式加

如果偏离了以上格式（如本约定演化、原文邮件特殊），在当日工作日志 `.workbuddy/memory/YYYY-MM-DD.md` 里记一笔说明。

## 不该做的事

- **不要**把新数据行插到 `DMCA-Records.md` 的**分隔行之前**。GFM 表格要求「表头 → 分隔行 → 数据行」，
  顺序错了整个表格会退化成纯文本（表头和内容混在一起）。
  正确做法：先定位分隔行下标 `sep = next(i for i, l in enumerate(lines) if set(l.replace('|','').replace(' ','')) == {'-'})`，
  再 `lines.insert(sep + 1, row)`；**不要**额外 new 一行 `|---|`，否则表头下会出现两行分隔行。改完必须用 `git diff` 复核。
- **不要**假设 `BitTorrent-Tracker-Blacklisted-InfoHash.md` 结尾没有换行：文件末尾是 ```` ``` ```` **加一个换行**，
  追加 hash 时锚定字符串要带上这个尾部换行，否则 assert 失败。

- **不要**在黑名单文件里做字母排序（项目历史有意保留时间倒序）
- **不要**让 `DMCA-Records.md` 表格的行保持各自不同的 padding：列宽必须全表统一，
  否则 monospace 视图里 `Already Blocked` / `Verification Needed` 那几行会整行错位。
  新增/修改行后，把每列宽度取全表最大值重排一次所有行（内容保持不变），并用 GFM 解析器验一遍行数。
- **不要**修改邮件原文以"美化"语法或拼写（DMCA-Records.md 也提到 preserve 原文措辞）
- **不要**在 `DMCA-Records.md` 表里写多份邮件合并行——一行 = 一封投诉
- **不要**为单条 infohash 跨条目拆表——一个邮件即使有多个 hash，仍视为一行

## 提交风格

- `git commit` 信息遵循项目既有惯例（参考 `git log` 已有的中文 commit message）
- 一次 DMCA 受理建议一次提交，便于回溯

## DMCA 回复邮件模板

向投诉方发送确认邮件时，**沿用下列模板**（仅替换 `[]` 占位符）。这是**带 markdown 标记的富文本邮件**：加粗与 `- ` 列表是刻意加的，markdown-aware 客户端会渲染成粗体与项目符号。

**注意**：不要用「从已发邮件里复制出来的纯文本」反推格式——复制过程会丢掉加粗和列表符号，看起来像裸文本。格式以本节为准。

> Dear [对方公司名] Team,
>
> Thank you for bringing this matter to our attention. We take intellectual property rights seriously and have acted promptly to comply with your DMCA takedown request regarding the alleged infringing material titled **“[作品标题]”**.
>
> Upon reviewing your notice, we have **successfully blacklisted** the following Infohash(es) on our BT Tracker server to prevent further dissemination of the reported content:
>
> - **[INFOHASH_1]**
> - **[INFOHASH_2]**
> - ...
>
> For transparency, the public DMCA repository is available at <https://github.com/renfei/DMCA>.
>
> This action was completed in accordance with our internal policies and the requirements of the DMCA. Please let us know if further steps are required from our side or if additional information is needed to resolve this matter.
>
> We remain committed to cooperating with copyright holders and authorized enforcement agencies to address infringement concerns. For future reports, feel free to contact us directly at [i@renfei.net] for expedited processing.
>
> Thank you for your understanding.
>
> Best regards,
> RenFei
> Owner & Administrator, RENFEI.NET
> Designated DMCA contact: i@renfei.net

写作要点：

- 称呼 `[对方公司] Team,`
- 作品标题：弯引号 `“”` 包裹，**整体加粗**（含引号），`titled` 不加粗、后面不加冒号
- **加粗元素**：作品标题、`successfully blacklisted`、每条 hash 列表项
- **hash 列表**：markdown `- ` 列表，每行一条（`- **[INFOHASH]**`）
- **GitHub 仓库**段固定在 hash 列表之后、`This action was completed...` 之前，URL 用 `<>` 包成自动链接
- 联系方式邮箱保留方括号 `[i@renfei.net]`
- 页脚两行：`Owner & Administrator, RENFEI.NET` / `Designated DMCA contact: i@renfei.net`
- 正文即全部，**不写邮件头表格、不引用原投诉全文**
- 不附 tracker 工作原理等技术性解释
- 对存在疑问的细节（权属、缺要素等）通过 `[ ]` 留白，等用户决策后再发

### 重复投诉变体（同一 Info Hash 已在黑名单）

**触发条件**：`DMCA-Records.md` 里 Status 记为 `Already Blocked`（hash 此前已受理过）。此时**不能**再用基础模板的
「have **successfully blacklisted**」——那是新拉黑的表述，套在重复投诉上说不通。改用下列变体，如实告知对方这是重复的。

> Dear [对方公司名] Team,
>
> Thank you for your notice dated [本次投诉日期] regarding the alleged infringing material titled **“[作品标题]”**.
>
> This is a **duplicate report**. The Infohash(es) listed below were **already blacklisted** on our BT Tracker server under an earlier notice received on [原投诉日期], and they remain blocked to this day. Our tracker enforces the blacklist automatically, so these entries cannot be announced or served to peers.
>
> Summary of the earlier submission:
>
> - Date received: [原投诉日期]
> - Sender: [原投诉方邮箱]
> - Notice ID: [原 Notice ID]
> - Info Hash: **[INFOHASH_1]**
> - Status: Already Blacklisted
> - Record: <https://github.com/renfei/DMCA/blob/master/records/[YYYY-MM-DD]-dmca-[00N].md>
>
> For transparency, the public DMCA repository is available at <https://github.com/renfei/DMCA>, and every notice we have processed is listed in `DMCA-Records.md`.
>
> Each notice we receive has to be reviewed, logged and answered by hand. Resending a report for an Info Hash that is already blacklisted therefore creates **substantial duplicated manual work** on our side without changing the outcome. We kindly ask that you **verify against the repository above before submitting**, and refrain from sending further notices for content that already appears there. Please escalate to us only Infohash(es) that are not yet listed.
>
> We remain committed to cooperating with copyright holders and authorized enforcement agencies to address infringement concerns. For future reports, feel free to contact us directly at [i@renfei.net] for expedited processing.
>
> Thank you for your understanding.
>
> Best regards,
> RenFei
> Owner & Administrator, RENFEI.NET
> Designated DMCA contact: i@renfei.net

变体要点：

- 去重结论放前面：第二句就点明 **duplicate report**，不藏在中间
- **加粗元素**：`duplicate report`、`already blacklisted`、摘要里的 hash、`substantial duplicated manual work`、`verify against the repository above before submitting`
- **摘要用 `- ` 列表而非表格**：要写明 Date received / Sender / Notice ID / Info Hash / Status / Record 六项；
  邮件客户端对 GFM 表格支持不一致，列表是更稳的降级选择
- 日期统一写成 `September 19, 2026` 这种「月份英文全拼 + 日 + 年」形式；多值时用 `- Earlier notice: ...` 列出每次的时间
- Record 一行用 `<>` 自动链接指向 GitHub 上的原始记录文件；同 hash 多次时应列**最早**那次
- 删掉基础模板里的 `This action was completed in accordance with...` 一句——本次没有新动作可陈述
- 措辞直给但不失礼：先陈述内容早已处于屏蔽状态，再说明重复投诉的实际代价（人工复核开销）
- 同一封里若混有新 hash 与已存在 hash，拆成两段陈述：`newly blacklisted` 一组、`already blacklisted` 一组，摘要只对后者

---

最后修订：2026-10-21
