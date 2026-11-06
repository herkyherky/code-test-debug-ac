# GDB 构建与验证

APK 默认使用 `prebuilt/gdb-18.1-minimal-arm64`。`prebuilt/gdb-build.json` 记录源代码和预编译件 SHA-256、configure 参数与交叉构建缓存。打包时保留上游许可并做等长路径重定位；运行时复用编译器的 libc++，不安装 Python、Guile 或它们的垃圾回收器。

普通 APK 重建仍运行 `sh build.sh`，不必重编 GDB。GNU 上游源代码、Readline 补丁和固定 Termux 提交的构建配方保存在 `third_party/`。APK 的公开测试签名没有改变。

如要从头重编 GDB 并重复原生集成测试，请使用一次性的 Linux x86_64 容器，准备 Python 3、JDK 17、make、gcc/g++、patch、tar、dpkg-deb、readelf、debugfs。测试准备阶段需要容器内 root 来建立 Android ELF 所需的临时路径；实际被测 GDB 和 C++ 程序以 UID 10000、零 capabilities 运行，不需要手机 root。

1. `sh build.sh` 下载并准备原编译工具链。
2. `python3 tools/setup_debug_build.py` 下载校验后的宿主 LLVM、QEMU、调试器依赖与测试镜像，提取并应用 GDB 的 Android 补丁。
3. `python3 tools/extract_debug_runtime.py` 提取 Google SDK 36 镜像中的 Bionic 库。
4. `python3 tools/rebuild_debugger.py` 交叉构建不含 Python/Guile 的 GDB。
5. `python3 tools/package_debugger.py`，然后 `python3 tools/build.py`。
6. `python3 tools/debug-tests/build_fixtures.py`、`python3 tools/debug-tests/make_vm.py`、`python3 tools/debug-tests/run_native_vm.py`。

最后一步使用独立 ARM64 Linux 内核，直接测试 APK 内的 GDB 与 Android 16 Bionic；不是完整的 Android 系统测试，不覆盖 Android 应用进程的 SELinux、zygote、seccomp 或厂商限制。完整输出在 `build/native-debug-results.txt`。

`tools/debug-tests/TranscriptCheck.java` 可使用随包的 `MiRecord`、`DebugTransport` 解析真实 GDB 记录，验证分块输入、嵌套变量、调用栈、停止事件与结果编号。界面和协议边界测试使用 `python3 tools/ui-tests/run_tests.py`。


## 0.5.1 实时容器测试

准备上述 ARM64 内核、Bionic、BusyBox / QEMU 并构建 APK 后：

```sh
python3 tools/debug-tests/build_fixtures.py stl
python3 tools/debug-tests/run_live_stl.py
```

`make_vm.py --interactive` 显式把原始 ttyAMA0 作为普通用户 GDB 的 stdin/stdout，串口关闭回显和行缓冲；宿主 Java 直接使用应用的 MI 传输与变量模型查询真实 GDB。容器元素通过 libc++ 字段和内存读取，不调用容器成员函数。`LiveStlCheck.java` 保留断言与完整交互记录；`StlUiTest.java` 验证刷新时复用 View、语法高亮、主题模板与隐式日志。ABI 展示器和默认 allocator 的限制见 0.5.1 验证报告。
