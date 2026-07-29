# Hue Dimmer: Adaptive Lighting + Scene Cycle

A Home Assistant automation blueprint for Philips Hue Dimmer remotes connected through the official Hue Bridge integration. Home Assistant takes over the On button and cycles it through Adaptive Lighting and four Hue scenes, while brightness and off keep using their native Hue actions.

## What it does

- First **On** press, or any On press after the timeout: **Adaptive Lighting** (current-time adaptive look).
- Each additional **On** press within the timeout advances the cycle: **Adaptive, Rest, Relax, Bright, Concentrate,** then wraps back to Adaptive.
- Default timeout between presses: **3 seconds** (configurable per room, 1 to 10 seconds).
- **Brightness Up / Down** keep using native Hue dimming, and they end the cycle window and pause only brightness adaptation so color keeps adapting.
- **Off** keeps using the native Hue action, ends the cycle window, and resets the mode back to Adaptive for the next press.
- Each room gets its own remote, grouped light, Adaptive Lighting instance, scenes, helpers, timeout, and transition.

## Requirements

Read these before importing. The blueprint will import without them, but it will not behave correctly.

- **Home Assistant 2024.6.0 or newer.**
- **The official Hue Bridge integration.** The remote must be paired to your Hue Bridge and exposed through the Hue integration in Home Assistant. This blueprint uses `domain: hue` device triggers on `initial_press`. It will **not** work with Zigbee2MQTT, ZHA, or deCONZ. If the remote is paired to one of those instead, it will not even appear in the remote selector, and a different blueprint would be needed.
- **Adaptive Lighting 1.30.0 or newer.** Older versions do not support pausing brightness adaptation while color keeps adapting, which the brightness buttons rely on.
- **`take_over_control` enabled on each Adaptive Lighting instance.** The blueprint uses `adaptive_lighting.set_manual_control`, which only functions when `take_over_control` is on. This is the default, but confirm it.
- **Home Assistant location and the sun configured.** Adaptive Lighting needs your latitude, longitude, and the sun integration to compute the current look. This is part of normal HA onboarding.
- **The grouped light you select must be the one managed by the Adaptive Lighting instance you select.** If the AL instance's `lights:` list does not include that grouped light, `set_manual_control` and `apply` will have nothing to act on.
- **One Hue scene each for Rest, Relax, Bright, and Concentrate,** targeting that room's lights.

## How the cycle works

The mode is tracked by a per-room Dropdown helper. Each On press:

1. (Re)starts the per-room Timer helper for the configured timeout.
2. If the room was off, or the timer had already expired, it selects **Adaptive**. Otherwise it advances to the next option in the dropdown.
3. Applies the newly selected mode.

Because step 2 advances by moving to the next dropdown option, **the cycle order is exactly the order the dropdown options are defined in.** This is why the dropdown order below is not optional.

Returning to Adaptive clears manual control and then calls `adaptive_lighting.apply` with `turn_on_lights` enabled. This forces the current-time adaptive look immediately, whether the room was off or already on, instead of relying on a bare turn-on being intercepted at the right instant. The fade uses the Adaptive transition value you set.

## Setup

### 1. Import the blueprint

1. Put `hue_dimmer_adaptive_scene_cycle.yaml` in a public GitHub repository.
2. Open the file on GitHub and copy the normal browser URL. It looks like:
   `https://github.com/YOUR-USERNAME/YOUR-REPOSITORY/blob/main/hue_dimmer_adaptive_scene_cycle.yaml`
3. In Home Assistant, go to **Settings, Automations and scenes, Blueprints**.
4. Select **Import blueprint**, paste the URL, preview, and import.

### 2. Create the helpers (one set per room)

**Dropdown helper (input_select).** Create one per room under **Settings, Devices and services, Helpers, Create helper, Dropdown**. Add exactly these five options, with this exact spelling, capitalization, and order:

1. `Adaptive`
2. `Rest`
3. `Relax`
4. `Bright`
5. `Concentrate`

> **Order matters.** The cycle follows the dropdown order. If you reorder these, the scenes cycle in the new order. If you rename or recapitalize any option, that mode will fail with an "option not found" error, because the blueprint selects and reads these exact strings.

**Timer helper.** Create one per room under **Helpers, Create helper, Timer**. Its configured duration does not matter, because the blueprint supplies the timeout each time it starts the timer. **Leave timer restore disabled.** If restore is on and Home Assistant restarts mid-cycle, the timer can come back "active" and the next On press will advance the cycle instead of restarting at Adaptive.

> **Do not share a Dropdown or Timer helper between two rooms or remotes.** Each automation reads and writes its own helper. Sharing them will cause the rooms to overwrite each other's mode and timing.

### 3. Configure the remote in the Hue app

Keep the remote paired to the Hue Bridge, then set the buttons in the Hue app:

- **On (top button): Do nothing / unassigned.** This is essential. If On still performs a native Hue action, pressing it will fire both the native Hue turn-on (last stored state) and this automation, which can flash the old look before the adaptive look lands.
- **Brightness Up:** keep the normal Hue action.
- **Brightness Down:** keep the normal Hue action.
- **Off (bottom button): keep the normal Hue action.** The blueprint never issues its own `light.turn_off`. Off turning the lights off depends entirely on the Hue app assigning the bottom button to a native off action.

> **RWL022 owners, verify your buttons.** On the 2021 dimmer (RWL022), the top button is an on/off toggle and the bottom button is the "Hue" button rather than a dedicated Off. The device still exposes the four buttons as subtypes 1 to 4 top to bottom, but confirm that the bottom button (subtype 4) actually performs a native off in the Hue app, and that you can set the top button to do nothing. On the RWL020 and RWL021 the default On and Off layout works as written.

### 4. Create one automation per room

Select the imported blueprint, choose **Create automation**, and set, for that room:

- Hue Dimmer remote
- Hue grouped room or zone light
- Adaptive Lighting instance switch
- Dropdown helper
- Timer helper
- Rest, Relax, Bright, and Concentrate scenes
- On-button cycle timeout
- Adaptive transition (see below)

Repeat for each room, using that room's own remote, helpers, light, AL instance, and scenes.

## The Adaptive transition setting

This controls the fade used when returning to Adaptive, whether the room was off or already on. Set it to match your Adaptive Lighting instance's `initial_transition` if you want the return-to-Adaptive fade to feel identical to a normal Adaptive Lighting turn-on. Default is 1 second.

## Caveats and warnings

- **Off is only as reliable as the Hue app assignment.** The automation does not turn lights off on its own. If the bottom button is not assigned to a native off, nothing turns the room off.
- **On must do nothing natively,** as described above, or you get a double action.
- **Brightness keeps adapting color, on the AL interval.** After a brightness press, brightness is held manual and color continues to adapt, but only at your Adaptive Lighting `interval` (default around 90 seconds), not instantly.
- **Rapid On presses can trail slightly.** The automation runs queued, so presses process in order and the cycle stays correct, but each press makes one or more calls to the Hue Bridge and to Adaptive Lighting. Under very fast pressing the visible scene changes can lag a beat behind your presses.
- **A lone On press after the timeout returns to Adaptive,** even if you were sitting in a scene. This is intended. To keep cycling, press within the timeout window.
- **Pressing On while already in Adaptive re-applies Adaptive.** Harmless, and occasionally a very brief re-adapt is visible.
- **Adaptive Lighting sleep mode affects the look.** If the instance's sleep mode switch is on, the applied Adaptive look uses your sleep brightness and color.
- **Scenes fight nothing, by design.** Before each scene, the light is marked manually controlled so Adaptive Lighting does not override it. This is why the modes hold until you cycle away.

## Troubleshooting

- **The scenes cycle in the wrong order.** The Dropdown options are in the wrong order. Fix the option order to Adaptive, Rest, Relax, Bright, Concentrate (or to whatever order you actually want, since the cycle follows it).
- **A mode does nothing and the automation trace shows an error.** A Dropdown option is misspelled or miscapitalized. The options must be exactly `Adaptive`, `Rest`, `Relax`, `Bright`, `Concentrate`.
- **On to Adaptive does nothing when the room was already off and Adaptive Lighting had been turned off manually.** The blueprint waits 100 milliseconds after turning the AL switch on before applying. If your setup needs longer, open the automation in YAML and raise the `milliseconds: 100` value in the Adaptive branch to `250`.
- **Off does not turn the lights off.** The bottom button is not assigned to a native off in the Hue app. On an RWL022, confirm the bottom button and its assignment.
- **The remote does not appear when creating the automation.** It is not on the official Hue Bridge integration. This blueprint requires that integration.
- **Nothing happens on any button.** Confirm the remote is on the Hue Bridge integration and that the button triggers show up under the device page. If you are on the legacy Hue integration (v1 API), switch to the current integration.
- **Adaptive looks stale or lands late.** Confirm `take_over_control` is enabled on the AL instance and that the selected grouped light is in that instance's `lights:` list.

## Version

v3. The Adaptive branch now forces the current-time look with `adaptive_lighting.apply` and `turn_on_lights` instead of relying on turn-on interception, which fixes rooms coming on at a stale look and makes returning to Adaptive responsive. The Adaptive transition input is active again and controls the return-to-Adaptive fade.

## References

- Adaptive Lighting services (`apply`, `set_manual_control`): https://adaptive-lighting.nijho.lt/services/
- Adaptive Lighting manual control and `take_over_control`: https://adaptive-lighting.nijho.lt/advanced/manual-control/
