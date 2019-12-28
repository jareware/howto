# Figuring out the self-path of a shell script

Quite often your shell scripts need to reference other files that are sitting next to them in the same dir. But the script might be invoked from a different dir.

After writing a 1000 different (but same) variations of this across as many shell scripts, it's time to write down my canonical versions of ways to figure out the self-path of the currently executing script.

## Goal

Assuming your script lives in `/home/jara/bin/script.sh`, we want `$SELF_DIR` to be `/home/jara/bin`, regardless of whether it's invoked as:

* `/home/jara/bin/script.sh`
* `./bin/script.sh`
* `/home/jara/bin/../bin/script.sh`
* or just `script.sh` (assuming it's on your `$PATH`)

## Method 1

```bash
SELF_DIR="$(realpath $(dirname "$0"))" # figure out the absolute path to the current script, regardless of pwd
```

**Pro:** This is the most intuitive solution.

**Con:** `realpath` may not be available on all systems by default (e.g. macOS).

## Method 2

```bash
SELF_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" >/dev/null 2>&1 && pwd)" # figure out the absolute path to the current script, regardless of pwd
```

**Pro:** Doesn't depend on external programs besides POSIX standard ones, but...

**Con:** ...depends on `bash`. So if you go with this, make sure to specifically use `#!/bin/bash` as your shebang line. Also, while this produces absolute paths, it doesn't resolve symlinks. Depending on what you want, this may be a bug or a feature.

## Method 3

```bash
SELF_DIR="$(perl -e 'use File::Basename; use Cwd "abs_path"; print dirname(abs_path(@ARGV[0]));' -- "$0")" # figure out the absolute path to the current script, regardless of pwd (perl is more cross-platform than realpath; https://stackoverflow.com/a/30795461)
```

**Pro:** This is the best solution (assuming you want symlinks resolved); `perl` is more likely to be available on the system your script runs on than many other programs.

**Con:** It's a bit... fugly.
