# MOVIN Quest Companion APK

**English** | [한국어](README.ko.md)

A Quest-side companion app that streams hand tracking data from a Meta Quest headset to MOVIN Studio running on a PC on the same Wi-Fi. This is not a Meta Store build, so it has to be installed once via USB from a PC.

---

## 1. Prerequisites

### Meta account / Developer Mode

> Quest requires **Developer Mode** to be enabled in order to run apps installed from outside the Meta Store.

1. Create a Meta developer account at https://developer.oculus.com (free, one-time organization registration required).
2. Install the **Meta Horizon app** on your phone and sign in.
3. In the app: `Menu` → `Devices` → select your Quest → `Developer Mode` → **ON**.
4. Reboot the Quest headset once.

### USB-C cable

You need a USB-C cable that supports data transfer (charge-only cables won't work). The cable that came with the Quest or an official Quest Link cable will do.

### APK file

Use the `MOVINQuestCompanion.apk` file located alongside this README. You can either download the file directly from GitHub web or clone the repository and use the file at the same path.

---

## 2. Install ADB

ADB (Android Debug Bridge) is the tool used to install the APK from your PC onto the Quest.

### Windows

**Option A — winget (simplest)**

```powershell
winget install --id=Google.PlatformTools -e
```

**Option B — Direct download**

1. Download the zip from https://developer.android.com/tools/releases/platform-tools.
2. Extract it to a path with no spaces or non-ASCII characters (e.g. `C:\platform-tools`).
3. Add `C:\platform-tools` to your PATH environment variable (`Win` → `Edit the system environment variables` → `Path` → `Edit` → `New`).

After installing, open a new PowerShell window and run `adb version` to verify.

### Linux

```bash
sudo apt install android-tools-adb       # Ubuntu / Debian
sudo dnf install android-tools           # Fedora
sudo pacman -S android-tools             # Arch
```

---

## 3. Connect the Quest to your PC

1. Plug the Quest into your PC with the USB-C cable.
2. **While wearing the Quest**, look for the "Allow USB debugging?" dialog inside the headset, check **`Always allow from this computer`**, and choose **`Allow`**.
3. Verify the device shows up on the PC:
   ```bash
   adb devices
   ```
   ```
   List of devices attached
   1WMHHxxxxxxxxx    device
   ```

If you see `unauthorized`, you need to accept the allow-dialog again inside the headset (reconnect the cable to make it appear).

---

## 4. Install the APK

From the folder containing the APK file:

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
2. Open `App Library` → use the category dropdown in the upper-right and choose **`Unknown Sources`**.
3. Launch **`MOVIN Quest Companion`**.
4. On first launch, allow both the **Hand Tracking** and **Local Network** permission prompts.

> Make sure `Settings` → `Movement Tracking` → `Hand Tracking` is enabled on the headset itself. If hand tracking permission isn't active, the app shows a `⚠ Hand tracking permission required` warning at the top of its UI.

---

## 6. How to use the app

The UI is **controller-only**. Hand tracking input is intentionally not used — that prevents the user's hands, which keep moving during mocap, from accidentally pressing UI elements. Once you've finished setting up the connection, you can put the controller down — that's when the mocap-side hand tracking activates.

### Controller interaction

- Pick up a controller and a ray appears, pointing forward (tilted 40° downward) — **cyan** at rest, **green** while the trigger is held.
- Aim the ray at a button or numpad key and pull the **trigger** to click.
- When you put the controller down, the ray disappears.

### First screen — Choose which PC to connect to

When the app launches, it scans the local Wi-Fi for any PC running MOVIN Studio.

- **When a PC is found** — A `Found N hosts:` header appears at the top of the screen, followed by up to 5 buttons each showing the PC's name and `IP:port`. Aim the ray at the host you want and pull the trigger to start the connection.
- **When no PC is found** — Type the PC's IP (e.g. `192.168.1.100`) using the numpad in the middle (1–9 / 0 / `.` / `⌫`) and press the **Connect** button below.

> The IP you enter is saved automatically and shows up the next time you launch the app.

### Connection status screen

Once you press Connect the screen switches to the connection status view.

| State | Meaning |
|------|------|
| `○ Waiting...` (gray) | Idle |
| `⟳ Connecting...` (blue, pulsing) | Sending packets, waiting for a reply (up to 5 seconds) |
| `● Connected` (green) | The PC is responding; hand data is being streamed |
| `⚠ No response from Studio` (yellow, pulsing) | No reply for 1.5+ seconds |

> If no response arrives within 5 seconds of pressing Connect, the screen also shows `No response. Check the IP and that the PC is on the same Wi-Fi.` as a hint.

Below the status label, the left/right hand tracking state is shown as **● L   ● R** (●=tracked, ○=not tracked).

To connect to a different PC, press the red **Disconnect** button at the bottom to return to the first screen.

### Panel position

The panel is head-locked — it stays in front of you wherever you turn or move. Mocap usage takes the headset off and wears it around the neck, so there's no concern about the UI obstructing the user's view during the actual capture.

---

## 7. Network settings

- Connect the Quest and the Studio PC to the **same Wi-Fi**.
- The first time you launch MOVIN Studio on the PC, Windows may show a network access prompt. Choose **'Allow access'**.
- Some corporate or public Wi-Fi networks block device-to-device traffic. In that case, use a separate router and connect only the Studio PC and the Quest to it.

---

## 8. Smoke test

1. On the PC, launch MOVIN Studio and enter the mocap scene.
2. Toggle `Hand Tracking` ON, and choose **Quest** from the `Hand Type` dropdown.
3. Put the Quest on and launch the companion app.
4. Confirm the character's hands follow yours on the PC screen.

For the full usage flow, see the GitBook guide — **MOVIN Studio Usage Guide → Hands → Quest**.

---

## 9. Troubleshooting

### Device doesn't show up in `adb devices`

| Symptom | Fix |
|---|---|
| Empty list | Verify the cable supports data transfer / try a different USB port |
| `unauthorized` | Accept the allow-dialog inside the Quest again (reconnect the cable to re-trigger it) |
| (Windows) Yellow ! mark | Install the [Oculus ADB Driver](https://developer.oculus.com/downloads/package/oculus-adb-drivers/) |

### Install errors (`INSTALL_FAILED_*`)

| Code | Fix |
|---|---|
| `ALREADY_EXISTS` | Reinstall with the `-r` flag |
| `UPDATE_INCOMPATIBLE` | `adb uninstall com.movin.questcompanion`, then install again |
| `INSUFFICIENT_STORAGE` | Free up space on the Quest (Settings → Storage) |

### Studio doesn't see the Quest

1. Make sure both devices are on the **exact same Wi-Fi (SSID)**.
2. If you ever clicked 'Block' on the Windows network access prompt, re-allow MOVIN Studio under `Control Panel → System and Security → Windows Defender Firewall → Allow an app through firewall`.
3. On corporate/public Wi-Fi, try a separate router with only the two devices on it.
4. Quit the companion app on the Quest and re-launch it.

---

## 10. Uninstall

```bash
adb uninstall com.movin.questcompanion
```
