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

- **不要**在黑名单文件里做字母排序（项目历史有意保留时间倒序）
- **不要**修改邮件原文以"美化"语法或拼写（DMCA-Records.md 也提到 preserve 原文措辞）
- **不要**在 `DMCA-Records.md` 表里写多份邮件合并行——一行 = 一封投诉
- **不要**为单条 infohash 跨条目拆表——一个邮件即使有多个 hash，仍视为一行

## 提交风格

- `git commit` 信息遵循项目既有惯例（参考 `git log` 已有的中文 commit message）
- 一次 DMCA 受理建议一次提交，便于回溯

## DMCA 回复邮件模板

向投诉方发送确认邮件时，**沿用下列模板**（仅替换 `[]` 占位符）。首次采用用户给 WARX 团队的历史回复作为基准，并由用户迭代加入以下要素：加粗关键词、`- ` 列表化 hash、页脚说明 + GitHub 仓库链接。

> Dear [对方公司名] Team,
>
> Thank you for bringing this matter to our attention. We take intellectual property rights seriously and have acted promptly to comply with your DMCA takedown request regarding the alleged infringing material titled **"[作品标题]"**.
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

- 称呼使用 `[对方公司] Team,`（沿用项目历史回复样本）
- 联系方式邮箱用 `[i@renfei.net]`（方括号包裹，沿用历史样本）
- **加粗元素**（markdown-aware 邮件客户端会渲染为粗体）：
  - 作品标题（仅加粗两侧引号包裹的名字，`titled` 不加粗）— `titled **"[作品标题]"**`
  - `successfully blacklisted` 一词
  - 每条 hash 列表项 — `- **[INFOHASH]**`
- **hash 列表**：使用 markdown ` - `（半角短横 + 空格），多个 hash 各占一行
- **GitHub 仓库**段：在 hash 列表之后、`This action was completed...` 之前，固定使用以下措辞：
  `For transparency, the public DMCA repository is available at <https://github.com/renfei/DMCA>.`
- **页脚**：在 `Best regards, RenFei` 之后追加两行：
  ```
  Owner & Administrator, RENFEI.NET
  Designated DMCA contact: i@renfei.net
  ```
- 末尾署名 `RenFei` 后已由页脚覆盖组织名 / 联系方式，不必重复
- 不附 tracker 工作原理等技术性解释
- 对存在疑问的细节（权属、同一 hash 已在黑名单、缺要素等）通过 `[ ]` 留白，等用户决策后再发

---

最后修订：2026-09-06
