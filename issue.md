### 1. 项目编译的中间产物被删除，导致 coverage 分析失败，但 fuzz driver 实际上可以运行成功

```
b'Running empty-fuzzer\r\n[2025-07-01 03:21:32,104 INFO] Finding shared libraries for targets (if any).\r\n[2025-07-01 03:21:33,279 INFO] Finished finding shared libraries for targets.\r\nCoverage error, creating log file: /out/fuzzer_stats/empty-fuzzer_error.log\r\n[2025-07-01 03:21:34,135 INFO] Finding shared libraries for targets (if any).\r\n[2025-07-01 03:21:34,144 INFO] Finished finding shared libraries for targets.\r\n\x1b[0;31merror: /out/src/libutf/runetype/isalnumrune.c: No such file or directory\r\n\x1b[0m\x1b[0;31mwarning: The file \'/src/libutf/runetype/isalnumrune.c\' isn\'t covered.\r\n\x1b[0m[2025-07-01 03:21:35,031 DEBUG] Finished generating per-file code coverage summary.\r\n[2025-07-01 03:21:35,031 DEBUG] Generating file view html index file as: "/out/report/linux/file_view_index.html".\r\nTraceback (most recent call last):\r\n  File "/opt/code_coverage/coverage_utils.py", line 829, in <module>\r\n    sys.exit(Main())\r\n             ^^^^^^\r\n  File "/opt/code_coverage/coverage_utils.py", line 823, in Main\r\n    return _CmdPostProcess(args)\r\n           ^^^^^^^^^^^^^^^^^^^^^\r\n  File "/opt/code_coverage/coverage_utils.py", line 780, in _CmdPostProcess\r\n    processor.PrepareHtmlReport()\r\n  File "/opt/code_coverage/coverage_utils.py", line 577, in PrepareHtmlReport\r\n    self.GenerateFileViewHtmlIndexFile(per_file_coverage_summary,\r\n  File "/opt/code_coverage/coverage_utils.py", line 450, in GenerateFileViewHtmlIndexFile\r\n    self.GetCoverageHtmlReportPathForFile(file_path),\r\n    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^\r\n  File "/opt/code_coverage/coverage_utils.py", line 422, in GetCoverageHtmlReportPathForFile\r\n    assert os.path.isfile(\r\n           ^^^^^^^^^^^^^^^\r\nAssertionError: "/src/libutf/runetype/isalnumrune.c" is not a file.\r\n'
```
runetype/%.c 生成 → 编译为 runetype/%.o → runetype/%.c 被自动删除

---

### 2. dynamic libraries 在运行时的缺失错误：
```
/out/empty-fuzzer: error while loading shared libraries: libvulkan.so.1: cannot open shared object file: No such file or directory
error while loading shared libraries: libGLdispatch.so.0: cannot open shared object file: No such file or directory
```
   * 动态依赖的间接依赖，在编译与链接阶段不会产生缺失错误 --> 要求 LLM 执行检查命令以确认所有依赖 ldd /out/empty-fuzzer
   * 错误的转义：export LDFLAGS="${LDFLAGS:-} -Wl,-rpath,'\$ORIGIN/lib'" --> export LDFLAGS="${LDFLAGS:-} -Wl,-rpath,\$ORIGIN/lib"
  
---

### 3. oss fuzz gen 阶段编译与链接非空的 `empty-fuzzer.c` 时才会出现的错误：
`/src/vulkan-headers/include/vulkan/vulkan_hpp_macros.hpp:28:4: error: "vulkan.hpp needs at least c++ standard version 11"`

```
2025-06-26 19:56:47 [Trial ID: 01] INFO [logger.info]: (build_result.compile_error) Errors: In file included from /src/fuzzers/empty-fuzzer.c:1:

In file included from /src/vulkan-headers/include/vulkan/vulkan.hpp:11:

/src/vulkan-headers/include/vulkan/vulkan_hpp_macros.hpp:28:4: error: "vulkan.hpp needs at least c++ standard version 11"

   28 | #  error "vulkan.hpp needs at least c++ standard version 11"

      |    ^

/src/vulkan-headers/include/vulkan/vulkan_hpp_macros.hpp:35:12: fatal error: 'ciso646' file not found

   35 | #  include <ciso646>

      |            ^~~~~~~~~
```

---

### 4. LLM 在 fuzz target 构建命令中加入了 -fno-lto 标志，阻止了 Fuzz Introspector 的分析
```
DataLoaderError: No fuzzer profiles
```

---

### 5. oss fuzz gen 阶段编译 empty-fuzzer.c 时出现找不到头文件相关的错误
要求 LLM 把system libraries 相关的头文件搜索路径加入到 fuzz target 编译的命令中
