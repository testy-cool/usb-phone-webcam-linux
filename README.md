# phonecam: use your Android phone as a webcam on Linux over USB

One command turns an Android phone into a webcam for Zoom, Google Meet, OBS and any other app on Linux. The phone stays plugged in over USB, so there is no Wi-Fi setup.

```bash
phonecam
```

```text
Phone camera is on. Pick Loopback video device in your app. Stop it with
phonecam stop.
```

It wakes the phone, opens the DroidCam app on it, connects over `adb`, and feeds the picture into a virtual camera. In your video app, choose **Loopback video device** as the camera.

## Why this exists

Newer Android phones can send their camera to Linux with `scrcpy`. Older ones cannot. On an Android 9 phone, `scrcpy --video-source=camera` stops with this error:

```text
[server] ERROR: Camera mirroring is not supported before Android 12
```

DroidCam works on those older phones. Its Linux client already does the hard part, so `phonecam` only removes the manual steps: finding the phone, waking it, opening the app, and starting the stream.

## What you get

- **One command to start and one to stop.** No app to open by hand on the phone.
- **USB, not Wi-Fi.** It connects through `adb`.
- **The right phone, found for you.** With one phone connected, nothing needs configuring.
- **Plain messages.** A locked phone, a missing phone or a failed connection each say what to do next.

## Requirements

- A Linux computer. It was tested on Linux Mint (Cinnamon, X11).
- An Android phone with **USB debugging** turned on and this computer allowed.
- The **DroidCam** app on the phone.
- The **DroidCam Linux client**, which provides `droidcam-cli` and the virtual camera. Get it from [dev47apps.com/droidcam/linux](https://www.dev47apps.com/droidcam/linux/).
- `adb` (on Debian and Ubuntu: `sudo apt install adb`).
- [uv](https://docs.astral.sh/uv/), which installs the two Python packages the script needs the first time you run it.

## Install

```bash
git clone https://github.com/testy-cool/phone-webcam-linux.git
cd phone-webcam-linux
install -m 755 phonecam ~/.local/bin/phonecam
```

## Use

1. Plug the phone in and unlock it.
2. Run `phonecam`.
3. In your video app, pick **Loopback video device**.
4. Run `phonecam stop` when you are done.

Prop the phone up in landscape, with the camera at eye level. A phone lying flat gives a sideways picture.

## Commands and options

```text
phonecam                 start the phone camera
phonecam stop            stop it
phonecam status          say whether it is running
phonecam --help          show every option
```

| Option | Meaning |
| --- | --- |
| `--serial` | The `adb` serial of the phone. Needed only when several phones are connected. Also read from `PHONECAM_SERIAL`. |
| `--device` | The virtual camera to write to. Default `/dev/video0`. |
| `--port` | The DroidCam port on the phone. Default `4747`. |
| `--hflip` | Mirror the picture left to right. |
| `--vflip` | Flip the picture upside down. |

## How it works

1. It finds the connected phone with `adb devices`.
2. It wakes the screen and stops if the phone is locked.
3. It opens the DroidCam app on the phone.
4. It starts `droidcam-cli` in the background, connected through `adb` on port 4747.
5. `droidcam-cli` writes the video into the virtual camera device.

## Troubleshooting

**"The phone is locked."** Unlock the phone and run `phonecam` again. The script cannot enter a PIN for you.

**"Connection reset! Is the app running?"** The DroidCam app was not in the foreground on the phone. Open it, then run `phonecam` again.

**"Several phones connected."** Run `adb devices`, then `phonecam --serial <serial>`.

**The camera is missing in your app.** Check which virtual cameras exist:

```bash
for d in /dev/video*; do echo "$d $(cat /sys/class/video4linux/$(basename $d)/name)"; done
```

Use the DroidCam device in the list with `--device`. Restart the video app after starting `phonecam`, because some apps only read the camera list when they open.

**The picture drops after a while.** The DroidCam app has to stay open and the phone screen has to stay unlocked. Set the phone to stay awake while charging in its developer options.

## What was and was not tested

Tested by the author:

- Galaxy S8 on Android 9 and Galaxy A54 on Android 15, both over USB.
- Linux Mint (Cinnamon, X11).
- The picture measured 640x480 with the free DroidCam app. The HD button in the app may give more, and it may need a paid version. That was not checked.

Not tested: other Linux distributions, Wayland, iPhones (`droidcam-cli` has an iOS mode that this script does not use), and audio.

## License

MIT. See `LICENSE`. This project is not affiliated with DroidCam or Dev47Apps.
