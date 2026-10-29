# create-chord-package

> [!NOTE]
> This section is largely AI-written, so parts of it may not be fully accurate. It will be replaced with a hand-written version when I get the time.

How to create a chord package for [Chord](https://github.com/KeyChord/Chord) (multi-letter shortcuts for macOS). A chord package is a folder of TOML chord files plus optional JavaScript handlers. This repo doesn't contain a CLI, and nothing is published to npm as `create-chord-package`, so you create the files by hand as described below.

## Quick start

1. Install Chord: download `Chord-aarch64.dmg` from the [latest release](https://github.com/KeyChord/Chord/releases/latest) (Apple Silicon only, signed and notarized), drag it into `Applications`, launch it, and grant it Accessibility access when asked.

2. Create a folder with these files:

   ```text
   my-chords/
   ├── package.json
   ├── vite.config.ts
   ├── chords/macos.toml
   └── src/js/say.ts
   ```

   `package.json`:

   ```json
   {
     "name": "@your-name/my-chords",
     "type": "module"
   }
   ```

   `chords/macos.toml` defines one global chord, `,s` (chords that start with a symbol work in every app):

   ```toml
   name = "My Chords"

   [on.say]
   file = "say.js"
   args = ["Hello from Chord"]

   [chords]
   ',s' = { name = "Say Hello", '@input:say' = [] }
   ```

   `src/js/say.ts`:

   ```ts
   export default function build(text: string) {
     return function say() {
       Bun.spawn(["say", text]);
     };
   }
   ```

   `vite.config.ts`:

   ```ts
   import { config } from "@keychord/config";

   export default config();
   ```

3. Install the build tools and build the handler:

   ```sh
   bun add -d @keychord/config vite-plus
   bunx vp pack
   ```

   This writes `js/say.js` and `js/package.json.js`. Chord only loads the built files in `js/`, never `src/`. See [@keychord/config](https://github.com/KeyChord/config) for build options, vendoring and Swift add-ons.

4. In Chord, open **Settings → Chords → Local Folders** and add the folder (**Add Folder**, or **Browse...**).

5. Hold `Caps Lock`, press `Space` and keep holding it, type `,s`, then release `Space`.

After changing the package, run `bunx vp pack` again (or keep `bunx vp pack --watch` running) and reload Chord's configuration with the `0,r` chord.

Chords that only press a shortcut (`shortcut = "cmd+n"`) or run a command (`shell = "open -a Terminal"`) don't need any JavaScript, so a package made only of those needs no build step.

### Sharing a package

Push the package to GitHub with its built `js/` folder committed: Chord clones the repo but doesn't install or build anything. Others can then add it with **Settings → Chords → Chord Repos → Add Repo** and `owner/name`. Add the `chordpack` topic to the repo so people can find it: Chord's **Settings → Browse** links to a GitHub search for that topic. A repo that contains several package folders named `chords-*`, like [KeyChord/chords](https://github.com/KeyChord/chords), is added with **Chord Monorepos → Add Monorepo** instead, and loads nothing until you tick packages under **Packages to use**.

If two loaded packages have the same name, a local folder wins over a repo, so a local checkout of a published package replaces it while you work on it.

## Structure

```
package.json
chords/
js/
bin/
```

### `package.json`

A chord package should have a `package.json` file with the following fields:

- `name`: The name of the chord package, conventionally in the form `@<username>/<slug>` (Chord doesn't check the format).

> If `package.json` is omitted, the chord package's name will instead be inferred from its folder name. Packages added from a Git repository are cloned into a folder named after the commit, so they should always contain a `package.json` file.

### `chords/`

A chord package's chord files live in a root `chords/` directory.

> [!NOTE]
> This section is largely AI-written, so parts of it may not be fully accurate. It will be replaced with a hand-written version when I get the time.

Every file under `chords/` is parsed as TOML. Chord matches a file to an app by its path, `chords/<bundle ID with dots replaced by slashes>/macos.toml`: for example, `chords/com/apple/finder/macos.toml` applies in Finder. Chords whose key starts with a symbol are global and work in every app; [@keychord/config](https://github.com/KeyChord/config) requires them to be defined in the package's root `chords/macos.toml`. Other files, such as a shared `base.toml`, take effect when a chord file imports them.

### `js/`

If a chord package uses JavaScript, they must be located in a root `js/` directory.

The files in the `js/` directory must be self-contained; they CANNOT import any external files or modules outside the `js/` folder.

> [!NOTE]
> This section is largely AI-written, so parts of it may not be fully accurate. It will be replaced with a hand-written version when I get the time.

Chord doesn't install dependencies (a package added from GitHub is only cloned), and Bun resolves a handler's imports the usual way, so everything except `chord`, `bun:*` and `node:*` has to be bundled into the handler. [@keychord/config](https://github.com/KeyChord/config) does this when it builds `src/js/*.ts`.

Handler `file` paths in chord files are relative to `js/`. To reuse another chord package's handlers and chord files, vendor it with `config({ vendor: ["@keychord/chords-menu"] })`, which copies its `js/` to `js/@keychord/chords-menu/js/` and its `chords/` to `chords/@keychord/chords-menu/chords/`. This handler then resolves to `js/@keychord/chords-menu/js/menu.js`:

```toml
[on.menu]
file = "@keychord/chords-menu/js/menu.js"
args = ["Safari"]
```

A vendored package has to be installed first. The official chord packages aren't published to npm, so install them from GitHub, for example `bun add -d github:KeyChord/chords-menu`.

### `bin/`

If a chord package has binaries, put them in a root `bin/` directory by convention (Chord doesn't treat it specially). Handlers can find them with `import.meta.chord.resolvePackageFile("bin/<name>")`.

## Chord files

A chord file typically contains the following top-level properties:

- `name`
- `meta`
- `on`
- `chords`
- `import`

### Handlers

> [!NOTE]
> This section is largely AI-written, so parts of it may not be fully accurate. It will be replaced with a hand-written version when I get the time.

A chord file may define zero or more handlers under `[on.<event-name>]`, and chords call them through the `@input:<event-name>` property on chord definitions. Its value is an array of arguments passed to the handler, and may contain regex captures such as `$1`.

```toml
[on.menu]
file = "@keychord/chords-menu/js/menu.js"
args = ["WhatsApp"]

[chords]
cvi = { name = "Chat: Video Call", '@input:menu' = ["Chat", "Video Call"] }
```

`file` is the handler module, relative to `js/`, and `args` are passed to its default export. When Chord loads the configuration, it calls the default export once for each `[on.<event-name>]` entry, with `this` set to `{ chordsFile, chordsFilePath, chordsFileAppId }`, and the export must return the handler function (or a Promise of one). Each time a chord fires, Chord calls that function with the chord's arguments and with `this` set to `{ focusedAppId, focusedWindowId, input, event }`. The [quick start](#quick-start) has a complete example.

When a chord fires `@input:<event-name>`, Chord first looks for a handler named `<package name>:<event-name>` (for example `[on.'@user/pkg:menu']`), then for `<event-name>`. A chord file's own handlers are checked before the handlers of the files it imports, so redefining `[on.<event-name>]` overrides an imported handler.

Use `@init:<event-name>` for initialization when a chord first matches, and `@keydown`, `@keypress`, or `@keyup` for live keyboard callbacks. Initialization resets when that chord stops matching. All five triggers can assign their handler's return value with `=<identifier>`:

```toml
[chords.'j,(.*)']
name = "Live clicks"
'@init:prepare=context' = []
'@keydown:update=latest' = ['$context', '$1']
'@keyup:released' = ['$latest']
```

Declare `prepare`, `update`, and `released` under `[on]` as usual. Return values persist for that chord's current matching interval, and Promise results are awaited before later callbacks. Handlers receive Web KeyboardEvent fields through `this.event`, so they can filter keys with `this.event.key === "Backspace"`. Chord delivers `@keypress` for every key, including Backspace and modifiers. See [the full event contract](https://github.com/KeyChord/Chord/blob/dev/scripting.md#chord-events-and-persistent-return-values).

### `chords`

A chord definition may consist of a regex pattern:

```toml
[chords]
'/d(\d+)' = { name = "Dock: Focus $1", '@input:dock' = ['$1'] }
```

> [!NOTE]
> This section is largely AI-written, so parts of it may not be fully accurate. It will be replaced with a hand-written version when I get the time.

Only the parenthesized parts of a key are regex; everything else is literal, and the pattern always has to match the whole sequence, so don't add `^` or `$`. Captures (`$1`, `$2`, ...) fill in `name` and the `@<trigger>:<event-name>` argument arrays, but not `shortcut` or `shell`, which run as written.

A chord file may also import chords from another file, which is useful for reusing definitions from other chord packages:

```toml
name = "Firefox"

[[import]]
file = "@keychord/chords-chromium/chords/base.toml"
```

> [!NOTE]
> This section is largely AI-written, so parts of it may not be fully accurate. It will be replaced with a hand-written version when I get the time.

Import paths are relative to the package's `chords/` folder, so this example needs `@keychord/chords-chromium` vendored (see [`js/`](#js)). Importing a chord file also imports its handlers. To vary an imported handler's arguments, the imported file declares `$`-prefixed values under `[meta]` and uses them in `args`:

```toml
# @keychord/chords-jetbrains/chords/base.toml
name = "JetBrains Base Chords"

[meta]
'$execPath' = ""

[on.command]
file = "jetbrains.js"
args = ['$execPath']
```

Each import can then override them:

```toml
name = "Android Studio"

[[import]]
file = "@keychord/chords-jetbrains/chords/base.toml"
[import.override.meta]
'$execPath' = "/Applications/Android Studio.app/Contents/MacOS/studio"
```

To replace an imported handler entirely, define `[on.<event-name>]` in the importing file.

## Recommended Third-Party Packages

Here are some curated packages that work well in chord handlers:

- [bplist-lossless](https://github.com/leonsilicon/bplist-lossless) - A binary plist parser specifically tailored for edits by avoiding loss of precision during parsing and re-serialization.
- [doctor-json](https://github.com/privatenumber/doctor-json) - A JSON editor that preserves all existing formatting/comments
- [keycode-ts2](https://github.com/KeyChord/keycode-ts) - A TypeScript port of the [Rust `keycode` crate](https://crates.io/crates/keycode) which uses the Chromium keycode names as the source of truth (_Chord_ uses these keycode names as the source of truth).
