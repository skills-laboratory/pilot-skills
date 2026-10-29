---
name: before-takeoff
description: Prepare the Zibo 737-800X on the ground for takeoff: verify weight-dependent data, flaps, trim, MCP settings, autobrake, speedbrake, transponder, rudder trim and Master Caution. Use when the engines are running and the airplane is on the ground before the brake release.
---

# Before takeoff

Use `fly-the-airplane` for how the tools work. The FMC route and performance data come from `fmc-setup`; the roll itself is `takeoff-straight-ahead`. While stopped, read every section of the `taxi` category before `takeoff` (`read_checklist` with `category`), even if already lined up and no taxi movement is needed: its configuration checks still apply. Do and verify unfinished applicable actions, mark only confirmed items, and review each section's `completed` and `total` before brake release; explain unavailable or inapplicable checks rather than falsely marking them. Then review the `takeoff` category and mark each item as soon as you have done it and confirmed it.

## Verify the data

Read `takeoff-data` (`gross-weight`, `fuel-total`, `trim-calculated`, `v1-set`, `vr-set`, `v2-set`) and `v-speeds.v1`, `vr`, `v2`: they must be non-zero and calculated for this weight. Do not reuse fuel quantity, cruise altitude or takeoff speeds from a previous flight without checking them with the user. Obtain the current departure and arrival charts: `list_documents` with kind `chart` and the airport ICAO, then `read_document`.

## Configure

1. Flap lever to the takeoff flaps and stabilizer trim to `trim-calculated` (`cockpit_flight_controls` `set-stabilizer-trim` once with the target in units; the server trims up or down itself, so never click trim in a loop; verify with `flight-controls.stabilizer-trim`). `flaps-match-fmc` and `trim-matches-fmc` must be true.
2. Autobrake RTO (`autobrake` 0), speedbrake armed (`speedbrake` 1), transponder TA/RA (`cockpit_lights_and_transponder` `transponder-mode` 5), lights as the checklist asks.
3. Flight directors on and autothrottle armed. These are toggles: read the state with `get_function_states`, press once, read again. Arm the autothrottle once (`a-t-arm-toggle`) and read `autothrottle-armed`; pressing it again switches it off.
4. MCP: speed to the V2 you read (`set-speed`), heading to the runway heading (`set-heading`), altitude to the cleared initial altitude (`set-altitude`). Without ATC use an altitude consistent with the published departure restrictions and terrain; do not reuse a previous setting.
5. On the ground `lnav` and `vnav` do not arm: select them after takeoff.
6. Nav radios: tune with `set-nav-1-standby-frequency` then `nav-1-swap`, never by writing the radio datarefs.
7. Read `flight-controls.rudder-trim` and `aileron-trim`: both must be 0 before the brake release (see `takeoff-straight-ahead`).
8. `master-caution` must read false: press `master-caution-captain` (it reports `verified` when the light goes out) and read it again. No six pack lamp may be lit without an explanation (see `cold-and-dark-start`, Master Caution and the six pack).

Release the parking brake only when the airplane is ready to roll. Then continue with `takeoff-straight-ahead`.
