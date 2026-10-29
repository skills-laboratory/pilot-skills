---
name: takeoff-straight-ahead
description: Take off the Boeing 737-800 in the simulator and climb straight ahead to a cleared altitude. Use when rolling, rotating, or climbing on runway heading with HDG SEL and LVL CHG, including yoke, pedal, thrust-lever, and rudder-trim checks before brake release.
---

# Takeoff straight ahead

Climb on the runway heading to the cleared altitude and level there. No turns and no other maneuvers.

Tool names below are the bare names; the client may prefix them with the server. Cockpit changes use the `cockpit_*` tools of the `flightsim-pilot` server (`cockpit_engines_and_thrust`, `cockpit_brakes_flaps_and_gear`, `cockpit_flight_controls`, `cockpit_mcp_and_autopilot`, `cockpit_lights_and_transponder`), each with an `action` and, for setpoints, a `value`. Read values with `read_reading` / `read_group` (ids such as `flight-controls.rudder-trim`). A `cockpit_*` call returns `ok`, `verified`, `before`, `after` and `detail`: `verified: true` means the change was observed. Read the airplane afterwards in any case. Read each tool schema before calling it.

The autopilot does not steer the nosewheel. Heading select with the autopilot off is guidance only. A roll that leaves the runway heading is already a failed takeoff; more thrust makes it a crash.

Check that the airplane is progressing straight. That check matters more than the flight computer. The FMC, the heading bug, and HDG SEL do not keep the nosewheel on the centerline. The pedals do.

## Before the parking brake is released

Pitch, roll, and rudder inputs only reach the airplane while `joystick-override` is on. Turn it on (`cockpit_flight_controls` `joystick-override`, a toggle) and read `flight-controls.joystick-override` back.

Read `flight-controls.pitch-control`, `roll-control`, `rudder-control`, and `engines-and-thrust.throttle-left` / `throttle-right` while the airplane is stopped. All of them must be zero. Set whatever is not zero back to zero and read it again. A yoke or a pedal left off center, or one lever left up from an earlier attempt, will yaw or roll the airplane as soon as it moves. Pitch trim is not one of these zeros; set that from the FMC takeoff trim below.

The `rudder-control` reading is the pedals, not the rudder trim. It can read 0 while the trim is holding the nose. Before the brake is released, read `flight-controls.rudder-trim` and `flight-controls.aileron-trim`. Both must be 0. On the Sydney 16R takeoff the airplane was lined up on heading 155 with the pedals at 0, both thrust levers at 0, and the engines matched, and rudder trim was -1. That is full left. As soon as the airplane rolled, the nose left the runway heading and ended near 50 degrees with the pedals still reading 0. Pedal input does not cancel that trim. Do not release the brake until `rudder-trim` reads 0. There is no action that sets the rudder or aileron trim; ask the user to fix it if it is not 0.

Prove the pedals work before the roll. Command a small `rudder-control` setpoint, read `rudder-control` back, then return it to 0 and read 0 again. A write can be accepted and still leave the previous value in place. As a second check, `flight-controls.rudder-input` (the real pedal position) must follow the same deflection. A timeout that leaves the pedals at 0 is not proof of steering. A timeout that leaves them off center is a pedal still deflected. Return both to 0 and read 0 before the brake is released. Do not roll with the test deflection still in.

Read the manual before the roll. Use `flightsim-pilot` `list_documents` with category `aircraft` and kind `manual`, then `read_document_chapter` on `aircraft-boeing_737_800-manual`. Takeoff pitch, the 400-foot roll-mode and flap step, and autopilot engagement are in chapters `section.61d8ed4d315b2284.1` and `section.e086c6cc41412562.1`. Do not invent a procedure from yoke, rudder, and trim alone.

Confirm the airplane is stopped and the heading is steady (`flight-instruments.heading`). That heading is the runway heading. Set the heading bug to it with `cockpit_mcp_and_autopilot` `set-heading`, and read `mcp-and-autopilot.selected-heading`. A bug a few degrees off will not be flown until the autopilot is engaged, but it must match the runway before the roll.

Set the altitude window to the cleared altitude with `set-altitude` and read `mcp-and-autopilot.selected-altitude`. It is a direct value (the old step clicks are not needed).

Set the speed window with `set-speed` (100 to 400 knots). The minimum is below a safe climb speed. For a light 737-800 (about 4,500 kg of fuel) with flaps 5, use about 160 knots until the flaps are up. Do not command 250 until the flaps are retracted. Flaps 5 is 37.5 percent on `brakes-flaps-and-gear.flap-lever`; leave the takeoff flap setting that the taxi checklist already accepted.

Read `flight-controls.pitch-trim`, `flight-controls.stabilizer-trim`, and the pitch attitude while the airplane is still stopped. On the crashed takeoff the pitch trim sat near -0.39 for the whole roll, and at about 136 knots the nose was still about half a degree below the horizon. That is not a takeoff trim. Speed without a nose-up stabilizer is not a stable runway run, and hauling the yoke against it will not fix it.

Check the takeoff trim against the FMC. `read_group` on `takeoff-data` returns `trim-calculated` (the FMC takeoff trim) and `trim-matches-fmc` (true once the stabilizer is at that trim) next to `gross-weight`, `fuel-total`, the entered `takeoff-flaps` with `flaps-match-fmc` (true once the flap lever is at those flaps), and the V-speeds (`v1-set`, `vr-set`, `v2-set` and the `-calculated` ones); `flight-controls.stabilizer-trim` is where the stabilizer actually is. They must agree for the current weight: a trim or V-speed that was calculated for another weight is not a takeoff setting. If the CG is `--.-%`, `trim-calculated` is 0 or the set V-speeds are 0, the takeoff data has not been entered: do step 10 of `fmc-setup` (the CG in % MAC, then accept V1, VR and V2) before the stabilizer and the brake. The CDU is typed with `cockpit_fmc_cdu` `cdu-type` and read from the `fmc-cdu` lines. Do not guess a ratio and do not release the brake on a value left over from the last flight. If you must change the elevator trim, `pitch-trim` (a ratio from -1 to 1) writes it; read `pitch-trim` back.

A verified `pitch-trim` of 0 is not a centered stabilizer. On the Zibo 737, writing the elevator trim to 0 made the ratio read 0, while `stabilizer-trim` stayed near 5.0 units and the stabilizer deflection stayed near -5.5 degrees. That airplane then rotated itself from about +1.5 degrees at 115 knots to about +34 degrees and 1,100 feet before the next pitch sample. Resetting the ratio is not the takeoff trim, and it does not prove the surface moved: check `stabilizer-trim`.

Arm the flight director, read it on, then arm the autothrottle and read the annunciator. These are toggles: read `flight-director` and `autothrottle-armed` first (`get_function_states`), press `flight-director-toggle` or `a-t-arm-toggle` only if off, then read again. Only then press `hdg-sel` by itself and read `fma-roll`. Roll must show `HDG SEL` and `mcp-and-autopilot.hdg-sel` must be on. A heading-select press sent together with the flight director, the autothrottle, the lights, and the transponder can be accepted and still leave the light off and the roll mode blank. Press it again, alone, until `fma-roll` shows HDG SEL. Do not press `lnav`: it follows a route and can turn. Do not press `alt-hld` on the ground: it holds the altitude you have now, which is the field. Do not press `vnav` or `lvl-chg` on the ground. Heading select is the roll mode and is guidance only; the autopilot is `cmd-a`. On the ground, with the autothrottle armed, LVL CHG can command climb thrust before you are tracking the centerline. The manual selects VNAV only after the flaps and slats are retracted.

`list_documents` and the checklist both still apply to lights, transponder TA/RA (`transponder-mode` with value 5), and the 40 percent N1 stabilization. 40 percent N1 is an engine indication, not a thrust-lever percent. A lever near 35 percent can already be about 60 percent N1. Read both `n1-left` and `n1-right`.

## The roll

Release the parking brake (`parking-brake-toggle`; read `brakes-flaps-and-gear.parking-brake` first and toggle only if it is set, i.e. at or above 0.5; a released brake reads about 0.2 on this Zibo, not 0) only when the yoke, pedals, both thrust levers, and rudder trim read zero, the rudder has read back, and the heading bug, pitch trim, altitude window, HDG SEL, flight director, and autothrottle arm are verified.

Bring both levers up together (`cockpit_engines_and_thrust` `throttle`, a percent that moves both levers) only while the heading stays on the runway heading and the nose is not drifting down. After each thrust change, read heading first, then `rudder-control`, both N1 values, pitch, and airspeed. From about 100 knots, also read `height-agl` on that same pass. The engines must match. A higher left N1 yaws the nose left. If the heading moves off the runway heading, move the pedals immediately to stop the yaw and read `rudder-control` back. Do not leave the pedals at 0 while the nose walks off, and do not add thrust until the heading is back on the runway heading. As airspeed comes alive, pitch should move toward the rotation attitude. If the nose stays on or below the horizon through flying speed, the stabilizer is still wrong: idle both levers and stop. Do not add thrust and do not fight a mistrimmed stabilizer with pitch control.

On the light 737-800 that leveled at 5,000 feet, the lever-to-N1 map with both levers together was about 18 percent lever for 45 percent N1, 32 percent lever for 61 percent N1, and 70 percent lever for 87 percent N1. Heading stayed on 155 while the pedals and rudder trim were 0. The miss was the pitch scan. At 115 knots the nose was only about +1.5 degrees. The next sample, still during a thrust step, was 130 knots, +34 degrees of pitch, and 1,100 feet height. That is already past a 15-degree rotation and well above the 400-foot autopilot gate. If pitch is rising through 10 degrees, stop adding thrust, command one pitch attitude aimed at 15 degrees, and read `pitch` back before another pitch command. The manual's check is the pitch indication: reaching a control ratio is not reaching 15°. Do not find a 30-degree nose at four-digit height. The 400-foot gate is where the autopilot takes over, together with the roll mode and the flap schedule; the answer to a nose that is too high is a small input, not a large one.

If the heading changes while `rudder-control` stays at 0 and both N1 values match, idle and stop while the heading is still near the runway heading, then read `rudder-trim` again. A full-left trim yaws the airplane with the pedals centered. Do not add thrust to steer out of it.

Moving a thrust lever by hand disarms the autothrottle: `autothrottle-armed` goes false and `fma-autothrottle` goes blank, while the readings of the levers and N1 look normal. `to-ga-left` does nothing useful with the autothrottle disarmed; it does not clear the arm itself. So take off thrust with one of two sequences, never a mix:

- TO/GA: leave both levers at idle, arm the autothrottle (read `autothrottle-armed` true), press `to-ga-left` once. Measured on the ground, the autothrottle goes to N1 mode, both levers move to about 89 percent and N1 reaches 98 percent in about 15 seconds, with the arm still set. This does not need the 40 percent N1 stabilization step by hand.
- Manual thrust: raise the levers with `throttle` and stabilize at 40 percent N1 as the checklist says. This disarms the autothrottle. Arm it again and read `autothrottle-armed` true before you press `to-ga-left`; the press then takes both engines to about 98 percent N1 in a few seconds.

After any TO/GA press, read `autothrottle-armed`, `fma-autothrottle` and both N1 values. If the arm is false, the autothrottle was disarmed by an earlier lever move: arm it again once and press `to-ga-left` once. If the arm is true and the levers did not move, do not press TO/GA again and do not push the levers to the stop to compensate. TO/GA also replaces HDG SEL with the takeoff roll mode; select `hdg-sel` again above 400 feet.

Abort the roll if any of these are true:

- Heading differs from the runway heading by more than about 5 degrees before rotation.
- Pitch stays on or below the horizon as airspeed reaches rotation.
- Rudder trim is not 0, a pedal input does not read back, or `rudder-control` stays at 0 while the heading is changing.
- Pitch trim has not been checked against a known takeoff trim.
- The engines are split, or either N1 is not what you commanded.
- Airspeed, heading, or attitude readings fail.

Abort means both thrust levers to idle (`throttle` 0), then stop. Do not rotate, and do not add power to "fly out of" a heading change. A departure that went from runway heading through 20 degrees off at about 70 knots, then 80 degrees off at rotation speed, was already lost; takeoff N1 near 100 percent on that heading left the pavement and crashed.

## After a stable liftoff

`pitch-control` is the elevator position as a fraction of full travel (-1 to 1; positive raises the nose). It is not a pitch angle and not a rate, and it stays where you put it until you write another value. The same value does far more at high speed: a modest input at about 195 knots gave +10,000 feet per minute and a stall at 90 knots within 15 seconds.

**Time is the problem, not the technique.** With TO/GA the airplane gains 6 to 8 knots per second and passes VR about 15 to 20 seconds after N1 reaches 90 percent. Every call takes 3 to 10 seconds, so the airspeed you read is already out of date when you act on it. Pilots that called `mark_checklist` or read other groups during the roll first rotated at 175 to 195 knots, pulled hard, stalled, pushed hard the other way, and then oscillated between a climb at 10,000 feet per minute and a dive until they crashed.

- Before releasing the brake, read `v-speeds.vr` and `v-speeds.v2` and keep them in mind.
- From brake release until the gear is up, make no other calls: no checklist, no MCP presses, no extra reading groups. Each cycle is one `read_group` with `flight-instruments.airspeed`, `flight-instruments.pitch` and `flight-instruments.height-agl`, then at most one control call.
- Start the rotation a little before VR: your call lands a few seconds after you decide, so use the airspeed `trend` and begin when the time to VR is about as long as a call takes. Do not wait to read VR exactly.
- Make the first input small, then read `pitch` and its `trend` straight after it, before another control or configuration call. Anticipate the pitch when your next change reaches the airplane: add the positive pitch trend times the seconds needed to read, decide and act. Ease the input back toward 0 before that projected pitch reaches 15 degrees; waiting to read 15 lets a held input pull through the target.
- Always base the next input on the reading you have just made, never on the previous input. One change per read, and never a large one: its effect grows with airspeed. If the nose is above 20 degrees or the speed is falling, reduce the remaining nose-up input toward 0 on the next control call and read pitch and speed immediately; configuration and checklist work must not delay this correction. Do not jump to a large value on the other side. If you rotated late the airplane is already flying fast, so a small input is enough.
- Cut `throttle` to about 70 percent once the climb is positive so the speed does not run away.

From 400 feet `height-agl`, with HDG SEL and the pitch mode set and the airplane following the flight director, engage `cmd-a`. The autopilot holds pitch and heading far better than hand inputs, and it is the way out of this manual phase: do not hand-fly beyond 400 feet. Do not wait for flap retraction to engage CMD A; retract flaps separately at their scheduled speeds. Release `joystick-override` and stop writing `pitch-control` and `roll-control` once CMD A is on (`mcp-and-autopilot.cmd-a` true). Heading select and the flight director alone are guidance and do not fly the airplane.

At a positive climb, gear up: `gear` with value 0. Read the three `*-gear-down` and `*-gear-transit` readings. Above 400 feet (`height-agl`), with the heading still the runway heading and the wings level, select or verify the roll mode and retract flaps as scheduled (`flaps` with value 1, then 0). The manual says not to treat 400 feet as an acceleration height or a fixed flap-retraction speed. For this straight-ahead climb the roll mode is HDG SEL. Reselect it if TO/GA or another mode replaced it. Flaps 5 reads 37.5 percent on the lever; flaps 1 reads 12.5 percent. Keep the speed window at about 160 until the flaps are up. Do not command 250 until they are retracted.

After the flaps and slats are actually retracted, select VNAV as the manual directs; keep the command channel already engaged at the 400-foot handover:

1. Press `vnav` and read `fma-pitch`. If performance init is blank, cruise altitude is empty, or the nav data is out of date, VNAV will not level at the altitude window (you cannot see the CDU scratchpad here; a pitch mode that is not VNAV after the press is the sign). Then press `lvl-chg` instead and confirm `fma-pitch` shows `MCP SPD`. On this airplane LVL CHG annunciates as MCP SPD, not the words LVL CHG.
2. Confirm `fma-roll` still shows HDG SEL.
3. Read the command light and the FMA. Engage the approved channel with `cmd-a` or `cmd-b` only if it is not already on; pressing an engaged channel disconnects it.

Do those steps one at a time. Do not send a `pitch-control` change in the same batch as `lvl-chg` or `cmd-a`. After the command light is on, stop moving the yoke and the pedals. A pitch command sent while CMD A is engaging fights the autopilot: on the 5,000-foot departure the nose was at +34 degrees, the yoke was zeroed in the same step as level change and CMD A, and pitch then hunted between about 17 and 20 degrees until altitude hold captured. The autopilot was already in heading select. The yoke was the interference.

Once Command A is on, it flies the modes already selected:

- Heading select holds the heading in the window. It does not steer the nosewheel, so on the ground it is guidance only.
- VNAV is the pitch mode the manual selects after the flaps are up. If that mode will not capture the altitude window, MCP SPD (level change) pitches to hold the speed in the window and climb to the altitude in the window.
- At the selected altitude the pitch mode becomes ALT HOLD and the airplane levels off. That is the capture. Do not press `lvl-chg` again once MCP SPD or ALT HOLD is showing, and do not press `alt-hld` on the ground to try to create it.
- The autothrottle, armed before brake release, shows MCP SPD in the climb and sets thrust to hold the speed window.

The mode that held 5,000 feet on runway heading 155 was CMD A, HDG SEL, ALT HOLD, and autothrottle MCP SPD. Do not turn and do not change the heading bug.

The autothrottle arm can drop while those mode buttons are pressed, and `fma-autothrottle` goes blank. Re-arm it once (read the state first: it is a toggle), then read `fma-autothrottle`. ARM and then MCP SPD means it is back. Do not press TO/GA to restore it.

If command mode will not engage, stay on the flight director with small pitch and roll inputs and keep the heading. Do not disconnect a mode that is already tracking the runway in order to hand-fly a correction.
