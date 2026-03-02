dukaci - SortingPlus
Patches for SortingPlus by RavenAscendant.

Requires: 110- SortingPlus - RavenAscendant
Place below SortingPlus in MO2 load order.

Changes:

1. Favorites sort to top of inventory grid, grouped by kind.
   Stock behavior lumps all favorites into one flat group.
   This keeps weapons with weapons, food with food, etc.
   Modified: zzz_rax_sortingplus_mcm.script (FindFreeCell)

2. Per-item keep limits on Put All.
   Right-click a favorited item -> "Keep Limit" -> enter a number.
   Put All then keeps only that many units and deposits the rest.
   Handles ammo stack splitting and multiuse item splitting.
   Limits persist across saves.
   Files: zzzz_rax_sortingplus_keepcaps.script,
          ui_sp_keepcaps.xml,
          ui_st_sp_keepcaps.xml
