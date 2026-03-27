# create-chord-package

A helper CLI for creating a new chord package.

## Structure

```
chords/
js/
bin/
```

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
