# MCL 启动器（C# / WinForms）

Minecraft 启动器，C# 重写版：WinForms 界面 + 完整的下载 / 安装 / 启动逻辑。
界面全部自绘（渐变背景、圆角卡片、圆角渐变进度条），不依赖任何第三方库；
**项目实现只使用 C# 和 x64 汇编**，没有 Python / PowerShell 运行或构建脚本。
SHA-1 默认使用内置的 x64 汇编实现，失败时自动回退到托管 C# 实现。

- 目标框架：.NET Framework 4.x（Windows 自带，无需安装运行时）
- 编译工具：`tools\roslyn\csc.exe`（Roslyn 4.8；系统自带的 csc 只支持 C# 5，编译不了主程序）
- 构建工具：`tools\BuildTool.exe`（C# 写的构建器，见下文）
- 成品：`dist\MCL启动器.exe`（约 394 KB）+ `dist\MCL启动器-cli.exe`（同代码，控制台版）
- 测试版：`tools\BuildTool.exe -Beta` 生成 `dist\MCL启动器-beta.exe` / `MCL启动器-cli-beta.exe`，版本 `1.18.2-beta.1`，窗口标题和副标题带 **BETA / 测试版**

## 目录结构

```
MCL-cs\
  src\
    Api.cs               配置(config.json) / 日志 / JSON(JavaScriptSerializer)
    Net.cs               下载器：镜像源、Cookie-less HTTP、断点续传 .part、SHA1、多线程
    Sha1.cs              SHA-1 流式哈希（托管实现 + x64 汇编调用）
    Sha1AsmBlob.cs       BuildTool 生成的汇编机器码（默认嵌入 393 字节汇编）
    Ui.cs                自绘控件：Draw 工具、CardPanel 圆角卡片、GradientProgress 进度条
    VersionMeta.cs       版本 JSON：规则判定、继承合并、库/资源清单、Java 版本要求
    Fabric.cs            Fabric Meta：加载器列表、解析 latest、profile
    Installer.cs         安装流程：版本文件/类库/资源/原生库/模组
    Mods.cs              Modrinth 搜索与安装（含依赖，默认走 MCIM 镜像）
    CurseForge.cs        CurseForge 搜索 / 文件解析 / 下载（MCIM 镜像，免 API Key）
    ModNames.cs          模组中文名：内置常见译名 + mod_names.json 自定义
    Launcher.cs          Java 检测、命令行拼装、进程启动
    JavaManager.cs       缺失 Java 时自动下载 JDK（Corretto / Microsoft / Adoptium）并解压到 runtime\
    MicrosoftAuth.cs     Microsoft / Xbox Live / Minecraft 正版登录与 token 刷新
    LoginForm.cs         正版登录窗口（浏览器授权码流程）
    LocalScan.cs         首次使用扫描本机 .minecraft / PCL / HMCL / Prism，识别版本与模组
    LocalScanForm.cs     本机版本扫描结果窗口（支持版本隔离目录）
    MainForm.cs          主窗口
    ModForm.cs           下载 Mod 窗口
    Instances.cs         多实例（instances\<名字> + instance.json，记录加载器类型）
    SafeZip.cs           zip-slip 防护解压（存档 / 实例 / Java）
    AccountStore.cs      多账号保存：微软正版 / 离线账号（accounts.json）
    Secret.cs            DPAPI 加密 config.json / accounts.json 里的登录令牌
    AccountsForm.cs      账号管理窗口（添加、切换、删除）
    NewInstanceDialog.cs 新建实例向导
    InstanceSettingsForm.cs 单个实例的内存 / Java / JVM 参数 / 窗口设置
    SettingsForm.cs      JVM/游戏参数、窗口大小、语言、自动下载 Java
    ModManage.cs         模组元数据解析、启用/禁用、删除、检查更新
    ModCheck.cs          模组依赖缺失 / 重复 modId 检查
    ModCheckForm.cs      模组依赖检查结果窗口
    ModManageForm.cs     模组管理窗口（中文名/原名、筛选、在线按名称查找）
    TextPromptForm.cs    暗色输入框（设置模组中文名等）
    Verify.cs            完整性校验与修复
    Forge.cs             Forge / NeoForge 安装（官方安装器）
    ModPack.cs           Modrinth .mrpack 解析与安装
    CurseForgePack.cs    CurseForge manifest.json 解析（转成可安装文件列表）
    Servers.cs           服务器列表（servers.json）
    ServersForm.cs       服务器列表窗口（状态查询 / 一键进服）
    ServerPing.cs        Minecraft Server List Ping（TCP 握手 + 状态 JSON）
    Saves.cs             存档列表（解析 level.dat NBT）/ 备份 / 还原 / 复制 / 重命名 / 导入导出
    NbtWriter.cs         保留标签类型的 level.dat 写入（重命名世界）
    SavesForm.cs         存档管理窗口
    Packs.cs             资源包 / 光影列表、启用、导入、Modrinth 下载
    PacksForm.cs         资源包 / 光影管理窗口
    PlayStats.cs         游玩次数 / 总时长统计
    Screenshots.cs       游戏截图索引
    LogDiagnostics.cs    latest.log / crash-reports 常见错误诊断
    ScreenshotForm.cs    截图浏览 / 预览 / 删除 / 复制到剪贴板
    InstanceTransferForm.cs 实例 zip 导入 / 导出
    ModpackBrowserForm.cs  在线整合包市场（Modrinth / CurseForge 搜索 / 下载）
    LogStatsForm.cs      游玩统计 + 日志诊断窗口
    JavaForm.cs          Java 运行时管理 / HTTP 代理设置
    LaunchService.cs     版本解析 + 启动 + 一键进服
    Program.cs           入口 + 命令行(versions/install/launch/search/addmod/mods/verify/
                         instances/servers/pack/modctl/saves/java)
    asm\sha1_x64.asm     汇编源（NASM 语法）
    asm\sha1_x64.bin     汇编产物（393 字节裸机器码，构建时直接嵌入）
  tests\TestSha1.cs      SHA-1 与系统 SHA1 对拍 + 汇编/托管对拍 + 性能测试
  tools\BuildTool.cs     构建器源码（纯 C#）
  tools\BuildTool.exe    构建器（用系统 csc 编译一次即可）
  tools\nasm\            NASM 2.16.03（仅重新汇编时用）
  tools\roslyn\          Roslyn C# 编译器
  dist\                  编译产物
  local_versions.json    扫描到的本机版本缓存（首次启动 / 「本地版本」生成）
  accounts.json          多账号列表（微软 refresh token / 离线账号）
  mod_names.json         自定义模组中文名（可选，右键模组管理里的模组会写入）
  game\                  默认游戏目录（首次启动时创建；runtime\ 放自动下载的 Java，backups\saves 放存档备份）
```

## 构建（只用 C#）

直接运行构建器：

```bat
tools\BuildTool.exe
```

它会用 `src\asm\sha1_x64.bin` 生成 `Sha1AsmBlob.cs`，编译 GUI 和 CLI，跑 SHA-1 自测，
最后合并 `CSharp源码-单文件.cs`。

如果 `tools\BuildTool.exe` 被删了，用 Windows 自带的 C# 编译器重建它（构建器本身只有 C# 5 语法）：

```bat
%WINDIR%\Microsoft.NET\Framework64\v4.0.30319\csc.exe /nologo /out:tools\BuildTool.exe tools\BuildTool.cs
```

参数：

- `-Asm`：重新用 NASM 汇编 `src\asm\sha1_x64.asm`（NASM 不认中文路径，构建器会自动拷到 `%TEMP%\mclasm`）
- `-NoAsm`：不嵌入汇编，SHA-1 全部走托管实现
- `-NoTest`：跳过 SHA-1 自测
- `-MergeOnly`：只重新生成单文件源码

默认使用已签入的 `sha1_x64.bin`，不依赖 NASM。

## 使用

双击 `dist\MCL启动器.exe`：选游戏版本 / Fabric 加载器 / 玩家名 / 内存，点「开始游戏」，
首次会自动下载客户端、类库、资源文件并安装 Fabric API、Thinkium 模组。
如果本机没有满足版本要求的 Java，启动器会自动下载 JDK
（优先 Amazon Corretto / Microsoft OpenJDK，失败再回 Adoptium Temurin；Java 8 / 17 / 21）
解压到 `game\runtime`，不再需要手动装 Java。下载支持断点续传（HTTP Range）和三次重试。

首次使用会询问是否扫描本机已安装的 Minecraft。扫描覆盖 `.minecraft`、PCL、HMCL、
Prism / MultiMC 实例等常见目录，并把找到的版本加入版本列表（带 `[本地]` 标记），
同时统计每个版本的模组数量；PCL 的“版本隔离”目录也能识别，启动时库/资源用外层目录、
`mods` / `saves` / `options.txt` 用隔离目录。
右上角一排按钮：

- **本地版本**：重新扫描本机，选择扫描到的版本后自动填入游戏目录和版本号，可直接启动。
- **账号管理**：右上角账号按钮 / 「工具 → 账号管理…」可保存多个微软正版账号和离线账号，支持添加、切换、删除。正版登录仍走微软浏览器授权码流程，token 过期启动时自动刷新；离线账号使用标准 `OfflinePlayer` UUID。账号保存在 `accounts.json`，刷新令牌用 DPAPI 加密，公用电脑请及时删除账号。
- **安全**：存档 / 实例 / Java 压缩包统一做 zip-slip 校验；登录令牌用 Windows DPAPI（CurrentUser）加密后写入 `config.json` / `accounts.json`；PinnedGet 会校验证书域名；TLS 只启用 1.2。
- **校验修复**：逐个核对客户端 jar / 类库 / 资源对象的 SHA-1，缺失或损坏的直接重下，日志会列出前 40 条问题。
- **设置**：额外 JVM 参数、额外游戏参数（每行/每参数一个，支持引号）、窗口宽高、游戏语言（写入 `options.txt`）、缺少 Java 时自动下载、启动前自动备份存档及每个存档保留份数；**版本通道**可选正式版 / 测试版（快照）/ 全部，版本列表会按通道过滤。
- **服务器**：维护服务器列表（存 `servers.json`），自动查询在线状态、MOTD、人数、延迟和版本；记录最近进入时间和次数，列表显示「最近」列；右键可收藏，收藏的服务器排在前面；双击一行即启动游戏并直连（`--server` / `--port`）。
- **资源包**：管理 `resourcepacks` / `shaderpacks`，支持导入 zip、启用/禁用（写 `options.txt` / `optionsshaders.txt`，兼容 Iris）、删除，以及从 Modrinth 搜索下载资源包和光影。
- **工具**：截图管理（浏览/预览/打开/复制到剪贴板/删除）、实例导入导出（整目录打 zip，支持重命名/复制）、游玩统计与日志诊断（累计启动次数/时长，读取 latest.log 和 crash-reports 找内存不足、Java 版本、模组依赖、Mixin 冲突等常见原因）、Java 运行时管理（查看本机/启动器安装的 Java，安装 8/17/21，设为默认、删除）和 HTTP 代理设置。
- **模组依赖检查**：扫描 mods 目录，检查缺失前置模组、被禁用的依赖、重复 modId；Fabric API 自带的 `fabric-*` 子模块不会误报。结果可一键复制成报告。
- **存档**：列出 `saves` 下的世界（名称/最后游玩/大小/模式/版本），可单个或全部备份成 zip（存 `backups\saves`）、从备份还原；还原时同名存档会先改名 `.mcl-old-时间` 保留。右键存档可进入此存档（1.20+ 快速进入）、重命名世界、复制、导出 zip、删除，另有「导入存档…」。
- **模组管理**：列出 `mods` 里所有模组，分「中文名 / 原名」两列显示，可按中英文名、文件名筛选；内置常见模组译名（钠、鸢尾花、机械动力、农夫乐事、应用能源2 等），右键可手动「设置中文名」写入 `mod_names.json`；「联网补全中文名」会到 MC 百科查询缺失译名并缓存。搜索/更新默认走 MCIM 镜像（`mod.mcimirror.top`），失败自动回官方源；Modrinth 搜索窗口也有「补全中文名」。
- **下载 Mod…**：搜索 Modrinth 并一键安装到 `game\mods`（含必需依赖）。窗口右上角可以切换 **CurseForge**，通过 MCIM 镜像免 API Key 搜索和下载；CurseForge 文件会连同必需前置一起安装。

主界面还有两项：

- **实例**：下拉切换多套独立游戏目录（`instances\<名字>`，各有自己的存档/模组/配置），「新建…」可起新实例，「删除」整目录清理。`instance.json` 记录 MC 版本、加载器类型/版本和独立的 Java / JVM 参数 / 游戏参数 / 窗口大小；「工具 → 实例设置…」编辑，切换实例时自动应用。
- **整合包**：支持 Modrinth `.mrpack` 和 CurseForge 整合包 `.zip`；「安装整合包…」装本地文件，工具菜单「在线整合包市场…」可搜索 Modrinth / CurseForge，下载后自动补齐 MC 与加载器并安装。CurseForge 包会先解析 `manifest.json`，再到 API 取每个文件的实际下载地址（作者禁止第三方下载的文件会跳过）。

命令行（全局参数必须写在子命令前面）：

```text
.\dist\MCL启动器-cli.exe versions --release
.\dist\MCL启动器-cli.exe --game-dir "D:\MC" --workers 24 install -v 1.20.1 --fabric
.\dist\MCL启动器-cli.exe launch -v 1.20.1 --username Steve --detach
.\dist\MCL启动器-cli.exe scan                    # 扫描本机 Minecraft，写入 local_versions.json
.\dist\MCL启动器-cli.exe login                   # 交互式 Microsoft 正版登录
.\dist\MCL启动器-cli.exe login --url "https://login.live.com/oauth20_desktop.srf?code=..."
.\dist\MCL启动器-cli.exe --game-dir "D:\.minecraft" launch -v "Fabulously Optimized" --local --launch-dir "D:\.minecraft\versions\Fabulously Optimized" --dry-run
.\dist\MCL启动器-cli.exe search sodium -v 1.20.1
.\dist\MCL启动器-cli.exe addmod sodium -v 1.20.1
.\dist\MCL启动器-cli.exe cfsearch jei -v 1.20.1 --limit 5
.\dist\MCL启动器-cli.exe cfinstall 238222 -v 1.20.1 --loader fabric
.\dist\MCL启动器-cli.exe accounts offline "Player"
.\dist\MCL启动器-cli.exe accounts list
.\dist\MCL启动器-cli.exe accounts use "Player"
.\dist\MCL启动器-cli.exe mods
.\dist\MCL启动器-cli.exe verify -v 1.20.1            # 完整性校验
.\dist\MCL启动器-cli.exe verify -v 1.20.1 --fix      # 校验并修复
.\dist\MCL启动器-cli.exe install -v 1.20.1 --loader-type forge
.\dist\MCL启动器-cli.exe instances list|create|delete
.\dist\MCL启动器-cli.exe instances export "实例名" "D:\实例.zip"
.\dist\MCL启动器-cli.exe instances import "D:\实例.zip"
.\dist\MCL启动器-cli.exe instances copy "实例名"
.\dist\MCL启动器-cli.exe instances rename "旧名字" "新名字"
.\dist\MCL启动器-cli.exe modctl check
.\dist\MCL启动器-cli.exe stats
.\dist\MCL启动器-cli.exe stats reset
.\dist\MCL启动器-cli.exe servers list|add|remove|join|ping
.\dist\MCL启动器-cli.exe pack "包.mrpack"
.\dist\MCL启动器-cli.exe modctl list|update|enable|disable|delete
.\dist\MCL启动器-cli.exe packs list
.\dist\MCL启动器-cli.exe packs install faithful --kind resourcepack
.\dist\MCL启动器-cli.exe packs install complementary --kind shader
.\dist\MCL启动器-cli.exe packs enable "Faithful.zip"
.\dist\MCL启动器-cli.exe saves list
.\dist\MCL启动器-cli.exe saves backup "新的世界"        # 单个存档打包到 backups\saves
.\dist\MCL启动器-cli.exe saves backup --all
.\dist\MCL启动器-cli.exe saves backups                # 列出备份
.\dist\MCL启动器-cli.exe saves prune --keep 5         # 每个存档只保留最近 5 份
.\dist\MCL启动器-cli.exe saves rename "旧名字" "新世界名"
.\dist\MCL启动器-cli.exe saves restore "新的世界-20261026-222100.zip"
.\dist\MCL启动器-cli.exe saves copy "新的世界"
.\dist\MCL启动器-cli.exe saves export "新的世界" "D:\新的世界.zip"
.\dist\MCL启动器-cli.exe saves import "D:\新的世界.zip"
.\dist\MCL启动器-cli.exe java list                    # 查看检测到的 Java
.\dist\MCL启动器-cli.exe java install --major 17      # 下载/安装 Java 17（自动选源）
.\dist\MCL启动器-cli.exe java install --major 21 --dry-run
```

没写 `--fabric` 时，优先用 config.json 里保存的加载器，其次用该游戏目录里已装的 Fabric；
要强制原版加 `--vanilla`。

## 界面

`src\Ui.cs` 里是整套主题和自绘控件，主窗口和各个窗口共用：

- `Theme` / `ThemedForm`：统一深色配色、字体、暗色滚动条（Windows 支持时）
- `RoundedButton`：圆角按钮、hover / 按下 / 禁用状态，替代系统灰按钮
- `ThemedCheckBox`：深色自绘复选框，替代白色系统勾选框
- ComboBox 使用 Windows `DarkMode_CFD` 主题，下拉箭头不再出现白色方块
- `Draw.Round` / `Draw.GradientBackground`：圆角矩形路径、竖向渐变底色
- `CardPanel`：圆角深色卡片，替代纯色 `Panel`
- `GradientProgress`：圆角凹槽 + 绿色渐变填充 + 居中百分比文字，替代系统 `ProgressBar`
- `DarkList`：`ListView` 深色自绘（表头 + 斑马行 + 选中高亮）与 `FillColumns` 列宽自适应

几个坑记一下：容器要设 `BackColor = Color.Transparent` 才能叠在渐变上；
`Control` 派生类设透明底色前必须 `SetStyle(ControlStyles.SupportsTransparentBackColor, true)`，
否则构造时抛「控件不支持透明的背景色」。

## 关于 x64 汇编

`src\asm\sha1_x64.asm` 实现 `void sha1_transform(uint32 state[5], const uint8 block[64])`：
Win64 调用约定（rcx=state, rdx=block），`BITS 64` 纯寄存器、位置无关、无重定位，
所以 `nasm -f bin` 直接得到可以原样执行的一段机器码（393 字节）。

运行期 `Sha1Native` 用 `VirtualAlloc(..., PAGE_EXECUTE_READWRITE)` 分配可执行内存、
拷入机器码，再 `Marshal.GetDelegateForFunctionPointer` 当普通方法调用；
任何一步失败就自动回落到托管实现（`CompressManaged`），不会因为汇编不可用而报错。

默认构建会把 `sha1_x64.bin` 嵌进 exe，`Sha1Native.Available` 为 true。
自测会同时校验标准 SHA-1 结果和汇编/托管 200 块压缩函数逐块对拍；
本机实测两者速度接近（32 MB 约 130–150 ms），汇编版主要作为可选实现与性能对照，
托管实现始终保留作回退。

## 常见问题

- **需要 Java**：1.17 要 16+，1.18–1.20.4 要 17+，1.20.5+ 要 21+。启动器会自动挑当前版本要求的可用 Java；一个都找不到时会自动下载 JDK 到 `game\runtime`（Corretto / Microsoft / Adoptium 三个源自动回退），也可以在「设置」里关掉自动下载。
- **下载慢**：默认走 BMCLAPI 镜像，源在官方与镜像之间自动回退；本机有 `.gradle\caches\fabric-loom` 缓存时类库与资源会自动复用，几乎不用下载。
- **登录**：点右上角账号按钮 → 「添加微软账号」，按窗口提示在浏览器完成 Microsoft 登录，把 `oauth20_desktop.srf?code=…` 回调地址粘回来即可；之后启动会自动使用正版会话，token 过期用 refresh token 刷新。没有正版账号时保持离线模式，`--uuid` 由玩家名 MD5 生成。如果微软内置客户端 ID 失效，可在 `config.json` 的 `msa_client_id` 填自己的 Azure 应用 ID（需要 `XboxLive.signin` 权限）。
- **NASM 中文路径**：不要直接在中文路径下调用 nasm；`tools\BuildTool.exe -Asm` 会自动拷到 `%TEMP%\mclasm` 再汇编。

## 免责声明

本启动器为第三方工具，与 Mojang / Microsoft 无关；请通过官方渠道获取 Minecraft 并遵守最终用户许可协议。使用本工具产生的风险由使用者自行承担。
