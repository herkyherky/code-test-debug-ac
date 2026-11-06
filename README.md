# AC — Code · Test · Debug on Android

**专为算法竞赛选手打造的 Android C++ IDE：大学水课、通勤、课间，拿出手机就能继续刷算法。**

原生离线编译、完整的 GDB 源码调试工作流、自动批量运行测试样例，以及 Codeforces 题目与样例导入。把竞赛刷题时真正需要的编辑、测试、排错和提交放进手机。

**[完整源码包](https://github.com/herkyherky/code-test-debug-ac/releases/download/v0.8.0/AC-0.8.0-source.zip) · [下载最新版 APK](https://github.com/herkyherky/code-test-debug-ac/releases/latest) · [v0.8.0 安装包](https://github.com/herkyherky/code-test-debug-ac/releases/download/v0.8.0/AC-0.8.0-arm64.apk) · [详细使用说明](USER_GUIDE.zh-CN.md) · [English](#english)**

## 为算法竞赛而做

| 需求 | AC 提供的能力 |
| --- | --- |
| 手机写 C++ | 本地 Clang 21 / LLD / libc++，支持 C++17 / C++20、`bits/stdc++.h`，编译、运行、调试无需服务器 |
| 完整 Debug 工作流 | 点击行号设置断点；继续、单步跳过、进入、跳出、暂停；调用栈、表达式求值、程序输出 |
| 看懂 STL 变量 | `vector`、`stack`、`queue`、`map`、`unordered_map` 等按元素展开，步进后自动刷新，支持嵌套与分页 |
| 自动跑样例 | 一次编译、逐组运行测试；AC / WA / TLE / RE / CE 等状态；实际输出与答案差异标红、相同内容标绿 |
| 每个文件独立样例 | `a.cpp` 和 `b.cpp` 分别关联自己的测试，类似 CPH 的工作方式；可选指定样例启动 Debug |
| CF 题目到提交 | 指定题号 / URL，导入题面与样例；带入编辑器最新代码、一键准备官网提交表单，官网最终确认 |
| 力扣函数题 | 由官网 C++ 初始模板配置独立本地测试驱动；数组、链表、树等常见参数；提交预览只含答题源码 |
| 用手机也舒服 | 紧凑 / 无顶栏模式、跟手切页、行号、缩进参考线、查找替换、代码提示、双指缩放、可重排快捷键 |
| 自由定制 | 深浅主题、自定义配色与模板、两行九列符号面板、40 种界面语言、独立错误 / 警告开关 |
| 刷题统计 | 去重题量、提交次数、年度热力图；公开主页查询或手动导入，数据来源与覆盖范围明确标注 |

> “完整 Debug”指已实现的上述源码调试工作流；不是桌面 IDE 所有扩展、终端、远程调试和任意 STL ABI 的承诺。输入在启动前通过样例提供，当前没有交互式终端。

## 快速开始

1. 从 [Releases](https://github.com/herkyherky/code-test-debug-ac/releases) 下载并安装 ARM64 APK，允许你的文件管理器安装此来源的应用。
2. 首次启动等待本地工具链解压。需要 Android 8.0+、ARM64 手机；建议预留 700 MB 可用空间用于首次安装和工作文件。
3. 在编辑器写代码，在“测试”中添加输入和预期输出，点击“运行”执行全部样例。
4. 点行号添加断点，点“调试”启动 GDB。每组样例也可单独作为 Debug 输入。
5. 想刷 CF 时，在“在线评测”启用 Codeforces，导入 `1907B` 或官网题目 URL。平台功能默认全部关闭，不影响离线 IDE。

安装包约 **78 MiB**；这不是安装后总占用，工具链首次解压后会占更多空间。无需 Termux、root、独立服务器、Node.js 或额外浏览器内核。

## Codeforces：题目、样例、本地测试、提交

**导入题号 / URL → 阅读题面 → 自动提取公开样例 → 当前编辑器写代码 → 本地一键跑样例 / Debug → 一键准备提交 → 官网确认 → 查看评测。**

官方网页登录，使用 Android System WebView 与 CookieManager。应用不记录密码，不把 Cookie 或认证令牌发送给第三方服务器。

“提交”会准备题目、语言和当前代码，**最终由用户在官网确认**；不会偷偷批量提交，也不会绕过验证码或反机器人验证。网页结构或平台规则限制自动填写时，可复制代码到官方页面完成正常操作。

| 平台 | 当前集成范围 |
| --- | --- |
| Codeforces | 公开题面 / 样例导入、官网表单填写、官方 API 评测查询、公开主页刷题统计 |
| AtCoder | 公开题面 / 样例、官网表单、提交详情；主页统计可选 AtCoder Problems 第三方数据源，需单独同意 |
| 洛谷 | 按需公开题面 / 样例、官网提交填写尝试及复制回退；评测官网查看，统计手动导入 |
| LeetCode / 力扣 | 官网阅读、用户提供模板和样例、常见函数题自动本地驱动、只复制答题代码提交；统计手动导入 |

四个平台可以独立启用。网站验证、OAuth、登录过期和公开数据限制均尊重平台机制。力扣 / 洛谷没有被包装成已实现的自动主页统计；不抓取力扣私有 GraphQL。

## 为什么保留原生工具链？

AC 走 Android ARM64 Clang + 本地 ELF 程序 + GDB 的路线。除了运行样例，也能真正停在 C++ 源码行、查看变量、展开 STL、逐步定位错误；这些能力是项目的重点。代价是工具链和头文件需要本地空间。

STL 展示器读取应用所带 libc++ 的字段 / 内存，不依赖 Python / Guile，不调用用户程序的方法。变量树有分页和深度上限；未初始化、不可读取或不支持的布局会明确提示。具体范围见 [使用说明](USER_GUIDE.zh-CN.md)。

## v0.8.0 新增

- 力扣模板分析、入口选择、独立 C++ 驱动、void 输出参数选择、可选外层无序 JSON 对比，载入代码需单独确认并支持撤销。
- 公开主页统计和提交热力图；去重 AC 题量与重复提交分开，未知日期不冒充零提交，缓存可离线阅读。
- 沿用之前的编译器和 GDB 原生资源，APK 较 0.7.1 增加约 36 KiB。

[力扣与统计操作指南](LEETCODE_AND_STATISTICS.md) · [0.8.0 验证与限制](VERIFICATION-0.8.0.md)

## 构建与架构

轻量原生 Android Java 工程；没有为了 OJ 接入捆绑 Node.js、Python 或新的浏览器内核。平台适配逻辑独立于主编辑器，统一使用现有编译、运行、测试与 Debug 模块。

```sh
unzip AC-0.8.0-source.zip
cd AC-0.8.0
sh build.sh
```

完整源码、测试及第三方构建材料通过 Release 的 `AC-0.8.0-source.zip` 提供；以下源码目录说明对应解压后的项目。

Linux x86_64 构建需要 Python 3、JDK 17（含 `jdk.compiler`）、`dpkg-deb`、`readelf` 和下载锁定依赖的网络。产物在 `deliverables/`。首次构建会生成本地测试签名；本仓库不包含已发布 APK 的签名私钥。

- `app/src/main/java/cn/gyf/cxx/`：编辑器、测试、GDB 与界面。
- `app/src/main/java/cn/gyf/cxx/oj/`：可选平台适配、力扣驱动、公开统计。
- `app/src/main/assets/oj/`：页面适配、题面渲染与本地函数驱动。
- `tools/ui-tests/`：宿主回归；`tools/oj-tests/`：网页适配测试。
- `third_party/`、锁定清单、`DEBUGGER_BUILD.md`：上游源码、补丁和构建材料。

当前保留已验证的预编译 ARM64 运行器；较早恢复材料缺少该小组件的 C 源码，不宣称其全源码可重建。应用使用 target SDK 28 和可调试构建以支持私有目录原生程序执行 / 调试；当前为侧载发行版本。

## 验证与贡献

v0.8.0 的 **128 项宿主 JUnit / Robolectric 测试通过**，另有网页 DOM / 编辑器适配测试和实际宿主 C++17 / C++20 驱动编译运行。原生编译器、GDB 和运行库与前版逐字节一致。Android 16 的原生 GDB 工作流此前已由使用者真机确认；本轮没有重新连接真机完成端到端验证。

欢迎通过 [Issues](https://github.com/herkyherky/code-test-debug-ac/issues) 提交问题、设备适配反馈和功能建议。提交错误报告时请带上系统版本、复现步骤和脱敏诊断；不要上传网站 Cookie、密码、Token 或私有代码。

首批欢迎贡献：真机兼容性测试、语言翻译、OJ 页面适配、触屏体验与更多 C++ 类型支持。请保持可选平台默认关闭，并复用现有编辑器、运行器与测试模块。

## 许可证

原创应用代码使用 [MIT](LICENSE)。LLVM / Clang / libc++、GNU GDB 及其他第三方组件分别遵循其上游许可证，不因本项目 MIT 许可而变更。完整说明见 [THIRD_PARTY_NOTICES.txt](THIRD_PARTY_NOTICES.txt)、`app/src/main/assets/licenses/` 和 `third_party/`。

## English

**AC is an Android C++ IDE built for competitive programmers. Code, test and debug from your phone between classes or on the go.**

- Native offline ARM64 Clang, C++17 / C++20, libc++ and GDB; no server or Termux required.
- Line breakpoints, step in / over / out, pause / continue, call stacks, expression evaluation and expandable STL variables.
- Batch sample tests with per-file cases, verdicts and highlighted output differences; launch existing Debug with a chosen sample.
- Import public Codeforces statements and samples, prepare official submission forms with the current code, then confirm on the official site.
- LeetCode starter-template configuration and separate local harnesses for common function problems; submit only your solution code.
- Optional public-profile statistics and heatmaps: official CF API, separately consented AtCoder Problems API, manual import for LeetCode / Luogu.
- Compact mobile UI, customizable themes / shortcut keys, find / replace and 40 UI languages (long untranslated help falls back to English).

**[Download the APK](https://github.com/herkyherky/code-test-debug-ac/releases/latest)** · Android 8.0+ · ARM64 · MIT app code; third-party licenses apply separately.

Website challenges, login limitations and platform automation rules are respected. This is an independently developed project and is not affiliated with any online judge platform.
