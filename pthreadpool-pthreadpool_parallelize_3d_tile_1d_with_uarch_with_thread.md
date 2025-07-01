# Bug Report: Division-by-zero in parallelize functions due to lack of input validation

## Issue Summary

A division-by-zero vulnerability has been identified in the `pthreadpool` library, leading to a Floating-Point Exception (FPE) and program termination. The bug originates in the internal utility function `divide_round_up` but is triggered by public API functions like `pthreadpool_parallelize_3d_tile_1d_with_uarch_with_thread`. The core issue is that the library does not validate fuzzer-provided numerical inputs before using them in division operations. The existing `assert(divisor != 0)` check is ineffective in release builds where `NDEBUG` is defined, leaving the library vulnerable to crash.

## Root Cause

1.  A user calls a public API function, such as `pthreadpool_parallelize_3d_tile_1d_with_uarch_with_thread`, providing `0` for a parameter that is internally used as a divisor (e.g., `tile_x` or `range_tile_x`).
2.  The library's public function passes this `0` value directly to the internal helper function `divide_round_up` as the `divisor` argument.
3.  Inside `divide_round_up`, an `assert(divisor != 0)` statement exists for debugging. However, during a standard `Release` build (configured via `-DCMAKE_BUILD_TYPE=Release`), the `NDEBUG` macro is defined. This causes the preprocessor to remove the `assert` statement entirely.
4.  With the assertion check compiled out, the function proceeds to execute a division or modulo operation (e.g., `dividend % divisor` or `dividend / divisor`) where the divisor is zero.
5.  This illegal operation triggers a hardware-level Floating-Point Exception (FPE), which is caught by the operating system, resulting in a program crash.

## Steps to Reproduce

The issue was identified using the following fuzzer:

```c
#include <stdint.h>
#include <stddef.h>
#include <pthreadpool.h>

// Dummy task function for fuzzing purposes
void dummy_task_3d_tile_1d_with_id_with_thread(void* argument, uint32_t uarch_index, size_t x, size_t y, size_t z, size_t tile_x, uint32_t thread_id) {
    // Task implementation can be empty for fuzzing
}

int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    // Initialize pthreadpool
    pthreadpool_t threadpool = pthreadpool_create(4); // Create a threadpool with 4 threads

    // Ensure data is not NULL and size is sufficient for fuzzing
    if (data == NULL || size < sizeof(uint32_t) * 2 + sizeof(size_t) * 5) {
        pthreadpool_destroy(threadpool);
        return 0;
    }

    // Extract parameters from data
    uint32_t uarch_index = *((uint32_t*)data);
    uint32_t thread_id = *((uint32_t*)(data + sizeof(uint32_t)));
    size_t x = *((size_t*)(data + sizeof(uint32_t) * 2));
    size_t y = *((size_t*)(data + sizeof(uint32_t) * 2 + sizeof(size_t)));
    size_t z = *((size_t*)(data + sizeof(uint32_t) * 2 + sizeof(size_t) * 2));
    size_t tile_x = *((size_t*)(data + sizeof(uint32_t) * 2 + sizeof(size_t) * 3));
    size_t range_tile_x = *((size_t*)(data + sizeof(uint32_t) * 2 + sizeof(size_t) * 4));

    // Call the function-under-test
    pthreadpool_parallelize_3d_tile_1d_with_uarch_with_thread(
        threadpool,
        dummy_task_3d_tile_1d_with_id_with_thread,
        NULL, // No specific argument needed for dummy task
        uarch_index,
        thread_id,
        x,
        y,
        z,
        tile_x,
        range_tile_x
    );

    // Clean up
    pthreadpool_destroy(threadpool);

    return 0;
}
```

```
// file: /src/pthreadpool/src/threadpool-utils.h

static inline size_t divide_round_up(size_t dividend, size_t divisor) {
  assert(divisor != 0); // 130
  if (dividend % divisor == 0) { // 131 
    return dividend / divisor;   // 132
  } else {
    return dividend / divisor + 1;
  }
}
```

The fuzzer triggers a crash with the following error and stack trace:

```
Run Error: AddressSanitizer: FPE on unknown address 0x55b51db283a7 (pc 0x55b51db283a7 bp 0x7ffc650a2f20 sp 0x7ffc650a2e40 T0)
SCARINESS: 10 (signal)
    #0 0x55b51db283a7 in divide_round_up /src/pthreadpool/src/threadpool-utils.h:131:16
    #1 0x55b51db283a7 in pthreadpool_parallelize_3d_tile_1d_with_uarch_with_thread_private_impl /src/pthreadpool/src/portable-api.c:4522:33
    #2 0x55b51db122b2 in LLVMFuzzerTestOneInput /src/fuzzers/empty-fuzzer.c:30:5
    #3 0x55b51d9c8de0 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #4 0x55b51d9c8605 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #5 0x55b51d9c9de5 in fuzzer::Fuzzer::MutateAndTestOne() /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:760:19
    #6 0x55b51d9cab75 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:905:5
    #7 0x55b51d9b8d6b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #8 0x55b51d9e4d92 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #9 0x7ff153747082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 5792732f783158c66fb4f3756458ca24e46e827d)
    #10 0x55b51d9ab5ed in _start (/out/empty-fuzzer+0xaf5ed)

DEDUP_TOKEN: divide_round_up--pthreadpool_parallelize_3d_tile_1d_with_uarch_with_thread_private_impl--LLVMFuzzerTestOneInput
```
