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

- `name`: The name of the chord package.

> For local chord packages, if `package.json` is omitted, the chord package's name will instead be inferred from its folder name. Remote chord packages (such as those from a public Git repository) MUST contain a `package.json` file or Chord will refuse to load them.

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

<<<<<<< HEAD
## Chord files

A chord file typically contains the following top-level properties:

- `name`
- `handlers`
- `chords`

### Handlers

A chord file may define zero or more handlers for custom events emitted by chords (through the `emit:<event-name>` property on chord definitions.

```toml
[handlers.menu]
type = "js"
file = "@keychord/chords-menu/js/menu.js"
args = ["WhatsApp"]

[chords]
cvi = { name = "Chat: Video Call", 'emit:menu' = ["Chat", "Video Call"] }
```

### `chords`

A chord definition may consist of a regex pattern:

```toml
[chords]
'/l(\d+)' = { name = "Lungo: Minutes", shell = "open --background 'lungo:activate?minutes=$1'" }
```

It may also "forward" to another file (useful for reusing definitions from other chord packages):

```toml
name = "Firefox"

[chords.'(.*)']
file = "@keychord/chords-chromium/chords/base.toml"
```

When forwarding to another file with chords that emit events, you may also register a custom handler for those events that takes priority:

```toml
name = "Android Studio"

[handlers.command]
type = "js"
file = "@keychord/chords-jetbrains/js/jetbrains.js"
args = ["/Applications/Android Studio.app/Contents/MacOS/studio"]

[chords.'(.*)']
file = "@keychord/chords-jetbrains/chords/base.toml"
'@command' = 'command'
```

## Recommended Third-Party Packages

Unfortunately, the LLRT runtime isn't 100% Node.js compatible, so some common npm packages won't work out of the box. Here are some curated ones that are known to work well:

- [nano-spawn-compat](https://github.com/leonsilicon/nano-spawn-compat) - A more ergonomic `child_process.spawn` that works in LLRT (whose `child_process` module only provides `spawn`).
- [bplist-lossless](https://github.com/leonsilicon/bplist-lossless) - A binary plist parser specifically tailored for edits by avoiding loss of precision during parsing and re-serialization.
- [doctor-json](https://github.com/privatenumber/doctor-json) - A JSON editor that preserves all existing formatting/comments
- [keycode-ts2](https://github.com/leonsilicon/keycode-ts2) - A TypeScript port of the [Rust `keycode` crate](https://crates.io/crates/keycode) which uses the Chromium keycode names as the source of truth (_Chord_ uses these keycode names as the source of truth).
