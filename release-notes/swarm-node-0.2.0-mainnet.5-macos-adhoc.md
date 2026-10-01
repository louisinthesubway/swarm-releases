**SWARM Node 0.2.0-mainnet.5 for macOS, ad hoc signed, not notarized.**

These are re-signed copies of the 0.2.0-mainnet.5 per-architecture macOS builds: the same application code as the released Windows and Linux 0.2.0-mainnet.5, nothing rebuilt. The original per-architecture CI builds were packaged with `mac.identity: null`, which left the app bundle with a signature that does not match its contents, so macOS reported them as "damaged and can't be opened". These copies carry a consistent ad hoc signature (identity `-`). They are not signed with an Apple Developer ID and are not notarized.

Network: SWARM **mainnet** build (`swarm-mainnet` profile, bundle id `green.swarm.node`, version `0.2.0-mainnet.5`).

## First open on a Mac

macOS asks once before it runs an app that is not notarized:

1. Open the DMG and drag **SWARM Node** to **Applications**.
2. Open SWARM Node from Applications. macOS says it cannot verify the app; click **Done** (not Move to Trash).
3. Open **System Settings > Privacy & Security**, scroll to Security, and click **Open Anyway** next to the SWARM Node message; confirm with your password or Touch ID.
   On macOS 14 and earlier, Control-click (or right-click) the app in Applications and choose **Open**, then **Open** again, does the same.

After that the app opens normally.

## Files

| File | Bytes | SHA-256 |
|---|---|---|
| `SWARM-Node-0.2.0-mainnet.5-mac-arm64-adhoc.dmg` (Apple silicon) | 172974105 | `6246723e497e43c29fdf68e1e87a8c939bcc17968ebfa7be8b004f9689ac347d` |
| `SWARM-Node-0.2.0-mainnet.5-mac-x64-adhoc.dmg` (Intel) | 179100906 | `8d52faecc9c8ee5a642dbb32fac356701faff9ae3c94b03050d401da9c736a71` |

`SHA256SUMS` lists both.

## Inputs (the unsigned per-architecture builds)

| File | Bytes | SHA-256 |
|---|---|---|
| `SWARM-Node-0.2.0-mainnet.5-mac-arm64.dmg` | 170264514 | `84ed1983f0d57ec6c7ca4e138bb75b35eae5854e0dd68597ad42914c1d79ed31` |
| `SWARM-Node-0.2.0-mainnet.5-mac-x64.dmg` | 175634183 | `0344c74c07033d4ef45733b8dac192e6649db7173e3487c76d8328fa02ce4a1f` |

Downloaded by the workflow from the project's download server and checked for size and SHA-256 before anything else ran.

## How they were made

Workflow `.github/workflows/mac-node-adhoc.yml` on branch `ci/mac-node-adhoc`, commit 89510a20034ae0f7eabbdea801e085565521b8ac.
Run: https://github.com/louisinthesubway/swarm-releases/actions/runs/36900065041 (arm64 on macos-14 / Apple silicon, x64 on macos-15-intel).

Per architecture: the original volume was copied out with `ditto`, extended attributes removed with `xattr -cr "SWARM Node.app"`, then signed inside out. Exact commands (the same 26 for both architectures; order within one path depth may differ):

1. Every Mach-O file in the bundle, deepest path first (17 files):
```
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Squirrel.framework/Versions/A/Resources/ShipIt"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Electron Framework.framework/Versions/A/Libraries/libvk_swiftshader.dylib"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Electron Framework.framework/Versions/A/Libraries/libffmpeg.dylib"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Electron Framework.framework/Versions/A/Libraries/libGLESv2.dylib"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Electron Framework.framework/Versions/A/Libraries/libEGL.dylib"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Electron Framework.framework/Versions/A/Helpers/chrome_crashpad_handler"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Squirrel.framework/Versions/A/Squirrel"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper.app/Contents/MacOS/SWARM Node Helper"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper (Renderer).app/Contents/MacOS/SWARM Node Helper (Renderer)"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper (Plugin).app/Contents/MacOS/SWARM Node Helper (Plugin)"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper (GPU).app/Contents/MacOS/SWARM Node Helper (GPU)"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/ReactiveObjC.framework/Versions/A/ReactiveObjC"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Mantle.framework/Versions/A/Mantle"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework"
codesign --force --sign - "SWARM Node.app/Contents/Resources/bin/swarm-node-daemon"
codesign --force --sign - "SWARM Node.app/Contents/Resources/bin/swarm-miner"
codesign --force --sign - "SWARM Node.app/Contents/MacOS/SWARM Node"
```
2. Every nested bundle, deepest first:
```
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Squirrel.framework"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper.app"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper (Renderer).app"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper (Plugin).app"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/SWARM Node Helper (GPU).app"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/ReactiveObjC.framework"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Mantle.framework"
codesign --force --sign - "SWARM Node.app/Contents/Frameworks/Electron Framework.framework"
```
3. The app:
```
codesign --force --sign - "SWARM Node.app"
```
4. The DMG, with the original volume name, contents (app, Applications link, background, volume icon, Finder layout) and image format; file system kept as in the original (arm64 APFS, x64 HFS+):
```
hdiutil create -volname "SWARM Node 0.2.0-mainnet.5" -srcfolder stage -fs APFS -format UDZO -imagekey zlib-level=9 SWARM-Node-0.2.0-mainnet.5-mac-arm64-adhoc.dmg
hdiutil create -volname "SWARM Node 0.2.0-mainnet.5" -srcfolder stage -fs HFS+ -format UDZO -imagekey zlib-level=9 SWARM-Node-0.2.0-mainnet.5-mac-x64-adhoc.dmg
```

## What was checked in the run

- `codesign --verify --deep --strict --verbose=2`: "valid on disk" and "satisfies its Designated Requirement", for the signed app, for the app inside the new DMG, and for a copy installed out of the new DMG.
- `codesign -dv`: `Signature=adhoc`, `Identifier=green.swarm.node`, `CFBundleShortVersionString` 0.2.0-mainnet.5; each of the 17 Mach-O files also verifies strictly on its own. CDHash of the app is identical before and after packaging (arm64 `abab263d0b2c8247ce3a55b3e1b0ab3de54aa85f`, x64 `f852929fad0d916d797cd8f855e2e07e70c8798f`).
- The bundled `swarm-node-daemon --version` prints `zebrad 6.3.0` (exit 0); `swarm-miner --help` exits 0.
- Smoke test: the installed copy was started on its own architecture, stayed up for 20 s, loaded its window (page title "SWARM Node"), and wrote no crash report.
- `spctl --assess` rejects the app, as expected for any app that is not notarized; that is the one-time "Open Anyway" above.
