# Electron cursor flickering demo

A minimal reproduction of cursor flickering on Windows when an always-on-top window uses `setIgnoreMouseEvents(true, { forward: true })`. Built from Electron's quick-start app.

Original bug report: [electron/electron#35414, setIgnoreMouseEvents on Windows / Flickering Cursor](https://github.com/electron/electron/issues/35414).

## What the demo does

[`main.js`](main.js) creates two overlapping windows:

- An 800 × 600 main window displaying [`index.html`](index.html), with a red **MOVE** area using `cursor: move` and a blue **POINTER** area using `cursor: pointer`.
- A 1200 × 800 frameless, transparent window displaying [`secondary.html`](secondary.html). Its background adds a faint dark tint over the window underneath.

The second window stays on top and lets mouse input pass through to the underlying window. Mouse movement is also forwarded to the overlay through this configuration:

```js
secondaryWindow.setIgnoreMouseEvents(true, { forward: true })
secondaryWindow.setAlwaysOnTop(true, 'screen-saver')
```

## Run the demo

Install Git and Node.js with npm, then run these commands in a Windows terminal:

```bash
git clone https://github.com/wapgear/demo-for-electron.git
cd demo-for-electron
npm ci
npm start
```

The repository declares Electron `^20.0.3`; `npm ci` installs the version recorded in the lockfile. The original issue reports the problem on Electron 18, 19, and 20, on Windows 10 and Windows 11.

## Reproduce the flicker

1. Keep the tinted overlay above the main window, covering the MOVE and POINTER areas.
2. Move the mouse around inside each area. Try clicking and moving again within the POINTER area.
3. Watch the cursor while it moves.

Expected behavior: the cursor consistently matches the underlying area's CSS, showing a move cursor over MOVE and a hand over POINTER.

Reported behavior: the cursor briefly switches between the default arrow and the cursor defined by the underlying area, producing visible flicker.

To compare with mouse forwarding disabled, change the call in `main.js` to `secondaryWindow.setIgnoreMouseEvents(true, { forward: false })`, restart the app, and repeat the steps. The overlay remains click-through, but no longer receives forwarded mouse movement.

## Issue and investigation

[Electron issue #35414](https://github.com/electron/electron/issues/35414) contains the original recording and subsequent reproduction reports. It was closed automatically due to inactivity, rather than a confirmed fix.

A [follow-up investigation](https://github.com/electron/electron/issues/35414#issuecomment-5777674802) reproduced the behavior with this demo on Electron 44.4.3 and Windows 11 ARM64 in Parallels. It recorded cursor flicker with forwarding enabled and none with forwarding disabled, and links to a proposed fix. See the issue for the investigation and validation status.

The upstream discussion is tracked in [Chromium issue #566069560, per-root Aura cursor suppression for click-through windows](https://issues.chromium.org/issues/566069560). It proposes suppressing cursor shape updates from the click-through overlay while preserving forwarded mouse events. The report requests feedback on the approach before preparing a Chromium change; validation with a patched Windows build is still pending.

## License

[CC0 1.0, Public Domain](LICENSE.md)
