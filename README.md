# AaTempSpoof

AaTempSpoof 是一个面向 Android Root 设备的温度伪装、去温控、充电优化和性能辅助模块。项目包含 Magisk/KernelSU/APatch 模块脚本、native 守护进程源码、WebUI 控制台、默认配置，以及可选的控制中心快捷开关 APK 项目。

当前模块版本：`v13.5`

> 警告：本项目会读写 `/sys`、`/proc`、`/data/adb/modules` 等 Root 路径，并会影响设备热控、充电、电池显示和性能限制策略。请只在了解风险的设备上使用。

## 功能

### 温度伪装

- CPU、GPU、DDR/内存、电池、机身外壳和其他温度节点伪装。
- 支持固定温度和动态区间温度。
- 支持电池充电过程温度曲线伪装。
- 支持电池循环次数、电量显示等信息伪装。

### 去温控与性能

- 热控配置覆盖。
- thermal governor 处理。
- cooling device 状态清理。
- 温度墙处理，GPU/DDR 温度墙可通过开关控制。
- HW 叠加层控制。
- ColorOS/OPlus 墓碑相关策略处理。

### 充电优化

- 处理慢充、夜间充电、灭屏降速、智能充电、充电保护和热控限流等限制。
- 支持 VOOC、SVOOC、UFCS、PD/PPS 等充电场景下的状态识别与辅助处理。
- 支持模拟插拔、协议判断、定时触发和充电状态伪装。
- 支持欧加 horae 控制和充电白名单。

### 自动控制

- 模块总开关。
- 前台应用白名单。
- horae 白名单。
- 兼容模式。
- 开发者模式。
- 开机自启和 watchdog 保活。

### WebUI

- 查看模块状态、设备信息、进程状态和实时温度。
- 调节 CPU/GPU/内存/电池/其他温度伪装值。
- 配置动态温度区间、电池循环、电量显示和充电温度曲线。
- 配置充电插拔、白名单、通知、触控采样率等功能。
- 查看运行日志。

### 快捷开关 APK

- Android Quick Settings Tile 快捷开关。
- 写入 `/data/adb/modules/AAaTempSpoof/AaTempSpoof/总开关.txt` 控制模块总开关。
- 支持 Root 检测、状态同步和独立 App 页面。

## 兼容性

- Android Root 环境。
- 支持 Magisk、KernelSU、APatch 一类模块管理器。
- 设备 ABI 需要为 `arm64`。
- 安装脚本要求内核版本 `>= 5.10`。
- 需要设备存在可用的温度、电池或热控节点。
- 部分功能依赖厂商实现，尤其是 OPlus/ColorOS、VOOC/SVOOC/UFCS、horae、`touchHidlTest` 等相关能力。

不建议和其他温度伪装、去温控、充电加速类模块同时使用。

## 目录结构

```text
.
├── AaTempSpoof/                 默认配置目录
├── bin/                         native 源码目录
│   ├── AaTempSpoof/
│   │   ├── main.c               主守护进程源码
│   │   └── d1x00_data.h
│   ├── cb.c                     充电插拔/旁路辅助进程
│   ├── charge_horae_daemon.c    horae 与前台白名单守护进程
│   ├── color_tombstone.c        ColorOS/OPlus 墓碑策略处理
│   └── touch_daemon.c           触控采样率守护进程
├── webroot/                     WebUI 与资源
├── apk/AAaTempSpoofTile/        快捷开关 APK 工程
├── customize.sh                 模块安装脚本
├── service.sh                   开机服务脚本
├── post-fs-data.sh              早期挂载与服务处理脚本
├── action.sh                    模块管理器动作脚本
├── uninstall.sh                 卸载恢复脚本
└── module.prop                  模块信息
```

## 重要说明

当前仓库是源码仓库，不是可直接刷入的 release zip。

`bin/AaTempSpoof/` 在源码仓库中是主程序源码目录；真正刷入模块时，`bin/AaTempSpoof` 必须是编译后的可执行文件。其他 native 源码也需要先编译成 `cb`、`charge_horae_daemon`、`color_tombstone`、`touch_daemon` 后再放入发布包的 `bin/` 目录。

## 构建 native 二进制

在 Termux 中安装依赖：

```sh
pkg install clang ndk-sysroot
```

从仓库根目录编译：

```sh
mkdir -p build/bin

clang bin/AaTempSpoof/main.c \
  -I bin/AaTempSpoof \
  -O2 -fPIE -pie -Wall -lm \
  -o build/bin/AaTempSpoof

clang bin/cb.c -O2 -fPIE -pie -Wall -o build/bin/cb
clang bin/charge_horae_daemon.c -O2 -fPIE -pie -Wall -o build/bin/charge_horae_daemon
clang bin/color_tombstone.c -O2 -fPIE -pie -Wall -o build/bin/color_tombstone
clang bin/touch_daemon.c -O2 -fPIE -pie -Wall -o build/bin/touch_daemon

strip build/bin/AaTempSpoof build/bin/cb build/bin/charge_horae_daemon build/bin/color_tombstone build/bin/touch_daemon
```

## 构建快捷开关 APK

快捷开关项目位于：

```text
apk/AAaTempSpoofTile/
```

推荐用 Android Studio 打开该目录构建。

命令行构建需要 Android SDK、JDK 和 Gradle 环境。当前仓库中没有完整 Gradle Wrapper 可执行脚本时，可先使用本机 Gradle，或自行补齐 wrapper 后执行：

```sh
cd apk/AAaTempSpoofTile
gradle assembleDebug
```

APK 输出通常位于：

```text
apk/AAaTempSpoofTile/app/build/outputs/apk/debug/app-debug.apk
```

发布模块时可将该 APK 放到：

```text
apk/AaTempSpoof.apk
```

安装脚本会在用户选择时尝试安装它。

## 打包 release zip

一个可刷入模块包应整理为类似结构：

```text
AAaTempSpoof/
├── module.prop
├── customize.sh
├── service.sh
├── post-fs-data.sh
├── action.sh
├── uninstall.sh
├── AaTempSpoof/
├── webroot/
├── apk/
│   └── AaTempSpoof.apk          可选
└── bin/
    ├── AaTempSpoof
    ├── cb
    ├── charge_horae_daemon
    ├── color_tombstone
    └── touch_daemon
```

示例整理流程：

```sh
rm -rf build/release
mkdir -p build/release/AAaTempSpoof/bin

cp module.prop customize.sh service.sh post-fs-data.sh action.sh uninstall.sh build/release/AAaTempSpoof/
cp -r AaTempSpoof webroot build/release/AAaTempSpoof/
cp build/bin/* build/release/AAaTempSpoof/bin/

# 可选：复制快捷开关 APK
mkdir -p build/release/AAaTempSpoof/apk
cp apk/AAaTempSpoofTile/app/build/outputs/apk/debug/app-debug.apk build/release/AAaTempSpoof/apk/AaTempSpoof.apk 2>/dev/null || true

cd build/release/AAaTempSpoof
zip -r ../AAaTempSpoof-v13.5.zip .
```

生成的 `AAaTempSpoof-v13.5.zip` 才是用于模块管理器刷入的包。

## 安装与使用

1. 在模块管理器中刷入 release zip。
2. 根据安装脚本提示选择高级模式或极简模式。
3. 高级模式可通过 WebUI 自定义配置。
4. 重启设备后模块生效。
5. 日志位于：

```text
/data/adb/modules/AAaTempSpoof/AaTempSpoof/tempspoof.log
```

主守护进程支持：

```sh
su -c '/data/adb/modules/AAaTempSpoof/bin/AaTempSpoof --status'
su -c '/data/adb/modules/AAaTempSpoof/bin/AaTempSpoof --stop'
```

## 默认配置

主配置文件：

```text
AaTempSpoof/AaTempSpoof.txt
```

默认包含 CPU/GPU/内存/电池温度、动态区间、电池循环、电量显示、充电温度曲线、温度墙等配置项。WebUI 保存配置后会写入该文件。

常见运行时文件：

```text
AaTempSpoof/总开关.txt
AaTempSpoof/cb.txt
AaTempSpoof/cbpz.txt
AaTempSpoof/horae.txt
AaTempSpoof/horaebai.txt
AaTempSpoof/jzwz.txt
AaTempSpoof/mubei.txt
AaTempSpoof/touch_opt.conf
AaTempSpoof/电池温度墙.txt
```

## 卸载与恢复

模块卸载时会执行 `uninstall.sh`，脚本会尝试停止相关进程、解除挂载、恢复热控/充电/电池相关节点和系统属性。

如果设备出现异常，建议先在模块管理器中禁用或卸载模块，然后重启。

## 风险提示

- 温度伪装和去温控可能导致设备发热、耗电增加、充电异常、性能波动或硬件保护降低。
- 不同 ROM、内核和厂商节点差异较大，功能不保证在所有设备上生效。
- 修改热控和充电策略有风险，请自行承担后果。
- 开源仓库中的源码和默认配置仅供学习、研究和自行构建使用。

## License

当前仓库尚未包含明确的开源许可证文件。正式开源前建议添加 `LICENSE`，明确授权范围、免责声明和二次分发规则。
