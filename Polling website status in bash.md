# Polling website status in bash

When doing website migrations, when the site doesn't have proper monitoring set up, `curl` in a bash loop may suffice. Or, at least I've written a variation of this (and read `man curl`) enough times to warrant writing it down somewhere.

Paste this function into your shell:

```bash
function poll_website_status {
  HOST="${1:-google.com}"
  SLEEP="${2:-10}"
  BASE="curl --silent --show-error --location --max-time $SLEEP"
  while true; do
    if [ "$3" == "--title" ]; then # print title as result
      RESULT="$($BASE $HOST 2>&1 | grep '<title>' | sed 's/[^<]*//' | cut -c -100)"
    else # print HTTP status code as result
66        RESULT="$($BASE --output /dev/null --write-out '%{http_code}' $HOST 2>&1 | tr '\n' ' ' | sed 's/More details here:.*//')"
    fi
    echo "$(date) --> $HOST --> $RESULT"
    sleep "$SLEEP"
  done
}
```

Now, you can use the `poll_website_status` command to check if `google.com` is up, every 10 seconds:

```console
$ poll_website_status google.com
Tue Dec 11 17:04:10 EET 2018 --> google.com --> 200
Tue Dec 11 17:04:20 EET 2018 --> google.com --> 200
Tue Dec 11 17:04:30 EET 2018 --> google.com --> curl: (6) Could not resolve host: google.com 000
Tue Dec 11 17:04:40 EET 2018 --> google.com --> curl: (6) Could not resolve host: google.com 000
Tue Dec 11 17:04:50 EET 2018 --> google.com --> 200
^C
```

Or, for example, you can check a full URL, every 60 seconds:

```console
$ poll_website_status https://github.com/about 60
Tue Dec 11 17:06:11 EET 2018 --> https://github.com/about --> 200
Tue Dec 11 17:07:12 EET 2018 --> https://github.com/about --> 200
Tue Dec 11 17:08:13 EET 2018 --> https://github.com/about --> 200
^C
```

Or, if the status code of the response isn't a trustworthy indicator of whether it's working or not:

```console
$ poll_website_status github.com 10 --title
Tue Jul  2 11:07:00 EEST 2019 --> github.com --> <title>The world’s leading software development platform · GitHub</title>
Tue Jul  2 11:07:10 EEST 2019 --> github.com --> <title>The world’s leading software development platform · GitHub</title>
Tue Jul  2 11:07:21 EEST 2019 --> github.com --> <title>The world’s leading software development platform · GitHub</title>
^C
```

Not much more to it!
