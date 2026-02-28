# Hytale Modding Documentation Index

**Created:** February 27, 2026  
**Source:** Decompiled HytaleServerJar + API Documentation  
**Purpose:** Comprehensive practical reference for mod developers

---

## 📚 Documentation Files

### 1. **PRACTICAL_MOD_PATTERNS.md** (Core Reference)

Complete, organized guide covering all major patterns with code examples.

**Contains:**

- Event Handler Implementations (with real examples)
- Inventory Operations (ItemStack, ItemContainer, transactions)
- World Interactions (querying blocks, spawning entities, chunks)
- Command Implementations (structure and registration)
- Plugin Lifecycle (onEnable/onDisable patterns)
- Networking Patterns (packet filtering and watching)
- Permission Checking patterns
- Asset Loading examples
- Complete crafting system example

**Use when:** You need to understand how to implement a specific feature.

**Format:** Organized by feature category with detailed explanations and 5-20 line code examples.

---

### 2. **QUICK_REFERENCE.md** (Developer Cheatsheet)

Fast lookup guide for common tasks, patterns, and APIs.

**Contains:**

- 5-minute pattern snippets (copy-paste ready)
- Common return types and their usage
- Event list with key methods
- Import paths for all major classes
- Best practices checklist
- Error handling examples
- Common gotchas and solutions

**Use when:** You need a quick answer or copy-paste code for a common task.

**Format:** Tables, code snippets, organized by use case.

---

### 3. **PROGRESSIVE_EXAMPLES.md** (Learning Path)

5 increasingly complex example plugins from beginner to advanced.

**Examples:**

- **Level 1:** Hello World Plugin (basic structure and logging)
- **Level 2:** Item Logger Plugin (accessing event data, string formatting)
- **Level 3:** Inventory Inspector Plugin (container operations, item manipulation)
- **Level 4:** Limited Resources Plugin (event cancellation, world interaction)
- **Level 5:** Mining Speed Modifier (commands, permissions, state tracking)

**Use when:** Learning Hytale modding from scratch or expanding your skill level.

**Format:** Complete, runnable plugin code with explanations and manifest files.

---

## 🎯 How to Use This Documentation

### For Beginners

1. Start with [PROGRESSIVE_EXAMPLES.md](PROGRESSIVE_EXAMPLES.md) - Levels 1-3
2. Reference [QUICK_REFERENCE.md](QUICK_REFERENCE.md) for syntax
3. Dive deeper into [PRACTICAL_MOD_PATTERNS.md](PRACTICAL_MOD_PATTERNS.md) as needed

### For Experienced Developers

1. Check [QUICK_REFERENCE.md](QUICK_REFERENCE.md) for API signatures
2. Reference [PRACTICAL_MOD_PATTERNS.md](PRACTICAL_MOD_PATTERNS.md) for implementation details
3. Use [PROGRESSIVE_EXAMPLES.md](PROGRESSIVE_EXAMPLES.md) as baseline for new feature types

### For Feature Implementation

1. Find your feature in [PRACTICAL_MOD_PATTERNS.md](PRACTICAL_MOD_PATTERNS.md) (sections 1-8)
2. Get quick syntax from [QUICK_REFERENCE.md](QUICK_REFERENCE.md)
3. See working example in [PROGRESSIVE_EXAMPLES.md](PROGRESSIVE_EXAMPLES.md)

---

## 📋 Pattern Quick Lookup

| Use Case | Document | Section |
| ---------- | ---------- | --------- |
| Listen to block breaks | PRACTICAL | 1.2 |
| Create ItemStack | QUICK_REFERENCE | 5-Minute Patterns |
| Manage inventory | PRACTICAL | 2.2 |
| Spawn entity | PRACTICAL | 3.2 |
| Query blocks | PRACTICAL | 3.3 |
| Create command | PRACTICAL | 4.1 |
| Check permissions | PRACTICAL | 7.1 |
| Load assets | PRACTICAL | 8.1 |
| Plugin startup | PRACTICAL | 5.1 |
| Plugin shutdown | PRACTICAL | 5.2 |
| Thread-safe world ops | QUICK_REFERENCE | Best Practices |
| Full inventory example | PROGRESSIVE | Level 3 |
| Commands + state | PROGRESSIVE | Level 5 |

---

## 🔑 Key Concepts Extracted from Decompiled JAR

### Core Architecture

- Plugins extend `JavaPlugin` and implement `onEnable()`/`onDisable()`
- Event system uses `EventRegistry.register(EventClass.class, handler)`
- World operations must occur inside `world.execute()` for thread safety
- All item operations use immutable transactions (`.execute()` to commit)

### Event System

Found in: `com.hypixel.hytale.server.core.event.events`

**Common Events:**

- `BreakBlockEvent` - Block destruction
- `PlaceBlockEvent` - Block placement
- `PlayerChatEvent` - Chat messages
- `PrepareUniverseEvent` - World ready for interaction
- `LoadAssetEvent` - Asset loading
- `ShutdownEvent` - Server shutdown

### Inventory System

Found in: `com.hypixel.hytale.server.core.inventory`

**Key Classes:**

- `ItemStack` - Immutable item representation
- `ItemContainer` - Inventory/chest abstraction
- `ItemStackTransaction` - Add/remove operations
- `MoveTransaction` - Item movement between containers

**Pattern:** All operations return Transactions that must be `.execute()`

### World System

Found in: `com.hypixel.hytale.server.core.universe.world`

**Key Classes:**

- `World` - Game world instance
- `BlockState` - Block at a position
- `Entity` - World entities
- `Chunk` - Terrain data section

**Pattern:** Use `world.execute(() -> { /* world ops */ })` for thread safety

### Command System

Found in: `com.hypixel.hytale.server.core.command`

**Key Classes:**

- `AbstractCommand` - Base for custom commands
- `CommandSender` - Player or console executing command
- `CommandManager` - Command registration

**Pattern:** Extend AbstractCommand, override execute(), validate permissions first

### Asset System

Found in: `com.hypixel.hytale.server.core.asset`

**Key Classes:**

- `HytaleAssetStore` - Asset repository
- `AssetModule` - Asset lifecycle management
- `LoadAssetEvent` - Asset loading notification

**Pattern:** Listen to LoadAssetEvent, use HytaleAssetStore for loading

### Permission System

Found in: `com.hypixel.hytale.server.core.permission`

**Key Classes:**

- `HytalePermissions` - Central permission registry
- `PermissionHolder` - Object holding permissions

**Pattern:** `sender.hasPermission(node)` before executing protected operations

---

## 🏗️ Directory Structure (Decompiled)

```nav
com/hypixel/hytale/server/core/
├── plugin/                    # Plugin lifecycle
├── event/                     # Event system
│   └── events/               # Event definitions
├── inventory/                # Item & container system
├── universe/world/           # World & block system
│   ├── entity/              # Entity management
│   └── block/               # Block operations
├── command/                  # Command system
├── io/                       # Networking (packets)
├── permission/               # Permission system
├── modules/                  # Built-in systems
│   ├── item/
│   ├── block/
│   ├── entity/
│   └── ... (20+ more)
└── asset/                    # Asset management
```

---

## 📝 Extracted Code Patterns (Summary)

### Total Patterns Documented

| Category | Count | Files |
| ---------- | ------- | ------- |
| Event handlers | 6 | PRACTICAL 1.x |
| Inventory operations | 3 | PRACTICAL 2.x |
| World interactions | 4 | PRACTICAL 3.x |
| Command implementations | 2 | PRACTICAL 4.x |
| Plugin lifecycle | 3 | PRACTICAL 5.x |
| Networking | 2 | PRACTICAL 6.x |
| Permission checking | 2 | PRACTICAL 7.x |
| Asset loading | 2 | PRACTICAL 8.x |
| Complete examples | 5 | PROGRESSIVE |
| Quick snippets | 8+ | QUICK_REFERENCE |

---

## 🚀 Getting Started (30-Minute Setup)

### Step 1: Create Plugin Directory Structure

```nav
Hytale GameFiles/
└── UserData/
    └── Saves/
        └── YourWorld/
            └── mods/
                └── MyPlugin.MyPlugin/
                    ├── manifest.json
                    └── src/
                        └── MyPlugin.java
```

### Step 2: Create manifest.json

```json
{
    "Group": "MyPlugin",
    "Name": "MyPlugin",
    "Version": "1.0.0",
    "Main": "MyPlugin.MyPlugin",
    "Authors": ["YourName"]
}
```

### Step 3: Write Your Plugin

Copy from Level 1 in PROGRESSIVE_EXAMPLES.md and modify as needed.

### Step 4: Reference Documentation

- Quick API look-up: QUICK_REFERENCE.md
- Implementation help: PRACTICAL_MOD_PATTERNS.md
- Learning: PROGRESSIVE_EXAMPLES.md

---

## ✅ Validation Checklist

When implementing a feature, verify:

- [ ] All world modifications inside `world.execute()`
- [ ] Permission check before command execution: `sender.hasPermission(...)`
- [ ] Event listeners registered in `onEnable()`
- [ ] Event listeners unregistered in `onDisable()`
- [ ] ItemStack operations store result: `ItemStack newItem = old.withQuantity(5)`
- [ ] Transactions are executed: `transaction.execute()`
- [ ] Error handling for failed transactions: check return value
- [ ] Logging important operations with `HytaleLogger`
- [ ] Proper Vector3i usage for blocks
- [ ] Proper Vector3d usage for entities
- [ ] Help text provided for commands

---

## 📚 Original Source Documents

All patterns extracted from:

1. **Decompiled JAR:** `UserData/API Docs/Decompiled/HytaleServerJar/`
   - Module implementations
   - Event definitions
   - System architecture

2. **API Documentation:** `UserData/API Docs/Java API/Java ReadMe.md`
   - Class signatures
   - Method documentation
   - Usage examples
   - Interface specifications

---

## 🔄 Updates & Maintenance

**Last Updated:** February 27, 2026

**Coverage:**

- ✅ Plugin lifecycle system
- ✅ Event system (core events)
- ✅ Inventory system
- ✅ World system (blocks, entities, chunks)
- ✅ Command system
- ✅ Permission system
- ✅ Asset loading
- ✅ Networking (basic patterns)
- ⚠️ Advanced mod systems (see modules/ for more)

**Future Additions:**

- Advanced entity component system
- Custom particle effects
- Advanced recipe system
- Clan/team systems
- Economy integration
- More networking examples

---

## 🤝 Contributing

To improve this documentation:

1. Test all code examples
2. Update with new patterns as discovered
3. Add version-specific notes
4. Link to official API changes

Current contributors: February 2026 API extraction

---

## 💡 Pro Tips from Extraction

1. **Always use `world.execute()`** - The decompiled code shows all world modifications wrapped in execute() calls for thread safety.

2. **Transaction pattern is important** - ItemContainer operations return Transaction objects. Not calling `.execute()` is a common mistake.

3. **Events are flexible** - The event system is the main extension point. Almost all gameplay can be controlled via event handlers.

4. **Permission checks first** - In command implementations, permission checking appears first in all official examples.

5. **Immutable ItemStack** - ItemStack operations never modify in-place; they return new instances.

6. **Logging is critical** - Even built-in modules extensively use HytaleLogger for debugging and monitoring.

7. **Vector3i vs Vector3d matters** - Block positions use integer vectors, entity positions use double for precision.

---

## 📖 Navigation Guide

**Start here:** [QUICK_REFERENCE.md](QUICK_REFERENCE.md)  
**Learn programming:** [PROGRESSIVE_EXAMPLES.md](PROGRESSIVE_EXAMPLES.md)  
**Implement features:** [PRACTICAL_MOD_PATTERNS.md](PRACTICAL_MOD_PATTERNS.md)  
**API Details:** [Java ReadMe.md](Java%20ReadMe.md)

---

## Happy modding! 🎮

For questions, check the relevant section of the documentation, then reference the decompiled source code for the most current implementation details.
