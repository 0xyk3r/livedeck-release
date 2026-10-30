# 安装说明

## 选安装版还是便携版

| | 安装版 `Setup.exe` | 便携版 `Portable.exe` |
| --- | --- | --- |
| 安装过程 | 有，可选安装位置 | 无，双击就运行 |
| 开始菜单 / 桌面图标 | 有 | 没有 |
| 数据存在哪 | `%AppData%\LiveDeck` | 可执行文件旁的 `LiveDeckData` 目录 |
| 卸载 | 控制面板里正常卸载 | 删掉 exe 和 `LiveDeckData` 目录 |
| 适合 | 自己的常用电脑 | U 盘、公用电脑、不想动系统的人 |

两个版本功能完全一样。数据目录互不干扰，所以同一台机器上装了安装版再跑便携版，两边的布局和设置是各自独立的。

**便携版迁移注意**：换位置时要把 exe 和它旁边的 `LiveDeckData` 目录**一起**移动。只搬 exe 会丢掉所有布局、设置和登录状态。

## 下载

官方地址只有一个：

    https://github.com/0xyk3r/livedeck-release/releases/latest

大陆网络直连 GitHub 下载可能很慢或失败，可以用公益加速服务，在原地址前面拼一个代理前缀即可。以 1.3.0 安装版为例（其他版本把两处版本号换掉）：

```
# 原始地址
https://github.com/0xyk3r/livedeck-release/releases/download/v1.3.0/LiveDeck-1.3.0-x64-Setup.exe

# 加速地址（在前面拼上代理域名）
https://gh-proxy.com/https://github.com/0xyk3r/livedeck-release/releases/download/v1.3.0/LiveDeck-1.3.0-x64-Setup.exe
```

可用的代理服务有 `gh-proxy.com`、`v6.gh-proxy.org`、`gh.llkk.cc`、`ghfast.top`，哪个快用哪个。这些服务由第三方免费提供，不受本项目控制，也可能随时失效。

> 用代理下载后**建议核对 SHA256**，见下一节。代理服务理论上有替换文件的能力，校验值是你确认文件完好的手段。

## 核对 SHA256

每个 Release 的说明里都列了文件的 SHA256 值。核对方法（PowerShell）：

```powershell
Get-FileHash .\LiveDeck-1.3.0-x64-Setup.exe -Algorithm SHA256
```

把输出的 `Hash` 与发布页上的值比对，一致即说明文件与开发者构建出来的完全相同。大小写不影响，PowerShell 输出大写，发布页是小写。

如果不一致，**不要运行这个文件**，换个地址重新下载。

## Windows 提示"Windows 已保护你的电脑"

本程序没有购买代码签名证书（一张证书每年数千元，个人项目暂时不做这项开销），所以 SmartScreen 会拦下未知发布者的程序。

处理方式：点提示框里的「更多信息」，再点「仍要运行」。

有些浏览器在下载阶段就会拦，需要在下载列表里选「保留」。

装完之后正常运行不会再提示。新版本因为文件变了，可能会再提示一次。

如果你对来源有顾虑，核对上一节的 SHA256 是比操作系统提示更可靠的验证方式。

## 杀毒软件报毒

个别国产杀软会对 Electron 打包的程序报"可疑行为"，这是启发式误报。常见触发点是程序会读写自己的数据目录、会发起网络请求、以及未签名。

如果确实被拦，可以把程序目录加进白名单。仍不放心的话，可以把安装包上传到 VirusTotal 之类的多引擎平台看具体是哪家报的、报的是什么。

## 系统要求

- 64 位 Windows 10（1809 及以上）或 Windows 11
- 建议 8GB 以上内存。同看的窗口越多、清晰度越高，内存和 CPU / GPU 占用越高
- 需要能访问所观看平台（斗鱼、虎牙、哔哩哔哩）的网络环境

暂无 32 位、ARM、macOS、Linux 版本。

## 卸载

**安装版**：设置 → 应用 → 已安装的应用，找到 LiveDeck 卸载。卸载不会删除 `%AppData%\LiveDeck` 里的配置，想彻底清干净就手动删掉这个目录。

**便携版**：删掉 exe 和同级的 `LiveDeckData` 目录即可，不写注册表。

## 数据目录里有什么

| 文件 | 内容 |
| --- | --- |
| `window-state.json` | 窗口大小、位置、是否最大化 |
| `app-prefs.json` | 关闭按钮行为等应用级偏好 |
| `update-state.json` | 上次检查更新的结果与公告已读状态 |
| `logs/` | 运行日志，排查问题时用 |
| Chromium Profile 相关目录 | 登录状态等浏览器数据 |

布局和大部分设置存在浏览器本地存储里，也位于上述数据目录内。

> `LiveDeckData` 目录里包含各平台的登录凭据，**不要**上传网盘或分享给他人。
