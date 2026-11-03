# MF50Config - MicroFox-50 Configuration Utility

A Windows replacement for Byonics' web-based MF-50 configurator
(https://byonics.com/mf50config/). It connects to the MF-50 over its
micro-USB data cable and reads and writes its settings.

This program is not provided by, supported by, or connected to Byonics, LLC.
This is a beta release: please report any problems (see "Reporting a
problem" below).

Download the latest version from the Releases page:
https://github.com/peteSIE/MF50Config/releases

## What you need

- Windows 10 or 11 (64-bit).
- A MicroFox-50 and a micro-USB **data** cable (a charge-only cable will not
  work).
- An amateur radio license: in most countries you need one to transmit on
  144-148 MHz. Your callsign goes in the morse message so the MF-50 identifies
  your station.
- Nothing to install: MF50Config.exe is a single file you can run from
  anywhere, including a USB stick.

## Quick start

1. Run MF50Config.exe. (If Windows says "Windows protected your PC", the
   program is not digitally signed: click More info, then Run anyway.)
2. Connect the MF-50 with its USB cable and turn it on. If no COM port
   appears, the USB driver is probably missing: see "Installing the USB
   driver" below.
3. Pick the COM port and click Connect.
4. Click Read Settings from MF-50 to fill the form with the unit's current
   values.
5. Change what you want, then click Write Settings to MF-50. The program
   checks every value first, then reads the settings back and warns you if
   anything did not stick.

## Using the app

- Connect / Disconnect opens or closes the serial port. Connecting does not
  prove that an MF-50 is on that port: with the wrong port, or with the unit
  switched off, the first Read shows a "no response" message.
- Read Version resets the MF-50 (its settings are kept) and shows its
  firmware version, e.g. "MicroFox-50 v0.5". It is optional, but a quick way
  to confirm that an MF-50 is answering.
- Defaults loads Byonics' recommended starting values into the form.
- Numeric settings have a slider next to the text box. Moving either one
  updates the other. The power label shows the approximate output in mW.
- The activity log at the bottom of the window shows everything sent to the
  MF-50 (">") and received from it ("<").
- The program remembers the COM port you last used.

### Tone sequence

Click Edit... next to the tone sequence (or choose Tools > Tone Sequence
Editor) to edit it as a list of tones: pick a note for each position, add,
remove or reorder tones. The raw character string is always shown and can be
edited directly. Play, in the editor or on the main window, plays one pass of
the sequence on your PC as a preview. It previews the tones only, not the
MF-50's exact sound or transmit timing.

### Profiles

File > Profiles saves the settings on the form under a name, and loads or
deletes saved ones. Loading only fills in the form: nothing goes to the MF-50
until you click Write Settings to MF-50, and you can write a profile
without reading the unit first. Profiles are stored in
%LOCALAPPDATA%\MF50Config\Profiles (File > Profiles > Open Folder shows it).

### Valid values

- tx freq: 144000 to 148000 (kHz), in 5 kHz steps (146565 = 146.565 MHz).
- morse message: required; your amateur radio callsign. If it does not look
  like a callsign (the factory default, MF50, does not), Write Settings asks
  you to confirm before sending.
- morse tone: 2 (220 Hz) to 49 (3323 Hz); the default 25 is 831 Hz.
- power: 0 (0 mW) to 50 (about 54 mW).
- send battery: No (0) or Yes (1).
- tone sequence: only characters from the tone table (the editor offers them).
- Every value: at most 64 characters, no line breaks. The other numeric
  settings are whole numbers, 0 or more.

## Installing the USB driver

The MF-50 connects through a CH340 chip. Windows usually finds a driver for
it, but on some PCs it does not. Choose Tools > Install CH341 Driver:

1. The program downloads the driver installer from WCH, the chip's maker.
   If their server does not answer, it retries, and you can choose a copy of
   CH341SER.EXE that you downloaded yourself, or open WCH's page in your
   browser.
2. Before running anything, Windows checks the installer's digital signature,
   and the program requires it to be signed by "Nanjing Qinheng
   Microelectronics Co., Ltd." A file that fails the check is not run.
3. Allow the Windows administrator prompt. In WCH's setup window click
   INSTALL, wait for it to report success, then close its window (it does not
   close by itself).
4. When asked, connect the MF-50 and turn it on, then click Retry. The program
   asks Windows whether the device and driver are working, and selects the new
   COM port for you.
5. Click Connect, then Read Settings.

A progress bar shows the current step, and the activity log records each one
(lines starting with "Driver:").

## Reporting a problem

Choose Help > Report a Problem... The program then:

1. asks whether to hide your morse message (usually your callsign) in the
   report. Only the morse message is hidden: read the details yourself
   before you post them;
2. copies the report details to the clipboard: the program version, your
   Windows version, the selected COM port and the activity log;
3. opens the problem-report page in your web browser. Describe what went
   wrong, and paste (Ctrl+V) the copied details where the form asks.

You need a free GitHub account to submit a report, and reports there are
public. To look at the details first, or to keep them, use Help > Copy Log or
Help > Save Log.

## Troubleshooting

- "The MF-50 stopped responding": check that the MF-50 is turned on, that the
  cable is a data cable, and that you chose the right COM port. Read Version
  is a good test.
- No COM port appears when you plug the MF-50 in: install the driver (above),
  try another USB port or cable, and look in Device Manager for a device with
  a warning icon.
- The driver download times out: WCH's server can be slow or unreachable from
  some networks. Use the options the program offers (choose a file you
  downloaded, or open WCH's page in your browser).
- Security software warns about the program: it starts a driver installer with
  administrator permission and downloads it from the internet, which some
  security software dislikes.
- To check a downloaded copy of the program, compare its SHA-256 hash with the
  one published with the release: run
  certutil -hashfile MF50Config.exe SHA256
  in a command prompt.

## Notes

- If a read or write fails part-way (timeout, unplugged cable), the program
  resets the MF-50 to get it out of its settings mode. The form keeps your
  values, but read again before writing, because they may no longer match the
  unit.
- Only the MicroFox-50 is supported. The MicroFox 15/15S, PicCon and MicroFox2
  use different configuration programs (https://byonics.com/mfconfig/).

## License

MF50Config is freeware. You may use it free of charge, and you may share the
unmodified program provided this notice stays with it and the author is
credited. You may not sell it, charge for it, rename it, present it as your
own work, or distribute modified versions.

This program is provided "as is", without warranty of any kind. The author is
not liable for any damage or loss arising from its use, including damage to
equipment or compliance with radio regulations.

Copyright (C) 2026 Peter Greene, N2LVI

Includes the Microsoft .NET runtime (MIT license).