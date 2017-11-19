# Pulling camera photos from Android

List all photos from 2017:

    adb shell 'find /storage/emulated/0/DCIM/Camera | grep -E "/[A-Z]+_201701"'

Pull all photos from 2017-01:

    adb pull $(adb shell 'find /storage/emulated/0/DCIM/Camera | grep -E "/[A-Z]+_201701"') .

Remove the same photos from device:

    adb shell 'find /storage/emulated/0/DCIM/Camera | grep -E "/[A-Z]+_201701" | xargs rm -f'
