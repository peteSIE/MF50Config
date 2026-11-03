# Byonics MicroFox-50 Configuration Utility (VB.NET, .NET 10)

A Windows Forms replacement for Byonics' web-based MF-50 configurator
(https://byonics.com/mf50config/). It connects over the MF-50's micro-USB
data cable and reads/writes the settings using the device's built-in serial
configuration session.

**Status: the read/write app has been confirmed working against a physical
MF-50. The log pane shows all serial traffic, which makes it easy to diagnose
any real-device differences.**

## Project layout

A single WinForms project, `src/MF50Config`:

| Location | Purpose |
|---|---|
| `Core/` | Non-UI logic: the MF-50 serial protocol (`Mf50Protocol.vb`), the serial line assembler (`LineAssembler.vb`), value validation (`SettingValidator.vb`), the tone code table (`ToneTable.vb`), the power-to-mW table (`PowerTable.vb`), the WAV preview builder (`WavBuilder.vb`), profile storage (`ProfileStore.vb`), and COM port inventory/diff/selection logic (`PortList.vb`). |
| `Form1`, `formToneSeqEditor`, `formProfilePickerForm`, `formProfileName`, `formAbout` | The user interface: menu bar, COM port selection, Read/Write buttons, setting controls with sliders, tone sequence editor, profile dialogs, activity log. |
| `Globals.vb` | Shared application state and the default setting values. |

## Building

Requires the .NET 10 SDK (Visual Studio 2026, or the command line).

- Visual Studio: open `MicroFox50.slnx`.
- Command line: `dotnet build MicroFox50.slnx`

### Making the single-file .exe

```
dotnet publish src/MF50Config -c Release
```

(or **Build > Publish** in Visual Studio). The result is
`src/MF50Config/bin/Release/net10.0-windows/win-x64/publish/MF50Config.exe`:
one self-contained 64-bit Windows file that runs on any Windows 10/11 PC
with nothing installed. Profiles are saved under `%LOCALAPPDATA%`, so the exe can
be run from anywhere, including a USB stick.

`Form1` can be edited in the Windows Forms Designer as usual (the designer
code is in `Form1.Designer.vb`).

## The MF-50 serial protocol

Derived from Byonics' own web configurator JavaScript
(https://byonics.com/mf50config/js/mf50.js), which runs the same settings
session over Web Serial, and the MF-50 manual v0.4
(https://byonics.com/downloads/MicroFox-50_Manual_v0.4.pdf):

- **Port settings: 115200 baud, 8 data bits, no parity, 1 stop bit** (the
  MF-50 manual's terminal instructions). No flow control. Keep DTR and RTS
  de-asserted so the MF-50's ESP8266 is not reset.
- Send `***` followed by a line feed to start a settings session.
- The device prints `Settings:`, then one line per setting in the form
  `name is value`, and finally `Settings Done`.
- After each setting line the device waits for one line of input:
  - an empty line (Enter) keeps the current value;
  - any other text replaces the value.
- During a write, the device prints the `name is value` line a second time
  (as confirmation) before moving to the next setting — so a writer must
  answer only every other settings line it receives.
- The firmware banner starts with `MicroFox-50 v` and is printed after a
  reset. A reset can be triggered over the USB cable with the DTR/RTS
  sequence: DTR=false, RTS=true, 50 ms, DTR=false, RTS=false.
- Values are limited to 64 characters; an empty value is sent as a single
  space (matching Byonics' tool).

The settings the MF-50 reports (the 11 listed in the manual's configuration
description and Byonics' configurator): tone speed, tone duration, loop time,
initial delay, tx freq, morse message, morse speed, morse tone, tone sequence,
power, send battery.

## Valid ranges (from the MF-50 manual v0.4)

The app validates all values before sending anything to the device:

| Setting | Rule |
|---|---|
| tone speed / tone duration / loop time / initial delay / morse speed | whole number, 0 or more (the manual states no ranges for these; this is the app's policy) |
| tx freq | 144000-148000 (kHz), in 5 kHz steps (e.g. 146565 = 146.565 MHz) |
| morse message | required (your amateur radio callsign) |
| morse tone | code 2 (220 Hz) to 49 (3323 Hz); default 25 = 831 Hz |
| tone sequence | only characters from the manual's tone code table (space = silence, codes 02-32 = `!` through `?`, codes 33-49 = `@` and `A` through `P`) |
| power | level 0 (0 mW) to 50 (~54 mW) |
| send battery | 0 (off) or 1 (on) |
| all | no line breaks, max 64 characters |

## Using the app

1. Connect the MF-50 with a **micro-USB data cable** (a charging-only cable
   will not work) and turn it on. The CH340 driver may be needed
   (https://byonics.com/drivers).
2. Pick the COM port and click **Connect**.
3. Click **Read Version**. This resets the MF-50 and waits for its startup
   banner (e.g. `MicroFox-50 v0.5`), which confirms that an MF-50 is on this
   COM port. If no banner arrives, check that the unit is on, that the cable
   is a data cable, and that this is the right COM port.
4. **Read Settings from MF-50** fills the form with the device's current
   values. Reading before writing is the normal flow - or load a profile and
   write it directly (see Profiles below).
5. Edit the values, then **Write Settings to MF-50**. The app validates every
   value first, then automatically reads the settings back afterwards and
   warns if anything did not stick.
6. **Defaults** loads Byonics' recommended starting values.

The log pane shows all serial traffic: `< ` for lines received from the MF-50
and `> ` for lines sent to it.

## What's new in this version

### Sliders

Every numeric setting (tone speed, tone duration, loop time, initial delay,
tx freq, morse speed, morse tone, power) now has a slider next to its text
box. Drag the slider and the text box follows; type a value and the slider
moves to it. The tx freq slider is stepped in the same 5 kHz channel raster
the validator enforces; the frequency text box always shows the exact kHz
value that will be sent. Next to the power value, a label shows the
approximate RF output in mW (interpolated from the manual's measured
table). **The slider ranges are a UI convenience, not firmware limits** —
the validation rules in the table above still apply when writing.

### Send battery

Shown as a dropdown of "No (0)" and "Yes (1)". The raw 0/1 is still what
goes over the wire; the labels are just friendlier.

### Tone sequence editor

Open it with the **...** button next to the tone sequence or
**Tools > Tone Sequence Editor**. It shows the exact character string the
MF-50 sends and receives — the raw string stays visible and editable at all
times, so nothing is hidden behind the friendly view. Below it, a grid lists
each position with its tone code, character, note, and frequency, and the
grid is directly editable: click a **Note** cell and pick a new tone from the
drop-down (the manual's full 49-entry table); **Add Tone** appends a new tone
(and opens its drop-down); select a row and press **Delete** (or Remove Row)
to remove it; Move Up/Move Down reorder tones. The raw string at the top
updates as you edit, and you can also edit it directly — the grid follows. **Play**
renders the sequence to a WAV and plays it on the PC as a preview: it uses
the tone speed from the form, one pass through the sequence, with brief
fade in/out. This is a preview of the tones only — it does not reproduce
the device's transmit schedule, loop timing, or exact hardware timbre.
Codes 33-49 use the characters `@` and `A` through `P`, per the manual's
tone code table.

### Profiles

**File > Profiles > Save As / Load / Delete** manages complete setting sets
as JSON files under
`%LOCALAPPDATA%\MF50Config\Profiles` (e.g.
`C:\Users\<you>\AppData\Local\MF50Config\Profiles`). Saving
captures every setting on the form (plus any unknown settings a newer
firmware reported, which are preserved with their device values). Loading
fills the form only — nothing is sent to the MF-50 until you click
**Write Settings to MF-50**. You can do that without reading the device
first: the app then quietly reads the device's setting order (the form
keeps your values), sends the values in exactly that order, and verifies
the result by reading back as usual. Profile files are written atomically
(write to a temporary file, then rename), and the format carries a version
number so future releases can migrate old files. **File > Exit** closes
the app (the normal Windows close button works too).

### COM port detection

The dropdown now shows each port the way Device Manager names it, e.g.
`COM3 (USB-SERIAL CH340)`, so the MF-50's adapter is easy to spot (friendly
names come from WMI; if that is unavailable the plain `COM3` form is
used). When a device is plugged in, unplugged, enabled or disabled in
Device Manager, Windows broadcasts a device-change message; the app
debounces it, rescans (off the UI thread, so the form never freezes), and
updates the dropdown automatically. (Hot-plug handling and the WMI
friendly names are Windows-runtime behavior that could not be exercised
in the build environment - only compile-tested.) When a port change is
detected the app also shows a message box listing what was added or
removed (suppressed while a session is running, to avoid stacking
dialogs), and every change is recorded in the **Activity Log** pane at
the bottom of the window with the same display names as the dropdown,
e.g. `COM port added: COM3 (USB-SERIAL CH340)`. The selection policy: the
first scan selects the first port; afterwards your current choice is kept
while that port exists; when it disappears the selection is cleared (not
silently moved to another port); when exactly one new port appears - the
classic unplug/replug identification trick, including a replug under a
different port number - that port is selected for you; with multiple
arrivals the choice is left to you; nothing is ever auto-connected. If the
connected port disappears while idle, the app disconnects cleanly (your
draft values stay on the form); if it disappears mid-session, the session
is cancelled, no recovery reset is attempted through the gone port, and
the connection is released once the session handler finishes. The manual
refresh button still works as before.

## Notes

- If a session fails mid-way (timeout, unplugged cable), the app resets the
  MF-50 via DTR/RTS to get it out of the settings session, since a device
  left waiting for a value would treat the next `***` as data. After a failed
  write the form keeps your draft values, but read again before writing. If the
  recovery reset also fails, Read/Write stay blocked until the device
  reports its startup banner again (Read Version performs that reset).
  After a failed session the form keeps your draft values — fix them,
  save them as a profile, or retry after recovery — but they may no longer
  match the device, so read again before trusting what is shown.
- Unknown setting names (e.g. from newer firmware) are logged and left
  unchanged when writing; they do not break the session.
- Only the MicroFox-50 is supported. The MicroFox 15/15S, MicroFox PicCon,
  and MicroFox2 use different configuration programs/protocols
  (https://byonics.com/mfconfig/).
