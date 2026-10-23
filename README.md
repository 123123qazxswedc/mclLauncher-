# MCL 启动器




## 快速开始

双击 `启动器.bat`（或 `py -3 mc_launcher.py`）打开图形界面：

1. 选择 **游戏版本**（如 `1.20.1`）与 **Fabric 加载器**（默认最新稳定版）；
2. 填写 **玩家名**（离线模式，无需正版账号）、**内存**、**Java**（默认自动挑选）；
3. 点 **开始游戏** —— 首次会自动下载客户端、库、资源（约 300 MB）后再启动。

> 下载源默认使用 BMCLAPI 国内镜像，失败会自动回落到 Mojang 官方源。
> 想用官方源，把"下载源"改成 `Mojang 官方源`。

## 功能

| 功能 | 说明 |
| --- | --- |
| 版本下载 | 从 Mojang 版本清单拉取任意正式版/快照，SHA1 校验，已存在文件跳过 |
| Fabric 支持 | 自动获取 fabric-loader profile 并与原版版本 JSON 合并（`inheritsFrom` 链） |
| 库与资源 | 按 Mojang 规则（`rules`：os/arch/features）筛选库，装配 classpath；下载 assets 对象并支持旧版 virtual 资源 |
| Native 库 | 老版本自动解压 `natives-*` jar（排除 `META-INF`、`extract.exclude`）到 `versions/<id>/natives` |
| Java 选择 | 扫描 Mojang 运行时、`JAVA_HOME`、Program Files 下的 JDK，按版本需求挑最合适的 |
| 离线登录 | 玩家名生成离线 UUID（`OfflinePlayer:<name>` 的 MD5 变体），`user_type=legacy` |
| 模组 | 可选安装 Fabric API（Modrinth 最新适配版）与随项目附带的 Thinkium 模组 |
| **下载 Mod** | 内置 Modrinth 浏览器：搜索/按当前版本与 Fabric 过滤、双击一键安装到 `mods/`，自动带上必需前置依赖（界面「下载 Mod…」按钮） |
| **本地缓存复用** | 装游戏前先在本机 `.minecraft\assets\objects`、`.gradle\caches\fabric-loom\assets\objects`、`.minecraft\libraries` 里找同样的文件直接复制，命中就不下载（可省数百 MB） |
| 实用项 | 24 线程并发下载、按字节的进度条 + 实时速度/剩余时间、实时日志（同时写入 `game/logs/mcl.log`）、取消运行、内存/玩家名配置持久化 |

## 命令行用法

```bat
py -3 mc_launcher.py versions --release           :: 列出可安装版本（* 表示本地已安装）
py -3 mc_launcher.py install -v 1.20.1 --fabric latest
py -3 mc_launcher.py launch  -v 1.20.1 --username Steve --memory 4096
py -3 mc_launcher.py launch  -v 1.20.4 --fabric none --no-mod --dry-run

py -3 mc_launcher.py search sodium -v 1.20.1   :: 在 Modrinth 搜索模组（留空看热门）
py -3 mc_launcher.py addmod sodium -v 1.20.1   :: 下载模组到 mods（含必需前置）
py -3 mc_launcher.py mods                      :: 列出 mods 目录里已安装的模组
```

常用参数：

| 参数 | 含义 |
| --- | --- |
| `--game-dir PATH` | 游戏目录（默认 `MCL/game`） |
| `--official` | 使用 Mojang 官方源（默认镜像） |
| `--fabric latest` / `none` / `0.19.5` | Fabric 加载器版本 |
| `--username NAME` | 离线玩家名 |
| `--memory 4096` | 最大内存 MB |
| `--java PATH` | 指定 java/javaw |
| `--resolution --width 1280 --height 720` | 自定义窗口 |
| `--dry-run` | 只打印启动命令与缺失文件，不启动 |
| `--detach` | 启动后立即返回，不转发游戏日志 |

## 目录结构

```
MCL/
├─ mc_launcher.py        # 入口：无参数→GUI，有参数→CLI
├─ 启动器.bat            # Windows 双击启动
├─ config.json           # 界面配置（首次运行后生成）
├─ mclaunch/
│  ├─ net.py             # 多源并发下载、SHA1 校验、进度聚合
│  ├─ version.py         # 规则求值、库/资源/参数解析、版本继承合并
│  ├─ fabric.py          # Fabric 加载器元数据
│  ├─ installer.py       # 安装流程、native 解压、模组安装
│  ├─ launcher.py        # Java 探测、启动命令拼装、进程管理
│  ├─ mods.py            # Modrinth 搜索、版本筛选、依赖解析与下载
│  ├─ modgui.py          # 「下载 Mod」窗口
│  ├─ cli.py             # 命令行界面
│  └─ gui.py             # Tkinter 图形界面
└─ game/                 # 默认游戏目录（.minecraft 结构）
   ├─ versions/  libraries/  assets/  mods/  logs/
```

## 说明与排查

- **Java 版本**：1.17 需要 Java 16+，1.18–1.20.4 需要 Java 17+，1.20.5+ 需要 Java 21+。
  启动器会自动挑选满足要求的最高优先级版本（优先精确匹配，其次最接近的更高版本）。
- **正版登录未实现**：目前只有离线模式，不要用它在在线服务器冒充他人账号。
- **下载很慢/卡住**：资源文件共约 700 MB（3598 个音效/语言对象），慢是网络问题而不是卡死 ——
  状态行会显示 `x.x MB/s · 剩余约 …`。如果本机装过官方启动器或用过 Gradle 构建 Fabric 模组，
  启动器会自动从这些缓存里直接复制对象文件（实测 3575 个文件 714 MB、约 11 秒完成）。
  也可切 `下载源` 或换更快的网络；重复安装只会补下缺的文件。
- **游戏崩溃**：查看 `game/logs/mcl.log` 与 `game/logs/latest.log`；`--dry-run` 可检查 classpath 是否完整。
- **模组未加载**：确认勾选了"安装 Fabric API"，`game/mods/` 下应同时有 `fabric-api-*.jar` 与 `thinkium-*.jar`。
- **下载 Mod 说明**：只收录「模组」（不含整合包/光影包）；`cdn.modrinth.com` 在部分网络会解析到不可达节点，
  启动器会自动改用同域名的 Cloudflare 备用 IP（`net.PINNED_HOSTS`），无需手动挂代理。
- **把 Mod 放进 mods 目录**：也可以直接从 Modrinth/CurseForge 手动下载 `.jar` 丢进 `game/mods/`，效果相同。

## 打包成 exe（可选）

