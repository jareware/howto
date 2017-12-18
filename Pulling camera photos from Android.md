# Pulling camera photos from Android

List all photos from e.g. 2017:

    adb shell 'find /storage/emulated/0/DCIM/Camera | grep -E "/[A-Z]+_2017"'

Pull all photos from 2017-01:

    adb pull $(adb shell 'find /storage/emulated/0/DCIM/Camera | grep -E "/[A-Z]+_201701"') .

Remove the same photos from device:

    adb shell 'find /storage/emulated/0/DCIM/Camera | grep -E "/[A-Z]+_201701" | xargs rm -f'

Note that `-r` is omitted from `rm` for safety. Burst photos (if you take any) will be stored as directories, so make note of any "is a directory" errors, and either remove them manually afterwards, or add the `-r`.

To revel in the glory of all the free space you now have:

    adb shell 'df -h /storage/emulated'
