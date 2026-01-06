# VFXSystem

## ✨ 特性

- **🚀 对象池技术** - 复用 VFX 实例，消除运行时实例化开销，减少 GC 压力
- **🔄 自动生命周期管理** - 基于 ParticleSystem 组件的智能时长计算，自动清理
- **🎯 灵活的 API** - 支持预制体引用和字符串名称两种访问方式
- **⚡ 性能优化** - 预热系统防止运行时卡顿，基于队列的高效池化
- **📦 零依赖** - 所有依赖包含在包内（SingletonDD、ObjectPool）
- **🏗️ 清晰的架构** - 门面模式隐藏复杂性，关注点分离易于维护

## 🏗️ 系统架构

VFXSystem 采用分层架构，职责清晰：

```
┌─────────────┐
│  VFXSystem  │ ← 公共 API
└──────┬──────┘
       │
┌──────▼──────────┐
│    VFXPool      │ ← 多池管理器
└──────┬──────────┘
       │
┌──────▼──────────┐     ┌─────────────────┐
│  ObjectPool<T>  │────▶│ VFXAutoRelease  │ ← 自动生命周期
└─────────────────┘     └─────────────────┘
```

### 组件职责

| 组件 | 职责 |
|------|------|
| **VFXSystem** | 高级 API 门面，向开发者隐藏对象池复杂性 |
| **VFXPool** | 管理多个对象池，支持名称和预制体双重访问 |
| **VFXAutoRelease** | 自动时长计算和自清理定时器 |
| **ObjectPool<T>** | 通用对象池基础，基于队列存储 |
| **SingletonDD** | DontDestroyOnLoad 单例基类，跨场景持久化 |

## 🚀 快速开始

### 第一步：设置

系统首次使用时自动初始化。可选地，可以添加 `VFXPool` 组件到 GameObject 并在 Inspector 中配置预设。

### 第二步：基本使用

```csharp
// 在指定位置播放 VFX
var vfxInstance = VFXSystem.Instance.PlayAt(explosionPrefab, hitPoint);

// 完全控制播放
var vfx = VFXSystem.Instance.Play(
    prefab: muzzleFlashPrefab,
    position: gunMuzzle.position,
    rotation: gunMuzzle.rotation,
    parent: gunMuzzle
);
```

### 第三步：配置预设（可选）

添加 `VFXPool` 组件到 GameObject 并配置预设 VFX 列表：
- 为字符串访问设置 VFX 名称
- 为常用特效配置预热数量
- 启用集合检查以辅助开发

## 📖 API 文档

### VFXSystem 主要方法

#### `Play()` - 完全控制 VFX 播放

```csharp
GameObject Play(
    GameObject prefab,          // 要播放的 VFX 预制体
    Vector3 position,           // 生成位置
    Quaternion rotation,        // 生成旋转
    Transform parent = null,    // 可选的父节点
    float durationOverride = -1f,  // 自定义时长（<=0 自动计算）
    bool resetScale = true      // 重置缩放为 (1,1,1)
)
```

**示例：**
```csharp
VFXSystem.Instance.Play(
    explosionPrefab,
    collision.contacts[0].point,
    Quaternion.identity,
    durationOverride: 3f
);
```

#### `PlayAt()` - 便捷方法（仅位置）

```csharp
GameObject PlayAt(
    GameObject prefab,
    Vector3 position,
    Transform parent = null,
    float durationOverride = -1f
)
```

**示例：**
```csharp
VFXSystem.Instance.PlayAt(hitEffectPrefab, hitPoint);
```

#### `Play(string)` - 按名称播放

```csharp
GameObject Play(
    string vfxName,             // 在 VFXPool 中注册的名称
    Vector3 position,
    Quaternion rotation,
    Transform parent = null,
    float durationOverride = -1f,
    bool resetScale = true
)
```

**示例：**
```csharp
// 需要在 VFXPool 中配置
VFXSystem.Instance.Play("Explosion", position, Quaternion.identity);
```

#### `Warmup()` - 预实例化 VFX 以优化性能

```csharp
void Warmup(GameObject prefab, int count)
void Warmup(string vfxName, int count)
```

**示例：**
```csharp
// 在加载界面调用
VFXSystem.Instance.Warmup(explosionPrefab, 10);
VFXSystem.Instance.Warmup("MuzzleFlash", 20);
```

#### `Release()` - 手动回收（很少需要）

```csharp
void Release(GameObject instance)
```

> **注意：** 系统通过 `VFXAutoRelease` 自动回收 VFX 实例。手动回收很少必要。

## ⚙️ 配置选项

### VFXPool Inspector 设置

| 设置 | 描述 | 默认值 |
|------|------|--------|
| `Default Pool Size` | 未注册 VFX 的默认池大小 | 20 |
| `Warmup Presets On Awake` | 启动时自动预热配置的效果 | true |
| `Pool Root` | 池化对象的父节点 | 自动创建 |

### 预设 VFX 列表

每个 `VFXConfig` 条目：

| 字段 | 描述 |
|------|------|
| `VFX Name` | 自定义标识符（为空则使用预制体名称） |
| `Prefab` | VFX 预制体引用 |
| `Prewarm Count` | 初始实例化数量 |
| `Collection Checks` | 启用防双重回收保护（推荐：true） |

## 🔧 高级特性

### 自动时长计算

系统通过分析所有 ParticleSystem 组件自动计算 VFX 时长：

```csharp
duration = main.duration + main.startLifetime.constantMax
```

这确保 VFX 完整播放后再回收，即使存在复杂的粒子生命周期。

### ParticleSystem 重启

每个 VFX 实例自动重启层级中的所有 ParticleSystem 组件：

```csharp
ps.Clear(true);              // 清除旧粒子
ps.Simulate(0f, true, true); // 重置模拟
ps.Play(true);               // 开始播放
```

这确保无论之前状态如何，VFX 播放一致。

### 双重访问模式

**按预制体引用**（灵活，无需注册）：
```csharp
VFXSystem.Instance.Play(explosionPrefab, pos, rot);
```

**按名称**（代码更简洁，需要 VFXPool 配置）：
```csharp
VFXSystem.Instance.Play("Explosion", pos, rot);
```

## ✅ 最佳实践

### 推荐做法：
- ✅ 在加载界面预热常用特效
- ✅ 对常用 VFX 使用基于名称的访问（代码更简洁）
- ✅ 开发期间启用集合检查
- ✅ 让自动回收处理清理
- ✅ 基于性能分析配置合理的池大小

### 不推荐做法：
- ❌ 直接访问 VFXPool（应使用 VFXSystem API）
- ❌ 手动销毁 VFX 实例（让池回收它们）
- ❌ 为不常用的特效创建巨大的池
- ❌ 在生产环境中未经充分测试就禁用集合检查

## 📊 性能考量

### 优势
- **消除运行时实例化开销** - 对象从池中复用
- **减少垃圾回收压力** - 无频繁分配/释放
- **维持稳定帧率** - 可预测的性能
- **内存高效** - 基于队列的存储，开销最小

### 内存使用

```
总内存 = (池大小 × VFX 预制体内存) + 活跃实例
```

- **非活跃实例**：存储在 Queue 中（已池化）
- **活跃实例**：在 HashSet 中跟踪（使用中）

### 优化建议

1. **分析池大小** - 监控 `pool.CountActive` 和 `pool.CountInactive`
2. **策略性预热** - 在加载界面而非游戏进行中预加载
3. **合理调整池大小** - 罕用 VFX 用小池，常用 VFX 用大池
4. **使用 Unity Profiler** - 检查内存使用和 GC 分配


## 📦 包内容

```
VFXSystem/
├── SingletonDD.cs       // DontDestroyOnLoad 单例基类
├── ObjectPool.cs        // 通用对象池实现
├── VFXSystem.cs         // 主 API 门面（使用这个！）
├── VFXPool.cs           // 池管理器（自动初始化）
└── VFXAutoRelease.cs    // 自动生命周期组件
```
