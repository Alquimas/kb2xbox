# kb2xbox

Convert a keyboard to one or more Xbox gamepads.

## Description

Linux treats every keyboard as the same input device, which makes local co-op games
with multiple players difficult. **kb2xbox** reads events from a dedicated keyboard
(typically a second or third one) and emulates Xbox controllers via `/dev/uinput`,
so each player can use their own keyboard as a controller.

## Requirements

- Linux
- Python 3.9+
- [python-libevdev](https://python-libevdev.readthedocs.io/en/latest/) (`pip install libevdev`)
- Write access to `/dev/uinput` (see [Setup](#setup))

## Setup

### 1. Create a virtual environment and install dependencies

```bash
python3 -m venv venv
source venv/bin/activate
pip install libevdev
```

### 2. Make `/dev/uinput` writable

```bash
sudo chmod 666 /dev/uinput
```

### 3. Find your keyboard

```bash
sudo python kb2xbox.py --list
```

Look for your second/third keyboard in the output and note its event device path
(e.g. `/dev/input/event5`).

## Usage

```bash
sudo python kb2xbox.py -d <DEVICE> <CONFIG_FILES...>
```

### Example

```bash
sudo python kb2xbox.py -d /dev/input/event5 config/xbox.cfg config/xbox2.cfg
```

This emulates 2 Xbox controllers on the same physical keyboard:

| Controller 1 | Controller 2 |
|---|---|
| Arrow keys -> Left Stick | E, S, D, F -> Left Stick |
| Right Alt -> A button | Left Shift -> A button |
| Henkan -> B button | Caps Lock -> B button |
| Katakanahiragana -> X button | Tab -> X button |

## Keyboard Controls

| Shortcut | Action |
|---|---|
| `Ctrl+F1` | Toggle grab (release keyboard back to system) |
| `Ctrl+Esc` | Quit |

While grab is active, keyboard input is captured by the emulated controllers and
won't affect other windows. Press `Ctrl+F1` to release the keyboard when you
need to type normally.

## Configuration Files

Config files map physical keyboard keys to Xbox controller buttons and axes.

### Format

```ini
NAME=<controller name>
VENDOR=<hex vendor ID>
PRODUCT=<hex product ID>
VERSION=<hex version>

# Buttons (EV_KEY)
BTN_SOUTH=KEY_<name>       # A
BTN_EAST=KEY_<name>        # B
BTN_NORTH=KEY_<name>       # X
BTN_WEST=KEY_<name>        # Y
BTN_TL=KEY_<name>          # Left bumper
BTN_TR=KEY_<name>          # Right bumper
BTN_SELECT=KEY_<name>      # Select / Back
BTN_START=KEY_<name>       # Start
BTN_THUMBL=KEY_<name>      # Left stick click
BTN_THUMBR=KEY_<name>      # Right stick click
BTN_MODE=KEY_<name>        # Xbox button

# D-Pad (EV_ABS)
ABS_HAT0X=KEY_<L>,KEY_<R>   # D-Pad left, right
ABS_HAT0Y=KEY_<U>,KEY_<D>   # D-Pad up, down

# Left thumb stick (EV_ABS)
ABS_X=KEY_<L>,KEY_<R>       # Left stick X axis
ABS_Y=KEY_<U>,KEY_<D>       # Left stick Y axis

# Right thumb stick (EV_ABS)
ABS_RX=KEY_<L>,KEY_<R>
ABS_RY=KEY_<U>,KEY_<D>

# Triggers (EV_ABS) — analog, supports split on multiple keys
ABS_Z=KEY_<name>[,KEY_...]  # Left trigger (LT)
ABS_RZ=KEY_<name>[,KEY_...] # Right trigger (RT)
```

- Leave a value empty to leave the button/axis unmapped.
- Lines starting with `#` are comments.
- Multiple comma-separated keys on an axis are split into equal steps.

### Keyboard key names

Use standard `KEY_*` names from Linux input-event-codes (e.g. `KEY_A`, `KEY_LEFT`,
`KEY_SPACE`, `KEY_LEFTSHIFT`, `KEY_HENKAN`).

### Included configs

| File | Game / Purpose |
|---|---|
| `config/xbox.cfg` | Generic Xbox controller layout |
| `config/xbox2.cfg` | Second generic Xbox controller |
| `config/Overcooked2-1.cfg` | Overcooked 2 — Player 1 |
| `config/Overcooked2-2.cfg` | Overcooked 2 — Player 2 |
| `config/Unrailed-1.cfg` | Unrailed! — Player 1 |
| `config/Unrailed-2.cfg` | Unrailed! — Player 2 |
| `config/TheEscapist2.cfg` | The Escapists 2 |

## Troubleshooting

**"Permission denied" on `/dev/uinput`**: run `sudo chmod 666 /dev/uinput`.

**No keyboards found with `--list`**: run with `sudo` — reading `/dev/input/event*`
requires root.

**Keys not working**: make sure the keyboard key name is correct (check with
`evtest`) and that your keyboard actually has that key.

## Network: use a keyboard from another machine

Forward a remote keyboard over TCP so it appears as a local device:

**Receiver** (the machine running kb2xbox):
```bash
nc -l -p 4444 > /dev/input/by-path/platform-i8042-serio-0-event-kbd
```

**Sender** (the remote keyboard):
```bash
cat /dev/input/by-path/platform-i8042-serio-0-event-kbd | nc <IP> 4444
```
