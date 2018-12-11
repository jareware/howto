# Polling website status in bash

When doing website migrations, when the site doesn't have proper monitoring set up, `curl` in a bash loop may suffice. Or, at least I've written this (and read `man curl`) enough times to warrant writing it down somewhere.

Paste this function into your shell:

```bash
function poll_website_status {
  HOST="${1:-google.com}"
  SLEEP="${2:-5}"
  while true; do
    RESULT="$(curl --silent --show-error --location --max-time $SLEEP --output /dev/null --write-out '%{http_code}' $HOST 2>&1 | tr '\n' ' ')"
    echo "$(date) --> $HOST --> $RESULT"
    sleep "$SLEEP"
  done
}
```

Now, you can use the `poll_website_status` command to check if `google.com` is up, every 5 seconds:

```console
$ poll_website_status google.com
Tue Dec 11 17:04:25 EET 2018 --> google.com --> 200
Tue Dec 11 17:04:30 EET 2018 --> google.com --> 200
Tue Dec 11 17:04:35 EET 2018 --> google.com --> curl: (6) Could not resolve host: google.com 000
Tue Dec 11 17:04:40 EET 2018 --> google.com --> curl: (6) Could not resolve host: google.com 000
Tue Dec 11 17:04:45 EET 2018 --> google.com --> 200
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

Not much more to it!
