# Agent notes: installing this fork as an unpacked extension

This branch (`local-install`) exists to install a patched build of Tab Suspender on a personal
machine in place of the Chrome Web Store version. It contains the fix from
`fix/ticker-interval-leak` (TabObserver.start() leaking setInterval timers). Do not open PRs from
this branch; the upstream PR comes from `fix/ticker-interval-leak`.

## Build

```bash
npm install
npm run build
```

Output goes to `build_dir/`. In `chrome://extensions`, enable Developer mode, click
**Load unpacked**, and select `build_dir/` (not the repo root).

## Before switching: ask the user how to handle the existing extension

If the Web Store version of Tab Suspender is already installed, **stop and present the user with
the two options below before installing anything.** Do not pick one on the user's behalf.

Why it matters: every suspended tab is a `chrome-extension://<id>/park.html` URL that only works
while an extension with that exact ID is installed. Settings, whitelist and saved tab screenshots
are also stored by ID. The repo's `manifest.json` has no `key` field, so by default the unpacked
build gets a **new** ID.

Present it roughly like this:

> **Option A: keep the existing extension's identity (preserves suspended tabs).** I copy the
> key from your installed Web Store version into this build so it gets the same ID. Your
> currently suspended tabs keep working, and settings, whitelist and screenshots carry over.
>
> **Option B: fresh install with a new ID.** Simpler, but every currently suspended tab must be
> unsuspended first or it becomes a dead page, and settings must be exported and re-imported.

Whichever option is chosen, first have the user click **Export Settings** on the Web Store
version's Settings page and save the file as a backup.

### Option A: reuse the Web Store key (same ID)

1. In `chrome://extensions`, find Tab Suspender and note its **ID** (32 letters a-p).
2. Read the installed store copy's manifest. Chrome adds a `"key"` field to it on install:
   - macOS: `~/Library/Application Support/Google/Chrome/<Profile>/Extensions/<ID>/<version>/manifest.json`
   - Windows: `%LOCALAPPDATA%\Google\Chrome\User Data\<Profile>\Extensions\<ID>\<version>\manifest.json`
   - Linux: `~/.config/google-chrome/<Profile>/Extensions/<ID>/<version>/manifest.json`

   `<Profile>` is usually `Default` (others are `Profile 1`, etc.; `chrome://version` shows the
   profile path). Only read this file; do not modify anything in the Chrome profile.
3. Copy the `"key"` value into the **root** `manifest.json` of this repo (not `build_dir/`, which
   is wiped on every build), as a top-level `"key": "..."` entry. The key is a public key, not a
   secret. Do not commit it to `fix/ticker-interval-leak` or any branch meant for an upstream PR.
4. `npm run build`, then confirm `build_dir/manifest.json` contains the key.
5. **Do not click "Remove" on the Web Store version.** Uninstalling deletes the data stored under
   that ID. With it still installed, **Load unpacked** `build_dir/`. Chrome should replace the
   store install with the unpacked build under the same ID and keep its data. This has not been
   tested on this project, so verify the next step carefully. Never run two extensions with the
   same ID.
6. Verify with the user: the extension card shows the same ID, settings and whitelist are intact,
   and existing suspended tabs still open and restore. If settings are missing, use
   **Import Settings** with the backup.

If the key cannot be found, tell the user and offer Option B instead.

### Option B: fresh install (new ID)

In order:

1. Export settings (above).
2. **Unsuspend all tabs** with the Web Store version (toolbar menu: "Unsuspend all tabs").
   Skipping this leaves suspended tabs pointing at a dead `park.html` URL.
3. Disable the Web Store version. Never run both at the same time; they will both try to suspend
   tabs.
4. Load unpacked `build_dir/`, then use **Import Settings** with the backup.

Under either option, do not delete the Web Store extension or its data until the user confirms
that tabs are restored and settings are intact.

## Updating

Unpacked extensions do not auto-update. To pick up changes: `git pull`, `npm run build` (under
Option A, check the `key` is still in the root `manifest.json`), then click the reload icon on the
extension in `chrome://extensions`. Once a Web Store release includes the fix, the user can switch
back: export settings first, then reinstall from the Web Store. Under Option A the ID is the same,
so suspended tabs keep working; under Option B, unsuspend all tabs before switching.
