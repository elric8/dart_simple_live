# Update Log

## 2026-05-10 22:35 Asia/Shanghai

### 本次已处理

- 按 issue #9 的预更新方向处理了抖音直播间打开失败相关防护：`444` 返回可读提示、HTML/数据结构保护、抖音请求节流，以及直播间快速重复进入防抖。
- 直播间关闭/切房时补充了播放器停止、弹幕定时器清理、贡献榜状态清理和已关闭页面的失效判断，降低退出后继续拉流或继续播放的概率。
- `SC` 显示默认值改为关闭。
- 弹幕屏蔽新增关键词编辑、用户编辑、分平台屏蔽用户、屏蔽预设导入/导出。
- 新增旧版本外部导出工具：
  - `tools/export_legacy_settings.ps1`
  - `simple_live_app/tool/export_legacy_settings.dart`
  - 本地 release 目录内另附 `export_legacy_settings_windows.exe`
- 导出内容包括：
  - `LocalStorage` 全量设置参数；
  - 当前屏蔽词；
  - 当前屏蔽用户，并按平台分组；
  - 屏蔽预设；
  - 关注列表；
  - 关注标签；
  - 历史记录。
- GitHub Actions 改为按需构建：
  - `publish_app_dev.yaml` 的 `dev_v*` tag 自动触发时，只默认跑 iOS/macOS；
  - Android/Windows/Linux/TV release workflow 去掉 tag push 自动构建，改为 `workflow_dispatch` 手动勾选；
  - 新增 `publish_app_release_ios_manual.yml`，用于手动构建 iOS unsigned IPA；
  - Android/Windows/Linux/TV 手动 workflow 均带 `ref` 与 `upload_release`，需要 GitHub 构建时再勾选并选择 tag/分支。
- 本地 release 新目录：
  - `release/v1.12.0-issue9-local-20260510`
  - 未覆盖旧 `release/v1.12.0` 和 `release/tv_v1.7.1`。
  - 已删除临时备份 `release - 副本`。

### 本次验证

- `dart analyze tool/export_legacy_settings.dart` 通过。
- `flutter analyze` 主 App 通过。
- `flutter analyze` TV App 通过。
- `tools/export_legacy_settings.ps1` 已用本机数据目录导出 JSON，通过。
- `release/.../tools/export_legacy_settings_windows.exe` 已用本机数据目录导出 JSON，通过。
- 已确认本地构建产物存在并复制到新 release 目录：
  - Windows: `windows-x64/Release/simple_live_app.exe`
  - Android: `android/simple_live_app-1.12.0+11200-issue9-local-20260510.apk`
  - Linux: `linux-x64/bundle/simple_live_app`
  - Android TV: `android-tv/SimpleLive-TV-*-issue9-local-20260510.apk`

### 下一步待修

- 自动化测试债务：`simple_live_app` 默认 widget test 仍需要补 GetX/服务初始化；`simple_live_core` 直播 API 测试依赖真实平台接口，后续要分离为 mock/集成测试。
- 自省：本轮优先做了迁移保护和发版流程止损，贡献榜和 Windows 全屏属于可复现性更强的下一批修复；后续每次构建前应固定新 release 子目录名，避免任何旧版本目录被覆盖。

## 2026-05-11 00:20 Asia/Shanghai

### 本次已处理

- 修复直播间右侧 `关注列表` 筛选后自动跳回 `全部`：筛选状态从构建函数局部变量挪到 `LiveRoomController`，避免关注列表刷新或弹窗重建时重置。
- 再次处理 Windows 最大化后双击全屏显示异常：进入全屏前只做最大化状态恢复和等待，不再在进入全屏路径里做窗口尺寸微调，避免全屏后被后续窗口 bounds 消息拉偏。
- 修复抖音贡献榜排名全为 `1` 的兜底逻辑：抖音接口返回的 `rank` 如果异常重复为 `1`，使用列表顺序生成展示名次。B 站、斗鱼贡献榜仍使用各自接口字段；虎牙当前未实现贡献榜。
- B4 弹幕行数自适应继续补强：引入本地 `canvas_danmaku` patch，给 `DanmakuOption` 增加 `lineHeight`，弹幕轨道高度随显示区域和用户设置动态计算，并在主 App 的有效区域与实际行数计算里使用同一套行距公式。
- 预发布迁移文档新增到 `docs/pre-release-update-and-sync.md`，说明旧版本导出脚本、WebDAV 同步和微力同步 / VerySync 迁移目录的推荐用法。
- GitHub Actions 继续改为按需构建：Android/Windows/Linux/TV release workflow 使用 `workflow_dispatch` 勾选；iOS/macOS 保留手动入口；release 上传显式指定 `tag_name`，避免手动 workflow 上传到错误 ref。

### 本次准备发布

- 本地新 release 目录固定为 `release/v1.12.0-issue9-local-20260510-next`。
- 主应用预发布 tag：`v1.12.0-issue9-local-20260510-next`。
- TV 预发布 tag：`tv_v1.7.1-issue9-local-20260510-next`。
- release 说明只写本次修复和更新前迁移教程，不写下一步 TODO。

### 本次验证

- `dart analyze tool/export_legacy_settings.dart` 通过。
- `flutter analyze` 主 App 通过。
- `flutter analyze` TV App 通过。
- Windows release 构建通过：`build/windows/x64/runner/Release/simple_live_app.exe`。
- Android release APK 构建通过：`build/app/outputs/flutter-apk/app-release.apk`。构建链因 `screen_brightness_android 2.1.4` 拉取 Kotlin 2.3 元数据，已同步升级主 App Android Kotlin 插件到 `2.3.21` 并改用新的 `compilerOptions` DSL。
- Android TV release split APK 构建通过：`armeabi-v7a / arm64-v8a / x86_64`。
- Linux WSL 原生目录构建通过，生成 `deb` 与 `zip`，并复制回 release 目录。
- `export_legacy_settings_windows.exe` 已用本机数据目录导出 JSON 通过；测试输出只用于本机验证，不进入 release。

### 后续待修

- 自动化测试债务：`simple_live_app` 默认 widget test 仍需要补 GetX/服务初始化；`simple_live_core` 直播 API 测试依赖真实平台接口，后续要分离为 mock/集成测试。
- 自省：本轮 Windows 全屏修复是基于窗口状态逻辑的二次收紧，仍建议用户在真实 Windows 最大化场景再复测；若仍异常，下一步应加 Windows 专用全屏状态探针，记录 `isMaximized/isFullScreen/bounds/titlebar` 的时间序列。

## 2026-05-12 21:13 Asia/Shanghai

### 本次已处理

- 处理 issue #10：Ubuntu 24.04.4 上 Linux deb/zip 打不开的根因是旧 Linux 包在 Ubuntu 22.04 环境链接了 `libmpv.so.1`，而 Ubuntu 24.04 侧通常是 `libmpv2` / `libmpv.so.2`。Linux release workflow 已改为 Ubuntu 24.04 runner，deb 也新增 `libgtk-3-0t64 | libgtk-3-0`、`libmpv2`、`libepoxy0`、`libasound2t64 | libasound2` 运行依赖。
- 处理 iPhone Air / 灵动岛界面问题：HyperOS windowed-mode 的异常 padding 兜底现在只对 Android 生效，避免 iOS 动态岛机型被误判为异常安全区。
- 直播间底部操作栏在 iOS 上收紧 bottom inset，上限压到 16，避免 home indicator 安全区把底栏撑得过高。
- 更新发布速查文档：明确 Ubuntu 22.04 WSL 构出的 Linux 包只适合本机冒烟，面向 Ubuntu 24.04 用户的公开 Linux 包必须用 Ubuntu 24.04 / `libmpv2` 环境构建。

### 本次验证

- `flutter analyze` 主 App 通过。

### 后续待修

- Linux zip 仍不能像 deb 一样自动安装系统依赖，后续 release 说明里需要单独提示 Ubuntu 24.04 用户安装 `libmpv2` 等运行库，或评估 AppImage/Flatpak 这类更完整的 Linux 分发方式。
- 若要继续覆盖 Ubuntu 22.04 用户，需要额外维护 `libmpv.so.1` 的 Linux legacy 产物；否则后续 Linux release 默认面向 Ubuntu 24.04。

## 2026-05-14 19:30 Asia/Shanghai

### 本次已处理

- 弹幕“显示几行”改为用户选择优先：设置值允许 `0..40`，实际显示行数会按当前显示区域、字体大小、视口高度计算最大可容纳行数，并把用户选择夹在 `0..最大行数` 内。
- 设置页把“显示几行”从加减/滑条改为底部单选菜单，提供“不显示弹幕、1 行、2 行……”直到当前最大可容纳行数，避免用户选到当前画面放不下的行数。
- `0` 行现在等价于不显示弹幕：直播间不会继续追加新弹幕，设置预览和当前弹幕层会清空已有弹幕，并同步隐藏滚动、顶部、底部和高级弹幕。
- 本地 `canvas_danmaku` 补强 `hideSpecial` 的 `copyWith`，隐藏高级弹幕时会清理已有高级弹幕；轨道数初始化为 `0` 并限制为非负数，避免极端小窗口或首次布局前加入弹幕时异常。
- 行数提示文案改为中性说明，只显示当前最大可容纳行数和当前实际显示行数，不再把用户主动选择低行数视为异常。

### 本次验证

- `flutter analyze` 主 App 通过。
- `flutter analyze` 本地 `canvas_danmaku` 可运行，但仍有 6 条既有 Flutter deprecated info，未发现本次引入的错误。

### 后续待修

- `canvas_danmaku` 仍有 `withOpacity` / `opacity` deprecated info，后续可以单独替换为新版 Flutter 推荐 API，减少分析噪声。

## 2026-05-14 21:08 Asia/Shanghai

### 本次已处理

- 新增直播间全局快捷键框架：直播间内 `F` 切换播放器全屏/退出全屏，`D` 开关弹幕并在关闭时清空当前弹幕层，`Esc` 优先退出直播间全屏/小窗，桌面端不在直播间时可退出系统全屏；文本输入框聚焦时不触发这些快捷键。
- 新增“允许后台继续播放”设置，默认开启；首次迁移会读取旧 `进入后台自动暂停` 值，旧值为开启时迁移为关闭后台继续播放。关闭该设置时沿用进后台清弹幕、记录恢复状态、回前台恢复的保守行为；开启时不主动清弹幕和取消延迟弹幕。
- 新增跨平台配置包 schema v2：普通导出包含设置、关注、关注标签、观看历史、弹幕屏蔽关键词、分平台屏蔽用户和屏蔽预设；普通导出排除首次运行状态、上次直播间恢复状态、WebDAV 地址/账号/密码/最近同步时间、B 站和抖音 Cookie。
- App 内新增“配置包导入导出”入口，支持合并/覆盖导入；“其他设置”里的导入导出也改为复用同一个配置包 schema，并保留旧 `simple_live` 配置文件导入兼容。
- 局域网同步新增 `/sync/profile`，设备同步页新增“同步完整配置包”，使用同一套 v2 JSON；WebDAV 备份 zip 内新增 `SimpleLive_Profile_v2.json`，恢复时优先使用 v2，读不到时继续走旧 follow/history/blocked_word/settings 文件。
- 继续修 iOS Air 横屏安全区：横屏全屏播放器只使用左右 `viewPadding`，top/bottom 清零；直播页横屏底部操作区不再叠加 iOS bottom inset，避免顶部横幅变小和底部控制区被撑高。
- 新增实时字幕实验框架：设置项包含启用开关、模型路径、语言、字号、位置；未选择有效本机模型路径时不允许开启；播放器新增字幕 overlay；服务层预留 `LiveSubtitleEngine` 抽象接口，当前只提供预览文本，不内置模型、不下载模型、不做真实 ASR 推理。

### Issue 记录

- issue #9 当前投票统计：`F1` 两票，`C5 + E5` 一票。本轮后台继续播放覆盖 `C5/E5` 方向；`F1` TV 黑屏/有声无画列为后续优先项，本轮不混入修复。
- issue #4 最新反馈：iPhone Air 重新 build 后仍然顶部和底部 UI 不对，本轮再次收紧横屏全屏安全区策略，仍需要真实 iPhone Air/iPad Air 复测。

### 后续待修

- TV `F1`：排查 TV 黑屏但有声音，优先看 Android TV 的视频输出驱动、surface attach、硬解配置和 media_kit 渲染层。
- 实时字幕：后续接入真实 ASR 时优先评估 `sherpa_onnx`，`whisper.cpp` 作为备选；先从 Windows/macOS/Linux 实验，再验证 Android/iOS/iPadOS 音频管线可行性。
- 继续补配置包导入导出的异常文件测试，包括未知 key、旧 schema、部分字段缺失和跨平台路径字段。

## 2026-05-17 Asia/Shanghai

### 本次已处理

- 抖音弹幕 WebSocket 请求头拼写修正、统一回浏览器风格请求头，TV 端握手不再被拒（`simple_live_core/lib/src/common/web_socket_util.dart`）。
- 抖音弹幕连接读取直播页动态 cookie，并保留用户自定义 cookie 优先级（`simple_live_core/lib/src/douyin_site.dart`、`simple_live_core/lib/src/danmaku/douyin_danmaku.dart`）。
- 抖音弹幕 WebSocket 增加多个备用节点：原节点优先，握手失败再串行回落到备用节点。
- 修复抖音弹幕 ACK 数据字段名错位的问题，长连接稳定性提升。
- 直播间补充弹幕连接失败提示，TV 端连接成功只写日志，不再打扰正常观看（`simple_live_app/lib/modules/live_room/live_room_controller.dart`）。
- TV 包升级到 `1.7.3 (10703)`，`assets/tv_app_version.json` 标记 `prerelease: true`；主 App 版本本轮不动。

### 本次准备发布

- TV 预发布 tag：`tv_v1.7.3`，3 个 split APK（`arm64-v8a / armeabi-v7a / x86_64`）。
- release 说明只写本轮抖音弹幕修复与已知问题，不混入主 App 改动。

### 本次验证

- `flutter analyze` 主 App 通过、TV App 通过。
- 本地 TV 模拟器（`Android Emulator - Medium_Phone / API 36 / x86_64`）连抖音弹幕复测正常出弹幕；备用节点回落用断网注入复测。

### 后续待修

- issue 12 中的 TV 关注闪退、"直播中"数量明显偏少留到下一轮单独处理。
- 抖音弹幕断流后是否需要主动重连而非只切节点，待真实长时间观看反馈。

## 2026-05-18 Asia/Shanghai

### 本次已处理

- 主 App 关注列表刷新修复 `updatedCount` 竞争：多 worker 并发 `++` 后再判断 `>= followList.length` 不可靠，可能造成 `updating.value` 卡 `true` 或 `filterData()` 漏调；改为 `await Future.wait(workers)` 完成后统一 `filterData()` + `updating.value = false`，移除 `updatedCount` 字段（`simple_live_app/lib/services/follow_service.dart`）。
- TV App 关注刷新同步迁移到主 App 11/22 的"自动并发数 + 按平台交错"策略：用户设置为 `0` 时按 `Platform.numberOfProcessors * 2.5` clamp 到 `4..20` 自动取并发数；按 `siteId` 分组后轮询交错排队，避免单一平台串行阻塞；用 `Queue` + 固定工人池替代旧 `length ~/ threadCount` 切片（`simple_live_tv_app/lib/services/follow_user_service.dart`）。
- README 完成改动节标题更新到本轮范围，下一步计划里被本轮覆盖的 issue 12 关注刷新条目同步勾选。
- TV 端 release 资产基于本轮修复重新构建并替换上传到 `tv_v1.7.3`，版本号保持 `1.7.3 (10703)`、文件名不变；release notes 增加 5/18 重构说明。
- 主 App 单独打 `v1.12.2` 预发布（Android apk + Windows zip + Linux deb/zip），版本号 `1.12.2 (11202)`，包含 5/14 配置包 v2 / 快捷键 / 后台播放 + 5/17 抖音弹幕修复 + 5/18 follow 修复。

### 本次准备发布

- TV 预发布 tag：`tv_v1.7.3`（资产替换，tag 不变）。
- 主 App 预发布 tag：`v1.12.2`，本地 release 目录 `release/v1.12.2/`。
- 主 App Linux 包由 GitHub Actions Ubuntu 24.04 runner 构建，避免本地 WSL 22.04 链接 `libmpv.so.1` 导致 24.04 用户不可用的问题。

### 本次验证

- `flutter analyze` 主 App、TV App 通过。
- 主 App 本地大关注列表（>200 条）手动复测：`updating` 状态在所有 worker 完成后稳定回落到 `false`，刷新中途取消页面不会留 `updating=true`。
- TV App 在本地 TV 模拟器跑同一份关注列表，自动并发数与平台交错排队均生效；`x86_64` APK 安装后进入虎牙 / 抖音直播间确认弹幕能连上。
- 主 App Windows zip 双击启动 + 进入虎牙直播间冒烟通过；Android APK 在本地模拟器上启动 + 关注刷新一次确认 `updating` 状态归零。

### 后续待修

- issue 12：TV 端恢复大量关注备份后闪退、重启仍闪退，需要单独排查关注存储读写与 TV 内存上限。
- 关注刷新过程中用户切换 tag 或筛选时是否需要中断当前 worker 池，待用户反馈。
- 主 App iOS IPA 本轮不附，等贡献者或后续 macOS / Actions iOS workflow 接入。

## 2026-05-23 Asia/Shanghai

### 本轮实施计划

- issue #13：Windows/macOS 桌面端新窗口白屏只处理桌面端；目标是保留多开能力，后开的实例使用本地 Hive 数据快照启动，避免多个进程抢同一份本地数据锁导致白屏。移动端和 TV 端不处理多开。
- issue #14：远程同步房间不再依赖作者的 `sync1.nsapps.cn`；改为自建 Cloudflare Workers + Durable Objects + WebSocket 后端，仍保留“创建房间、扫码/输入房间号、手动发送关注/历史/屏蔽词/B 站账号”的 600 秒临时房间同步体验，不改成常驻云同步。
- issue #16：Android 被其他应用小窗、侧边栏或系统覆盖层短暂打断时，不再把 `inactive` 当作后台，不清空弹幕、不取消延迟弹幕；只有明确进入 `paused/hidden` 且关闭“允许后台继续播放”时才执行保守清理。
- issue #17：“开源主页”改为当前仓库 `https://github.com/June6699/dart_simple_live`，并同步更新主 App 版本元数据，避免更新检查或关于页间接读到旧 `1.12.0`。
- Android 后台播放断音：主 App 增加 Android 前台服务和媒体播放通知，播放中进入后台时提高进程保活优先级；Flutter 播放器在开始播放、停止、错误、退出房间时同步启动或关闭服务，现有“允许后台继续播放”继续作为开关。
- 字体上限：主 App 弹幕字体设置上限从 `48` 提高到 `72`，仍由视口高度和行数计算自动收紧，避免大字体溢出。
- Supabase：本轮只作为后续“云端持续同步”方案占位，不引入依赖、不写 URL/key、不建表。后续若接入，优先同步关注列表、关注标签、观看历史、屏蔽词/屏蔽用户/预设、普通设置；Cookie、WebDAV 密码等敏感项默认不同步。入口应独立于现有局域网/WebDAV/临时房间同步，同步稳定后再考虑迁移。

### 本轮验证目标

- `flutter analyze` 主 App 与 TV App 通过。
- Android 真机或模拟器验证：开启“允许后台继续播放”后，直播播放时切后台或锁屏，声音持续；回前台后视频和声音继续可用。
- Android 覆盖层验证：开启弹幕后打开系统小窗、侧边栏或其他应用浮层，弹幕不被清空，回到应用后继续显示。
- Windows 10 验证：重复打开应用可以多开，新窗口不白屏。
- 同步验证：Windows、Android、TV 创建房间失败时显示具体原因；两端连接后同步关注、历史、屏蔽词不崩溃。
- issue #17 验证：关于页版本显示等于安装包版本，“开源主页”打开当前 repo。
- 字体验证：弹幕字体可调到 `72`，大字体时弹幕行数自动收紧，页面不溢出。

### 2026-05-23 远程同步后端调整

- 确认作者的 `sync1.nsapps.cn` 不能作为本 fork 的稳定依赖：旧 SignalR 房间接口会在 `CreateRoom/JoinRoom` 阶段失败，后续发布不再使用该域名。
- 新建独立后端仓库：`C:\Files\项目\simple-live-sync-server`。
- 后端采用 Cloudflare Workers + Durable Objects + WebSocket，恢复现有“600 秒临时房间同步”，不接入数据库、不保存关注/历史/Cookie/屏蔽词内容。
- 协议改为 JSON envelope：客户端 `createRoom/joinRoom/sendFavorite/sendHistory/sendShieldWord/sendBiliAccount`，服务端 `roomCreated/roomJoined/userUpdated/*Received/ack/error/roomDestroyed`。
- 单房间最多 8 个连接；单条消息最大 1 MB；创建者断开或房间过期时销毁房间。
- 主 App 与 TV App 的远程同步服务从 `signalr_netcore` 改为 `web_socket_channel`，保留原 `SignalRService` 外部 API，减少控制器改动。
- 房间号统一为 6 位大写数字/字母，扫码和手动输入校验已同步调整。
- 主 App “其他设置”新增“同步服务地址”，可改成自部署 `ws://` / `wss://` 地址；TV 关于页显示当前同步服务地址。
- TV 远程同步二维码显示不再依赖局域网 HTTP 服务状态，创建房间成功即可显示房间二维码。

### 2026-05-23 本轮已验证

- 后端：`npm install --proxy=http://127.0.0.1:51888 --https-proxy=http://127.0.0.1:51888 --registry=https://registry.npmjs.org/` 成功。
- 后端：`npm run typecheck` 通过。
- 后端：本地 `wrangler dev --ip 127.0.0.1 --port 8787` 验证 `/health` 正常。
- 后端：两个 WebSocket 客户端本地验证创建房间、加入房间、`userUpdated.isSelf`、`sendFavorite` 转发和 `ack` 均正常。
- 主 App：`flutter pub get` 通过，旧 `signalr_netcore` 依赖从 lockfile 移除。
- TV App：`flutter pub get` 通过，旧 `signalr_netcore` 依赖从 lockfile 移除。
- 主 App：`flutter analyze` 通过。
- TV App：`flutter analyze` 通过。

### Cloudflare 待操作

- 当前本机 Wrangler 未登录，`wrangler whoami` 提示需要 `wrangler login`，因此本轮没有擅自部署线上 Worker。
- 已部署到 Cloudflare Workers：`https://simple-live-sync.3439394104.workers.dev`。
- 主 App 和 TV App 的 `SignalRService.kDefaultUrl` 已改为 `wss://simple-live-sync.3439394104.workers.dev/sync`。
- 线上 `/` 和 `/health` 已返回 `status: true`。
- 线上 WebSocket 已通过 51888 代理探针验证：创建房间、加入房间、`userUpdated.isSelf`、`sendFavorite` 转发和 `ack` 均正常。
