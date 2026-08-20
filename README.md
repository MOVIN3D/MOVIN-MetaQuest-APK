# MOVIN Quest Companion APK

**English** | [한국어](README.ko.md)

[![Latest release](https://img.shields.io/github/v/release/MOVIN3D/MOVIN-MetaQuest-APK?display_name=tag&label=latest)](https://github.com/MOVIN3D/MOVIN-MetaQuest-APK/releases/latest)

A Quest-side companion app that streams hand tracking data from a Meta Quest headset to MOVIN Studio running on a PC on the same Wi-Fi. This is not a Meta Store build, so it has to be installed once via USB from a PC.

> MOVIN Studio runs on **Windows and Linux**. The one-time APK installation requires only a PC with `adb`, and it does not have to be the PC that runs Studio — installing from a Windows PC and then streaming to Studio on Linux works.

---

## 1. Prerequisites

### Headset

Quest 2, Quest 3, Quest 3S, or Quest Pro. The build targets Android API 32, so the first-generation Quest cannot run it.

### Meta account / Developer Mode

> Quest requires **Developer Mode** to be enabled in order to run apps installed from outside the Meta Store.

1. Create a Meta developer account at https://developers.meta.com/horizon/ (free, one-time organization registration required).
2. Install the **Meta Horizon app** on your phone and sign in.
3. In the app: `Menu` → `Devices` → select your Quest → `Developer Mode` → **ON**.
4. Reboot the Quest headset once.

### USB-C cable

You need a USB-C cable that supports data transfer (charge-only cables won't work). The cable that came with the Quest or an official Quest Link cable will do.

### APK file

**[⬇ Download MOVINQuestCompanion.apk](https://github.com/MOVIN3D/MOVIN-MetaQuest-APK/releases/latest/download/MOVINQuestCompanion.apk)** — this link always serves the newest build.

Older builds are on the [Releases](https://github.com/MOVIN3D/MOVIN-MetaQuest-APK/releases) page. Download the APK, not the `Source code` archives GitHub attaches automatically — those hold nothing but these README files.

---

## 2. Install ADB

ADB (Android Debug Bridge) is the tool used to install the APK from your PC onto the Quest.

### Windows

**Option A — winget (simplest)**

```powershell
winget install --id=Google.PlatformTools -e
```

**Option B — Direct download**

1. Download the ZIP from https://developer.android.com/tools/releases/platform-tools.
2. Extract it to a path with no spaces or non-ASCII characters (e.g. `C:\platform-tools`).
3. Add `C:\platform-tools` to your PATH environment variable (`Win` → `Edit the system environment variables` → `Path` → `Edit` → `New`).

After installing, open a new PowerShell window and run `adb version` to verify.

### Linux

```bash
sudo apt install adb                     # Ubuntu / Debian
sudo dnf install android-tools           # Fedora
sudo pacman -S android-tools             # Arch
```

Linux also needs a udev rule before your user account can talk to the headset — without it `adb devices` lists the Quest as `no permissions`. On Ubuntu/Debian, installing this package adds the rules for you:

```bash
sudo apt install android-sdk-platform-tools-common
```

On other distributions, add the rule by hand and make sure your user is in the `plugdev` group. `2833` is Meta's USB vendor ID:

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="2833", MODE="0666", GROUP="plugdev"' \
  | sudo tee /etc/udev/rules.d/51-android.rules
sudo udevadm control --reload-rules && sudo udevadm trigger
sudo usermod -aG plugdev $USER    # log out and back in for this to take effect
adb kill-server
```

---

## 3. Connect the Quest to your PC

1. Plug the Quest into your PC with the USB-C cable.
2. **While wearing the Quest**, look for the `Allow USB debugging?` dialog inside the headset, check `Always allow from this computer`, and choose `Allow`.
3. Verify the device shows up on the PC:
   ```bash
   adb devices
   ```
   ```
   List of devices attached
   1WMHHxxxxxxxxx    device
   ```

If you see `unauthorized`, reconnect the cable to bring the dialog back and allow USB debugging again.

---

## 4. Install the APK

From the folder you downloaded the APK into:

```bash
adb install MOVINQuestCompanion.apk
```

To overwrite an existing build with a new one (preserving app data):

```bash
adb install -r MOVINQuestCompanion.apk
```

---

## 5. Launch the app on the Quest

1. Put the headset on.
2. Open `App Library` → use the category dropdown in the upper-right and choose `Unknown Sources`.
3. Launch `MOVINQuestCompanion`.
4. On first launch, allow the `Hand Tracking` permission prompt. That is the only prompt the app raises; network access is granted at install time, so no other permission prompt appears.

> Make sure `Settings` → `Movement Tracking` → `Hand Tracking` is enabled on the headset itself. If hand tracking isn't available to the app, a red `⚠ Hand tracking permission required — allow it in Quest Settings` line appears under the title about 3 seconds after launch.

> The app runs boundary-less on purpose: while it is in the foreground the Guardian boundary is not drawn, and the headset will not switch to passthrough when you move beyond the boundary. Mocap movement would otherwise interrupt the hand tracking pipeline constantly.

---

## 6. How to use the app

### Pointing and clicking

Controllers and bare hands both work, and each hand is independent — you can hold a controller in one hand and point with the other.

- **Controller** — pick one up and a **cyan** ray appears, angled 40° downward so it points naturally at the panel in front of you. Pull the **trigger** to click; the ray turns **green** while the trigger is held.
- **Hand** — with no controller in that hand, a **white** ray projects from your hand along the aim direction the headset computes. **Pinch** thumb and index together to click; the ray turns **green** while pinching.

A pinch only registers while the ray is resting on a button, so normal hand movement during mocap does not trigger UI actions. A ray disappears for a hand that is neither holding a controller nor being tracked.

Once the connection is up you can put the controller down — nothing on the status screen needs any further input.

### First screen — pick the PC to connect to

Both ways of choosing a target are on the screen at the same time.

**Top — automatic discovery.** The app broadcasts on the local network once per second and lists every PC running MOVIN Studio that answers.

- The header reads `Searching for MOVIN Studio...` until something answers, then `Found 1 host:` / `Found N hosts:`.
- Up to 5 hosts are shown, most recently seen first, each as `hostname   ip:port`. A host that goes quiet for 5 seconds drops off the list.
- Click a host to connect straight away, using the port that host advertised.

**Bottom — manual entry**, under the `— Or type IP manually —` separator.

- Type the PC's IP with the numpad (`1`–`9`, `0`, `.`, `⌫`; 15 characters max) and press `Connect`.
- An empty field shows `Enter the host PC's IP address`; a malformed address shows `Invalid IP address` and turns the IP text red.
- Manual connections always use port 14044.

> The IP is saved whenever you connect and comes back pre-filled next launch. On a headset that has never connected, the field starts at `192.168.1.100`.

### Connection status screen

The top line shows the selected target as `ip:port`.

| State | Meaning |
|------|------|
| `○ Waiting...` (gray) | Idle |
| `⟳ Connecting...` (blue, pulsing) | Packets are going out, no reply yet (up to 5 seconds) |
| `● Connected` (green) | Studio is replying — last reply within 1.5 s |
| `⚠ No response from Studio` (yellow, pulsing) | No reply for over 1.5 s, or none at all within 5 s of pressing Connect |

The line below the status changes with the state:

- **While connected** it shows hand tracking as `● L   ● R` (● = the headset cameras see that hand, ○ = they don't). The app keeps sending keepalive packets even with no hands in view, so `● Connected` alongside `○ L   ○ R` is normal.
- **When not connected** it shows a diagnostic counter line instead, e.g. `disc 1 · sent 72/s · ack 0`, followed by a one-line hint. Section 9 explains both.

The red `Disconnect` button at the bottom stops sending and returns you to the first screen, where you can pick a different PC.

### Panel position

The panel floats about 1 m in front of you, slightly below eye level. It is **not** rigidly head-locked: it stays where it is while you turn your head within roughly 20°, and only eases back in front of you once you turn past that, settling as soon as it is facing you again.

---

## 7. Network settings

- Connect the Quest and the Studio PC to the **same Wi-Fi**.
- The first time you launch MOVIN Studio on the PC, Windows may show a network access prompt. Choose `Allow access`.
- Some corporate or public Wi-Fi networks block device-to-device traffic. In that case, use a separate router and connect only the Studio PC and the Quest to it.

The app uses these ports; open them in your firewall if necessary:

| Port | Direction | Purpose |
|---|---|---|
| UDP 14044 | Quest → PC | Hand data. Studio's replies arrive back on the same socket. |
| UDP 14045 | Quest → broadcast | Discovery request |
| UDP 14046 | PC → Quest | Discovery reply |

Blocking 14045/14046 only breaks the automatic host list — typing the IP by hand still connects. Blocking 14044 breaks the connection itself.

---

## 8. Smoke test

1. On the PC, launch MOVIN Studio and enter the mocap scene.
2. Toggle `Hand Tracking` ON, and choose `Quest` from the `Hand Type` dropdown.
3. Put the Quest on and launch the companion app.
4. Pick your PC from the discovered host list (or type its IP) and connect.
5. Confirm the status reads `● Connected` with `● L   ● R`, and that the character's hands follow yours on the PC screen.

For the full usage flow, see the GitBook guide — **MOVIN Studio Usage Guide → Hands → Quest**.

---

## 9. Troubleshooting

### Device doesn't show up in `adb devices`

| Symptom | Fix |
|---|---|
| Empty list | Verify the cable supports data transfer / try a different USB port |
| `unauthorized` | Reconnect the cable to bring the dialog back, then allow USB debugging inside the Quest again |
| (Windows) Yellow exclamation mark | Install the [Oculus ADB Driver](https://developers.meta.com/horizon/downloads/package/oculus-adb-drivers/) |
| (Linux) `no permissions` | The udev rule is missing — see the Linux part of section 2, then re-plug the cable |

### Install errors (`INSTALL_FAILED_*`)

| Code | Fix |
|---|---|
| `ALREADY_EXISTS` | Reinstall with the `-r` flag |
| `UPDATE_INCOMPATIBLE` | `adb uninstall com.movin.questcompanion`, then install again |
| `INSUFFICIENT_STORAGE` | Free up space on the Quest (Settings → Storage) |

### The app can't reach Studio

When the connection isn't healthy the status screen shows a counter line and a hint. The counters are:

| Counter | Meaning |
|---|---|
| `disc` | Number of Studio PCs currently visible to discovery |
| `sent` | Packets per second leaving the headset |
| `ack` | Replies received from Studio since you pressed Connect |

Then find the hint shown underneath them in the table below.

| Hint on the headset | What it means | Fix |
|---|---|---|
| `Not sending — socket/target issue.` | The app never got a packet out | Press `Disconnect` and connect again; re-check the IP you entered |
| `No reply → same Wi-Fi? firewall? Studio in the mocap scene?` | Nothing discovered and nothing replying | Put both devices on the same SSID, check that the network doesn't isolate clients, and make sure Studio is in the mocap scene |
| `PC found but silent → on Studio enable hand-tracking + Quest, or firewall on UDP 14044.` | Discovery found the PC, but it isn't answering the data stream | In Studio, turn `Hand Tracking` ON and set `Hand Type` to `Quest`; allow UDP 14044 through the PC firewall |
| `Reply intermittent — Wi-Fi/latency.` | Replies arrive but keep dropping out | Weak Wi-Fi — move closer to the router or switch to 5 GHz |

If you clicked `Block` on the Windows network prompt at some point, re-allow MOVIN Studio under `Control Panel → System and Security → Windows Defender Firewall → Allow an app through firewall`.

### Other app issues

| Symptom | Fix |
|---|---|
| No hosts ever appear in the list | Discovery (UDP 14045/14046) is blocked or the two devices are on different subnets — type the IP by hand instead |
| `⚠ Hand tracking permission required` stays on screen | Enable `Settings → Movement Tracking → Hand Tracking` on the headset, then restart the app |
| Connected, but `○ L   ○ R` | Hands are out of the headset cameras' view, or hand tracking is off on the headset — the network link itself is fine |
| The ray doesn't appear | Controllers: pick one up so the headset registers it as held. Hands: they have to be tracked, and the hand ray only shows for a hand that isn't holding a controller |

---

## 10. Uninstall

```bash
adb uninstall com.movin.questcompanion
```
