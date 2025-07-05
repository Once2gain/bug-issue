## 目前 end-to-end 实验已经成功，感觉可以持续 Fuzzing 的函数

#### [cardboard](https://github.com/googlevr/cardboard)
Open source Cardboard SDK and samples. This SDK provides everything you need to create your own Virtual Reality (VR) experiences for Google Cardboard.

1. LoadObjFile
```
/**
 * Loads obj file from assets folder from the app.
 *
 * This sample uses the .obj format since .obj is straightforward to parse and
 * the sample is intended to be self-contained, but a real application
 * should probably use a library to load a more modern format, such as FBX or
 * glTF.
 */
bool LoadObjFile(AAssetManager* mgr, const std::string& file_name, std::vector<GLfloat>* out_vertices, std::vector<GLfloat>* out_normals, std::vector<GLfloat>* out_uv, std::vector<GLushort>* out_indices)
```

#### [openxr-sdk](https://github.com/khronosgroup/openxr-sdk)
Generated headers and sources for OpenXR loader.

1. JSON清单文件解析 CreateIfValid (https://github.com/KhronosGroup/OpenXR-SDK/blob/main/src/loader/manifest_file.cpp)

```
void ApiLayerManifestFile::CreateIfValid(ManifestFileType type, const std::string &filename, std::istream &json_stream, LibraryLocator locate_library, std::vector<std::unique_ptr<ApiLayerManifestFile>> &manifest_files)
```

2. OpenXR实例创建 xrCreateInstance（"Cyclomatic complexity": 148）(https://github.com/KhronosGroup/OpenXR-SDK/blob/main/src/loader/loader_core.cpp)

```
LOADER_EXPORT XRAPI_ATTR XrResult XRAPI_CALL xrCreateInstance(const XrInstanceCreateInfo *info, XrInstance *instance)
```

#### [smhasher](https://github.com/aappleby/smhasher)
SMHasher is a test suite designed to test the distribution, collision, and performance properties of non-cryptographic hash functions.

1. lookup3 (https://github.com/aappleby/smhasher/blob/master/src/lookup3.cpp)
```
uint32_t lookup3 ( const void * key, int length, uint32_t initval )
```

2. MurmurHash64A / MurmurHash2 (https://github.com/aappleby/smhasher/blob/master/src/MurmurHash2.cpp)
```
uint32_t MurmurHash2 ( const void * key, int len, uint32_t seed )
```
```
// MurmurHash2, 64-bit versions, by Austin Appleby

// The same caveats as 32-bit MurmurHash2 apply here - beware of alignment 
// and endian-ness issues if used across multiple platforms.

// 64-bit hash for 64-bit platforms
uint64_t MurmurHash64A ( const void * key, int len, uint64_t seed )
```

3. Hash128 (https://github.com/aappleby/smhasher/blob/master/src/Spooky.cpp)
```
// do the whole hash in one call
void SpookyHash::Hash128(const void *message, size_t length, uint64 *hash1, uint64 *hash2)
```

#### [brltty](https://github.com/brltty/brltty)
BRLTTY is a background process (daemon) providing access to the Linux/Unix console (when in text mode) for a blind person using a refreshable braille display.

1. brlapi__openConnection 涉及网络、配置解析和文件操作 (https://github.com/brltty/brltty/blob/master/Programs/brlapi_client.c) 
```
/* Function: brlapi_openConnection
 * Creates a socket to connect to BrlApi */
brlapi_fileDescriptor BRLAPI_STDCALL brlapi__openConnection(brlapi_handle_t *handle, const brlapi_connectionSettings_t *clientSettings, brlapi_connectionSettings_t *usedSettings)
```

2. brlapi__enterTtyModeWithPath 涉及动态内存分配和对环境变量的解析
```
/* Function : brlapi_enterTtyModeWithPath */
/* Takes control of a tty path */
int BRLAPI_STDCALL brlapi__enterTtyModeWithPath(brlapi_handle_t *handle, const int *ttys, int nttys, const char *driverName)
```

3. ignore_accept_keys 
```
/* Function : ignore_accept_keys */
/* Common tasks for ignoring and unignoring keys */
/* what = 0 for ignoring !0 for unignoring */
static int ignore_accept_keys(brlapi_handle_t *handle, int what, brlapi_rangeType_t r, const brlapi_keyCode_t *code, unsigned int n)
```

4. brlapi__doWaitForPacket 接收和分发来自服务器的所有数据包
```
/* brlapi_doWaitForPacket */
/* Waits for the specified type of packet: must be called with brlapi_req_mutex locked */
/* deadline can be used to stop waiting after a given date, or wait forever (NULL) */
/* If the right packet type arrives, returns its size */
/* Returns -1 if a non-fatal error is encountered */
/* Returns -2 on end of file */
/* Returns -3 if the available packet was not for us */
/* Returns -4 on timeout (if deadline is not NULL) */
/* Calls the exception handler if an exception is encountered */
static ssize_t brlapi__doWaitForPacket(brlapi_handle_t *handle, brlapi_packetType_t expectedPacketType, void *packet, size_t packetSize, struct timeval *deadline)
```

#### [pthreadpool](https://github.com/google/pthreadpool)
Portable (POSIX/Windows/Emscripten) thread pool for C/C++

1. pthreadpool_parallelize_4d_tile_2d_dynamic_with_uarch 未发现的复杂度: 115, 圈复杂度: 21 (https://github.com/google/pthreadpool/blob/main/src/portable-api.c)
```
PTHREADPOOL_WEAK void pthreadpool_parallelize_4d_tile_2d_dynamic_with_uarch(pthreadpool_t threadpool, pthreadpool_task_4d_tile_2d_dynamic_with_id_t function, void* context,uint32_t default_uarch_index, uint32_t max_uarch_index, size_t range_i, size_t range_j, size_t range_k, size_t range_l, size_t tile_k, size_t tile_l, uint32_t flags)
```

2. pthreadpool_create 入口函数 (https://github.com/google/pthreadpool/blob/main/src/pthreads.c)
```
PTHREADPOOL_WEAK struct pthreadpool* pthreadpool_create(size_t threads_count)
```

#### [libaddressinput](https://github.com/google/libaddressinput)
Google’s postal address library, powering Android and Chromium

1. FillAddress 圈复杂度 103，调用函数数量 1029 (https://github.com/google/libaddressinput/blob/master/cpp/src/address_input_helper.cc)
```
// Fill in missing components of an address as best as we can based on
// existing data. For example, for some countries only one postal code is
// valid; this would enter that one. For others, the postal code indicates
// what state should be selected. Existing data will never be overwritten.
//
// Note that the preload supplier must have had the rules for the country
// represented by this address loaded before this method is called - otherwise
// an assertion failure will result.
//
// The address should have the best language tag as returned from
// BuildComponents().
void AddressInputHelper::FillAddress(AddressData* address)
```

2. Normalize (https://github.com/google/libaddressinput/blob/master/cpp/src/address_normalizer.cc)
```
// Converts the names of different fields in the address into their canonical
// form. Should be called only when supplier->IsLoaded() returns true for
// the region code of the |address|.
void AddressNormalizer::Normalize(AddressData* address)
```

#### [libsecret](https://gitlab.gnome.org/GNOME/libsecret.git)
A GObject-based library for storing and receiving secrets.

1. secret_service_search_sync 根据一组属性在密钥服务中进行同步搜索 (https://gitlab.gnome.org/GNOME/libsecret/-/blob/main/libsecret/secret-methods.c)
```
GList *secret_service_search_sync (SecretService *service, const SecretSchema *schema, GHashTable *attributes, SecretSearchFlags flags, GCancellable *cancellable, GError **error)
```

#### [libutf](https://github.com/cls/libutf)
This is a C89 UTF-8 library, with an API compatible with that of Plan 9's libutf, but with a number of improvements

1. utfnlen 计算一个 UTF-8 字符串中包含多少个 Rune 字符 (https://github.com/cls/libutf/blob/master/utf/utflen.c)
```
size_t utfnlen(const char *s, size_t len)
```

2. charntorune 核心解析引擎，负责将一个字节序列转换为一个 Rune (https://github.com/cls/libutf/blob/master/utf/chartorune.c)
```
int charntorune(Rune *p, const char *s, size_t len)
```

## 仍未编译成功的 Google chrome 项目，准备跑新一轮的自动编译实验 new-projects

- https://chromium.googlesource.com/angle/angle
- https://github.com/google/anonymous-tokens
- https://github.com/chromium/content_analysis_sdk
- https://chromium.googlesource.com/crashpad/crashpad
- https://dawn.googlesource.com/dawn
- https://github.com/google/ink.git
- https://chromium.googlesource.com/deps/inspector_protocol/
- https://chromium.googlesource.com/codecs/libgav1/
- https://github.com/google/mediapipe
- https://gitlab.freedesktop.org/mesa/mesa
- https://chromium.googlesource.com/chromiumos/platform/minigbm
- https://github.com/google/perfetto/
- https://github.com/google/private-join-and-compute
- https://github.com/Nicoshev/rapidhash.git
- https://github.com/google/ruy
- https://github.com/google/securemessage
- https://github.com/google/shell-encryption
- https://skia.googlesource.com/skia.git
- https://github.com/tensorflow/text.git
- https://github.com/tensorflow/tflite-support
- https://github.com/google/ukey2
- https://chromium.googlesource.com/chromium/src/+archive/main/chrome/updater.tar.gz
- https://chromium.googlesource.com/external/webrtc
