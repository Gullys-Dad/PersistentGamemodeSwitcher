# PersistentGamemodeSwitcher

A fork of MagicSweet's client-side Fabric mod, published at `github.com/Gullys-Dad/PersistentGamemodeSwitcher`. It lets the F4 gamemode switcher screen open and work on a server where the client has no gamemode permission. Read `GULLYCRAFT.md` at the workshop root first.

This is a **client mod**, not a server plugin. Nothing in here runs on GULLYCRAFT. The fork exists to keep the mod working on current Minecraft; `gradle.properties` pins Minecraft 26.2, Fabric loader 0.19.3 and Fabric API 0.155.2, while upstream's own links still point at MagicSweet's CurseForge page and issue tracker.

## What it actually does

Vanilla's gamemode switcher checks a permission client-side and sends a `ServerboundChangeGameModePacket`. Two mixins change that:

- `GameModeSelectionScreenMixin` redirects the permission check to return `true` so the screen is usable, and redirects the packet send into `/gamemode <mode>` sent as a chat command instead.
- `KeyboardMixinGameModeSwitch` intercepts `handleDebugKeys` so the mod's own keybinds take over F4 and the cycle key.

Keybinds come from `KeyboardHotkeys`: F4 opens the switcher, and a second unbound keybind cycles spectator and back to the previous mode. Both are registered in a static initialiser, which is why `initialize()` exists as an empty method, called from the client entrypoint just to force class loading.

**The permission bypass is cosmetic.** It only fakes the client's own check. The actual gamemode change is a `/gamemode` command, which the server authorises normally. A player without permission to run `/gamemode` gains nothing but an open menu. Say that plainly if it ever comes up, because the mod's name and description both suggest more than it does.

The server entrypoint exists only to log two warnings saying the mod is client-side and can be deleted.

## Layout

```
PersistentGamemodeSwitcher.java              server entrypoint, logs a warning and nothing else
client/PersistentGamemodeSwitcherClient.java client entrypoint, open and cycle actions
client/KeyboardHotkeys.java                  the two keybinds and their tick handlers
client/mixin/GameModeSelectionScreenMixin    permission redirect, packet to command redirect, close handling
client/mixin/KeyboardMixinGameModeSwitch     takes over the debug key path
```

`persistentgamemodeswitcher.mixins.json` lists both mixins, client side only, `defaultRequire: 1`, so a mixin that stops matching is a hard crash at launch rather than a silent no-op.

## Build

```
./gradlew build
```

Jar lands in `build/libs`. Version `1.0.3.14`, group `com.magicsweet` (the fork kept upstream's package and group).

**Mixins are the fragile part.** Both target obfuscation-sensitive internals: `GameModeSwitcherScreen.switchToHoveredGameMode`, `PermissionCheck#check`, `ClientPacketListener#send`, `KeyboardHandler#handleDebugKeys`. Every Minecraft update can break any of them, and the symptom is a crash on launch with a mixin apply failure, not a quiet regression. Updating this mod means re-checking those four targets against the new mappings. `keyEvent.key() == 78` and `event.key() == 292` are raw GLFW key codes (N and F4) left from upstream; they are magic numbers but they are correct.

## Keeping it honest with the server

`other/approved-mods.md` in the wiki is the player-facing list of what is allowed on GULLYCRAFT. A client mod that bypasses a client-side permission check is exactly the kind of thing players will ask about. If this mod is for Cole's own use, it does not belong on that page; if it is ever approved for players, the page and the server's actual `/gamemode` permissions have to agree. Do not add it to the wiki without asking.

MIT licensed, upstream's license file kept. Keep the attribution to MagicSweet in `fabric.mod.json` if you touch it.
