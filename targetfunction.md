---
[cardboard](https://github.com/googlevr/cardboard)

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

---

[openxr-sdk](https://github.com/khronosgroup/openxr-sdk)

1. JSON清单文件解析 CreateIfValid (https://github.com/KhronosGroup/OpenXR-SDK/blob/main/src/loader/manifest_file.cpp)

```
void ApiLayerManifestFile::CreateIfValid(ManifestFileType type, const std::string &filename, std::istream &json_stream, LibraryLocator locate_library, std::vector<std::unique_ptr<ApiLayerManifestFile>> &manifest_files)
```

2. OpenXR实例创建 xrCreateInstance（"Cyclomatic complexity": 148）(https://github.com/KhronosGroup/OpenXR-SDK/blob/main/src/loader/loader_core.cpp)

```
LOADER_EXPORT XRAPI_ATTR XrResult XRAPI_CALL xrCreateInstance(const XrInstanceCreateInfo *info, XrInstance *instance)
```

---

[smhasher](https://github.com/aappleby/smhasher)

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

---

[brltty](https://github.com/brltty/brltty)

brlapi__openConnection 涉及网络、配置解析和文件操作 (https://github.com/brltty/brltty/blob/master/Programs/brlapi_client.c) 
```
/* Function: brlapi_openConnection
 * Creates a socket to connect to BrlApi */
brlapi_fileDescriptor BRLAPI_STDCALL brlapi__openConnection(brlapi_handle_t *handle, const brlapi_connectionSettings_t *clientSettings, brlapi_connectionSettings_t *usedSettings)
```

brlapi__enterTtyModeWithPath 涉及动态内存分配和对环境变量的解析
```
/* Function : brlapi_enterTtyModeWithPath */
/* Takes control of a tty path */
int BRLAPI_STDCALL brlapi__enterTtyModeWithPath(brlapi_handle_t *handle, const int *ttys, int nttys, const char *driverName)
```

ignore_accept_keys 
```
/* Function : ignore_accept_keys */
/* Common tasks for ignoring and unignoring keys */
/* what = 0 for ignoring !0 for unignoring */
static int ignore_accept_keys(brlapi_handle_t *handle, int what, brlapi_rangeType_t r, const brlapi_keyCode_t *code, unsigned int n)
```

brlapi__doWaitForPacket 接收和分发来自服务器的所有数据包
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

---

[pthreadpool](https://github.com/google/pthreadpool)

1. pthreadpool_parallelize_4d_tile_2d_dynamic_with_uarch 未发现的复杂度: 115, 圈复杂度: 21 (https://github.com/google/pthreadpool/blob/main/src/portable-api.c)
```
PTHREADPOOL_WEAK void pthreadpool_parallelize_4d_tile_2d_dynamic_with_uarch(pthreadpool_t threadpool, pthreadpool_task_4d_tile_2d_dynamic_with_id_t function, void* context,uint32_t default_uarch_index, uint32_t max_uarch_index, size_t range_i, size_t range_j, size_t range_k, size_t range_l, size_t tile_k, size_t tile_l, uint32_t flags)
```

2. pthreadpool_create 入口函数 (https://github.com/google/pthreadpool/blob/main/src/pthreads.c)
```
PTHREADPOOL_WEAK struct pthreadpool* pthreadpool_create(size_t threads_count)
```

---

[cpu_features](https://github.com/google/cpu_features)

---

[libaddressinput](https://github.com/google/libaddressinput)

---

[libsecret](https://gitlab.gnome.org/GNOME/libsecret.git)

---
