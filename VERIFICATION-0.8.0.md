# AC 0.8.0 验证记录

验证日期：2026-11-05（UTC）。基线：已保存的 AC 0.7.1 源码与 APK。应用包名 `cn.gyf.cxx`，版本 `0.8.0`，versionCode `14`，ARM64，min SDK 26，target SDK 28。

## 实际完成的验证

- 重新构建 Java / DEX / APK，SDK 35、JDK 17。APK CRC 全量检查、4 字节对齐和 v2 / v3 签名验证通过。
- 新 APK 与 0.7.1 的工具链 ZIP、GDB ZIP、调试器版本标记、直接打包的 `libc++_shared.so` 逐字节相同。签名证书 SHA-256 同为 `5c27d1d28b927082c24d9601ba407c0e78621f5e0eeec5b5722a8aaf7bc21fcb`，保留覆盖升级条件。
- 最终源码运行完整 JUnit / Robolectric 回归：**128 tests，全部通过**。含原有 101 项和新增力扣 14 项、统计 13 项。真实输出见 `verification/0.8.0/ui-test-results.txt`、`verification/0.8.0/ui-test-summary.json` 和根目录 `UI_TEST_RESULTS.txt`。
- 力扣测试实际调用宿主 g++ 编译并运行 C++17 / C++20 驱动，覆盖 vector、嵌套布尔 vector、字符串 / 转义 / Unicode、链表、树、void 输出参数、整数溢出、独立驱动持久化、提交源码边界、显式载入与撤销、JSON 比较和大输出。
- JSON 比较覆盖 12 万项完整数组及显示范围之外的差异，外层无序比较保留重复数量和内层次序。测试同时覆盖错误格式、字符串空白和非法 Unicode 转义。
- 统计测试覆盖精确官方主页解析、域名仿冒拒绝、作者匹配、ID 去重、重复 AC 去重、分页重叠、AtCoder 时间边界停滞、空响应未知、取消请求、旧主页写回拒绝、手动来源不可冒充官网核验、缓存隔离、时区、闰日与热力图位置。
- 原有编辑器、波浪诊断、括号缩进、快捷键、主题、每文件样例、存储维护、GDB MI 传输 / 变量展示和 OJ 流程的宿主回归均包含在上述 128 项中。
- 运行 `node tools/oj-tests/test_dom.cjs` 通过。覆盖已有 CF / AtCoder / 洛谷样例和 DOM、Ace / Monaco / CodeMirror / textarea、页面歧义、未知登录、语言选择失败、禁止自动点击最终提交、真实 Java 生成脚本的可信 URL / 恶意域名 / 路径变化校验、打包的 KaTeX 公式渲染。详细输出：`verification/0.8.0/oj-dom-results-0.8.0.txt`。Node 和 JSDOM 仅作为宿主测试依赖，不随 APK 分发。
- 检查模板对话框和统计页深浅主题的原生测试渲染。图片为合成测试账号和手动统计数据，不能用作真实账号或网络统计成功的证据。
- 语言目录检查通过：40 种语言 + 跟随系统；中文、英文、俄语、繁中顺序；核心控件词条与可重复序列化。新功能提供英语帮助回退；不宣称所有长说明都有 40 种人工翻译。

## 公开接口连通性

对公开示例用户名分别完成单次请求：Codeforces 官方 `user.status` 返回 HTTP 200 / 1 条记录；AtCoder Problems 第三方 API 返回 HTTP 200 / 500 条并按时间排序。仅验证当前可访问性和响应形状，未冒称完整历史导入。结果见 `verification/0.8.0/public-api-smoke-0.8.0.json`。

分页、限频调度、作者匹配、去重、覆盖范围和错误流程主要通过可控响应测试；第三方数据的完整性仍由其数据源决定。统计不读取登录 Cookie。力扣和洛谷本版没有自动主页统计查询，只能官网查看或手动导入，界面已明确标注。

## 产物与边界

安装包 `AC-0.8.0-arm64.apk`：81,572,113 字节，较 0.7.1 增加 36,940 字节（约 36.1 KiB）。SHA-256：`c34e27a8d647c235bc05a86b8962ff8719d04e4199a8166bd4bb569542af515d`。机器可读记录及原生资源散列在 `verification/0.8.0/artifact-facts-0.8.0.json`。

本次环境没有连接 Android 真机：没有重新声称实际安装升级、手机端 Clang / GDB 端到端、软键盘触摸体验、真实平台登录持久化或最终官网提交已经验证。原有 GDB 的 Android 16 真机成功来自先前用户反馈；本次保持相同原生资源，宿主测试不等于新增真机验证。没有进行跨版本手机性能基准，原生资源相同不代表所有路径性能已获实测证明。

力扣模板解析和自动驱动不是通用特殊判题器；设计题、任意自定义类型、多合法答案、浮点容差等可能需要自定义驱动，具体见 `LEETCODE_AND_STATISTICS.md`。官网 WebView、验证码、OAuth 与平台规则的既有限制继续保留，不绕过安全机制。

源码包包括完整 Java、C++ 驱动、测试、文档、构建脚本和原有预编译运行器 / 第三方源码；排除宿主 SDK、测试依赖、Node 模块、OpenCC 宿主依赖和重复生成的原生资源 ZIP。可通过 `build.sh` 按锁定依赖重建。较早版本已缺少运行器 C 源码，保留原二进制，不将此描述为该组件全源码可重建。

## 重现

```sh
sh build.sh
python3 tools/ui-tests/run_tests.py
node tools/oj-tests/test_dom.cjs
python3 tools/l10n/check_catalog.py
python3 tools/package_source.py
```

测试需要先安装项目锁定的宿主依赖；宿主测试环境准备方式沿用 `tools/ui-tests/`、`tools/oj-tests/` 内的说明。构建和依赖下载需要网络，APP 的本地编译 / 测试 / 调试仍可离线使用。
