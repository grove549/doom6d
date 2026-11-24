# Doom550D — Canon EOS 6D 按键适配

这个目录是 [lauris-nl/doom550d](https://github.com/lauris-nl/doom550d) 的 6D 按键适配版，并增加了模块级音量设置。改动在 `module/doom550d.c` 的输入层和 `module/doom_audio_ml.c` 的音量输出；显示、palette、音频和 FIO 仍按 6D 平台验证。

## 改动内容

- 增加 `DOOM_TARGET_6D` 编译分支，按键码取自魔灯 `platform/6D.116/gui.h`：
  - 方向/斜方向：`0x2E/0x2F/0x30/0x31/0x32/0x34/0x35/0x1C`
  - 方向统一松开：`0x36`（6D 只发这一个 UNPRESS_UDLR）
  - `SET 0x04/0x05`、`MENU 0x06`、`INFO 0x07`、`PLAY 0x0B`、`TRASH 0x0D`
  - `AF 点选择松开 0x0F`、`放大 0x12/0x13`、`Q 0x1D`
  - 滚轮 `0x00/0x01/0x02/0x03`
  - 半按/全按快门 `0x50..0x53`
- 6D 没有单独的 `UNPRESS_UP/DOWN/LEFT/RIGHT`，所以方向键按下后保持到 `UNPRESS_UDLR`；同时支持 6D 发来的斜方向事件。
- 6D 没有 zoom-out 键，因此：
  - **按住放大键（Zoom In）奔跑**；
  - **轻按 AF 点选择键切换奔跑**（防止放大键在某些拍摄模式下不发事件）；
  - **快控转盘左右 = 侧移**；
  - **快控转盘上下 = 上一/下一把武器**。
- 输入队列、暂停/存档/读档和退出时会清理方向、奔跑、侧移状态，避免粘键。

## 6D 操作表

| 6D 按键 | Doom 动作 |
| --- | --- |
| 方向键 / 多功能控制器 | 前进/后退、左右转身；支持斜方向 |
| `SET` | 开火；菜单中确认 |
| `PLAY` | 开门/按开关 |
| 按住放大键（Zoom In） | 奔跑 |
| 轻按 AF 点选择键（AF Point） | 切换奔跑（6D 无 zoom-out） |
| 快控转盘左/右 | 左/右侧移 |
| 快控转盘上/下 | 上一/下一把武器 |
| `INFO` | 自动地图；菜单中返回 |
| `MENU` | 打开/关闭 Doom 菜单（启动后 5 秒内忽略，避免误触） |
| `TRASH` 连按三次 | 打开隐藏作弊菜单 |
| `Q` | 确认退出 Doom |
| 半按/全按快门 | 模块只消费事件，不映射为 Doom 动作；避免 EOS 对焦/快门抢控制 |

> 如果 6D 上快控转盘的左右/上下与预期相反，交换 `DOOM_BGMT_WHEEL_LEFT/RIGHT` 与 `DOOM_BGMT_WHEEL_UP/DOWN` 的处理即可；如果某个拍摄模式下放大键不发事件，使用 AF 点键切换奔跑。

## 音量

在 `Games → Doom` 菜单里：

- `Volume` 可调 `1` 到 `ASIF_MAX_VOL`（6D 为 10，550D 为 5）；
- `1` 最轻，数值越大越响；
- 音量在启动 Doom 时应用，运行中修改需要先 `Q` 退出再重新启动。

Doom 自己的 `F4` 声音菜单（SFX / Music）仍然有效，控制的是 Doom 混音器里的音效和音乐比例；模块的 `Volume` 控制的是 Canon 音频输出的总音量。

## 构建

把 `module/` 放到魔灯源码树：

```sh
cp -r doom550d-6d/module magiclantern/modules/doom550d
cd magiclantern
make -C modules/doom550d DOOM_6D=1 -j4
```

输出：

```text
modules/doom550d/build/doom.mo
```

`DOOM_6D=1` 会传入 `-DDOOM_6D`，启用 6D 分支。模块构建默认是相机无关的，不一定会定义 `CONFIG_6D`，所以不要省略这个参数。若在定义了 `CONFIG_6D` 的 6D 平台构建中编译，也会自动启用。

把生成的 `doom.mo` 放到 6D SD 卡：

```text
ML/MODULES/doom.mo
```

`autoexec.bin`、`6D_116.sym` 和 `doom.mo` 必须来自同一套 6D.116 构建；**不要**把 550D 的 `doom.mo` 放到 6D。

## 已构建的产物

本目录包含一次实际编译结果：

```text
build/doom.mo   524296 bytes
build/doom.dep  883 bytes
build/doom.sym  35549 bytes
```

- 工具链：xPack GNU Arm Embedded GCC 15.2.1
- 编译命令带 `DOOM_6D=1`，反汇编符号中可确认 `doom6d_press_direction` / `doom6d_press_diagonal` 已编译进去
- `doom.mo` SHA-256：`D47CC3AB92B36F137BC0395D67A5BC2A32AE1EF48652BB8B7BCD511C82076BE6`
- 模块是 ELF32 ARM EABI（ARMv5TE），与 6D.116 的 `PLATFORM_ARCH = armv5te` 一致
- `build/doom.dep` 里 72 个未解析符号，在 6D.116 发布包的 `ML/modules/6D_116.sym` 中全部找到（0 missing）
- 同一份源码去掉 `DOOM_6D=1` 也编译通过（550D 分支没有回归）
- **已在 Canon EOS 6D 实机测试通过**：模块可加载，按键映射和游戏运行正常（用户反馈）

## 重要限制

1. **按键层已适配并实机验证**；显示、palette 和音频仍沿用上游 550D/DIGIC 4 实现。当前 6D.116 + 本模块的组合实测可用，但更换 ML 版本、固件或相机平台后需要重新验证。
2. 6D 的音频符号在 6D.116 的 `.sym` 中全部存在；当前构建实测有声音。若换用其它 ML 构建，仍需确认 `doom.dep` 中的音频符号没有缺失。
3. 6D 的 `BGMT_AFPAT_UNPRESS 0x0F` 也被魔灯的“方向键模式” tweak 使用；Doom 运行时会消费它，退出 Doom 后恢复正常。
4. 6D 只发 `BGMT_UNPRESS_UDLR`，如果某固件版本在菜单/回放模式下不发这个事件，方向可能保持按下；按 `Q`/`MENU` 退出或重新进入 Doom 会清理。
5. 现有 6D 的 3× 放大/数字变焦与 Doom 的奔跑键冲突：Doom 运行期间按放大键只作为奔跑，不执行 Canon 放大。退出 Doom 后恢复。

## 实机验证

- 相机：Canon EOS 6D
- ML：6D.116
- 模块：[`build/doom.mo`](build/doom.mo)
- 结果：**模块正常加载，按键映射与游戏运行正常**（用户实机反馈）
- 构建 SHA-256：`D47CC3AB92B36F137BC0395D67A5BC2A32AE1EF48652BB8B7BCD511C82076BE6`

如果后续修改代码或更换 ML 构建，请重新跑下面的测试清单。

## 测试清单

> 实机整体已通过；下面保留为逐项回归检查。

- [ ] 四个方向各按/放 20 次，确认不粘键；
- [ ] 八方向斜按，确认不会同时卡住两个正交方向；
- [ ] 按住放大键奔跑、松开停止；
- [ ] 轻按 AF 点键切换奔跑，再按取消；
- [ ] 快控转盘左右侧移、上下切武器；
- [ ] `SET` 开火/菜单确认，`PLAY` 开门，`INFO` 自动地图/返回；
- [ ] `MENU` 开关 Doom 菜单，`TRASH` 三连开作弊菜单，`Q` 退出确认；
- [ ] 存档、读档、退出再进后输入状态正常；
- [ ] 半按/全按快门不会卡住开火键，也不应触发 Doom 动作；
- [ ] 开启 Debug logging 后查看 `ML/LOGS/DOOMRAW.LOG`，确认 6D 实际发出的按键码与 `platform/6D.116/gui.h` 一致。

## 补丁

只改 `module/doom550d.c` 的补丁：

```sh
patch -p1 < doom550d-6d-keys.patch
```

`module/Makefile` 另外加了一个可选的 `DOOM_6D=1` 编译开关；如果不用该开关，也可以手动在 CFLAGS 里加 `-DDOOM_6D`。

## 许可

与上游一致：GPL-2.0。Doom 引擎与 WAD 数据的版权/许可遵循上游说明。
