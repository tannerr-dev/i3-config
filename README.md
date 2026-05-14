# i3wm Configuration Additions

This document describes the custom additions made to this i3wm configuration.

## Table of Contents

1. [Kensington SlimBlade Trackball Button Remapping](#kensington-slimblade-trackball-button-remapping)
2. [Bluetooth Headphone Play/Pause Button](#bluetooth-headphone-playpause-button)
3. [MPRIS Media Player Script](#mpris-media-player-script)

---

## Kensington SlimBlade Trackball Button Remapping

### What
The Kensington SlimBlade Trackball has 4 buttons. By default, the layout doesn't match intuitive left/right clicking.

### Goal
- **Left side** (both top and bottom buttons) = **Left click**
- **Right side** (both top and bottom buttons) = **Right click**

### Physical Button Mapping

| Physical Button | Original Function | Remapped To |
|----------------|------------------|-------------|
| Bottom-Left (1) | Left click | **Left click** |
| Top-Left (2) | Middle click | **Left click** |
| Bottom-Right (3) | Right click | **Right click** |
| Top-Right (8) | Back button | **Right click** |

### Code Added to `config`

Add this line to your i3 config (recommended near other input device settings):

```i3
# Kensington SlimBlade Trackball - left side = left click, right side = right click
# Top-Left (btn 2) -> left click, Bottom-Left (btn 1) -> left click
# Top-Right (btn 8) -> right click, Bottom-Right (btn 3) -> right click
exec xinput set-button-map "Kensington Kensington Slimblade Trackball" 1 1 3 4 5 6 7 3
```

### Where to Place

Recommended location in `~/.config/i3/config`:

```i3
#custom mouse / trackpad settings
exec xinput set-prop "Elan Touchpad" "libinput Tapping Enabled" 1
# Kensington SlimBlade Trackball - left side = left click, right side = right click
exec xinput set-button-map "Kensington Kensington Slimblade Trackball" 1 1 3 4 5 6 7 3
```

---

## Bluetooth Headphone Play/Pause Button

### What
Enable the play/pause button on Bluetooth headphones (like Beats) to control media playback.

### Code Added to `config`

Add this line to your i3 config (recommended near other audio key bindings):

```i3
# Bluetooth headphone play/pause button (Beats)
# Controls whichever MPRIS player is active: Brave, Chromium, Firefox, Spotify, VLC
# Note: Waterfox does not expose MPRIS and cannot be controlled
bindcode 209 exec --no-startup-id $HOME/.local/bin/mpris_playpause.sh
```

### Why `bindcode` Instead of `bindsym`?

Bluetooth media keys often bypass X11 keysym mapping. Using `bindcode 209` (raw keycode) is more reliable than `bindsym XF86AudioPlay` or `bindsym XF86AudioPause` for Bluetooth devices.

### How to Find Your Keycode

If your headphones use a different keycode, run:

```bash
timeout 15 xinput test-xi2 --root 2>&1 | grep -E "detail:|KeyPress|KeyRelease"
```

Then press your headphone button and note the `detail:` number.

### Where to Place

Recommended location in `~/.config/i3/config`:

```i3
set $refresh_i3status killall -SIGUSR1 i3status
bindsym XF86AudioRaiseVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ +10% && $refresh_i3status
bindsym XF86AudioLowerVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ -10% && $refresh_i3status
bindsym XF86AudioMute exec --no-startup-id pactl set-sink-mute @DEFAULT_SINK@ toggle && $refresh_i3status
bindsym XF86AudioMicMute exec --no-startup-id pactl set-source-mute @DEFAULT_SOURCE@ toggle && $refresh_i3status
# Bluetooth headphone play/pause button (Beats)
bindcode 209 exec --no-startup-id $HOME/.local/bin/mpris_playpause.sh
```

---

## MPRIS Media Player Script

### What
A lightweight shell script that controls MPRIS-compatible media players via D-Bus. It uses only `dbus-send` (pre-installed on all Linux systems).

### Features

- Controls whichever player is currently **Playing**
- If nothing is playing, resumes the **last paused player** (remembers state)
- Supports: Brave, Chromium, Firefox, Spotify, VLC
- **Does NOT support Waterfox** (MPRIS not exposed)

### Installation

Copy the script to `~/.local/bin/`:

```bash
mkdir -p ~/.local/bin
cp mpris_playpause.sh ~/.local/bin/
chmod +x ~/.local/bin/mpris_playpause.sh
```

### Script Location

```
~/.local/bin/mpris_playpause.sh
```

### How It Works

1. Queries D-Bus for all `org.mpris.MediaPlayer2.*` services
2. Checks each player's `PlaybackStatus`
3. If a player is `Playing` → sends `PlayPause` to that player
4. If nothing is `Playing` → reads `/tmp/mpris_last_paused` and resumes that player
5. If no state file exists → falls back to the first player found
6. Saves the controlled player's dbus name to `/tmp/mpris_last_paused`

### Dependencies

- `dbus-send` (pre-installed on all Linux distributions)

No additional packages needed.

---

## Full Example: All Additions in Context

Here's how all the additions look together in `~/.config/i3/config`:

```i3
# ============================================
# AUDIO KEYS
# ============================================
set $refresh_i3status killall -SIGUSR1 i3status
bindsym XF86AudioRaiseVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ +10% && $refresh_i3status
bindsym XF86AudioLowerVolume exec --no-startup-id pactl set-sink-volume @DEFAULT_SINK@ -10% && $refresh_i3status
bindsym XF86AudioMute exec --no-startup-id pactl set-sink-mute @DEFAULT_SINK@ toggle && $refresh_i3status
bindsym XF86AudioMicMute exec --no-startup-id pactl set-source-mute @DEFAULT_SOURCE@ toggle && $refresh_i3status

# Bluetooth headphone play/pause button (Beats)
# Controls whichever MPRIS player is active: Brave, Chromium, Firefox, Spotify, VLC
# Note: Waterfox does not expose MPRIS and cannot be controlled
bindcode 209 exec --no-startup-id $HOME/.local/bin/mpris_playpause.sh

# ============================================
# INPUT DEVICES
# ============================================
#custom mouse / trackpad settings
exec xinput set-prop "Elan Touchpad" "libinput Tapping Enabled" 1
# Kensington SlimBlade Trackball - left side = left click, right side = right click
# Top-Left (btn 2) -> left click, Bottom-Left (btn 1) -> left click
# Top-Right (btn 8) -> right click, Bottom-Right (btn 3) -> right click
exec xinput set-button-map "Kensington Kensington Slimblade Trackball" 1 1 3 4 5 6 7 3
```

---

## Git Checklist

When checking this config into git, ensure these files are committed:

- [ ] `~/.config/i3/config` (with the additions above)
- [ ] `~/.local/bin/mpris_playpause.sh`
- [ ] `~/.local/bin/mpris_playpause.README.md` (optional but recommended)

**Note:** `~/.local/bin/` may not be in your git repo. Consider creating a `bin/` folder in your dotfiles repo and symlinking:

```bash
ln -s /path/to/dotfiles/bin/mpris_playpause.sh ~/.local/bin/mpris_playpause.sh
```

Or simply keep the script alongside your i3 config and reference it with a relative path.

---

## Troubleshooting

### Script says "No MPRIS media players found"

Make sure a supported media player is running and actively playing or ready to play media.

### Button doesn't do anything

Your headphones may emit a different keycode. Test with:

```bash
timeout 15 xinput test-xi2 --root 2>&1 | grep -E "detail:|KeyPress|KeyRelease"
```

Then update the `bindcode` number in your i3 config.

### Trackball buttons not remapped after reboot

Make sure the `exec xinput set-button-map` line is in your i3 config and i3 is reloaded (`$mod+Shift+c` or `i3-msg reload`).

---

## License

Public domain / use however you want.
