# Hue Dimmer: Adaptive Lighting + 3-Second Scene Cycle

A Home Assistant automation blueprint for Philips Hue Dimmer remotes connected through the official Hue Bridge integration.

## Behavior

- First **On** press, or a press after the timeout: **Adaptive Lighting**
- Additional **On** presses within the timeout: **Relax → Rest → Bright → Concentrate → Adaptive**
- Default timeout: **3 seconds**
- Brightness buttons continue using native Hue dimming
- Off continues using the native Hue action
- Each room can select its own remote, grouped light, Adaptive Lighting instance, scenes, timeout, and transition

## Files

Upload this file to a public GitHub repository:

- `hue_dimmer_adaptive_scene_cycle.yaml`

This `README.md` is optional but useful as the repository landing page.

## Import into Home Assistant

1. Open `hue_dimmer_adaptive_scene_cycle.yaml` in your GitHub repository.
2. Copy the normal browser URL. It should look like:

   `https://github.com/YOUR-USERNAME/YOUR-REPOSITORY/blob/main/hue_dimmer_adaptive_scene_cycle.yaml`

3. In Home Assistant, go to **Settings → Automations & scenes → Blueprints**.
4. Select **Import blueprint**.
5. Paste the GitHub file URL, preview it, and import it.
6. Select the imported blueprint and choose **Create automation**.

## Hue app setup

Keep the remote paired with the Hue Bridge.

- **On:** Do nothing / unassigned
- **Brightness Up:** keep the normal Hue action
- **Brightness Down:** keep the normal Hue action
- **Off:** keep the normal Hue action

## Helpers required for each room

Create one **Dropdown** helper containing these options with exact capitalization:

- Adaptive
- Relax
- Rest
- Bright
- Concentrate

The order does not matter.

Create one **Timer** helper for each room. Its configured duration does not matter because the blueprint supplies the timeout whenever it starts the timer. Leave timer restoration disabled.

## Create one automation per room

For each automation, select that room's:

- Hue Dimmer remote
- Hue grouped room or zone light
- Adaptive Lighting switch
- Dropdown helper
- Timer helper
- Relax, Rest, Bright, and Concentrate Hue scenes
- Cycle timeout
- Adaptive transition
