# NH WeChat Vehicle Sales Collector — User Guide

Collector code version: NH-Unified-20261006-R1. Source guide revised: 2026-10-06. English edition: 2026-10-09.

The package includes three desktop collection modes, Python and OCR dependencies, the saved sales database and task checkpoints, and the phone project's code and screenshot records. Extract the entire ZIP before running a CMD file; do not run it from inside an archive viewer.

## 1. Use a dedicated virtual machine

**Pure GUI collection controls the mouse, scrolls the page, and keeps the mini program in the foreground. While it runs, this computer is not suitable for office work, chatting, or operating other windows at the same time.** Moving the mouse, switching pages, or covering the window can pause collection.

A dedicated Windows virtual machine is recommended: run WeChat and the collector inside the VM, and use the host computer normally. A spare physical computer is another option.

Keep WeChat signed in and the VM's interactive desktop available. Do not lock it, put it to sleep, or disconnect its interactive desktop. Use the VM console and avoid taking over the mouse or switching pages inside the VM while collection runs. Keep the host awake as well. Combined mode can switch to GUI and needs the same setup; authentication recovery in pure API mode can also operate a WeChat window.

### Current display requirements

The current GUI calibration uses **1600×1200, one display, and 100% Windows scaling**. The mini-program window is fixed at **575×1154**, with the exact title `NH乘用车销量库`.

| Display setting | Current status |
|---|---|
| 1600×1200 at 100% scaling | The calibration used by this package. Use this setting in the VM where possible. |
| 1920×1080 (16:9, 1080p) | Not adapted yet. The original 1154-pixel-high window does not fit a 1080-pixel-high display. The window must be shortened and the footer buttons, scrolling, and screenshot area recalibrated. |
| 2560×1440 (16:9, commonly called 2K) | Not accepted through testing yet. The original window can fit, but display checks and WeChat recovery still contain 1600×1200 restrictions and need adjustment and testing. |
| 125% or 150% scaling, or multiple displays | Requires separate calibration. Coordinates calibrated at 100% scaling cannot simply be reused. |

The code already uses window-relative coordinates and control detection, so 1080p and 2K profiles can be added after measurement. Relaxing the display check alone is insufficient: date, city, model, Back and Confirm controls, screenshots, and OCR also need checking. Initial calibration can use selection pages and existing screenshots without pressing Query. This revision changes documentation only; the original display configuration remains in place.

## 2. First-time installation

1. Extract the full ZIP, for example to `D:\NH-Unified-20261006`.
2. Double-click **00-Set-Root.cmd** and choose `C:\NH-Crawler`, `D:\NH-Crawler`, `E:\NH-Crawler`, or another absolute directory. The default is `C:\NH-Crawler`.
3. Double-click **01-Install.cmd**. After checking the files, the installer copies the code, Python, database, and evidence to that directory and adjusts runtime paths. The destination must be empty; no directory junction is used.
4. Double-click **02-Check-Install-Environment.cmd**. If the x64 Visual C++ runtime is missing, it downloads the Microsoft installer, verifies its signature, and requests administrator approval. Restart Windows first if the installer requests a restart.
5. Open `NH乘用车销量库` in WeChat, use the display settings above, and choose a collection mode.

Python 3.12, OCR, OpenCV, ONNX Runtime, and window-control dependencies are bundled. The Visual C++ installer is downloaded online; it is not included as an offline installer. If other dependencies are damaged, the environment check lists the failed items instead of automatically replacing library versions. **03-Repair-VCpp.cmd** reinstalls or repairs the Visual C++ runtime.

Installed launchers are under `<root>\UNIFIED\`; use those for subsequent runs. If the destination already contains data, back it up and choose a new empty directory. A partly installed directory left by an interrupted installation also needs separate attention.

You can also run this from the extracted directory:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\Unified.ps1 -Action install -Root 'E:\NH-Crawler'
```

This installs the package without starting collection. Old drive letters in historical evidence are retained; locate the original image under the new root using the same evidence-relative path.

## 3. Choose a collection mode

### 10-Start-Pure-GUI.cmd: pure GUI

Uses simulated clicks in the mini program, reads its window, takes screenshots, and runs OCR on the computer before writing to the existing queue and database. Screenshots come from the target window, not downloaded sales images. This launcher does not start the API sales collector.

The existing input settings, window lock, supervisor, and obstruction recovery are retained. Each query's finishing stage includes a random 10–20-second wait. Review existing screenshots first if OCR fails. Reading an API checkpoint file locally is not an API request.

### 11-Start-GUI-API.cmd: API with GUI handoff

Runs API collection first. When authentication recovery attempts are exhausted and the handoff conditions are satisfied, API collection exits before GUI starts. While GUI runs, the adapter only reads local token candidates. When a candidate appears, it stops GUI first and then lets an API request validate the candidate.

The two modes run sequentially, not concurrently, and use the same desktop database and output directory. The complete handoff in this newly integrated package still needs acceptance testing in a real WeChat environment. Existing waits, HOLD states, and authentication failures are retained and shown in status and logs.

### 12-Start-Pure-API.cmd: pure API

Runs only the API sales collector and does not fall back to GUI sales collection after authentication failure. Local token discovery and WeChat H5 refresh/reopen authentication recovery are retained.

Healthy requests wait a random 12–20 seconds. Abnormal backoff may extend to 120 seconds; long cooldowns are stored separately. If pacing state has not been initialized, the original B05 logic may wait 100 minutes first. Restarting preserves the remaining wait.

The default query window is 24 months. When a national response contains fewer than 35 rows, one response can include several cities and months. At 35 rows, it splits by province and then city. A city response still reaching 35 rows is split by month. If a single-city, single-month query still reaches the limit, it is recorded as `UNFINISHED`.

The current API mode does not use 35-month windows. The number 35 is a response-row cap, not a month count. Completion depends on coverage, not on whether a row dated 2026 appears. The original model-name-to-API-ID mapping is retained; manufacturer attribution for identically named models needs checking against the catalog.

If this project already has a collection process, clicking Start again shows its status. Startup stops if a collector in another directory is found or process identity cannot be confirmed. Stop the current mode before switching to avoid running multiple collectors.

## 4. Scheduled breaks

GUI breaks are based on cumulative active collection time:

- After one hour of collection, take a random 10–20-minute break.
- After four cumulative hours, take a random 45–101-minute break. The long break takes precedence; a short break is not added.
- Break time is excluded from active collection time.

| Launcher | Action |
|---|---|
| **30-Long-Rest-OFF.cmd** | Disable future four-hour scheduled long breaks |
| **31-Long-Rest-ON.cmd** | Re-enable four-hour scheduled long breaks |
| **32-Short-Rest-OFF.cmd** | Disable future hourly scheduled short breaks |
| **33-Short-Rest-ON.cmd** | Re-enable hourly scheduled short breaks |

The settings are `scheduled_long_rest_enabled` and `scheduled_short_rest_enabled` in `<root>\GUI-ONLY\duty-cycle.json`. The controller reads them on its next run cycle; repeated restarts are unnecessary. Disabling a schedule does not end an already reserved break early, and re-enabling it does not reset cumulative runtime.

The current configuration has no automatic cooldown every 40 minutes. The 40-minute figure in older material may refer to monitoring frequency.

These switches control scheduled breaks only. Per-query waits, API backoff, server-directed waits, insufficient permissions or quota, and STOP/HOLD follow the original logic. Do not substitute the old `GUI-DUTY-OFF` command for these switches.

## 5. Data and checkpoints

Main database:

```text
<root>\nh-sales-v3\output\adaptive.sqlite3
```

| Table or directory | Contents |
|---|---|
| `sales` | Collected sales, retaining the original manufacturer, brand, model, category, year-month, city/province, and other fields |
| `tasks` | Query scopes, parent/child tasks, status, completion types, and partition skips, including verified empty results |
| `attempts` | Attempts, leases, and submission records used to determine whether a query can be retried after interruption |
| `evidence`, `row_sources` | Screenshot-evidence indexes and row provenance |
| `output\v3\` | Runtime state, heartbeats, and stop reasons |
| `output\evidence2\` | Screenshot evidence |
| `GUI-ONLY\dutycycle\`, the kit's `runtime\` | Breaks, HOLD states, gates, and mutual-exclusion state |
| `output\api-collect\done.txt` | Completion checkpoints generated by current API runs; scope, unfinished-task, and authentication records are in the same directory |

The database snapshot contains **97,241 sales rows, 50,324 tasks, and 12,238 attempt records**. There are 2,576 `verified_empty_scope*` tasks recording verified empty scopes; zeros are not inserted into the sales table. This is a saved local snapshot, not today's combined totals from all VMs.

The partition is **P1 / [0,504]**, ordered as `p1_cut_forward_circular_p2_p3_p4_p5`. The original GUI date range and exclusion of 2026 are retained. `partition_skipped` means another partition is responsible; it is not a missing task on this machine.

Older API checkpoints are in `snapshots\api-unbound\` and `archives\API-checkpoints-unbound.zip`. They have not been bound to this machine and are inactive. Before importing them, check machine identity, model/manufacturer, months, city scope, and the database. Do not directly merge `done.txt` from different partitions.

The database passes physical-integrity checks; historical foreign-key issues remain. Older versions may have false-empty, interrupted, or unfinished records. Packaging has not cleaned those checkpoints.

The full SQL export is `sql\adaptive-full.sql.gz` and contains all tables. `DATA-SNAPSHOT.json` records counts and hashes; `SOURCES.json`, `SUPPLEMENTS.json`, and `EXTRA-HISTORICAL-FILES.json` record provenance. Missing historical screenshots and catalogs were supplied from an older migration package without overwriting the newer database or state.

## 6. Progress, stopping, and resuming

- **20-Status.cmd**: displays database counts, stop conditions, and waits. Look at recent queries, sales additions, and task changes—not just processes or heartbeats.
- **21-Stop-This-Project.cmd**: stops this project while retaining data, checkpoints, and remaining waits. Collectors in other directories are unaffected.
- **22-Resume-Manual-Stop-Only.cmd**: after manual confirmation, enter `RESUME` to release the marker written by this package's manual Stop launcher. Then choose a mode separately. Platform HOLD states and other waits are not cleared with it.

Log locations:

```text
API logs: <root>\nh-sales-v3\output\api-collect\run-*.log
API probe: probe-events-*.jsonl and probe-state.json in the same directory
GUI logs: kit\logs\, GUI-ONLY\logs\, GUI-ONLY\dutycycle\, output\v3\
```

The probe records the context of existing requests; it sends no extra test requests. Check timestamps in the latest log. A model or city in an old log is not necessarily the current task.

### Common issues

| Issue | Check first |
|---|---|
| Window or control not found | Display resolution, scaling, exact title, correct foreground mini program, and obstructions |
| `interactive_desktop_unavailable` | Whether the computer or VM is locked, sleeping, or has lost its interactive desktop |
| OCR failure or incomplete table | Existing screenshots and the current scope; avoid repeating the same submitted query |
| Token not found or HTTP 401 | WeChat login and the correct H5 session; a temporary URL code is not a permanent token, and an interface response still needs to validate a refreshed token |
| Long wait after `SAVED` | Abnormal backoff, retained waits, server-directed delays, or long cooldown in the log |
| Query count exhausted or no permission | Confirm account permissions and quota manually before addressing the stop reason |
| Damaged evidence | New evidence is written to evidence2; check the disk, because changing directories does not repair a damaged disk |

## 7. Backup and migration

Stop collection before backing up:

- The complete `nh-sales-v3\output`, including the database, catalogs, evidence2, adaptive-attempts, v3, and api-collect.
- The kit's `runtime` directory.
- GUI `dutycycle`.
- Current code and configuration.

A writing SQLite database may also have `-wal` and `-shm` files; do not copy only the main database file. This package uses a checked static snapshot. If the current VM has newer data, export those files first rather than overwriting them with this snapshot.

For migration, stop the old machine, install into an empty directory on the new machine, and resume the same partition. Do not copy one P1 checkpoint to two machines and run overlapping tasks simultaneously.

SQL is for restoring into a new empty database or auditing data. Decompress `adaptive-full.sql.gz`, import it, and check all table counts and integrity. For normal use, keep `adaptive.sqlite3` directly. Restore results are in `SQL-RESTORE-VERIFICATION.json`.

## 8. Phone-project files

```text
archives\NH-Honor-P1P2-D-20261001-Code-Patch.zip
archives\NH-Honor-Click-Screenshot-Evidence-20261002.zip
phone\latest-stopped-runtime\
phone\CONTINUITY.json
phone\PROGRESS.json
```

The phone workflow uses native phone screenshots, USB transfer to the computer, and computer-side OCR, retaining the 35-month plan and execution ledger. The latest batch attempt exited because a date-text receipt was missing; automated batch collection has not passed acceptance testing. State is `USER_STOPPED`; the two older monitors are paused.

A single Bijie/Wenjie M9 query previously produced 15 reviewed rows in the phone's separate `auto_sales` table; they were not merged into the desktop database. **40-Phone-Package-Status.cmd** displays archived state only and does not start phone collection.

The evidence archive preserves native screenshots and program-input receipts for reviewing the operation sequence. Simulated touch is a program action; menu screenshots do not count as new sales queries.

## 9. Platform, networking, and query records

The platform is the “乘用车销量/NH乘用车销量库” mini program and related WeChat H5 pages. The saved API code uses maibasi.com business interfaces.

The purchase terms you supplied include member queries by hand and screenshot sharing. Interface and automated batch-query quotas depend on the platform's specific terms. Keep purchase information and customer-service responses with your records.

Historical problems include HTTP 401, empty HTTP responses, permission/quota messages, network failures, and window-positioning, token-recovery, and OCR issues. Match logs, page messages, and account state at the same time. Slow responses alone do not establish anti-bot detection or poor server performance.

The “3 ms/4 ms” field in the backend screenshot is **processing duration**, not request spacing. Use operation timestamps for the same account or the probe's `start_interval_seconds` to check frequency.

Different private VM IPs may share the same public egress address as the host; NAT commonly shares egress. Changing a nickname does not necessarily change a member or session identifier. Before network migration or troubleshooting, stop collection, retain checkpoints, and verify WeChat login and actual egress. The package does not rotate IPs or accounts automatically.

`HTTP_PROXY` and `HTTPS_PROXY` environment variables do not prove that a WeChat mini program uses the proxy. Phone USB debugging controls the phone and transfers images; phone routing and proxy settings determine the network used for business traffic.

Random waits and backoff control frequency and reduce duplicate requests. There is no fixed “ban-proof interval.” Follow the logged stop or wait when permissions, quota, or rate limiting are explicit. There is no established fixed token lifetime or conclusion that several tokens can remain valid simultaneously.

References: [Scrapy throttling](https://docs.scrapy.org/en/latest/topics/autothrottle.html), [Android ADB](https://developer.android.com/tools/adb).

## 10. Files and verification records

```text
00-Set-Root.cmd                  Choose the installation root
01-Install.cmd                   Install into an empty directory
02-Check-Install-Environment.cmd  Check dependencies; download/install missing VC++
03-Repair-VCpp.cmd                Repair VC++
10-Start-Pure-GUI.cmd             Pure GUI
11-Start-GUI-API.cmd              API first, sequential GUI handoff
12-Start-Pure-API.cmd             Pure API
20-Status.cmd                    Show status
21-Stop-This-Project.cmd          Stop this project
22-Resume-Manual-Stop-Only.cmd     Release this package's manual Stop marker
30/31-Long-Rest-OFF/ON.cmd        Long-break switches
32/33-Short-Rest-OFF/ON.cmd       Short-break switches
40-Phone-Package-Status.cmd       Show phone-project state
90/91/92-Offline-Plan-*.cmd       View plans for the three modes offline
payload/                        Code, Python, desktop database, and evidence
sql/                            Full SQL export
phone/                          Phone files at the time of stopping
archives/                       Releases and evidence archives
snapshots/api-unbound/           Older API checkpoints not yet bound to a machine
```

The integrated package passed offline entry-point, path-adaptation, dependency, and existing tests, as well as SQL restoration and data/evidence hash checks. It has not undergone a new live WeChat collection acceptance test. Prepare the environment using the check results before running it.

`VERIFICATION.txt` records tests and hashes; `ROLES.json` lists the original artifacts and subsequent supplements. Rollback restores only the specified package copy, not a production database or installed directory.

The full package contains databases, screenshots, and diagnostics that may include account, member, and IP information. Use it for private backup and migration; inspect its contents before sharing it.
