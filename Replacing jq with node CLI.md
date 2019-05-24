# Replacing jq with node CLI

Interacting with JSON in a shell is traditionally a pain. [`jq`](https://stedolan.github.io/jq/) makes it a tiny bit easier, but I never remember any of its more advanced syntax.

I do, however, remember most of the standard JS [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) and [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) API's by heart.

## Examples

Pretty-printing [the public GitHub gists API](https://api.github.com/gists/public) with node instead of `jq`:

```bash
curl -s 'https://api.github.com/gists/public' | node -p 'JSON.parse(fs.readFileSync(0))'
```

If you're writing a script, consider making a function:

```bash
function json-select {
  node -p "JSON.parse(fs.readFileSync(0))$1" 2> /dev/null # error silently when the selector doesn't match
}
```

This makes things nicer to read, and gives you less escaping headaches if you need to capture the output to a variable, for example:

```bash
curl -s 'https://api.github.com/gists/public' | json-select '[0].owner.repos_url'
```

And remember that you have the full JavaScript Array/Object API at your disposal:

```bash
curl -s 'https://api.github.com/gists/public' | json-select '
  .filter(g => g.public)
  .map(g => g.owner.login + " has " + Object.keys(g.files).length + " files")
  .join("\n")
'
```

This would give you somthing like:

```
imarklee has 1 files
aarighi has 1 files
mymizan has 3 files
Rodge1979 has 1 files
billyxs has 1 files
...
```

## How it works

This works because [`readFileSync`](https://nodejs.org/api/fs.html#fs_fs_readfile_path_options_callback) can, instead of a filename, take a file descriptor as its first argument, and a node process running on the CLI will automatically have `0` pointing to its `stdin`.

To be honest, I'm not entirely sure why the `fs` module is available by default in REPL mode, couldn't find any docs mentioning it. But hey, not going to complain.
