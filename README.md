# KARI-DF26-Skill

**语言 / Language:** [简体中文](#简体中文) · [English](#english)

## 简体中文

> [统一使用文档](https://github.com/xlle-er/KARI-DF26) · 默认安装最新 KARI 稳定固件；离线部署可使用下文 --assets 指向配套本地构建结果。

> [KARI 统一使用文档](https://github.com/xlle-er/KARI-DF26) · 本次为新应用身份与新设备配置，需要重新安装和配置。当前验证结果见 [KARI 验证状态](https://github.com/xlle-er/KARI-DF26/blob/main/docs/validation.md)，历史验证不计入本次通过项。

用一句话让 Codex 或其他 Agent 部署 KARI ESP32-S3 底盘；也可直接使用跨平台 Python 工具。支持 N16R8、N8R8 的 DevKitC-1 兼容开发板，官方默认接线和自定义 GPIO 共用同一份固件，接线保存在设备中。

默认安装最新官方稳定固件，参与者无需选择版本。支持活动推荐开发板快捷安装，也支持其他兼容开发板和自定义接线。固件及网页安装源码见 [KARI-DF26-Firmware](https://github.com/xlle-er/KARI-DF26-Firmware)。本仓库不包含固件二进制。

## 一句话安装并烧录

把下面这一句话发给 Codex，Agent 会安装 skill 并继续引导首次烧录，无需再发第二条部署指令：

> 请直接从 https://github.com/xlle-er/KARI-DF26-Skill 拉取 main 分支最新代码，安装其中 skills/kari-deploy 的 Skill，并使用该仓库配套工具引导我首次烧录最新官方稳定固件。

Agent 会直接从 GitHub 获取最新 Skill 和配套部署工具，无需下载教程附件。已有干净仓库会快进更新；存在本地修改或分支分歧时会保留原内容并使用新目录。

Agent 首先询问“你使用的是活动方推荐的开发板吗？”（是 / 否 / 不确定）。选择“是”默认采用 N16R8 和官方接线，只需[按文档接线](https://github.com/xlle-er/KARI-DF26-Firmware#官方接线)，烧录时断开电机电源，无需逐项回答 GPIO。选择“否”或“不确定”时，Agent 协助确认板型，仅自定义接线需要提供六个 GPIO。已有有效接线默认保留，改用官方接线前会询问。

随后可选择让 Agent 帮你烧录或使用网页；已提供的信息不会重复询问。Agent 会检查 USB 设备、下载校验最新稳定固件、烧录并回读验证。多个串口时需要选择目标设备；依赖安装在项目独立环境中。不会自动点动电机、擦除 NVS 或覆盖未知分区。

**[打开网页安装入口](https://xlle-er.github.io/KARI-DF26-Firmware/)。** [网页安装源码](https://github.com/xlle-er/KARI-DF26-Firmware/tree/main/installer)可本地运行，使用电脑 Chrome / Edge，通过 HTTPS 或 localhost 打开；也可使用 Agent 烧录。

## 手动使用

Python 3.10+。macOS/Linux：

```sh
git clone https://github.com/xlle-er/KARI-DF26-Skill.git
cd KARI-DF26-Skill
python3 -m venv .venv
.venv/bin/python -m pip install -e .
.venv/bin/python -m kari_deploy.cli --json doctor
.venv/bin/python -m kari_deploy.cli --json inspect --port /dev/cu.YOUR_DEVICE
.venv/bin/python -m kari_deploy.cli --json install --port /dev/cu.YOUR_DEVICE --board n16r8 --use-default-pins
```

Windows PowerShell：

```powershell
git clone https://github.com/xlle-er/KARI-DF26-Skill.git
cd KARI-DF26-Skill
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e .
.\.venv\Scripts\python.exe -m kari_deploy.cli --json doctor
.\.venv\Scripts\python.exe -m kari_deploy.cli --json inspect --port COM3
.\.venv\Scripts\python.exe -m kari_deploy.cli --json install --port COM3 --board n8r8 --use-default-pins
```

以上串口仅为示例。始终使用 `doctor` 中实际属于目标开发板的串口；多个设备时不能猜测。`--use-default-pins` 仅在实际接线符合下表时使用。已有配置的重装省略该参数即可保留；未配置设备需要明确选择默认接线或提供配置文件。

| L298N | GPIO |
|---|---:|
| ENA（左） | 17 |
| IN1 | 16 |
| IN2 | 15 |
| IN3 | 7 |
| IN4 | 6 |
| ENB（右） | 5 |

使用 GPIO 编号，不是排针序号。ESP32 与 L298N 共地，拔掉 ENA/ENB 跳帽，安装和配置时断开电机电源。活动推荐板确认后默认 N16R8 和官方接线；其他开发板需要确认模块型号和引脚布局。Flash 检测无法判断 PSRAM 或排针布局。首次配置前 GPIO 不启用输出；若电机驱动输入无下拉，不能依赖浮空引脚保证电机电气状态，需保持电机断电或使用适当的使能下拉。

自定义配置文件示例 `my-wiring.json`：

```json
{"version":1,"board":"n16r8","pins":{"ena":17,"in1":16,"in2":15,"in3":7,"in4":6,"enb":5}}
```

```sh
.venv/bin/python -m kari_deploy.cli --json configure --port /dev/cu.YOUR_DEVICE --board n16r8 --profile my-wiring.json
.venv/bin/python -m kari_deploy.cli --json verify --port /dev/cu.YOUR_DEVICE --board n16r8 --profile my-wiring.json
# 维护者使用已构建的配套 USB 清单和分段文件：
.venv/bin/python -m kari_deploy.cli --json install --port /dev/cu.YOUR_DEVICE --board n16r8 --profile my-wiring.json --assets ../KARI-DF26-Firmware/dist
```

首次安装及初始接线配置无需创建或输入设备密码，`install`、`configure` 自动使用固件内置初始凭据。若旧设备已由使用者改过凭据，工具会停止并保留配置；仅此已有设备场景使用 `--password-stdin` 从标准输入提供当前凭据，不会重置凭据。勿把个人凭据放进命令参数、配置文件、日志或 Git。`.kari-deploy/` 仅保存设备 ID、固件版本、接线和验证时间，默认忽略。

退出码：`0` 成功；`2` 参数、配置或兼容性错误；`3` 依赖、文件或网络错误；`4` USB 传输错误；`5` 设备拒绝或回读失败。所有命令 stdout 均为 JSON；`--json` 为显式机器输出标记，放在子命令之前。

## 连接故障和升级

- macOS：优先 `/dev/cu.*`，关闭 Arduino 串口监视器及其他占用工具。
- Windows：确认设备管理器中出现 COM 口；仅从开发板或桥接芯片厂商安装对应 USB 驱动。
- Linux：确认当前用户有串口设备权限；按发行版要求加入串口组后重新登录，不使用 `chmod 777`。
- 连接失败：检查数据线，按住 BOOT、按一下 RST，再松开 BOOT。原生 USB 可能重新枚举；重新 `doctor` 并确认同一块板后使用新串口继续 `configure` / `verify`，不要反复烧录。
- 未知分区：工具停止且不写入。先单独调查并备份 NVS，再决定迁移方式；工具不提供绕过检查的参数。
- 保留接线的兼容官方 OTA 可继续使用。擦除 NVS 后需要重新确认接线。旧自定义固件不能可靠读出其 GPIO 常量，第一次迁移必须重新填写。
- 安装后连接回读返回的实际 `KARI-名字` 或 `KARI-识别码` 热点，访问 `http://192.168.4.1/` 配网、核对接线并在车轮架空时进行短时方向校准。

## 开发与验证

```sh
.venv/bin/python -m unittest discover -s tests -v
```

`kari_deploy/boards.json` 来自固件仓库 `hardware/boards.json`；修改硬件支持范围时同步生成，固件仓库的 contracts 检查两者一致。.github/workflows/test.yml 在 macOS、Windows、Linux 上运行 Python 3.10/3.12 的测试和 doctor。CI 不能替代各系统的真实 USB、两种模块、供电、电机和 OTA 验收。本次本地验证为 macOS，Windows/Linux 真机 USB 与电机测试仍待完成。

## English

[Unified guide](https://github.com/xlle-er/KARI-DF26) · [Current validation](https://github.com/xlle-er/KARI-DF26/blob/main/docs/validation.md). New KARI application/device namespaces require installation and configuration again.

Install the KARI ESP32-S3 chassis with one Agent prompt or the cross-platform Python tool. DevKitC-1-compatible N16R8/N8R8 boards use one firmware for official/custom GPIO wiring, persisted on the device. Latest official stable firmware is the default; participants do not choose a version. Firmware/install-site source is in [KARI-DF26-Firmware](https://github.com/xlle-er/KARI-DF26-Firmware); this tool repository includes no firmware binaries.

### One-prompt installation

> Fetch the latest main branch of https://github.com/xlle-er/KARI-DF26-Skill, install its skills/kari-deploy skill, and use the matching tools to guide my first installation of the latest official stable firmware.

The Agent fetches current skill/tool source directly. Clean checkouts fast-forward; edits/divergence are preserved by using a fresh directory. It first asks whether this is the activity's recommended board (yes/no/unsure). Yes selects N16R8 and [official wiring](https://github.com/xlle-er/KARI-DF26-Firmware#官方接线); disconnect motor power, without answering six GPIO questions. Other boards require identification; only custom wiring needs all six GPIOs. Existing valid wiring is preserved unless you choose otherwise. Already supplied answers are reused.

Choose Agent-assisted USB or the [live web installer](https://xlle-er.github.io/KARI-DF26-Firmware/). [Installer source](https://github.com/xlle-er/KARI-DF26-Firmware/tree/main/installer) also runs locally in desktop Chrome/Edge over HTTPS/localhost. The Agent checks USB, downloads/verifies stable assets, flashes and reads back. Multiple ports require target selection. Dependencies use a project environment. It never automatically moves motors, erases NVS or overwrites unknown partitions.

### Manual operation

Python 3.10+, macOS/Linux:

```sh
git clone https://github.com/xlle-er/KARI-DF26-Skill.git
cd KARI-DF26-Skill
python3 -m venv .venv
.venv/bin/python -m pip install -e .
.venv/bin/python -m kari_deploy.cli --json doctor
.venv/bin/python -m kari_deploy.cli --json inspect --port /dev/cu.YOUR_DEVICE
.venv/bin/python -m kari_deploy.cli --json install --port /dev/cu.YOUR_DEVICE --board n16r8 --use-default-pins
```

Windows PowerShell:

```powershell
git clone https://github.com/xlle-er/KARI-DF26-Skill.git
cd KARI-DF26-Skill
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e .
.\.venv\Scripts\python.exe -m kari_deploy.cli --json doctor
.\.venv\Scripts\python.exe -m kari_deploy.cli --json inspect --port COM3
.\.venv\Scripts\python.exe -m kari_deploy.cli --json install --port COM3 --board n8r8 --use-default-pins
```

Ports above are examples: use the actual target from doctor and never guess between devices. Use --use-default-pins only for the wiring below. Reinstallation omits both profile options to preserve valid wiring; unconfigured devices need an explicit default/profile choice.

| L298N | GPIO |
| --- | --- |
| ENA (left) | 17 |
| IN1 | 16 |
| IN2 | 15 |
| IN3 | 7 |
| IN4 | 6 |
| ENB (right) | 5 |

Use GPIO numbers, not header positions. Share GND, remove ENA/ENB jumpers, disconnect motor power for install/configuration. Recommended-board confirmation selects N16R8/default pins; other boards need module/pinout confirmation. Flash detection cannot establish PSRAM or header layout. GPIO output is not enabled before first wiring confirmation; floating motor-driver inputs require disconnected power or appropriate enable pull-downs.

Custom my-wiring.json example:

```json
{"version":1,"board":"n16r8","pins":{"ena":17,"in1":16,"in2":15,"in3":7,"in4":6,"enb":5}}
```

```sh
.venv/bin/python -m kari_deploy.cli --json configure --port /dev/cu.YOUR_DEVICE --board n16r8 --profile my-wiring.json
.venv/bin/python -m kari_deploy.cli --json verify --port /dev/cu.YOUR_DEVICE --board n16r8 --profile my-wiring.json
# Maintainers may select matching built USB manifests and segments:
.venv/bin/python -m kari_deploy.cli --json install --port /dev/cu.YOUR_DEVICE --board n16r8 --profile my-wiring.json --assets ../KARI-DF26-Firmware/dist
```

First install/configure automatically uses the firmware's built-in initial credential; beginners need not create/enter a password. If a previously configured device rejects it, stop and preserve settings. Only this recovery case accepts the owner's current credential via --password-stdin. Never put personal credentials in command arguments, profiles, logs or Git. Ignored .kari-deploy/ records contain only device ID/version/wiring/verification time.

Exit codes: 0 success; 2 arguments/config/compatibility; 3 dependency/file/network; 4 USB transport; 5 rejection/readback failure. All commands emit JSON on stdout; place explicit --json before the subcommand. Verification compares the device against the downloaded version dynamically, board/memory/six GPIOs and hardware_ready=true; independent verify needs no fixed --version. Maintainers may explicitly request --release TAG (including a prerelease) or --assets DIRECTORY; failures never silently choose preview or bypass checks.

### Recovery and updates

- macOS: prefer /dev/cu.*, close Arduino Serial Monitor/other port owners.
- Windows: identify COM in Device Manager; install drivers only from the board/bridge vendor.
- Linux: use distribution serial-group permissions and log in again; no chmod 777.
- Check a data-capable cable. Hold BOOT, press/release RST, release BOOT for download mode. Native USB may re-enumerate: doctor again, identify the same physical board, then resume configure/verify instead of repeatedly flashing.
- Unknown partitions stop without writes; separately investigate/back up NVS before migration. There is no bypass option.
- Compatible official OTA retains wiring; NVS erasure requires confirmation again. GPIO constants in arbitrary custom firmware cannot be reliably recovered.
- Join the actual returned KARI-name / KARI-ID hotspot, open http://192.168.4.1/, provision Wi-Fi and verify wiring. Lift wheels for brief direction calibration; software readback is separate from real motor tests.

### Development and validation

```sh
.venv/bin/python -m unittest discover -s tests -v
```

kari_deploy/boards.json is generated from the firmware's hardware/boards.json; firmware contracts check consistency. .github/workflows/test.yml runs Python 3.10/3.12 tests and doctor on macOS/Windows/Linux. CI cannot establish real USB behavior, both module types, power/motors or OTA. Original local testing was macOS; Windows/Linux physical USB and motor acceptance remain separate from central maintainer-confirmed results.
