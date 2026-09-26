# DiceRPG (Fabric Mod)

A Fabric mod for Minecraft 1.21.11: 12 elemental dice items, each with its own unique roll effects.
This is a rewrite of an earlier Paper-plugin version of the same idea - this one is a real Fabric
mod that produces a `DiceRPG-FABRIC-1.21.11-1.0.0.jar` style file for your server's `mods/` folder.

## How it works
- New players are auto-granted a random dice on first join (toggle with `giveDiceOnFirstJoin` in
  `DiceRPGFabric.java`).
- `/dice` (requires permission level 2, i.e. op) opens a menu where the player running it picks
  one of the 12 dice.
- `/reroll [player]` (op only) gives a Reroll Token. Right-clicking it swaps your current dice for
  a new random one.
- Right-clicking a dice item rolls 1-10:
  - **1** → a strong negative effect (unique per dice)
  - **2-4** → a weaker negative effect
  - **5-6** → a mild positive effect
  - **7-9** → a stronger positive effect
  - **10** → each dice's ultimate effect
  - There's a 60s cooldown between rolls per player (`CooldownManager.cooldownSeconds`).

## The 12 dice
Wind, Fire, Water, Ice, Feather, Spider, Magma, Air, Earth, Space, Lightning, Void.

Each dice's roll logic lives in `DiceType.java` - search for the dice's name (e.g. `FIRE(`) to
tweak its potion effects or messages. Items are registered in `ModItems.java`, and their in-game
icons currently reuse vanilla textures (feather, blaze powder, obsidian, etc.) via the model JSONs
in `src/main/resources/assets/dicerpg/models/item/` - swap those for real custom textures whenever
you want a unique look.

## Building
Requires JDK 21 and an internet connection (Gradle will download Minecraft, Yarn mappings, and
Fabric API automatically - no separate install needed beyond the JDK).

```bash
./gradlew build
```

*(On Windows, use `gradlew.bat build`. You'll need a `gradlew`/`gradlew.bat` wrapper - if it's not
included, run `gradle wrapper` once with any local Gradle install, or open the project in IntelliJ
IDEA with the Fabric plugin, which generates it automatically.)*

The compiled mod will be at `build/libs/DiceRPG-FABRIC-1.21.11-1.0.0.jar`. Drop that into your
server's `mods/` folder alongside Fabric Loader and Fabric API, then restart.

**This mod is server-side only logic** (it doesn't touch rendering/client code), but it does
register real custom items, so for players to see proper names/icons in their inventory they do
need the mod installed client-side too, the same way you installed the Mocap mod. If you'd rather
players NOT need to install anything, that's possible but requires a different approach (reusing
vanilla item types instead of registering new ones) - let me know if you want that version instead.

## Important note on this build
I don't have internet access in the environment I built this in, so I could not run
`./gradlew build` myself to confirm it compiles against the exact 1.21.11 Yarn mappings. I used
the version numbers and API shapes that were current as of my research, but Fabric's internal APIs
shift fairly often between versions. If the build fails, the error will point at one specific
line/method - the most likely trouble spots, in order of likelihood, are:
1. `Item#use` method signature (in `DiceItem.java` / `RerollItem.java`) - some versions return
   `ActionResult`, others `TypedActionResult<ItemStack>`.
2. `ServerCommandSource#getPlayerOrThrow()` - may be named `getPlayer()` in some mappings.
3. `SoundEvents.UI_BUTTON_CLICK` - may need `.value()` appended (it's a `RegistryEntry` in some
   versions).

Paste me the exact compiler error and I'll fix it in one pass.

## Customizing
- **Cooldown / first-join behavior**: top of `DiceRPGFabric.java` / `CooldownManager.java`
- **Permissions**: change `hasPermissionLevel(2)` in `DiceCommands.java`
- **Dice effects**: `DiceType.java`
- **Item appearance**: `assets/dicerpg/models/item/*.json` and `ModItems.java`
