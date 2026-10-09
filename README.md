# WeChat-Mini-Program-NH-Passenger-Vehicle-Sales-Data-Collector-nh-
这是一个支持win端和安卓手机端通过微信小程序（#小程序://乘用车销量/7HQVrFbWXL5mREf）购买会员（58元可随意搜索平台规定可以截图分享数据）可以自动化采集城市-年月-车型层面汽车销量的方法分享，使用模拟点击的方式更适合放在虚拟机中运行，可以无人监管，自驱执行对这个小程序数据爬取。
[README.md](https://github.com/user-attachments/files/33238182/README.md)
# NH 微信销量采集使用说明

采集代码版本：NH-Unified-20261006-R1。说明修订：2026-10-06。

本包包含电脑版三种采集入口、Python 和 OCR 环境、已保存的销量数据库、任务断点，以及手机版代码和截图资料。先完整解压，再运行 CMD；不要直接在压缩软件里双击。

## 1. 建议在专用虚拟机中运行

**纯 GUI 采集会控制鼠标、滚动页面，并把小程序保持在前台。采集期间，这台电脑不适合同时用来办公、聊天或操作其他窗口。** 鼠标被移动、页面被切走或窗口被遮挡，都可能让采集暂停。

建议给采集器单独准备一台 Windows 虚拟机：在虚拟机里运行微信和采集器，宿主机继续正常使用。也可以用一台闲置电脑。

虚拟机内需要保持微信登录，桌面不能锁屏、休眠或黑屏断开。使用虚拟机控制台，运行后不要在虚拟机里抢鼠标或切换页面。宿主机也不要进入休眠。混合模式会切换到 GUI，按同样要求准备；纯 API 的认证恢复也可能操作微信窗口。

### 当前分辨率要求

当前 GUI 校准环境是 **1600×1200、单显示器、Windows 缩放100%**，小程序窗口固定为 **575×1154**，标题为 `NH乘用车销量库`。

## 2. 第一次安装

1. 解压完整 ZIP，例如放到 `D:\NH-Unified-20261006`。
2. 双击 **00-Set-Root.cmd**，选择 `C:\NH-Crawler`、`D:\NH-Crawler`、`E:\NH-Crawler` 或其他绝对目录。默认是 `C:\NH-Crawler`。
3. 双击 **01-Install.cmd**。安装器校验文件后，把代码、Python、数据库和证据复制到选定目录，并调整运行路径。目标目录必须为空，不使用目录联接。
4. 双击 **02-Check-Install-Environment.cmd**。缺少 VC++ x64 时会从微软下载，验证签名后请求管理员权限安装；如提示重启，先重启 Windows。
5. 在微信中打开 `NH乘用车销量库`，按上面的分辨率要求设置，再选择启动模式。

Python 3.12、OCR、OpenCV、ONNX Runtime 和窗口操作依赖已经随包提供。VC++ 安装器需要联网下载，不是离线内置。其他依赖损坏时，环境检查会列出失败项，不自动更换库版本。**03-Repair-VCpp.cmd** 用于重新安装或修复 VC++。

安装后的入口在 `<根目录>\UNIFIED\`，以后从这里启动即可。若目标目录已有数据，先备份并选择新的空目录；安装中断留下的半成品目录也应另行处理。

也可以从解压目录执行：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\Unified.ps1 -Action install -Root 'E:\NH-Crawler'
```

这条命令只安装，不启动采集。历史证据里的旧盘符保留原样，查看原图时按 evidence 相对路径到新目录寻找。

## 3. 选择采集模式

### 10-Start-Pure-GUI.cmd：纯 GUI

通过小程序界面模拟点击、读取窗口、截图，再在电脑上 OCR 并写入原队列和数据库。截图来自目标窗口，不是下载销量图片。这个入口不启动 API 销量采集器。

保留原有模拟输入参数、窗口锁、守护和遮挡恢复。每次查询收尾随机等待10–20秒。OCR失败时先复核已有截图；本地读取 API 断点文件不等于发送 API 请求。

### 11-Start-GUI-API.cmd：API 与 GUI 接续

先运行 API。认证恢复尝试耗尽且满足交接条件后，API退出，再启动 GUI。GUI运行期间只读本地 token 候选；发现候选后先停 GUI，再交给 API 请求验证。

两种方式顺序运行，不并发。它们使用同一电脑数据库和输出目录。新合包的完整接续流程还需要实际微信环境验收；现有等待、HOLD或认证失败会保留，并在状态和日志中显示。

### 12-Start-Pure-API.cmd：纯 API

只运行 API 销量采集器，认证失败后不切换到 GUI 销量采集。保留自动寻找本地 token 候选，以及微信 H5 刷新、重开的认证恢复流程。

健康请求随机等待12–20秒；异常退避可延长至120秒，长冷却另行保存。首次没有初始化等待状态时，原 B05 逻辑可能先等100分钟。重启保留剩余等待。

默认查询窗口为24个月：全国结果不足35行时，一次可以包含多个城市和月份；达到35行后拆省，再拆城市；城市仍达到35行则二分月份。单城单月仍达到上限时，记录为 `UNFINISHED`。

当前 API 不采用35个月窗口。35是返回行数上限，不能直接当作35个月；任务结束按范围完整性判断，不按是否出现2026年判断。原车型名到 API ID 的映射保留，同名车型的厂商归属需要结合目录核对。

同一项目已有采集进程时，再点启动只显示状态。发现其他目录的采集器，或无法确认进程身份时，启动会停止。换模式前先使用停止入口，避免同时启动多个采集器。

## 4. 休息设置

GUI默认按累计实际采集运行时间计算：

- 每运行1小时，随机休息10–20分钟。
- 累计运行4小时，随机休息45–101分钟，优先执行长休息，不再叠加短休息。
- 休息时间不计入累计运行时间。

| 入口 | 用途 |
|---|---|
| **30-Long-Rest-OFF.cmd** | 关闭后续4小时计划长休息 |
| **31-Long-Rest-ON.cmd** | 恢复4小时计划长休息 |
| **32-Short-Rest-OFF.cmd** | 关闭后续每小时计划短休息 |
| **33-Short-Rest-ON.cmd** | 恢复每小时计划短休息 |

设置保存在 `<根目录>\GUI-ONLY\duty-cycle.json` 的 `scheduled_long_rest_enabled`、`scheduled_short_rest_enabled`，下一轮运行读取，不需要反复重启。正在进行的休息不会因关闭开关提前结束；重新开启也不清零累计时间。

当前配置没有“每40分钟自动冷却”这一项，旧资料中的40分钟可能指监控频率。

这些开关只控制计划休息。每次查询等待、API退避、服务端指定等待、权限或额度不足、STOP/HOLD仍按原逻辑处理。不要用旧版 `GUI-DUTY-OFF` 替代这两个开关。

## 5. 数据和断点

主数据库：

```text
<根目录>\nh-sales-v3\output\adaptive.sqlite3
```

| 表或目录 | 内容 |
|---|---|
| `sales` | 已采集销量，包含厂商、品牌、车型、类别、年月、城市/省份等原字段 |
| `tasks` | 查询范围、父子任务、状态、完成类型和分片跳过记录，含已确认空结果 |
| `attempts` | 尝试、租约和提交记录，用来处理中断后是否可以重查 |
| `evidence`、`row_sources` | 截图证据索引和数据来源 |
| `output\v3\` | 运行状态、心跳和停止原因 |
| `output\evidence2\` | 截图证据 |
| `GUI-ONLY\dutycycle\`、kit的`runtime\` | 休息、HOLD、门禁和互斥状态 |
| `output\api-collect\done.txt` | 当前 API 运行生成的完成断点，同目录还有范围、未完成任务和认证日志 |

本包数据库快照有 **97,241条销量、50,324条任务、12,238条尝试记录**。其中2,576个 `verified_empty_scope*` 任务记录已确认空范围，销量表不补零。这是本机保存的快照，不是所有虚拟机截至今天的汇总。

分片为 **P1 / [0,504]**，顺序 `p1_cut_forward_circular_p2_p3_p4_p5`。原 GUI 时间范围和排除2026的配置保留。`partition_skipped` 表示由其他分片负责，不是本机漏采。

API旧断点在 `snapshots\api-unbound\` 和 `archives\API-checkpoints-unbound.zip`，尚未绑定当前机器，因此没有自动启用。导入前要核对机器、车型/厂商、月份、城市范围及数据库；不同分片的 `done.txt` 不要直接合并。

数据库物理完整性检查正常，历史外键问题仍保留。旧版本可能存在误空、误停或未完成记录，合包本身没有清洗这些断点。

全库 SQL 位于 `sql\adaptive-full.sql.gz`，包含所有表。`DATA-SNAPSHOT.json`记录计数和哈希；`SOURCES.json`、`SUPPLEMENTS.json`、`EXTRA-HISTORICAL-FILES.json`记录资料来源。缺失的历史截图和目录资料来自旧迁移包，未覆盖较新的数据库或状态。

## 6. 查看进度、停止和恢复

- **20-Status.cmd**：查看数据库计数、停止和等待状态。判断进展时看最近查询、销量新增和任务变化，不只看进程或心跳。
- **21-Stop-This-Project.cmd**：停止本项目，保留数据、断点和剩余等待；其他目录的采集器不受影响。
- **22-Resume-Manual-Stop-Only.cmd**：人工确认后输入 `RESUME`，解除由本包手动停止入口写入的标记，再另选模式启动。平台HOLD和其他等待不会一并清除。

日志位置：

```text
API日志：<根目录>\nh-sales-v3\output\api-collect\run-*.log
API探针：同目录 probe-events-*.jsonl、probe-state.json
GUI日志：kit\logs\、GUI-ONLY\logs\、GUI-ONLY\dutycycle\、output\v3\
```

探针记录已有请求的上下文，不额外发送测试请求。看最新日志的时间，旧日志里的车型或城市不代表当前任务。

### 常见问题

| 情况 | 先检查什么 |
|---|---|
| 找不到窗口或控件 | 分辨率、缩放、精确标题、前台是否为正确小程序，有无窗口遮挡 |
| `interactive_desktop_unavailable` | 虚拟机或电脑是否锁屏、休眠，交互桌面是否断开 |
| OCR失败或表格不完整 | 先复核已有截图和当前范围，避免重复提交同一查询 |
| token找不到或401 | 微信是否登录，是否打开正确H5会话；链接中的临时code不是永久token，刷新后的有效性仍由接口响应确认 |
| `SAVED`后等待较长 | 日志里的异常退避、保留等待、服务端等待或长冷却 |
| “查询次数已用完”“无权使用” | 先手工确认账户权限和额度，再处理停止原因 |
| evidence损坏 | 新证据写入evidence2；检查磁盘状况，换文件夹不能修复磁盘本身 |

## 7. 备份和迁移

先停止采集，再备份：

- 完整 `nh-sales-v3\output`，包括数据库、catalogs、evidence2、adaptive-attempts、v3和api-collect。
- kit的 `runtime`。
- GUI的 `dutycycle`。
- 当前代码和配置。

SQLite正在写入时，可能还有 `-wal`、`-shm` 文件，不要只复制主数据库。本包使用核验后的静态快照。当前虚拟机若已有更新数据，先导出其最新文件，不能用本包快照覆盖。

迁移时停止旧机器，在新机器的空目录安装，再接续同一分片。不要复制同一P1断点后让两台同时跑重叠任务。

SQL用于恢复到新的空数据库或核对数据：解压 `adaptive-full.sql.gz`，导入后检查全部表计数和完整性。日常使用直接保留 `adaptive.sqlite3` 即可。SQL恢复结果在 `SQL-RESTORE-VERIFICATION.json`。

## 8. 手机版资料

```text
archives\NH-Honor-P1P2-D-20261001-Code-Patch.zip
archives\NH-Honor-Click-Screenshot-Evidence-20261002.zip
phone\latest-stopped-runtime\
phone\CONTINUITY.json
phone\PROGRESS.json
```

手机方案采用手机原生截图、USB传回电脑、电脑OCR，保留35个月方案和执行账本。最新批量尝试因缺少日期文字回执退出，自动批量采集尚未验收。状态为 `USER_STOPPED`，两个旧监控暂停。

曾有毕节/问界M9单次查询的15条审核结果，保存在手机独立 `auto_sales` 表，未合并到电脑主库。`40-Phone-Package-Status.cmd`只查看资料状态，不启动手机采集。

截图包保留原生截图和程序输入回执，可用于核对操作过程。模拟触摸属于程序操作，菜单截图不计为新销量查询。

## 9. 平台、网络和查询记录

平台是“乘用车销量/NH乘用车销量库”小程序及相关微信H5，保存的API代码使用maibasi.com业务接口。

你提供的购买规则包括会员手工查询和截图分享。接口及批量自动查询的额度，以平台给出的具体说明为准，购买说明和客服回复可以一起留档。

历史问题包括401、空HTTP响应、页面权限或查询次数提示、网络错误，也包括窗口定位、token恢复和OCR问题。排查时对照同一时间的日志、页面提示和账户状态；不能只凭响应慢就判断是反爬或服务器性能问题。

后台截图里的“3毫秒/4毫秒”是**消耗时间**，不是请求间隔。频率看同一账户的操作日期，或探针里的 `start_interval_seconds`。

虚拟机私网IP不同，公网出口可能仍与宿主机相同，NAT通常共用出口。更换昵称也不等于更换会员或会话标识。需要迁移网络或排障时，先停止采集、保留断点，再确认微信登录和实际出口；本包不自动切换IP或账号。

`HTTP_PROXY`、`HTTPS_PROXY`环境变量不能作为微信小程序已经走代理的判断依据。手机USB调试用于控制和传图，业务流量走哪条网络，要看手机实际路由和代理设置。

随机等待和退避用于控制频率、减少重复请求，没有固定的“不封号间隔”。遇到明确权限、额度或限流提示，按日志停止或等待；token有效期和多个token能否同时使用，目前没有固定规律结论。

相关资料：[Scrapy节流说明](https://docs.scrapy.org/en/latest/topics/autothrottle.html)、[Android ADB文档](https://developer.android.com/tools/adb)。

## 10. 文件与验证记录

```text
00-Set-Root.cmd                  选择安装目录
01-Install.cmd                   安装到空目录
02-Check-Install-Environment.cmd  环境检查，缺VC++时下载并安装
03-Repair-VCpp.cmd                修复VC++
10-Start-Pure-GUI.cmd             纯GUI
11-Start-GUI-API.cmd              API优先，GUI顺序接续
12-Start-Pure-API.cmd             纯API
20-Status.cmd                    查看状态
21-Stop-This-Project.cmd          停止本项目
22-Resume-Manual-Stop-Only.cmd     解除本包手动停止标记
30/31-Long-Rest-OFF/ON.cmd        长休息开关
32/33-Short-Rest-OFF/ON.cmd       短休息开关
40-Phone-Package-Status.cmd       查看手机版状态
90/91/92-Offline-Plan-*.cmd       离线查看三种模式计划
payload/                        代码、Python、电脑数据库和证据
sql/                            完整SQL
phone/                          手机版停止时资料
archives/                       发布包与证据归档
snapshots/api-unbound/           尚未绑定机器的API旧断点
```

整合包完成了离线入口、目录适配、依赖和原有测试，以及SQL恢复和数据/证据哈希核对；没有重新进行实际微信采集验收。运行前按环境检查结果准备。

`VERIFICATION.txt`保存测试和哈希，`ROLES.json`记录原交付及后续补充文件。回滚脚本只恢复指定的包副本，不修改正式数据库或已安装目录。

完整版含数据库、截图及诊断，可能带有账户、会员、IP等信息。用于私人备份和迁移，分享给别人前先检查内容。
