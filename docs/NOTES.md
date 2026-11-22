# FFXI Addon — Valkyrie

> [!NOTE]
> **Quick Reference**
> - **Path:** `C:\Windower\addons\Valkyrie\`
> - **Author / Version:** Areint / Alzade / `1.0.1`
> - **Commands:** `//valk`, `//valkyrie`
> - **Load Status:** On-Demand
> - **Origin:** Third-Party (`https://github.com/Tny5989/Valkyrie.git`)

Object-oriented packet-automation addon that streamlines Einherjar chamber registration, Glowing Lamp entry, and lamp disposal at the Hazhalm Testing Grounds Entry Gate.

## What the Code Does
- **Command & State Machine (`valkyrie.lua`, `command/`):** Accepts one complex command at a time (`NilCommand` guard) built by `CommandFactory.CreateCommand(cmd, p1)`, wrapping callbacks for success/failure back to idle.
- **Data Models (`data/`):**
  - `Npcs.lua`: Defines `Entry Gate` (`id = 17097342`, `zone = 78` Hazhalm Testing Grounds) and finds the closest NPC in the current zone.
  - `chambers.lua`: Coordinates/rotation and `idx` (`1..9`) / `tier` (`1..3`) for the 9 Wing I–III chambers (Rossweisse's through Brunhilde's; Odin's chambers are omitted per `README.md`).
  - `lamps.lua`: Item IDs for `Smouldering Lamp` (`5413`) and `Glowing Lamp` (`5414`).
- **Register Flow (`RegisterDialogue`):** Trades a `Smouldering Lamp` (`0x036`) to `Entry Gate`, blocks incoming menu packets (`0x037`, `0x034`/`0x032`, `0x05C`), parses chamber availability from `0x05C` bits `32..40` (`Chamber free` vs `Chamber occupied`), sends menu choices (`0x05B` via `Choice`), and completes on `0x02A`.
- **Enter Flow (`EnterDialogue`):** Trades a `Glowing Lamp` (`0x036`), reads the target chamber from menu packets (`0x034`/`0x032` & `0x05C`), injects `0x05C` (`Warp`) with the chamber's `x`, `y`, `z`, `r` coordinates, confirms via `0x05B` (`Choice`), and completes on `0x036`.
- **Drop Flow (`DropCommand`):** Loops through inventory dropping every `Glowing Lamp` (`5414`) via outgoing `0x028` (`Bag = 0`, `Count = 1`) with `1`-second sleeps.
- **Deduplication (`util/packets.lua`):** Tracks packet IDs on `0x034`, `0x032`, `0x05C`, `0x052`, `0x036`, `0x02A` to ignore duplicate incoming chunks.

## Commands & Usage
- `//valk register <chamber>` — Register a chamber using a Smouldering Lamp (`<chamber>` matches `en` in `data/chambers.lua`, e.g., `"Rossweisse's Chamber"`, auto-translate supported).
- `//valk enter` — Trade a Glowing Lamp to `Entry Gate` and warp into the registered chamber.
- `//valk drop` — Drop all Glowing Lamps from inventory.

## Events, Packets & Technical Hooks
- **Windower Events:** `load`, `addon command`, `incoming chunk`, `outgoing chunk`.
- **Packets:**
  - Outgoing (injected): `0x036` (lamp trade), `0x05B` (menu choice), `0x05C` (chamber warp coordinates), `0x028` (drop Glowing Lamp).
  - Incoming (monitored/blocked during dialogue): `0x032`, `0x034`, `0x036`, `0x037`, `0x052`, `0x05C`, `0x02A`.
- **Dependencies / Libraries:** `logger`, `extdata`, `packets`, `config` (in `util/settings.lua`), plus modular `command/*`, `data/*`, `model/*`, `util/*` files.

## Local Settings & Customizations
From `data/settings.xml`:
- `<maxdistance>25</maxdistance>` (default).

## Related Notes
- `FFXI Windower Overview`
- `FFXI Windower Addons Catalog`
