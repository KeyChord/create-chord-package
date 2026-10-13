# create-chord-package

A helper CLI for creating a new chord package.

## Structure

```
package.json
chords/
js/
bin/
```

### `package.json`

A chord package must have a `package.json` file with the following fields:

- `name`: The name of the chord package. Must be in the form `@<username>/<slug>`.

> For local chord packages, if `package.json` is omitted, the chord package's name will instead be inferred from the device name + its folder name. Remote chord packages (such as those from a public Git repository) MUST contain a `package.json` file or Chord will refuse to load them.

### `chords/`

A chord package must have a root `chords/` directory.

### `js/`

If a chord package uses JavaScript, they must be located in a root `js/` directory.

The files in the `js/` directory must be self-contained; they CANNOT import any external files or modules outside the `js/` folder.

To allow for de-duplication of external packages, any non-builtin package imports are resolved from the root `js/` folder. For example, the following import:

```js
// js/example.js
import buildMenuHandler from '@keychord/chords-menu/js/menu.js';
```

Will resolve at runtime to:

```js
// js/example.js

import buildMenuHandler from './@keychord/chords-menu/js/menu.js';
```

### `bin/`

If a chord package has binaries, they must be located in a root `bin/` directory.

## Chord files

A chord file typically contains the following top-level properties:

- `name`
- `handlers`
- `chords`
- `import`

### Handlers

A chord file may define zero or more handlers for custom events emitted by chords through the `@input:<event-name>` property on chord definitions. Its value is an array of arguments passed to the handler, and may contain regex captures such as `$1`.

```toml
[on.menu]
file = "@keychord/chords-menu/js/menu.js"
args = ["WhatsApp"]

[chords]
cvi = { name = "Chat: Video Call", '@input:menu' = ["Chat", "Video Call"] }
```

In order to avoid conflicts, `@input:<event-name>` will first emit an event namespaced to the package name (`@user/pkg:<event-name>`), and if that isn't handled, then it will emit the non-namespaced event (`<event-name>`). This allows chord files that import other files to override their handlers.

Use `@init:<event-name>` for initialization when a chord first matches, and `@keydown`, `@keypress`, or `@keyup` for live keyboard callbacks. Initialization resets when that chord stops matching. All five triggers can assign their handler's return value with `=<identifier>`:

```toml
[chords.'^j,(.*)$']
name = "Live clicks"
'@init:prepare=context' = []
'@keydown:update=latest' = ['$context', '$1']
'@keyup:released' = ['$latest']
```

Declare `prepare`, `update`, and `released` under `[on]` as usual. Return values persist for that chord's current matching interval, and Promise results are awaited before later callbacks. Handlers receive Web KeyboardEvent fields through `this.event`, so they can filter keys with `this.event.key === "Backspace"`. Chord also delivers `@keypress` for non-character keys. See [the full event contract](https://github.com/KeyChord/Chord/blob/dev/scripting.md#chord-events-and-persistent-return-values).

### `chords`

A chord definition may consist of a regex pattern:

```toml
[chords]
'/l(\d+)' = { name = "Lungo: Minutes", shell = "open --background 'lungo:activate?minutes=$1'" }
```

A chord file may also import chords from another file, which is useful for reusing definitions from other chord packages:

```toml
name = "Firefox"

[[import]]
file = "@keychord/chords-chromium/chords/base.toml"
```

Note that importing other chord files also imports their handlers as well. If you need to override their handlers, you can explicitly register a handler for events from that chord file by scoping the event name with the package name:

```toml
name = "Android Studio"

[on.command]
file = "@keychord/chords-jetbrains/js/jetbrains.js"
args = ["/Applications/Android Studio.app/Contents/MacOS/studio"]

[[import]]
file = "@keychord/chords-jetbrains/chords/base.toml"
':command' = command
```

## Recommended Third-Party Packages

Unfortunately, the LLRT runtime isn't 100% Node.js compatible, so some common npm packages won't work out of the box. Here are some curated ones that are known to work well:

- [nano-spawn-compat](https://github.com/leonsilicon/nano-spawn-compat) - A more ergonomic `child_process.spawn` that works in LLRT (whose `child_process` module only provides `spawn`).
- [bplist-lossless](https://github.com/leonsilicon/bplist-lossless) - A binary plist parser specifically tailored for edits by avoiding loss of precision during parsing and re-serialization.
- [doctor-json](https://github.com/privatenumber/doctor-json) - A JSON editor that preserves all existing formatting/comments
- [keycode-ts2](https://github.com/leonsilicon/keycode-ts2) - A TypeScript port of the [Rust `keycode` crate](https://crates.io/crates/keycode) which uses the Chromium keycode names as the source of truth (_Chord_ uses these keycode names as the source of truth).
