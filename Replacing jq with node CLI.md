# Replacing jq with node CLI

Interacting with JSON in a shell is traditionally a pain. [`jq`](https://stedolan.github.io/jq/) makes it a tiny bit easier, but I never remember any of its more advanced syntax.

I do, however, remember most of the standard JS [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) and [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) API's by heart.

## Examples

Pretty-printing [the public GitHub gists API](https://api.github.com/gists/public) with node instead of `jq`:

```bash
$ curl -s 'https://api.github.com/gists/public' | node -p 'JSON.parse(fs.readFileSync(0))'
```

Or doing something less trivial:

```bash
$ curl -s 'https://api.github.com/gists/public' | node -p '
  JSON.parse(fs.readFileSync(0))
    .filter(g => g.public)
    .map(g => g.owner.login + " has " + Object.keys(g.files).length + " files")
    .join("\n")
'
```

```
carol00 has 1 files
Untrusted-Game has 1 files
wei3hua2 has 1 files
weicks has 1 files
wawhal has 1 files
Jigar1900 has 1 files
adeweb has 4 files
kooksee has 1 files
SimonDev54 has 7 files
sshnaidm has 1 files
neutrinog has 1 files
matthiasbaldi has 1 files
natthakan159 has 1 files
Untrusted-Game has 1 files
hmatsu47 has 3 files
chocopowwwa has 2 files
dumbest has 1 files
Jigar1900 has 1 files
rtoIedo has 1 files
rtoIedo has 1 files
binaryoung has 1 files
nathan-lapinski has 1 files
nikita8 has 1 files
mrdogra007 has 1 files
anuragambuja has 1 files
noemi-dresden has 1 files
RosiAtanasova has 1 files
PerJoy has 1 files
mi2428 has 2 files
OnyxM has 2 files
```

## How it works

This works because [`readFileSync`](https://nodejs.org/api/fs.html#fs_fs_readfile_path_options_callback) can, instead of a filename, take a file descriptor as its first argument, and a node process running on the CLI will automatically have `0` pointing to its `stdin`.

To be honest, I'm not entirely sure why the `fs` module is available by default in REPL mode, couldn't find any docs mentioning it. But hey, not going to complain.
