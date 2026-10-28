# LAVA STUDIO 应用台 · 一键复刻包（通用所有 LAVA 设备）

把一台全新的 LAVA STUDIO 做成「全屏应用台」的完整流程，共 5 步。
所有步骤都以**当前最终可行版本**为准，不依赖具体软件名（应用列表由设备实时扫描，装了什么显示什么）。

| 步骤 | 内容 | 在哪操作 |
|---|---|---|
| 1 | Via 地址打开设置 → 手动开 ADB | 设备 Via + 电脑 |
| 2 | Via 生成应用台（含全部优化） | 电脑起服务，设备 Via 打开 |
| 3 | 系统自带 Apple Music 换壳成应用台 | 电脑一条命令 |
| 4 | 替代自带测试软件的方式安装其他软件 | 电脑一条命令一个 APK |
| 5 | 新装软件自动进应用台 | 无需操作，自动完成 |

---

## 第 0 步：准备（电脑只做一次，macOS / Windows 通用）

**Windows 用户注意**：所有命令都在 **Git Bash** 里执行（装 [Git for Windows](https://git-scm.com/download/win) 自带），
不要用 CMD / PowerShell。macOS 直接用「终端」。

| 依赖 | macOS | Windows |
|---|---|---|
| adb | `brew install --cask android-platform-tools` | `winget install Google.PlatformTools`（或解压 SDK platform-tools 后把目录加入 PATH） |
| Python 3.9+ | 官网 / brew 安装 | 官网安装，**勾选 Add to PATH** |
| Git Bash | —（自带终端即可） | 装 [Git for Windows](https://git-scm.com/download/win) |
| JDK 11+（第 3、4 步用） | `brew install openjdk@17` 或 Android Studio 自带 | 装 JDK（如 Adoptium），安装器会自动设置 JAVA_HOME |
| Android SDK build-tools 34 + platforms/android-34（第 3、4 步用） | Android Studio 自带 | Android Studio 自带 |

脚本会自动在这两个平台上找工具链（含 Windows 的 `.exe` / venv `Scripts` 目录 / AppData SDK 路径）；
`tools/apktool.jar` 缺失时自动下载。

设备：Android 7.0+，和电脑同一局域网，先用**数据线**（非充电线）连电脑。

---

## 第 1 步：通过 Via 地址打开设置，手动打开 ADB

LAVA 固件屏蔽了大部分设置入口，用跳转页从 Via 直达开发者选项：

1. 电脑上起跳转页服务：

   ```bash
   python3 adb_page/lava_server.py     # 端口 8000，任意路径都返回跳转页
   ```

2. 设备上打开 **Via 浏览器**，地址栏输入电脑局域网地址（如 `192.168.31.36:8000`），
   点 **「③ 直达主设置界面」**（组件名 `com.android.settings/.Settings`，实测最稳定；
   开发者选项直达按钮在本机无效，勿用）。

3. 在设置里找到**开发者选项**，**手动打开「USB 调试」**
   （如果没有开发者选项：先去「关于设备」连点「版本号」7 次解锁）。

4. 电脑上确认连接，并顺手打开网络 ADB（之后可拔线）：

   ```bash
   adb devices            # 首次如弹授权框，勾选「始终允许」
   ./setup.sh device      # 自动开 5555 → 读设备 IP → 切 WiFi ADB → 校验
   ```

> 设备重启后 5555 会关闭（无 root 不能自启）。**装一次看门狗就不用再管了**：
>
> ```bash
> ./setup.sh watchdog     # macOS：装成常驻任务（LaunchAgent，每 20 秒检查，Mac 重启后自动运行）
> ```
>
> 原理：设备 USB 线插着电脑时，看门狗（`app_portal/adb_watchdog.sh`）检测到「USB 在线 + 5555 掉线」
> 就自动执行 `adb tcpip 5555` 并回连（IP 取设备自报的 wlan0，兜底上次成功记录）。日志在同目录 `adb_watchdog.log`。
> **硬限制**：设备无 root，USB 线没插电脑时无法远程恢复（安卓安全机制，任何 App 都绕不开）。
> Windows 用户：用「任务计划程序」每 1 分钟定时运行 `app_portal/adb_watchdog.sh`（Git Bash）。
>
> 第 3 步装好壳之后就不再依赖 ADB，这条只是维护通道。

---

## 第 2 步：Via 生成应用台（含全部优化，最终效果）

1. 电脑上起应用台服务：

   ```bash
   ./setup.sh serve -d     # 后台常驻，打印 http://<电脑IP>:8900/
   ```

   首次启动自动建虚拟环境、装依赖（flask / pillow / qrcode / pyaxmlparser / numpy）、生成 10 张内置壁纸。

2. 设备 Via 打开打印出的地址，建议在 Via 设置里把主页设为该地址（通用 → 主页 → 自定义）。
   > 第 3 步装壳后，**桌面「Apple Music」图标是主入口**（免 ADB）；Via 主页作为浏览器备用通道。

**当前最终效果清单**（开箱即含，无需额外配置）：

- 全屏壁纸：必应每日 / 10 张内置 / 手机扫码上传，壁纸库网格可切换、可删上传；双层淡入淡出切换不闪黑
- **壁纸锁定**：设置里打开「锁定壁纸」后，每日一图永远不会覆盖你的选择（已修复每日壁纸误覆盖选择的问题）
- 「我的最爱」行居中置顶，长按图标 550ms 快速加/删最爱
- 设置面板排序：常用在前（我的最爱 / 强杀应用），服务器地址 / 壁纸 / 屏保依次在后
- 「全部应用」「系统设置」默认收起，点击展开 / 再点收回，焦点自动跟随
- 系统设置 11 个快捷入口（全部设置 / Wi-Fi / 蓝牙 / 应用管理 / 开发者选项 / 显示 / 声音 / 存储 / 语言和输入法 / 日期和时间 / 关于设备）
- **强杀应用**：设置里两段确认强制结束进程；运行中的应用红框高亮（按 oom 判定，缓存进程不算）
- **屏保**：等待 5/10/20/30 分钟、1 小时可选；遮罩浓度浅/中/深三档；**四款时钟样式可选**——翻页时钟、极简数字、表盘（指针）、中文文字钟，24 小时制
- 应用点击优先走壳内桥（免 ADB）→ ADB → 系统 intent 三级兜底
- 性能已针对 RK3399 级芯片优化：无 backdrop-filter / 无文字阴影 / 列表展开无高度动画 / 轮询降频
- 页面自动更新：服务端改版后设备 30 秒内自动刷新，无需手动清缓存

3. **USB 调试自动恢复**（一次性授权，装完第 3 步的壳后生效）：

   ```bash
   adb shell pm grant com.apple.android.music android.permission.WRITE_SECURE_SETTINGS
   ```

   效果：每次打开应用台，壳自动检测 USB 调试开关，**被关掉就自动打开**（实测验证，无需再进开发者选项）。
   说明：`./setup.sh shell` 装壳时会自动执行这条授权，此命令用于手动补做或老壳升级；换壳用了其他包名时，把命令里的包名替换即可。
   注意：网络 ADB（5555 端口）是固件级锁死，任何 App 无法自动恢复；装了看门狗（`./setup.sh watchdog`）后，只要 USB 线插着电脑就会自动恢复，无需手动操作。

---

## 第 3 步：用系统自带 Apple Music 换壳成应用台

LAVA 桌面是固件级固定清单，新装应用不显示图标；但桌面槽位会随包增删刷新——
用 **Apple Music 的身份**（包名 `com.apple.android.music`）装壳，桌面原图标点击即开应用台：

```bash
./setup.sh shell --pkg com.apple.android.music
```

脚本自动完成：**备份原 APK 到 `backup/`** → 写入服务地址 → 构建 APK → 签名 → 安装 → **授予 WRITE_SECURE_SETTINGS 权限**。
完成后点桌面「Apple Music」图标就是应用台，界面显示 **「壳模式 · 免ADB」**，断电重启照样能用。
授权后壳会**自动恢复被关掉的 USB 调试开关**（每次打开应用台时检测，关着就自动打开）；
注意：网络 ADB（5555 端口）固件级锁死无法自动恢复，设备重启后仍需插 USB 跑 `./setup.sh device` 或 `fix_adb.sh` 一次。

**换网络不用重装壳**：服务器地址保存在设备本地，电脑 IP 变了有两种改法——

1. 壳**连不上时自动弹出原生「改地址」界面**（输入框已预填当前地址），填新地址点「保存并重连」即可；
2. 应用台 **⚙ 设置 → 服务器地址 → 点「📡 自动获取」**，自动填入当前可达地址，再点「保存并重连」。

地址永久保存，重启不丢。

- 非交互：`./setup.sh shell --pkg com.apple.android.music --ip 192.168.31.36`
- 想顶替其他系统应用：把 `--pkg` 换成设备上已存在的包名（`adb shell pm list packages -s` 查看）
- 还原原版：`adb install -r backup/com.apple.android.music-<时间>.apk`
- 注意：部分定制桌面按包名硬编码图标名，改壳的显示名不会生效（桌面行为，不影响使用）

---

## 第 4 步：替代自带测试软件的方式安装其他软件

LAVA 桌面有看门狗：不在固件白名单里的应用，启动约 0.1 秒就被强杀。
白名单是**固定名单**（出厂测试应用等），所以安装要走「把 APK 改包名成白名单里的包名」这条路，已封装成一条命令：

```bash
./install_app.sh 某个应用.apk                      # 自动挑一个可用槽位（出厂测试应用包名）
./install_app.sh 某个应用.apk --pkg <槽位包名>      # 指定槽位
./install_app.sh 某个应用.apk --yes                # 免确认
```

脚本自动完成：

1. aapt2 读原包名
2. apktool 解包 → 补全相对类名（`".MainActivity"` 补成全名，否则改包名后必崩）→ Manifest / 资源 / smali 全局替换包名
3. 重打包 → zipalign → portal.jks 签名
4. 卸载冲突原包 → 安装 → 5 秒启动验证（活不下来会提示换槽位）
5. 清应用台图标缓存

**槽位规则**：一个槽位 = 一个固件认识、当前没装的包名。槽位用完就卸一个不用的应用腾出包名。**不要用自造包名**（如 `xxx.slot1`），必被杀。

**固件白名单全部包名**（逆向桌面 `LauncherUtil.ALL_APP` 硬编码名单，状态为 2026-10-28 实测）：

| 分类 | 包名 | 说明 / 状态 |
|---|---|---|
| 测试类槽位（实测可用） | `com.ayst.stresstest` | 压力测试，**空闲**（优酷槽位版曾用 ✅） |
| 测试类槽位（已占用） | `com.lava.spitest` | SPI 测试，已占用：影视仓 |
| 测试类槽位（已占用） | `com.lava.music.me.loop.music.test` | 循环播放测试，已占用：小白文件管理器 |
| 测试类槽位（空闲未实测） | `com.lavamusic.factory.cpufinder` | CPU 检测工具 |
| 测试类槽位（空闲未实测） | `com.lavamusic.me.screendetection` | 屏幕检测 |
| 固件应用名（本机未装，可当槽位未实测） | `com.lavamusic.me.drummer / flash / mymusic / recorder / smart / studiotrainer / home / jam / mvsearcher`、`com.lavamusic.linkphone` | 架子鼓、点歌、我的音乐、录音、智能陪练、陪练、主页（谨慎）、即兴、MV 搜索、连手机 |
| 已占用·勿动 | `com.lavamusic.me.tuner / tempo / daw / effecter / myalbum / loop / browser / appcenter / setting / studiocommunity / studiotabs / update / user / vision / activation / keyboard / launcher / guide / logcollecteur / configinit` 等 | 固件在用应用，卸了丢功能；`com.android.*` 系统包名勿用 |
| ⚠️ 实测被杀 | `com.lavamusic.me.rank`、`com.test.abc` | 勿再使用 |

**约定：以后装任何第三方应用都用第 4 步槽位方式**（`./install_app.sh xx.apk` 自动挑空槽位、自动跳过已占用；优先用空闲的测试类槽位）。

**限制**：加壳 APK（dex2c / 360 / 腾讯乐固 / **爱加密 ijiami** 等）重签名后会崩，不能用此流程；
`install_app.sh` 已内置加固检测（`assets/ijm_lib`、`libjiagu`、`libexec.so` 等特征），命中会警告并要求 `--yes` 确认。
实测案例：有谱么 APK（爱加密加固）在 RK3399 固件上必崩，改用**网页版替代**——把官网地址加进应用台「系统设置」入口，用 Via 打开即可。

普通设备（无白名单固件）直接 `adb install -r xx.apk` 即可。

---

## 第 5 步：安装的软件自动添加到应用台

无需任何操作：

- 服务后台每 30 秒扫描一次设备应用，新装的（含第 4 步装的应用）**30 秒内自动出现在应用台**，名字和图标自动抓取
- 前端每 12 秒轮询刷新；应用列表按包名+APK 路径增量缓存（`cache/apps.json`），服务重启也不丢
- 卸载的应用自动消失

---

## 常见问题

**应用台打开白屏 / 转圈**：电脑服务没起（`lsof -i:8900`），或电脑 IP 变了——**不用重装壳**：断连时壳会自动弹出改地址界面；或设置 → 服务器地址 → 「📡 自动获取」→ 保存并重连。

**设置的壁纸老被每日一图覆盖**：设置 → 壁纸 → 打开「锁定壁纸」；「恢复每日一图」会自动解锁。

**设备重启后网页版打不开应用**：5555 随重启关闭；装了看门狗（`./setup.sh watchdog`，一次性）且 USB 线插着时 20 秒内自动恢复；没装就插 USB 跑 `./setup.sh device`；或直接用壳版打开（壳不需要 ADB）。

**覆盖安装失败（签名不一致）**：`adb uninstall <包名>` 后重装；原应用可从 `backup/` 装回。

**点应用没反应**：看界面通道标识——「ADB 离线」说明走的是浏览器通道，改用壳版打开。

**电脑重启后**：手动跑一次 `./setup.sh serve -d`。

---

## 目录结构

```
lava-portal-kit/
├── setup.sh               总控脚本（serve / device / shell / watchdog / all）
├── install_app.sh         装新应用（白名单槽位方式：改包名+重签+安装+启动验证）
├── adb_page/              第 1 步的设置跳转页（lava_server.py + lava.html，端口 8000）
├── tools/apktool.jar      改包名用的反编译工具（缺失自动下载）
├── app_portal/            服务端：扫描应用、界面、壁纸、二维码上传
│   ├── deploy.sh          一键部署（建 venv、装依赖、起服务）
│   ├── server.py          Flask 服务
│   ├── app.html           单页界面（全部界面逻辑）
│   ├── fix_adb.sh         开启/修复 WiFi ADB（自动读设备 IP）
│   ├── adb_watchdog.sh    WiFi ADB 看门狗（配合 ./setup.sh watchdog 常驻）
│   └── wallpapers/        10 张内置壁纸
├── portal_shell/          壳工程（含免ADB桥 + 强杀应用桥）
│   ├── build.sh           一键构建（自动找 build-tools / android.jar / JDK）
│   ├── AndroidManifest.xml
│   ├── java/...           SplashActivity（WebView + JS 桥）
│   └── portal.jks         签名库（口令 android，重建请保留）
├── site/                  GitHub Pages 指南页（推到仓库开启 Pages 即可发布）
└── backup/                被顶替应用的原始 APK 备份（脚本自动生成）
```
