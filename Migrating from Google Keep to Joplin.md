# Migrating from Google Keep to Joplin

As part of a continuing effort of de-Googling my life, I recently migrated my notes from [Google Keep](https://www.google.com/keep/) to [Joplin](https://joplinapp.org/). I'm already a heavy user of Dropbox, so after discovering Joplin uses Dropbox as its sync backend, and notes are stored in simple Markdown files, I was sold.

## Exporting

Anyway, there's not a ready-made migration tool for this task, and I didn't end up making one, either, but here's my process, for posterity:

1. Use [Google Takeout](https://takeout.google.com/) to export your Keep notes into a file
1. Use [`google-keep-exporter`](https://github.com/vHanda/google-keep-exporter) to convert the notes to Markdown
1. If you can live with the notes being a bit messy, you're done!

## Reformatting

If you want more compatible Markdown formatting (e.g. checkbox lists) and creation dates and titles being imported correctly, you need to do a bit of massaging, still.

In retrospect, patching `google-keep-exporter` with these changes would have made more sense, but I had already written most of the following shell script when I reached that conclusion. Oh well!

```bash
function keep_md_to_joplin_md {
  notebookUuid="$1"
  uuid="$2"
  file="$3"
  date="$(cat $file | grep '^date: ' | sed 's/^date: //g' | sed "s/[ ']//g")"
  headerLines="$(cat $file | sed -n '/---/,/---/p' | wc -l)"
  title="$(
    if (cat $file | grep '^title: >-' > /dev/null); then # multi-line title
      cat $file | sed -n '/^title: >-/,/^[^ ]/p' | head -n -1 | tail -n +2 | xargs
    elif (cat $file | grep '^title: ' > /dev/null); then # single-line title
      cat $file | grep '^title: ' | sed 's/^title: //' | xargs
    else # no title
      cat $file | tail -n "+$headerLines" | tail -n +3 | head -n 1 # i.e. just use the first body line
    fi
  )"
  body="$(cat $file | tail -n "+$headerLines" | tail -n +3 | sed 's/^\\- /- /g' | sed 's/^\[\(.\)\] /- [\1] /g')"
  echo -n "\
$title

$body

id: $uuid
parent_id: $notebookUuid
created_time: $date
updated_time: $date
is_conflict: 0
latitude: 0.00000000
longitude: 0.00000000
altitude: 0.0000
author: 
source_url: 
is_todo: 0
todo_due: 0
todo_completed: 0
source: joplin
source_application: net.cozic.joplin-mobile
application_data: 
order: 0
user_created_time: $date
user_updated_time: $date
encryption_cipher_text: 
encryption_applied: 0
markup_language: 1
type_: 1"
}

notebookNow="$(date --iso-8601=seconds --utc | sed 's/\+.*/.000Z/')"
notebookUuid="$(uuid | sed 's/-//g')"

rm -rf import-to-joplin
mkdir import-to-joplin

echo -n "\
import-to-joplin

id: $notebookUuid
created_time: $notebookNow
updated_time: $notebookNow
user_created_time: $notebookNow
user_updated_time: $notebookNow
encryption_cipher_text: 
encryption_applied: 0
parent_id: 
type_: 2" > "import-to-joplin/$notebookUuid.md"

for f in $(ls *.md); do
  noteUuid="$(uuid | sed 's/-//g')"
  echo "$f"
  keep_md_to_joplin_md "$notebookUuid" "$noteUuid" "$f" > "import-to-joplin/$noteUuid.md"
done
```

That is, the script reads all `.md` files from the current dir, reformats them to the Joplin "raw" format, and outputs the reformatted (and renamed) files to a dir called `import-to-joplin`.

## Importing

This is the simple part: in Joplin (the desktop app), select "Import" and "RAW - Joplin Export Directory". Navigate to your `import-to-joplin` dir, and hit OK.

After the process completes (may take a while if you have a lot notes), you should end up with a new notebook called "import-to-joplin". Rename it to something sensible, and you're done.

## Gotchas

* One thing I noticed was a few errors about unterminated quotes when running the script for the first time. This happened in notes/files that had quotes (double or single) in their title. Since the issue manifested in only a few files, I was too lazy to tweak the script, and just corrected the problem by hand where needed. You probably want to do this beforehand.
* The script was ran on a Linux box. It may require some minor tweaks on macOS; particularly the macOS flavors of `sed` and `date` always require ever-so-slightly different arguments to achieve the same effects.
