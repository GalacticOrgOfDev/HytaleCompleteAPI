# Hytale Mod Development: Quick Reference

## Getting Started

### 1. Create a Plugin Class

```java
public class MyPlugin extends JavaPlugin {
    @Override
    public void onEnable() {
        // Plugin is loading
        getEventRegistry().register(BreakBlockEvent.class, this::onBreak);
    }
    
    @Override
    public void onDisable() {
        // Plugin is unloading
    }
}
```

### 2. Create Plugin Manifest

```json
{
    "Group": "MyPlugin",
    "Name": "MyPlugin",
    "Version": "1.0.0",
    "Main": "MyPlugin.MyPlugin",
    "Authors": ["YourName"]
}
```

---

## 5-Minute Patterns

### Listen to Block Break

```java
getEventRegistry().register(BreakBlockEvent.class, event -> {
    Vector3i pos = event.getTargetBlock();
    World world = event.getWorld();
    
    logger.info("Block broken at " + pos);
});
```

### Listen to Block Place

```java
getEventRegistry().register(PlaceBlockEvent.class, event -> {
    Vector3i pos = event.getTargetBlock();
    ItemStack item = event.getItemInHand();
    
    // Cancel by setting invalid position
    event.setTargetBlock(new Vector3i(0, -1, 0));
});
```

### Create an ItemStack

```java
ItemStack cobaltIngot = new ItemStack("hytale:Ingredient_Bar_Cobalt", 40);
ItemStack cyanShard = new ItemStack("hytale:Ingredient_Crystal_Cyan", 100);

// Immutable operations
ItemStack halfStack = cobaltIngot.withQuantity(32);
ItemStack enchanted = cyanShard.withMetadata("Enchant", value);
```

### Add Item to Inventory

```java
ItemContainer inventory = new ItemContainer();
ItemStack item = new ItemStack("hytale:apple", 100);

ItemStackTransaction trans = inventory.addItemStack(item);
if (trans.execute()) {
    logger.info("Item added");
}
```

### Spawn Entity in World

```java
world.execute(() -> {
    Entity entity = new Entity();
    Vector3d pos = new Vector3d(100.5, 64.5, 200.5);
    world.addEntity(entity, pos, Vector3f.ZERO, AddReason.SPAWN);
});
```

### Query Block at Position

```java
world.execute(() -> {
    Vector3i pos = new Vector3i(100, 64, 200);
    BlockState state = world.getBlockState(pos);
    if (state != null) {
        BlockType type = state.getBlockType();
    }
});
```

### Check Permission

```java
if (!sender.hasPermission("mycmd.use")) {
    sender.sendMessage("Permission denied");
    return;
}
```

### Create a Command

```java
public class MyCommand extends AbstractCommand {
    @Override
    public String getName() { return "mycommand"; }
    
    @Override
    public String getDescription() { return "Does something"; }
    
    @Override
    public void execute(CommandSender sender, String[] args) {
        sender.sendMessage("Command executed");
    }
}
```

### Thread-Safe World Operations

```java
// Always use world.execute() for world modifications
world.execute(() -> {
    // Safe to modify world state here
    world.setBlockState(pos, blockState);
});
```

---

## Common Return Types

| Type | Meaning | Usage |
| ------ | --------- | ------- |
| `ItemStackTransaction` | Item add/remove operation | `.execute()` to commit |
| `ListTransaction` | Multi-item operation | `.execute()` to commit |
| `MoveTransaction` | Item movement | `.execute()` or `.rollback()` |
| `BlockState` | Block in world | Read/modify block properties |
| `Vector3i` | Integer coordinates | Block positions |
| `Vector3d` | Decimal coordinates | Entity positions |
| `CompletableFuture<T>` | Async operation | `.thenAccept()` for result |

---

## Event List (Common)

| Event | What Triggers | Key Methods |
| ------- | --------------- | ------------- |
| `BreakBlockEvent` | Player breaks block | `getTargetBlock()`, `getBlockType()` |
| `PlaceBlockEvent` | Player places block | `getTargetBlock()`, `getRotation()` |
| `PlayerChatEvent` | Player sends message | `getMessage()`, `setMessage()` |
| `PlayerConnectEvent` | Player joins server | `getPlayer()`, `getWorld()` |
| `PlayerDisconnectEvent` | Player leaves server | `getPlayer()`, `getReason()` |
| `EntityRemoveEvent` | Entity despawns | `getEntity()`, `getReason()` |
| `PrepareUniverseEvent` | World is ready | `getWorld()` |
| `ShutdownEvent` | Server shutting down | (no methods) |
| `LoadAssetEvent` | Asset loading | `getAssetId()`, `getAssetType()` |
| `PlayerInteractEvent` | Player interacts | `getTargetEntity()`, `getTargetBlock()` |

---

## Key Classes (Import Paths)

```java
// Core Plugin
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.logger.HytaleLogger;

// Items & Inventory
import com.hypixel.hytale.server.core.inventory.ItemStack;
import com.hypixel.hytale.server.core.inventory.container.ItemContainer;
import com.hypixel.hytale.server.core.inventory.transaction.*;

// World & Entities
import com.hypixel.hytale.server.core.universe.world.World;
import com.hypixel.hytale.server.core.entity.Entity;
import com.hypixel.hytale.component.AddReason;

// Events
import com.hypixel.hytale.server.core.event.EventRegistry;
import com.hypixel.hytale.server.core.event.events.ecs.*;

// Commands
import com.hypixel.hytale.server.core.command.AbstractCommand;
import com.hypixel.hytale.server.core.command.CommandSender;

// Utilities
import org.joml.Vector3i;
import org.joml.Vector3d;
import org.joml.Vector3f;
```

---

## Best Practices Checklist

- [ ] All world modifications inside `world.execute()`
- [ ] Always check `sender.hasPermission()` in commands
- [ ] Use `try-catch` around `.execute()` for transactions
- [ ] Register events in `onEnable()`
- [ ] Clean up in `onDisable()`
- [ ] Store EventRegistration refs for later unregistering
- [ ] Use `.rollback()` on failed transactions
- [ ] ItemStack operations are immutable (return copies)
- [ ] Use `Vector3i` for blocks, `Vector3d` for entities
- [ ] Always provide help text in commands
- [ ] Log important operations with `HytaleLogger`
- [ ] Validate arguments in command execution

---

## Error Handling Examples

```java
// Transaction failure
ItemStackTransaction trans = container.addItemStack(item);
boolean success = trans.execute();
if (!success) {
    trans.rollback();
    logger.error("Failed to add item");
}

// Async loading
world.getChunkAsync(x, z)
    .thenAccept(chunk -> { /* use chunk */ })
    .exceptionally(error -> {
        logger.error("Chunk load failed: " + error);
        return null;
    });

// Null checking
BlockState state = world.getBlockState(pos);
if (state != null) {
    // Safe to use
}

// Missing permission
if (!sender.hasPermission("action")) {
    sender.sendMessage("Access denied");
    return;
}
```

---

## Debugging Tips

1. **Enable all log levels in onEnable**

    ```java
    HytaleLogger logger = HytaleLogger.forEnclosingClass();
    // Use logger.info(), logger.warn(), logger.error(), logger.debug()
    ```

2. **Check event registration**

    ```java
    // Verify events are firing
    getEventRegistry().register(MyEvent.class, event -> {
        logger.info("Event fired!");  // Should appear in logs
    });
    ```

3. **Verify world thread**

    ```java
    // If things don't update, you're probably not on world thread
    world.execute(() -> {
        // Now you're on world thread
        world.setBlockState(pos, state);
    });
    ```

4. **Transaction debugging**

    ```java
    boolean canAdd = container.canAddItemStack(item);
    logger.info("Can add: " + canAdd);

    if (!canAdd) {
        logger.debug("Container full or incompatible");
    }
    ```

---

## Common Gotchas

❌ **Wrong:** `container.addItemStack(item)` without `.execute()`
✅ **Right:** `container.addItemStack(item).execute()`

❌ **Wrong:** Modifying world directly (not thread-safe)
✅ **Right:** `world.execute(() -> { /* modify world */ })`

❌ **Wrong:** ItemStack operations modify in-place
✅ **Right:** `ItemStack newStack = oldStack.withQuantity(5);`

❌ **Wrong:** No permission checking in commands
✅ **Right:** `if (!sender.hasPermission("cmd")) return;`

❌ **Wrong:** Not storing EventRegistration references
✅ **Right:** `EventReg ref = registry.register(...);` then `ref.unregister();` in onDisable

---

**Last Updated:** February 27, 2026
**API Version:** Hytale 2026

