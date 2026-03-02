# dukaci - SortingPlus

Patches for `110- SortingPlus - RavenAscendant`.

## Features

1. **Favorites sort to top by kind** — favorited items form their own category, optionally grouped by kind (weapons with weapons, food with food) or all in one row
2. **Per-item keep limits on Put All** — right-click a favorited item → set limit → Put All deposits excess. Splits ammo stacks and multiuse items at the boundary. Persists across saves
7. **Stock Up button** — appears next to Take All on stash containers. Takes favorited items from container to fill up to each keep limit. Splits ammo/multiuse at the boundary. Only shows on stash containers (not corpses/companions)
3. **Per-kind MCM sort order** — each category has a configurable sort value with optional within-row priority (decimal part)
4. **Junk category** — mark items as junk, sort to end, highlight/icon separately
5. **Context menu** — toggle favorites/junk via right-click (if functors enabled)
6. **ARM magazine support** — loaded magazines get their own sort category

## Files

| File | Purpose |
|------|---------|
| `gamedata/scripts/zzz_rax_sortingplus_mcm.script` | MCM menu builder, sort comparators, settings loader, FindFreeCell override |
| `gamedata/scripts/zzzz_rax_sortingplus_keepcaps.script` | Keep-limit logic for Put All + Stock Up button |
| `gamedata/scripts/rax_dynamic_custom_functor.script` | Right-click menu support |
| `gamedata/scripts/rax_icon_layers.script` | Dynamic icon layer rendering |
| `gamedata/scripts/rax_persistent_highlight.script` | Persistent cell coloring |
| `gamedata/configs/sortingplus.ltx` | Sort defaults, groups, overrides, within-kind priority |
| `gamedata/configs/ui/ui_sp_keepcaps.xml` | Keep-limit dialog + Stock Up button layout |
| `gamedata/configs/text/eng/ui_st_sortingplus_unique.xml` | English MCM strings |
| `gamedata/configs/text/eng/ui_st_sp_keepcaps.xml` | Keep-limit + Stock Up strings (eng) |
| `gamedata/configs/text/rus/ui_st_sp_keepcaps.xml` | Keep-limit + Stock Up strings (rus) |
| `gamedata/configs/text/rus/ui_st_sortingplus_unique.xml` | Russian MCM strings |
| `gamedata/textures/ui/sortingplus/icon-favourite.dds` | Favorite marker overlay |
| `gamedata/textures/ui/sortingplus/icon-junk.dds` | Junk marker overlay |

## Architecture

### MCM Menu (`on_mcm_load`)

Built dynamically:
1. Reads kind defaults from `[sortingplus_kind_defaults]` in LTX
2. Reads display groups from `[sortingplus_kind_groups]` in LTX (ordered)
3. Scans `ini_sys` for undiscovered kinds → adds to "Discovered" group
4. Generates widgets: title per group, number input per kind

Labels resolved by MCM via `ui_mcm_SortingPlus_<id>` string lookup — no hooks.

### Sort Value System

Each item category has a numeric sort value (e.g. `2.1`):
- **Whole number** = row group (same number = same inventory row)
- **Decimal** = position within that row (2.1 before 2.2)
- `subkindprio` toggle: when OFF, `math.floor()` flattens values so decimals are ignored

### Sort Comparators

Five functions override `utils_ui`:

| Function | Primary sort | Fallback |
|----------|-------------|----------|
| `sort_by_kind` | Kind value | Size |
| `sort_by_sizekind` | Width → height | Kind → within-kind priority → alpha → condition |
| `sort_by_size` | Within-kind priority | Width → height → alpha |
| `sort_by_props` | Same-section only: ammo count → upgrades → condition |
| `sort_by_index` | Index-based |

### FindFreeCell Override

Tracks kind-based row separation in `sort_method == "kind"`:
- Favorites with `FAVSKINDROWS`: sort order = `-1000 - kind_value` (above all, keeps kind separation)
- Favorites without kind rows: sort order = `-1000` (single row)
- Compares integer parts to detect kind change → starts new row

### Settings (`loadsettings`)

Called on MCM change. Clears caches (`item_order`, `ab_k`, `item_sort_prio`, `kind_overrides`). Rebuilds sort order from either LTX (`useltx`) or MCM inputs. Applies `math.floor()` if `subkindprio` is off.

### Keep Limits (Put All Override)

1. Pre-scan actor inventory for favorited items with limits
2. Group by section, sort by condition then units
3. Accumulate greedily; mark excess for deposit
4. Call original Put All
5. Split border items: trim kept units, create new item with excess in NPC stash
6. Excludes artefact melter (`itm_artefactskit`) from splitting

### Stock Up Button

Monkey-patches `InitControls`, `InitCallbacks`, and `Reset` on `UIInventory`:
1. Shrinks Take All to 90px, adds Stock Up button (90px) to its right
2. Only visible on stash containers (`npc_is_box`), not corpses or companions
3. `LMode_StockUp` — inverse of Put All:
   - Count player units per favorited+limited section
   - Calculate deficit (limit − player count)
   - Scan container for matching items, sort best condition first
   - Take greedily; split ammo/multiuse at boundary (same exclusions as Put All)
   - Sections with no explicit limit (nil = "keep all") are skipped

### Highlighting & Icons

- `rax_persistent_highlight`: favorites = green (`ARGB 50,176,238,26`), junk = red (`ARGB 50,204,0,0`)
- `rax_icon_layers`: overlays 15x15 icons (top-left for favorites, top-right for junk)

### Keybinds

- `K` (DIK_K): Toggle favorite
- `J` (DIK_J): Toggle junk

## MCM Settings

| ID | Type | Default | Description |
|----|------|---------|-------------|
| `hilitefavs` | check | true | Highlight favorited items |
| `iconfavs` | check | true | Show favorite marker icon |
| `sortfavs` | check | true | Favorites as separate sort category |
| `favskindrows` | check | true | Favorites grouped by kind (vs single row) |
| `putallfavs` | check | true | Block Put All for favorites |
| `hilitejunk` | check | true | Highlight junk items |
| `iconjunk` | check | true | Show junk marker icon |
| `sortjunk` | check | true | Junk as separate sort category |
| `selljunk` | check | true | Auto-move junk to trade pane |
| `playerinv` | check | true | Sort player inventory |
| `playertrade` | check | true | Sort player trade inventory |
| `npcinv` | check | true | Sort NPC/container inventory |
| `npctrade` | check | true | Sort NPC trade inventory |
| `resort` | check | false | Re-sort immediately on mark |
| `usefunctors` | check | true | Right-click menu for favorites/junk |
| `useltx` | check | false | Use LTX sort order instead of MCM inputs |
| `subkindprio` | check | true | Within-row priority (decimal part matters) |

Plus dynamic per-kind number inputs (`ko_<kind>`) built from LTX defaults.

## LTX Sections (`sortingplus.ltx`)

| Section | Purpose |
|---------|---------|
| `[item_kind_order]` | Legacy sort order (used when `useltx` checked) |
| `[kind_overrides]` | Force items into different kinds (+ automatic prefix splits: `prt_w_*` → `i_part_w`, `prt_o_*` → `i_part_o`) |
| `[item_sort_priority]` | Within-kind ordering for individual items (lower = first, default 1000) |
| `[sortingplus_kind_defaults]` | Default sort value per kind (populates MCM inputs) |
| `[sortingplus_kind_groups]` | Display grouping for MCM UI (order = display order) |

## Adding a New Kind / MCM Group

No script changes needed — the system is fully data-driven.

### 1. Register the kind in `sortingplus.ltx`

Add an entry to each of these sections:

```ini
[item_kind_order]
my_new_kind = 15

[sortingplus_kind_defaults]
my_new_kind = 15
```

To put it in an existing group, append it to that group's line in `[sortingplus_kind_groups]`:

```ini
[sortingplus_kind_groups]
grp_equipment = i_device,i_tool,i_repair,i_kit,my_new_kind
```

To create a new group, add a new line (position in the section = position in MCM):

```ini
[sortingplus_kind_groups]
grp_mygroup = my_new_kind,another_kind
```

### 2. Add XML strings

In `ui_st_sortingplus_unique.xml` (both `eng/` and `rus/`):

```xml
<!-- Group header (only if adding a new group) -->
<string id="ui_mcm_SortingPlus_grp_mygroup">
    <text>My Group</text>
</string>

<!-- Kind label (shown next to the number input) -->
<string id="ui_mcm_SortingPlus_ko_my_new_kind">
    <text>My New Kind</text>
</string>

<!-- Kind tooltip (shown on hover) -->
<string id="ui_mcm_SortingPlus_ko_my_new_kind_desc">
    <text>Description of what this kind contains.</text>
</string>
```

### String ID convention

| Element | Pattern |
|---------|---------|
| Group header | `ui_mcm_SortingPlus_` + group id (e.g. `grp_mygroup`) |
| Kind label | `ui_mcm_SortingPlus_ko_` + kind name |
| Kind tooltip | `ui_mcm_SortingPlus_ko_` + kind name + `_desc` |

### Auto-discovery fallback

If a mod adds items with a `kind` value not in `[sortingplus_kind_defaults]`, the script discovers it at runtime by scanning `ini_sys`, assigns default sort value `30`, and appends it to a "Discovered" group at the bottom of MCM. It works without config — it just won't have a nice label.

## Global State

| Variable | Purpose |
|----------|---------|
| `item_order` | kind → sort value (float) |
| `kind_overrides` | section → overridden kind |
| `item_sort_prio` | section → within-kind priority |
| `favorite_itms` | section → "favorites" (persisted) |
| `junk_itms` | section → "junk" (persisted) |
| `favorite_limits` | section → keep count (persisted, in loadout limits script) |
| `ab_w`, `ab_h`, `ab_k` | item dimension/kind caches |

## Callbacks

| Callback | Purpose |
|----------|---------|
| `load_state` / `save_state` | Persist favorites, junk marks, keep limits |
| `actor_on_first_update` | Initial settings load |
| `on_option_change` | MCM change → reload settings |
| `ActorMenu_on_item_before_move` | Block favorites from Put All, allow excess deposits |
| `GUI_on_show` | Auto-move junk to trade pane |
| `on_key_press` | K/J keybinds |
