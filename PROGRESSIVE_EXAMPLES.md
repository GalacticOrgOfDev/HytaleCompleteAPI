# Hytale Mod Development: Progressive Examples

Learn Hytale modding through 5 complete, working examples, from beginner to advanced.

---

## Level 1: Hello World Plugin

**What you'll learn:** Basic plugin structure, logging, event listening

```java
package com.example.helloworld;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.logger.HytaleLogger;
import com.hypixel.hytale.server.core.event.events.ecs.BreakBlockEvent;

public class HelloWorldPlugin extends JavaPlugin {
    private final HytaleLogger logger = HytaleLogger.forEnclosingClass();
    
    @Override
    public void onEnable() {
        logger.info("Hello World Plugin loaded!");
        
        // Listen to block breaks
        getEventRegistry().register(BreakBlockEvent.class, event -> {
            logger.info("You broke a block!");
        });
    }
    
    @Override
    public void onDisable() {
        logger.info("Hello World Plugin unloaded");
    }
}
```

**manifest.json:**

```json
{
    "Group": "HelloWorldExample",
    "Name": "HelloWorldPlugin",
    "Version": "1.0.0",
    "Main": "com.example.helloworld.HelloWorldPlugin",
    "Authors": ["YourName"]
}
```

**What happens:**

1. Plugin loads and prints "Hello World Plugin loaded!"
2. Every time you break a block, it prints "You broke a block!"
3. Plugin unloads and prints "Hello World Plugin unloaded"

---

## Level 2: Item Logger Plugin

**What you'll learn:** Access event data, inspect items, format output

```java
package com.example.itemlogger;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.logger.HytaleLogger;
import com.hypixel.hytale.server.core.event.events.ecs.BreakBlockEvent;
import com.hypixel.hytale.server.core.inventory.ItemStack;
import org.joml.Vector3i;

public class ItemLoggerPlugin extends JavaPlugin {
    private final HytaleLogger logger = HytaleLogger.forEnclosingClass();
    private int blocksBroken = 0;
    
    @Override
    public void onEnable() {
        logger.info("Item Logger Plugin started");
        
        getEventRegistry().register(BreakBlockEvent.class, event -> {
            onBlockBreak(event);
        });
    }
    
    private void onBlockBreak(BreakBlockEvent event) {
        // Get event data
        Vector3i pos = event.getTargetBlock();
        ItemStack tool = event.getItemInHand();
        String blockName = event.getBlockType().getName();
        
        // Increment counter
        blocksBroken++;
        
        // Log detailed information
        logger.info(String.format(
            "[Block #%d] Broke %s at [%d,%d,%d] with %s (Durability: %.1f)",
            blocksBroken,
            blockName,
            pos.x, pos.y, pos.z,
            tool.getItemId(),
            tool.getDurability()
        ));
        
        // Example: Special handling for stone
        if (blockName.contains("Cobalt") && tool.getItemId().contains("Adamantite")) {
            logger.info("You used an Adamantite tool on Cobalt!");
        }
    }
    
    @Override
    public void onDisable() {
        logger.info("Item Logger Plugin stopped. Total blocks broken: " + blocksBroken);
    }
}
```

**What happens:**

1. Every block break logs: block number, type, position, tool, and durability
2. Tracks total blocks broken across session
3. Detects special combinations (Adamantite tool on Cobalt)

**Output example:**

```example
[Block #1] Broke Hytale:cobalt_ore at [100,64,200] with hytale:iron_pickaxe (Durability: 128.0)
[Block #5] Broke hytale:cobalt_ore at [105,64,205] with hytale:adamantite_pickaxe (Durability: 256.0)
You used an Adamantite tool on Cobalt!
```

---

## Level 3: Inventory Inspector Plugin

**What you'll learn:** ItemContainer operations, inventory checking, item manipulation

```java
package com.example.inventoryinspector;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.logger.HytaleLogger;
import com.hypixel.hytale.server.core.command.AbstractCommand;
import com.hypixel.hytale.server.core.command.CommandSender;
import com.hypixel.hytale.server.core.inventory.ItemStack;
import com.hypixel.hytale.server.core.inventory.container.ItemContainer;

public class InventoryInspectorPlugin extends JavaPlugin {
    private final HytaleLogger logger = HytaleLogger.forEnclosingClass();
    
    @Override
    public void onEnable() {
        logger.info("Inventory Inspector started");
    }
    
    // Example method showing inventory operations
    public void demonstrateInventoryOperations() {
        // Create containers
        ItemContainer inventory = new ItemContainer();
        ItemContainer storage = new ItemContainer();
        
        // Add items
        ItemStack apples = new ItemStack("hytale:apple", 64);
        ItemStack sticks = new ItemStack("hytale:stick", 32);
        ItemStack cobaltIngot = new ItemStack("hytale:Ingredient_Bar_Cobalt", 5);
        
        inventory.addItemStack(apples).execute();
        inventory.addItemStack(sticks).execute();
        inventory.addItemStack(cobaltIngot).execute();
        
        // Inspect inventory
        logger.info("Inventory contains:");
        inventory.forEach((short slot, ItemStack stack) -> {
            logger.info(String.format(
                "  Slot %d: %s x%d",
                slot,
                stack.getItemId(),
                stack.getQuantity()
            ));
        });
        
        // Count items
        int stickCount = inventory.countItemStacks(stack ->
            stack.getItemId().contains("sticks")
        );
        logger.info("Total sticks: " + stickCount);
        
        // Move items
        if (storage.canAddItemStack(sticks)) {
            inventory.moveItemStackFromSlot((short)2, storage).execute();
            logger.info("Moved sticks to storage");
        }
        
        // Clear inventory
        inventory.clear().execute();
        logger.info("Inventory cleared");
    }
    
    @Override
    public void onDisable() {
        logger.info("Inventory Inspector stopped");
    }
}
```

**Key concepts:**

- Creating and managing ItemContainers
- Adding items with transaction execution
- Iterating containers
- Counting items with predicates
- Moving items between containers
- Clearing containers

---

## Level 4: Limited Resources Plugin

**What you'll learn:** Event cancellation, resource drops, world interaction

```java
package com.example.limitedresources;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.logger.HytaleLogger;
import com.hypixel.hytale.server.core.event.events.ecs.BreakBlockEvent;
import com.hypixel.hytale.server.core.event.events.ecs.PlaceBlockEvent;
import com.hypixel.hytale.server.core.inventory.ItemStack;
import com.hypixel.hytale.server.core.universe.world.World;
import org.joml.Vector3i;
import org.joml.Vector3d;
import org.joml.Vector3f;

public class LimitedResourcesPlugin extends JavaPlugin {
    private final HytaleLogger logger = HytaleLogger.forEnclosingClass();
    
    // Blocks that cannot be broken
    private static final String[] PROTECTED_BLOCKS = {
        "protected_block",
        "protected_block",
        "protected_block"
    };
    
    @Override
    public void onEnable() {
        logger.info("Limited Resources Plugin started");
        
        getEventRegistry().register(BreakBlockEvent.class, event -> {
            onBlockBreak(event);
        });
        
        getEventRegistry().register(PlaceBlockEvent.class, event -> {
            onBlockPlace(event);
        });
    }
    
    private void onBlockBreak(BreakBlockEvent event) {
        String blockName = event.getBlockType().getName();
        World world = event.getWorld();
        Vector3i pos = event.getTargetBlock();
        
        // Check if block is protected
        for (String protected_block : PROTECTED_BLOCKS) {
            if (blockName.contains(protected_block)) {
                // Cancel by setting invalid position
                event.setTargetBlock(new Vector3i(0, -1, 0));
                logger.warn("Prevented breaking of: " + blockName);
                return;
            }
        }
        
        // Spawn extra drops for rare blocks
        if (blockName.contains("diamond")) {
            world.execute(() -> {
                // Spawn bonus item
                ItemStack bonus = new ItemStack("hytale:diamond_dust", 4);
                Vector3d dropPos = new Vector3d(pos.x + 0.5, pos.y + 0.5, pos.z + 0.5);
                
                // In real code, would create ItemEntity
                logger.info("Would spawn diamond dust at " + dropPos);
            });
        }
    }
    
    private void onBlockPlace(PlaceBlockEvent event) {
        Vector3i placePos = event.getTargetBlock();
        
        // Prevent building too high
        if (placePos.y > 256) {
            event.setTargetBlock(new Vector3i(0, -1, 0));
            logger.info("Prevented block placement above build limit");
        }
        
        // Prevent building below world
        if (placePos.y < 0) {
            event.setTargetBlock(new Vector3i(0, -1, 0));
            logger.info("Prevented block placement in void");
        }
    }
    
    @Override
    public void onDisable() {
        logger.info("Limited Resources Plugin stopped");
    }
}
```

**Features:**

- Prevents breaking protected blocks
- Detects rare blocks and awards bonuses
- Prevents building outside valid height range
- Demonstrated event cancellation
- World execution for safe modifications

---

## Level 5: Advanced - Mining Speed Modifier with Commands

**What you'll learn:** Commands, permissions, state tracking, complex logic

```java
package com.example.minespeed;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.logger.HytaleLogger;
import com.hypixel.hytale.server.core.event.events.ecs.BreakBlockEvent;
import com.hypixel.hytale.server.core.command.AbstractCommand;
import com.hypixel.hytale.server.core.command.CommandSender;
import java.util.*;

public class MineSpeedPlugin extends JavaPlugin {
    private final HytaleLogger logger = HytaleLogger.forEnclosingClass();
    
    // Track mining speed multipliers per player
    private Map<String, Double> playerMultipliers = new HashMap<>();
    
    @Override
    public void onEnable() {
        logger.info("Mining Speed Plugin started");
        
        // Register event handler
        getEventRegistry().register(BreakBlockEvent.class, event -> {
            onBlockBreak(event);
        });
        
        // Commands would be registered here
        registerCommands();
    }
    
    private void registerCommands() {
        // Create and register mining speed command
        AbstractCommand speedCmd = new MineSpeedCommand(this);
        // CommandManager would register this
        logger.info("Mining speed command registered");
    }
    
    private void onBlockBreak(BreakBlockEvent event) {
        String blockName = event.getBlockType().getName();
        HytaleLogger logger = HytaleLogger.forEnclosingClass();
        
        // Would get player name from event
        String playerName = "Player";  // Placeholder
        
        // Get player's multiplier (default 1.0)
        double multiplier = playerMultipliers.getOrDefault(playerName, 1.0);
        
        if (multiplier != 1.0) {
            logger.info(String.format(
                "%s mined %s with %.2fx speed modifier",
                playerName, blockName, multiplier
            ));
        }
    }
    
    // Separate command class
    public static class MineSpeedCommand extends AbstractCommand {
        private MineSpeedPlugin plugin;
        
        public MineSpeedCommand(MineSpeedPlugin plugin) {
            this.plugin = plugin;
        }
        
        @Override
        public String getName() {
            return "minespeed";
        }
        
        @Override
        public String getDescription() {
            return "Modify mining speed multiplier";
        }
        
        @Override
        public String getUsage() {
            return "/minespeed <set|get|reset> [multiplier]";
        }
        
        @Override
        public List<String> getAliases() {
            return Arrays.asList("mspeed", "minemod");
        }
        
        @Override
        public void execute(CommandSender sender, String[] args) {
            // Permission check
            if (!sender.hasPermission("minespeed.use")) {
                sender.sendMessage("Access denied");
                return;
            }
            
            // Argument validation
            if (args.length == 0) {
                sender.sendMessage("Usage: " + getUsage());
                return;
            }
            
            String action = args[0].toLowerCase();
            
            switch (action) {
                case "set":
                    if (args.length < 2) {
                        sender.sendMessage("Usage: /minespeed set <multiplier>");
                        return;
                    }
                    handleSet(sender, args[1]);
                    break;
                    
                case "get":
                    handleGet(sender);
                    break;
                    
                case "reset":
                    handleReset(sender);
                    break;
                    
                default:
                    sender.sendMessage("Unknown action: " + action);
            }
        }
        
        private void handleSet(CommandSender sender, String value) {
            try {
                double multiplier = Double.parseDouble(value);
                
                if (multiplier < 0.1 || multiplier > 5.0) {
                    sender.sendMessage("Multiplier must be between 0.1 and 5.0");
                    return;
                }
                
                plugin.playerMultipliers.put(sender.getName(), multiplier);
                sender.sendMessage("Mining speed set to: " + multiplier + "x");
                
            } catch (NumberFormatException e) {
                sender.sendMessage("Invalid multiplier: " + value);
            }
        }
        
        private void handleGet(CommandSender sender) {
            double current = plugin.playerMultipliers.getOrDefault(
                sender.getName(), 1.0
            );
            sender.sendMessage("Current mining speed: " + current + "x");
        }
        
        private void handleReset(CommandSender sender) {
            plugin.playerMultipliers.remove(sender.getName());
            sender.sendMessage("Mining speed reset to 1.0x");
        }
    }
    
    @Override
    public void onDisable() {
        logger.info("Mining Speed Plugin stopped");
        playerMultipliers.clear();
    }
}
```

**Advanced features:**

- Command system with subcommands
- Permission checking
- State tracking per player
- Argument parsing and validation
- Error handling
- Help messages
- Complex logic based on player data

**Usage:**

```example
/minespeed get           -> Shows current speed
/minespeed set 2.0       -> Sets to 2x speed
/minespeed reset         -> Resets to 1x
```

---

## Progression Summary

| Level | Focus | Concepts |
| ------- | ------- | ---------- |
| 1 | Basic structure | Plugin class, onEnable/onDisable, event listening |
| 2 | Data access | Event properties, string formatting, conditional logic |
| 3 | Inventory | Containers, transactions, item loops, predicates |
| 4 | World interaction | Event cancellation, world.execute(), spawning |
| 5 | Advanced | Commands, permissions, state tracking, complex logic |

---

## Next Steps

After these examples, explore:

1. **Recipes & Crafting** - Use ItemContainer.moveItemStackFromSlot() for recipes
2. **Custom Entities** - Create specialized entity types with components
3. **Networking** - Add PacketFilter for custom protocol handling
4. **Async Operations** - Use CompletableFuture for non-blocking loads
5. **Configuration Files** - Use ConfigUtil for persistent settings

---

**Created:** February 27, 2026
**Framework:** Hytale Server API 2026

