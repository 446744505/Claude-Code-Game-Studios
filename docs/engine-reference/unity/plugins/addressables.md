# Unity 6.3 — Addressables

**上次核对：** 2026-02-13  
**状态：** 可用于生产  
**包：** `com.unity.addressables`（Package Manager）

---

## 概述

**Addressables** 是 Unity 的高级资源管理系统，用异步加载、远程内容与更精细的内存控制替代 `Resources.Load()`。

**适合用 Addressables 的场景：**
- 异步加载资源（不阻塞主线程）
- DLC 与远程内容
- 内存优化（按需加载/卸载）
- 资源依赖管理
- 资源量很大的项目

**不适合用 Addressables 的场景：**
- 体量很小的项目（引入成本不划算）
- 启动瞬间就必须立即可用的资源（请使用直接引用）

---

## 安装

### 通过 Package Manager 安装

1. `Window > Package Manager`
2. Unity Registry > 搜索 “Addressables”
3. 安装 `Addressables`

---

## 核心概念

### 1. **可寻址资源（Addressable Assets）**
- 勾选为 “Addressable” 的资源（分配唯一键）
- 运行时可通过键加载

### 2. **资源组（Asset Groups）**
- 组织资源（例如 “UI”、“Weapons”、“Level1”）
- 组决定构建设置（本地 vs 远程）

### 3. **异步加载**
- 加载均为异步（非阻塞）
- 返回 `AsyncOperationHandle`

### 4. **引用计数**
- Addressables 会跟踪资源使用情况
- 用完后须手动释放资源

---

## 配置

### 1. 将资源标记为可寻址

1. 在 Project 窗口选中资源
2. Inspector > 勾选 “Addressable”
3. 指定键（例如 `"Enemies/Goblin"`）

**或通过脚本：**
```csharp
#if UNITY_EDITOR
using UnityEditor.AddressableAssets;
using UnityEditor.AddressableAssets.Settings;

AddressableAssetSettings.AddAssetEntry(guid, "MyAssetKey", "Default Local Group");
#endif
```

---

### 2. 创建组

`Window > Asset Management > Addressables > Groups`

- **Default Local Group**：随构建打包
- **Remote Group**：托管在服务器（CDN）

---

## 基础加载

### 异步加载资源

```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

public class AssetLoader : MonoBehaviour {
    async void Start() {
        // ✅ 异步加载资源
        AsyncOperationHandle<GameObject> handle = Addressables.LoadAssetAsync<GameObject>("Enemies/Goblin");
        await handle.Task;

        if (handle.Status == AsyncOperationStatus.Succeeded) {
            GameObject prefab = handle.Result;
            Instantiate(prefab);
        } else {
            Debug.LogError("Failed to load asset");
        }

        // ⚠️ 重要：用完后释放
        Addressables.Release(handle);
    }
}
```

---

### 加载并实例化

```csharp
async void SpawnEnemy() {
    // ✅ 一步完成加载与实例化
    AsyncOperationHandle<GameObject> handle = Addressables.InstantiateAsync("Enemies/Goblin");
    await handle.Task;

    GameObject enemy = handle.Result;
    // 使用 enemy...

    // ✅ 销毁时释放
    Addressables.ReleaseInstance(enemy);
}
```

---

### 批量加载资源

```csharp
async void LoadAllWeapons() {
    // 加载所有带 "Weapons" 标签的资源
    AsyncOperationHandle<IList<GameObject>> handle = Addressables.LoadAssetsAsync<GameObject>("Weapons", null);
    await handle.Task;

    foreach (var weapon in handle.Result) {
        Debug.Log($"Loaded: {weapon.name}");
    }

    Addressables.Release(handle);
}
```

---

## 资源标签（Labels）

### 指定标签

1. `Window > Asset Management > Addressables > Groups`
2. 选中资源 > Inspector > Labels > 添加标签（例如 `"Level1"`、`"UI"`）

### 按标签加载

```csharp
// 加载所有带 "Level1" 标签的资源
Addressables.LoadAssetsAsync<GameObject>("Level1", null);
```

---

## 远程内容（DLC）

### 配置远程组

1. 新建组：`Window > Addressables > Groups > Create New Group > Packed Assets`
2. 组设置：
   - **Build Path**：`ServerData/[BuildTarget]`
   - **Load Path**：`http://yourcdn.com/content/[BuildTarget]`

### 构建远程内容

1. `Window > Asset Management > Addressables > Build > New Build > Default Build Script`
2. 将 `ServerData/` 目录上传到 CDN
3. 游戏从远程服务器加载资源

---

## 预加载 / 缓存

### 下载依赖

```csharp
async void PreloadLevel() {
    // 下载组内全部资源但不载入内存
    AsyncOperationHandle handle = Addressables.DownloadDependenciesAsync("Level1");
    await handle.Task;

    // 此时 "Level1" 资源已缓存，后续可快速加载
    Addressables.Release(handle);
}
```

### 查询下载体积

```csharp
async void CheckDownloadSize() {
    AsyncOperationHandle<long> handle = Addressables.GetDownloadSizeAsync("Level1");
    await handle.Task;

    long sizeInBytes = handle.Result;
    Debug.Log($"Download size: {sizeInBytes / (1024 * 1024)} MB");

    Addressables.Release(handle);
}
```

---

## 内存管理

### 释放资源

```csharp
// ✅ 用完后务必释放
Addressables.Release(handle);

// ✅ 对已实例化的对象
Addressables.ReleaseInstance(gameObject);
```

### 查看引用计数

```csharp
// Addressables 使用引用计数
// 当 refCount == 0 时卸载资源
```

---

## 资源引用（Inspector 中指定）

### 使用 AssetReference

```csharp
using UnityEngine.AddressableAssets;

public class EnemySpawner : MonoBehaviour {
    // ✅ 在 Inspector 中赋值（拖拽）
    public AssetReference enemyPrefab;

    async void SpawnEnemy() {
        AsyncOperationHandle<GameObject> handle = enemyPrefab.InstantiateAsync();
        await handle.Task;

        GameObject enemy = handle.Result;
        // 使用 enemy...

        enemyPrefab.ReleaseInstance(enemy);
    }
}
```

---

## 场景

### 加载可寻址场景

```csharp
using UnityEngine.SceneManagement;

async void LoadScene() {
    AsyncOperationHandle<SceneInstance> handle = Addressables.LoadSceneAsync("MainMenu", LoadSceneMode.Additive);
    await handle.Task;

    SceneInstance sceneInstance = handle.Result;
    // 场景已加载

    // 卸载场景
    await Addressables.UnloadSceneAsync(handle).Task;
}
```

---

## 常见模式

### 懒加载（按需加载）

```csharp
Dictionary<string, AsyncOperationHandle<GameObject>> loadedAssets = new();

async Task<GameObject> GetAsset(string key) {
    if (!loadedAssets.ContainsKey(key)) {
        var handle = Addressables.LoadAssetAsync<GameObject>(key);
        await handle.Task;
        loadedAssets[key] = handle;
    }
    return loadedAssets[key].Result;
}
```

---

### 场景卸载时清理

```csharp
void OnDestroy() {
    // 释放所有 handle
    foreach (var handle in loadedAssets.Values) {
        Addressables.Release(handle);
    }
    loadedAssets.Clear();
}
```

---

## 内容目录更新（在线更新）

### 检查目录更新

```csharp
async void CheckForUpdates() {
    AsyncOperationHandle<List<string>> handle = Addressables.CheckForCatalogUpdates();
    await handle.Task;

    if (handle.Result.Count > 0) {
        Debug.Log("Updates available");
        await Addressables.UpdateCatalogs(handle.Result).Task;
    }

    Addressables.Release(handle);
}
```

---

## 性能建议

- 在启动时**预加载**常用资源
- 不用时**立即释放**资源
- 使用**标签**批量加载相关资源
- **缓存**远程内容以支持离线使用

---

## 调试

### Addressables Event Viewer

`Window > Asset Management > Addressables > Event Viewer`

- 显示所有加载/释放操作
- 各资源的内存占用
- 引用计数

### Addressables Profiler

`Window > Asset Management > Addressables > Profiler`

- 实时资源使用情况
- Bundle 加载统计

---

## 从 Resources 迁移

```csharp
// ❌ 旧：Resources.Load（同步，会卡住帧）
GameObject prefab = Resources.Load<GameObject>("Enemies/Goblin");

// ✅ 新：Addressables（异步，不阻塞）
var handle = await Addressables.LoadAssetAsync<GameObject>("Enemies/Goblin").Task;
GameObject prefab = handle.Result;
```

---

## 参考来源
- https://docs.unity3d.com/Packages/com.unity.addressables@2.0/manual/index.html
- https://learn.unity.com/tutorial/addressables
