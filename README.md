# VFXSystem

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
