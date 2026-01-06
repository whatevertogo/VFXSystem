# VFXSystem

[![Unity](https://img.shields.io/badge/Unity-2020.3%2B-black?logo=unity)](https://unity.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A lightweight, high-performance visual effects management system for Unity that uses intelligent object pooling to optimize VFX playback performance.

## ✨ Features

- **🚀 Object Pooling** - Reuse VFX instances to eliminate runtime instantiation overhead and reduce GC pressure
- **🔄 Automatic Lifecycle** - Self-cleaning VFX with intelligent auto-duration calculation based on ParticleSystem components
- **🎯 Flexible API** - Both prefab-reference and string-based access patterns for different use cases
- **⚡ Performance Optimized** - Prewarming system to prevent runtime hitches, Queue-based pooling for efficiency
- **📦 Zero Dependencies** - All dependencies included in the package (SingletonDD, ObjectPool)
- **🏗️ Clean Architecture** - Facade pattern hides complexity, separation of concerns for maintainability

## 🏗️ Architecture

VFXSystem follows a layered architecture with clear separation of concerns:

```
┌─────────────┐
│  VFXSystem  │ ← Public API (Facade)
└──────┬──────┘
       │
┌──────▼──────────┐
│    VFXPool      │ ← Multi-Pool Manager
└──────┬──────────┘
       │
┌──────▼──────────┐     ┌─────────────────┐
│  ObjectPool<T>  │────▶│ VFXAutoRelease  │ ← Auto Lifecycle
└─────────────────┘     └─────────────────┘
```

### Component Responsibilities

| Component | Responsibility |
|-----------|---------------|
| **VFXSystem** | High-level API facade that hides pooling complexity from developers |
| **VFXPool** | Manages multiple object pools with dual access (name & prefab) |
| **VFXAutoRelease** | Automatic duration calculation and self-cleanup timer |
| **ObjectPool<T>** | Generic pooling foundation with Queue-based storage |
| **SingletonDD** | DontDestroyOnLoad singleton base for scene persistence |

## 🚀 Quick Start

### Step 1: Setup

The system auto-initializes on first use. Optionally, add a `VFXPool` component to a GameObject and configure presets in the Inspector.

### Step 2: Basic Usage

```csharp
// Play VFX at position
var vfxInstance = VFXSystem.Instance.PlayAt(explosionPrefab, hitPoint);

// Play with full control
var vfx = VFXSystem.Instance.Play(
    prefab: muzzleFlashPrefab,
    position: gunMuzzle.position,
    rotation: gunMuzzle.rotation,
    parent: gunMuzzle
);
```

### Step 3: Configure Presets (Optional)

Add a `VFXPool` component to a GameObject and configure the preset VFX list:
- Set VFX names for string-based access
- Configure prewarm counts for frequently used effects
- Enable collection checks for development

## 📖 API Documentation

### VFXSystem.Main Methods

#### `Play()` - Full control over VFX playback

```csharp
GameObject Play(
    GameObject prefab,          // VFX prefab to play
    Vector3 position,           // Spawn position
    Quaternion rotation,        // Spawn rotation
    Transform parent = null,    // Optional parent transform
    float durationOverride = -1f,  // Custom duration (<=0 for auto-calculation)
    bool resetScale = true      // Reset scale to (1,1,1)
)
```

**Example:**
```csharp
VFXSystem.Instance.Play(
    explosionPrefab,
    collision.contacts[0].point,
    Quaternion.identity,
    durationOverride: 3f
);
```

#### `PlayAt()` - Convenience method for position-only

```csharp
GameObject PlayAt(
    GameObject prefab,
    Vector3 position,
    Transform parent = null,
    float durationOverride = -1f
)
```

**Example:**
```csharp
VFXSystem.Instance.PlayAt(hitEffectPrefab, hitPoint);
```

#### `Play(string)` - Play by registered name

```csharp
GameObject Play(
    string vfxName,             // Name registered in VFXPool
    Vector3 position,
    Quaternion rotation,
    Transform parent = null,
    float durationOverride = -1f,
    bool resetScale = true
)
```

**Example:**
```csharp
// Requires VFXPool configuration
VFXSystem.Instance.Play("Explosion", position, Quaternion.identity);
```

#### `Warmup()` - Pre-instantiate VFX for performance

```csharp
void Warmup(GameObject prefab, int count)
void Warmup(string vfxName, int count)
```

**Example:**
```csharp
// Call during loading screens
VFXSystem.Instance.Warmup(explosionPrefab, 10);
VFXSystem.Instance.Warmup("MuzzleFlash", 20);
```

#### `Release()` - Manual release (rarely needed)

```csharp
void Release(GameObject instance)
```

> **Note:** The system automatically releases VFX instances via `VFXAutoRelease`. Manual release is rarely necessary.

## ⚙️ Configuration

### VFXPool Inspector Settings

| Setting | Description | Default |
|---------|-------------|---------|
| `Default Pool Size` | Default pool size for unregistered VFX | 20 |
| `Warmup Presets On Awake` | Auto-warmup configured effects on start | true |
| `Pool Root` | Parent transform for pooled objects | Auto-created |

### Preset VFX List

Each `VFXConfig` entry:

| Field | Description |
|-------|-------------|
| `VFX Name` | Custom identifier (uses prefab name if empty) |
| `Prefab` | VFX prefab reference |
| `Prewarm Count` | Initial instances to create |
| `Collection Checks` | Enable double-release protection (recommended: true) |

## 🔧 Advanced Features

### Automatic Duration Calculation

The system automatically calculates VFX duration by analyzing all ParticleSystem components:

```csharp
duration = main.duration + main.startLifetime.constantMax
```

This ensures VFX plays completely before recycling, even with complex particle lifetimes.

### ParticleSystem Restart

Every VFX instance automatically restarts all ParticleSystem components in the hierarchy:

```csharp
ps.Clear(true);              // Clear old particles
ps.Simulate(0f, true, true); // Reset simulation
ps.Play(true);               // Begin playback
```

This ensures consistent VFX playback regardless of previous state.

### Dual Access Patterns

**By Prefab Reference** (flexible, no registration needed):
```csharp
VFXSystem.Instance.Play(explosionPrefab, pos, rot);
```

**By Name** (cleaner code, requires VFXPool configuration):
```csharp
VFXSystem.Instance.Play("Explosion", pos, rot);
```

## ✅ Best Practices

### DO:
- ✅ Warmup frequently used effects during loading screens
- ✅ Use name-based access for common VFX (cleaner code)
- ✅ Enable collection checks during development
- ✅ Let auto-release handle cleanup
- ✅ Configure reasonable pool sizes based on profiling

### DON'T:
- ❌ Access VFXPool directly (use VFXSystem API)
- ❌ Destroy VFX instances manually (let pool release them)
- ❌ Create huge pools for rarely used effects
- ❌ Disable collection checks in production without thorough testing

## 📊 Performance Considerations

### Benefits
- **Eliminates runtime instantiation overhead** - Objects reused from pool
- **Reduces garbage collection pressure** - No frequent allocations/deallocations
- **Maintains consistent frame rate** - Predictable performance
- **Memory-efficient** - Queue-based storage with minimal overhead

### Memory Usage

```
Total Memory = (Pool Size × VFX Prefab Memory) + Active Instances
```

- **Inactive instances**: Stored in Queue (pooled)
- **Active instances**: Tracked in HashSet (in-use)

### Optimization Tips

1. **Profile pool sizes** - Monitor `pool.CountActive` and `pool.CountInactive`
2. **Warmup strategically** - Preload during loading screens, not during gameplay
3. **Right-size pools** - Small pools for rare VFX, large pools for frequent VFX
4. **Monitor with Unity Profiler** - Check memory usage and GC allocations

## 💻 Example Use Cases

### Explosion on Impact

```csharp
void OnCollisionEnter(Collision collision)
{
    var contact = collision.contacts[0];
    VFXSystem.Instance.PlayAt(explosionVFX, contact.point);
}
```

### Muzzle Flash Effect

```csharp
void Shoot()
{
    // Spawn at gun muzzle, attach to gun, short duration
    VFXSystem.Instance.Play(
        muzzleFlashPrefab,
        gunMuzzle.position,
        gunMuzzle.rotation,
        parent: gunMuzzle,
        durationOverride: 0.1f
    );

    // Fire bullet...
}
```

### Named VFX with Prewarming

```csharp
void Awake()
{
    // Preload common effects during level load
    VFXSystem.Instance.Warmup("Explosion", 10);
    VFXSystem.Instance.Warmup("HitEffect", 20);
    VFXSystem.Instance.Warmup("MuzzleFlash", 30);
}

void PlayExplosion(Vector3 position)
{
    // Clean string-based API
    VFXSystem.Instance.Play("Explosion", position, Quaternion.identity);
}
```

### VFX with Custom Duration

```csharp
// Override auto-calculated duration
VFXSystem.Instance.PlayAt(
    longEffectPrefab,
    position,
    durationOverride: 10f  // Force 10 second duration
);
```

## 🔍 Troubleshooting

### VFX Not Appearing

**Possible causes:**
- Prefab doesn't contain ParticleSystem components
- VFXPool not initialized (check `VFXPool.Instance`)
- Duration calculation failed (check console warnings)

**Solution:**
```csharp
// Verify VFXPool is ready
if (VFXPool.HasInstance)
{
    VFXSystem.Instance.PlayAt(vfxPrefab, position);
}
```

### Performance Issues (Frame Drops)

**Possible causes:**
- Pool sizes too small (runtime instantiation)
- Not using warmup for frequently used effects
- Too many active VFX instances

**Solution:**
```csharp
// Increase pool sizes and warmup
void Awake()
{
    VFXSystem.Instance.Warmup(frequentEffect, 50);
}
```

### Memory Growing Over Time

**Possible causes:**
- Pool sizes too large for usage patterns
- Auto-release not working correctly
- Manual instantiation bypassing pool

**Solution:**
```csharp
// 1. Review pool sizes in VFXPool configuration
// 2. Check console for collection check warnings
// 3. Ensure all VFX go through VFXSystem API
```

## 📋 System Requirements

- **Unity Version**: Unity 2020.3 or later (uses `FindFirstObjectByType`)
- **Dependencies**: None (all included)
- **Namespaces**: `CDTU.Utils` (included in package)

## 📦 Package Contents

```
VFXSystem/
├── SingletonDD.cs       // DontDestroyOnLoad singleton base
├── ObjectPool.cs        // Generic object pool implementation
├── VFXSystem.cs         // Main API facade (use this!)
├── VFXPool.cs           // Pool manager (auto-initialized)
└── VFXAutoRelease.cs    // Auto lifecycle component
```

## 🎓 Design Philosophy

VFXSystem follows these principles:

1. **Simplicity** - Clean, intuitive API that hides complexity
2. **Performance** - Object pooling for production-ready optimization
3. **Safety** - Collection checks and robust error handling
4. **Flexibility** - Multiple access patterns for different use cases
5. **Zero Dependencies** - Self-contained package for easy integration

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 Code Comments

> **Note:** Code comments are in Chinese (中文). The API and documentation are in English for international accessibility.

---

**Made with ❤️ for Unity developers**
