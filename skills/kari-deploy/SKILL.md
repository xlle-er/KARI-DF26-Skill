---
name: kari-deploy
description: Guide activity participants through their first KARI DF26 firmware installation, using Agent-assisted USB flashing or the web installer, with recommended-board defaults or custom wiring.
---

**语言 / Language:** [简体中文](#简体中文) · [English](#english)



## English

# First installation

Help beginners and experienced participants install the latest official stable firmware. Do not ask participants to choose a firmware version or require a fixed version number. Reuse information they already supplied and skip answered questions.

First ask: “你使用的是活动方推荐的开发板吗？” (是 / 否 / 不确定).
- Yes: use N16R8 and official default wiring. Say “请按文档接线，烧录时断开电机电源” and link the [wiring documentation](https://github.com/xlle-er/KARI-DF26-Firmware#官方接线). This confirmation is sufficient: do not additionally ask for module markings or each GPIO.
- No / unsure: help identify an ESP32-S3 N16R8 or N8R8 module with a DevKitC-1-compatible pinout. Flash capacity alone cannot establish PSRAM or wiring. Do not guess unsupported pinouts. Ask whether they use documented official wiring or custom wiring; only custom wiring needs all six GPIOs, collected in one question.

Ask once: “你想给 KARI 起什么名字？支持中文，也可以保留默认识别码。” Reuse any name already provided. Names must contain visible text and fit 24 UTF-8 bytes (up to 8 common Chinese characters). Never truncate a character. The firmware derives both the robot's BLE name and hotspot as `KARI-名字`; skipped naming keeps the current name or the default `KARI-识别码`.

Then ask whether they prefer Agent-assisted USB installation or the web installer, unless their request already makes this clear. Choosing a route does not require a second deployment approval.

## Agent-assisted USB installation

Fetch the latest `main` source directly from the [KARI-DF26-Skill repository](https://github.com/xlle-er/KARI-DF26-Skill): `git clone --branch main https://github.com/xlle-er/KARI-DF26-Skill.git`. Install or refresh the skill from `skills/kari-deploy/` in that checkout and use its matching `kari_deploy/` tool. Do not require a tutorial attachment or use a bundled snapshot as the source of updates. If reusing a clean checkout, fetch `origin main` and update with `git pull --ff-only origin main`; preserve local edits and stop on divergence rather than resetting them. Use a fresh checkout when updating would conflict with local work. Create a project `.venv` and run `python -m pip install -e .` there. Python 3.10+ is required; do not install system-wide packages. Use `.venv/bin/python` on macOS/Linux or `.venv\Scripts\python.exe` on Windows.

Run `python -m kari_deploy.cli --json doctor`, identify the target USB port, then `inspect --port PORT`. Ask which port belongs to KARI if there are multiple candidates. Disconnect motor power before inspection, which can reset an older board. Never start motor tests automatically.

For a first installation on the recommended board, run `python -m kari_deploy.cli --json install --port PORT --board n16r8 --use-default-pins`. Add `--name NAME` only when the participant chose a name; the tool writes it over USB and verifies both BLE/status name and hotspot after reboot. Omit `--release`: the tool downloads and verifies the latest official stable assets. If unavailable or incompatible, explain the failure; do not silently use a prerelease or bypass compatibility checks. Only use an explicit `--release TAG` or `--assets DIRECTORY` when requested by a maintainer.

Inspect existing wiring before choosing installation options. Preserve an existing valid hardware profile by omitting both profile options. If it differs from the recommended board/default pins, explain that briefly and ask whether to keep it or switch to official wiring; default to preservation. Actual hardware conflicting with the selected board must stop installation for clarification, never trigger automatic reflashing with another board profile.

For custom wiring, write a local JSON profile with `version: 1`, `board: "n16r8"` or `"n8r8"`, and `pins: {ena,in1,in2,in3,in4,enb}` containing integers. Validate with the package's `validate_profile`, show the resulting table, then use `--profile FILE`. GPIO numbers differ from header positions. Official pins are ENA=17, IN1=16, IN2=15, IN3=7, IN4=6, ENB=5; A=left, B=right. Share GND and remove ENA/ENB PWM jumpers. Do not rebuild firmware to change wiring.

First-time participants have not set a personal device password. Do not ask them to create or enter one, and do not add a password-choice step. The Python tool automatically supplies the firmware’s built-in initial credential for wiring and naming; run the normal install command without `--password-stdin`. If an existing device rejects initial authentication, stop and explain that it was previously configured; do not reset its credentials or erase NVS. Only in that existing-device recovery flow may the owner supply their current credential through `--password-stdin`; never put personal credentials in argv, files, logs, reports or Git.

Success requires the tool's verified result: device version matches the downloaded manifest dynamically, board/physical memory/all six pins match, and `hardware_ready=true`. An independent `verify --port PORT --board BOARD` does not need `--version`; never add a fixed version gate. Keep the password-free `.kari-deploy` record local. Report the installed version as a result, not a prerequisite.

## Web installation

The public [KARI web installer](https://xlle-er.github.io/KARI-DF26-Firmware/) is deployed and verified. Direct participants there; the [installer source](https://github.com/xlle-er/KARI-DF26-Firmware/tree/main/installer) also supports local hosting when requested.

The web route uses the same recommended-board question and N16R8/default-wiring shortcut. Other boards expose model/wiring controls; existing configurations are preserved unless the participant chooses to replace them. Participants select the USB device and click Install in desktop Chrome/Edge over HTTPS or localhost. Do not ask them to select a firmware version. First-time installation has no device-password field: the wizard handles the initial credential internally. If an older published installer still requires that field, use the updated installer source or the Agent route instead of adding a password question for participants.

## Recovery and next steps

Never erase-flash, overwrite unknown partitions or bypass hardware/OTA checks. Unknown layouts require a separate migration investigation and NVS backup. If native USB re-enumerates, run doctor, identify the same physical board, and resume inspect/configure/verify instead of flashing repeatedly. Check occupied ports and data cables first; driver, serial permission and BOOT/RST guidance is in the repository README.

For wiring-only changes use `configure` with the same profile options without reflashing. After installation direct participants to the hotspot name returned in `ap_ssid` (now `KARI-名字` or `KARI-识别码`) and `http://192.168.4.1/` for Wi-Fi and direction calibration with wheels lifted. Distinguish software verification from real motor testing. Compatible official OTA preserves wiring; clearing NVS requires configuring it again.

## 简体中文

### 首次安装

帮助新手与熟悉设备的参与者安装最新官方稳定固件，不让参与者选择版本或填写固定版本号。复用已提供的信息，跳过已回答的问题。

先问“你使用的是活动方推荐的开发板吗？”（是/否/不确定）。是：采用 N16R8 与官方接线，说明“请按文档接线，烧录时断开电机电源”，链接[接线文档](https://github.com/xlle-er/KARI-DF26-Firmware#官方接线)；确认足够，不追加逐个 GPIO 或模组丝印问题。否/不确定：协助识别 ESP32-S3 N16R8/N8R8 和 DevKitC-1 兼容布局。Flash 容量不能确认 PSRAM 或接线，不猜测未支持板型。询问默认/自定义接线，仅自定义时一次收集六个 GPIO。

问一次“你想给 KARI 起什么名字？支持中文，也可以保留默认识别码。”复用已有答案；名字需可见文字、最多 24 UTF-8 字节（常见汉字最多约 8 个），不截断字符。固件广播/热点均为 KARI-名字；跳过则沿用现名或 KARI-识别码。

除非请求已明确，选择 Agent USB 烧录或网页安装；选择路径不需要第二次部署批准。

### Agent USB 安装

直接从 https://github.com/xlle-er/KARI-DF26-Skill 的最新 main 获取：

```sh
git clone --branch main https://github.com/xlle-er/KARI-DF26-Skill.git
```

安装/刷新该目录 skills/kari-deploy/ 的 Skill，使用配套 kari_deploy/。不要求教程附件，不把随附件旧快照当更新源。干净目录可 fetch origin main 后 git pull --ff-only origin main；存在修改或分歧时保留，用新目录而非重置。Python 3.10+，创建项目 .venv，用 python -m pip install -e .；不要全局安装。macOS/Linux 用 .venv/bin/python，Windows 用 .venv\Scripts\python.exe。

先运行 python -m kari_deploy.cli --json doctor，再 inspect --port PORT。多个串口须确认目标；检查前断开电机电源，因为旧板可能复位。不自动测试电机。

推荐板首次安装：

```sh
python -m kari_deploy.cli --json install --port PORT --board n16r8 --use-default-pins
```

仅用户选择名字时加 --name NAME，工具通过 USB 保存并重启回读 BLE/状态名字和热点。省略 --release，动态下载校验最新稳定资产；不可用/不兼容时明确失败，不暗用预发布或绕过检查。只有维护者明确要求时才用 --release TAG / --assets DIRECTORY。

先检查已有接线。有有效配置时省略两种 profile 选项保留；与默认不同则简要说明，询问保留或切换，默认保留。实际硬件与所选板型冲突时停止澄清，不自动换板反复烧录。

自定义配置使用本地 JSON：version=1、board 为 n16r8/n8r8，pins 为整数 ena/in1/in2/in3/in4/enb。用工具 validate_profile 校验、展示表后 --profile FILE。GPIO 不是排针序号；默认 ENA=17、IN1=16、IN2=15、IN3=7、IN4=6、ENB=5；A 左/B 右，共地，拔 ENA/ENB PWM 跳帽。改接线无需重编固件。

首次参与者没有设置个人密码，不增加创建/输入密码步骤；工具自动带初始凭据，普通安装不要 --password-stdin。已有设备拒绝初始凭据时停止，说明曾配置，不重置凭据或擦 NVS。仅恢复已有设备时由所有者经 --password-stdin 提供当前密码；不放 argv、文件、日志、报告或 Git。

成功必须使用工具 verified 结果：设备版本动态匹配下载清单，板型/物理内存/六 GPIO 匹配且 hardware_ready=true。独立 verify --port PORT --board BOARD 无需 --version，不加固定版本门槛。无密码 .kari-deploy 记录仅本地保存；版本作为结果报告。

### 网页安装

[公开安装页](https://xlle-er.github.io/KARI-DF26-Firmware/) 已部署并验证可用；需要时可使用[源码](https://github.com/xlle-er/KARI-DF26-Firmware/tree/main/installer)本地托管。沿用推荐板问题和 N16R8 默认接线快捷路径；其他板显示板型/接线控制，未明确替换则保留已有配置。

参与者在电脑 Chrome/Edge 的 HTTPS/localhost 页面选择 USB、点击安装，不需要选固件版本。首次安装不输入设备密码，向导内部处理初始凭据；若旧网页仍要求密码，使用更新源码或 Agent 路径，不额外追问首次用户密码。

### 恢复和后续

不擦整片 Flash、不覆盖未知分区、不绕过硬件/OTA 检查。未知布局须另行调查并备份 NVS。原生 USB 重新枚举后 doctor、确认同一块板，继续 inspect/configure/verify 而不是反复刷写。先排查占用与数据线，驱动/串口权限/BOOT-RST 见 README。

仅改接线用 configure 和同一组选项，无需重刷。安装后指向回读 ap_ssid（KARI-名字或KARI-识别码）和 http://192.168.4.1/，车轮架空进行配网/方向校准。区分软件回读和真实电机测试。兼容官方 OTA 保留接线；清 NVS 后需重新配置。
