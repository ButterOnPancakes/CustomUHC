# CustomUHC

CustomUHC is a modular Minecraft Ultra Hardcore (UHC) plugin project for Spigot. It provides the shared systems needed to host a UHC game, while allowing separate plugins to add game modes and content. The repository includes a Ninjago-themed mode as an example.

> This project is a work in progress and targets the legacy Spigot 1.8.8 API.

## Project layout

- `UHC/UHCAPI` — public Java interfaces and utility classes for interacting with the UHC framework.
- `UHC/UHCBase` — the main plugin implementation: game and world configuration, player state, timers, scenarios, menus, scoreboards, and other shared systems.
- `UHC/Modules/Ninjago` and `UHC/Modules/NinjagoV2` — game-mode plugins built on top of `UHCBase` and `UHCAPI`.

## Build

Build all Maven modules from the repository root:

```sh
mvn clean package
```

If you are developing an add-on outside this Maven project, run `mvn install` from the root first so the `UHCAPI` and `UHCBase` artifacts are available in your local Maven repository.

The project is configured for Java 8 and Spigot 1.8.8. Maven also declares WorldEdit 6.1.4 as a provided dependency for compilation. The generated plugin JARs are written to each module's `target/` directory. Install `UHCBase` on the server; install a mode plugin such as `Ninjago` alongside it to enable that mode. The mode plugin declares `UHCBase` as a dependency and will load after it.

## Using the API

Add `UHCAPI` to your plugin's Maven dependencies with `provided` scope. A mode plugin can also depend on `UHCBase` with `provided` scope, as the included Ninjago module does. This makes the API available at compile time without packaging duplicate framework classes in the add-on JAR.

For example, the dependencies in a module's `pom.xml` can look like this:

```xml
<dependency>
    <groupId>me.butter</groupId>
    <artifactId>UHCAPI</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>me.butter</groupId>
    <artifactId>UHCBase</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
```

Declare the base plugin as a dependency in your add-on's `plugin.yml` so Bukkit loads it first:

```yaml
depend:
  - UHCBase
```

`UHCAPI.getInstance()` is the entry point to the running framework. Its handler methods expose the main systems, including players, game configuration and state, worlds, timers, scenarios, items, menus, scoreboards, tabs, structures, and custom entities. For example, a Bukkit listener in an add-on can use it to find a player's UHC wrapper:

```java
import me.butter.api.UHCAPI;
import me.butter.api.player.UHCPlayer;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.player.PlayerJoinEvent;

public final class JoinListener implements Listener {
    @EventHandler
    public void onJoin(PlayerJoinEvent event) {
        UHCPlayer player = UHCAPI.getInstance()
                .getPlayerHandler()
                .getUHCPlayer(event.getPlayer());

        // Use the UHCPlayer API to interact with this participant.
    }
}
```

Add-on plugins can contribute content through the relevant handlers. For instance, scenarios are registered with `getScenarioHandler().addScenario(...)`; a full game mode can implement the `Module` interface and register it with `getModuleHandler().setModule(...)`. See `UHC/Modules/Ninjago` for a working example of a plugin that registers a module, scenarios, commands, and listeners.

The public API sources are under `UHC/UHCAPI/src/main/java/me/butter/api`. The framework is designed for the Bukkit/Spigot 1.8.8 environment, so newer server APIs may require code changes.
