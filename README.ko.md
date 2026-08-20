# MOVIN Quest Companion APK

[English](README.md) | **한국어**

[![Latest release](https://img.shields.io/github/v/release/MOVIN3D/MOVIN-MetaQuest-APK?display_name=tag&label=latest)](https://github.com/MOVIN3D/MOVIN-MetaQuest-APK/releases/latest)

Meta Quest 헤드셋의 손 추적 데이터를 같은 Wi-Fi에 있는 PC의 MOVIN Studio로 전송하는 Quest용 컴패니언 앱입니다. 정식 스토어 빌드가 아니라서 한 번은 PC에서 USB로 직접 설치해야 합니다.

> MOVIN Studio는 **Windows와 Linux**를 지원합니다. APK 설치에는 `adb`가 설치된 PC만 있으면 되고, 설치용 PC와 Studio 실행 PC는 서로 달라도 됩니다 — Windows PC에서 설치한 뒤 Linux의 Studio로 데이터를 보내는 것도 정상 동작합니다.

---

## 1. 사전 준비

### 헤드셋

Quest 2 / Quest 3 / Quest 3S / Quest Pro. 빌드가 Android API 32 기준이라 1세대 Quest에서는 실행되지 않습니다.

### Meta 계정 / 개발자 모드

> Meta 스토어 외부에서 설치한 앱을 Quest에서 실행하려면 **개발자 모드(Developer Mode)**를 켜야 합니다.

1. [Meta 개발자 사이트](https://developers.meta.com/horizon/)에서 Meta 개발자 계정 생성 (무료, 조직 등록 1회 필요).
2. 모바일에 **Meta Horizon 앱**을 설치하고 로그인.
3. 앱 → `메뉴` → `디바이스` → 사용 중인 Quest 선택 → `개발자 모드` → **ON**.
4. Quest 헤드셋을 한 번 재부팅.

### USB-C 케이블

데이터 전송이 가능한 USB-C 케이블이 필요합니다 (충전 전용은 안 됨). Quest 박스 동봉 케이블이나 Quest Link용 공식 케이블을 그대로 쓰시면 됩니다.

### APK 파일

**[⬇ MOVINQuestCompanion.apk 다운로드](https://github.com/MOVIN3D/MOVIN-MetaQuest-APK/releases/latest/download/MOVINQuestCompanion.apk)** — 이 링크에서 항상 최신 빌드를 받을 수 있습니다.

이전 버전은 [Releases](https://github.com/MOVIN3D/MOVIN-MetaQuest-APK/releases) 페이지에 있습니다. GitHub이 자동으로 붙이는 `Source code` 압축 파일에는 이 README 문서들만 들어 있으니, APK만 받으시면 됩니다.

---

## 2. ADB 설치

ADB(Android Debug Bridge)는 PC에서 Quest로 APK를 설치하기 위한 도구입니다.

### Windows

**옵션 A — winget (가장 간단)**

```powershell
winget install --id=Google.PlatformTools -e
```

**옵션 B — 직접 다운로드**

1. [Platform Tools 페이지](https://developer.android.com/tools/releases/platform-tools)에서 ZIP 다운로드.
2. 압축을 `C:\platform-tools`처럼 공백이나 비ASCII 문자가 없는 경로에 풀기.
3. 환경변수 PATH에 `C:\platform-tools` 추가 (`Win` → `시스템 환경 변수 편집` → `Path` → `편집` → `새로 만들기`).

설치 후 새 PowerShell에서 `adb version` 실행해 동작 확인.

### Linux

```bash
sudo apt install adb                     # Ubuntu / Debian
sudo dnf install android-tools           # Fedora
sudo pacman -S android-tools             # Arch
```

Linux에서는 udev 규칙이 있어야 사용자 계정으로 헤드셋에 접근할 수 있습니다. 규칙이 없으면 `adb devices`에서 Quest가 `no permissions`로 표시됩니다. Ubuntu/Debian에서는 다음 패키지를 설치하면 필요한 규칙이 함께 추가됩니다.

```bash
sudo apt install android-sdk-platform-tools-common
```

다른 배포판에서는 규칙을 직접 추가하고 사용자를 `plugdev` 그룹에 넣어주세요. 여기서 `2833`은 Meta의 USB 벤더 ID입니다.

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="2833", MODE="0666", GROUP="plugdev"' \
  | sudo tee /etc/udev/rules.d/51-android.rules
sudo udevadm control --reload-rules && sudo udevadm trigger
sudo usermod -aG plugdev $USER    # 적용을 위해 로그아웃 후 재로그인
adb kill-server
```

---

## 3. Quest 헤드셋을 PC에 연결

1. USB-C 케이블로 Quest와 PC 연결.
2. **Quest를 착용한 상태에서** 헤드셋에 표시되는 `USB 디버깅 허용?` 대화상자에서 `이 컴퓨터에서 항상 허용` 체크 후 `허용`.
3. PC에서 인식 확인:
   ```bash
   adb devices
   ```
   ```
   List of devices attached
   1WMHHxxxxxxxxx    device
   ```

`unauthorized`로 나오면 케이블을 다시 연결해 대화상자를 띄운 뒤 USB 디버깅을 다시 허용해주세요.

---

## 4. APK 설치

APK를 다운로드한 폴더로 이동한 뒤:

```bash
adb install MOVINQuestCompanion.apk
```

기존 버전 위에 새 빌드를 덮어쓸 때 (데이터 보존):

```bash
adb install -r MOVINQuestCompanion.apk
```

---

## 5. Quest에서 앱 실행

1. Quest 헤드셋 착용.
2. `앱 라이브러리` → 우측 상단 카테고리 드롭다운에서 `Unknown Sources` 선택.
3. `MOVINQuestCompanion` 실행.
4. 첫 실행 시 뜨는 `Hand Tracking` 권한 요청을 허용해주세요. 앱이 표시하는 권한 요청은 이것 하나뿐이고, 네트워크 권한은 설치 시점에 부여되어 따로 묻지 않습니다.

> 헤드셋 본체에서 `설정` → `움직임 추적` → `핸드 트래킹`이 켜져 있는지 확인해주세요. 앱에서 손 추적을 사용할 수 없으면 실행 약 3초 뒤 제목 아래에 빨간 `⚠ Hand tracking permission required — allow it in Quest Settings` 문구가 표시됩니다.

> 이 앱은 의도적으로 경계선(Guardian) 없이 동작합니다. 앱이 켜져 있는 동안에는 경계선이 그려지지 않고, 경계 밖으로 나가도 패스스루로 전환되지 않습니다. mocap 중 움직임 때문에 손 추적이 계속 끊기는 것을 막기 위한 설정입니다.

---

## 6. 앱 사용법

### 포인팅과 클릭

컨트롤러와 맨손 모두 사용할 수 있고, 양손이 독립적으로 동작하므로 한 손에 컨트롤러를 들고 다른 손으로 포인팅할 수도 있습니다.

- **컨트롤러** — 손에 들면 **청록색** 광선이 나타납니다. 40° 아래로 기울어져 있어 자연스럽게 정면의 패널을 가리킵니다. **트리거**를 당기면 클릭되고, 당기는 동안 광선이 **초록색**으로 바뀝니다.
- **맨손** — 그 손에 컨트롤러를 들고 있지 않으면 헤드셋이 계산한 조준 방향으로 손에서 **흰색** 광선이 뻗어 나갑니다. 엄지와 검지를 **핀치(pinch)**하면 클릭되고, 핀치하는 동안 광선이 **초록색**으로 바뀝니다.

핀치는 광선이 실제로 버튼 위에 올라가 있을 때만 클릭으로 인식되므로, mocap 중 손이 움직이는 것만으로는 UI가 작동하지 않습니다. 컨트롤러를 들고 있지도 않고 추적도 되지 않는 손에는 광선이 표시되지 않습니다.

연결된 뒤에는 컨트롤러를 내려놓으셔도 됩니다 — 상태 화면에서는 더 이상 입력할 것이 없습니다.

### 첫 화면 — 연결할 PC 선택

두 가지 선택 방법이 한 화면에 함께 표시됩니다.

**위쪽 — 자동 검색.** 앱이 1초에 한 번 로컬 네트워크로 브로드캐스트를 보내고, 응답한 MOVIN Studio PC를 목록으로 보여줍니다.

- 응답이 없는 동안에는 `Searching for MOVIN Studio...`, 응답이 오면 `Found 1 host:` / `Found N hosts:`로 헤더가 바뀝니다.
- 최근에 응답한 순서로 최대 5개까지 `호스트명   ip:포트` 형태로 표시됩니다. 5초 동안 응답이 없는 호스트는 목록에서 사라집니다.
- 항목을 클릭하면 그 호스트가 알려준 포트로 즉시 연결됩니다.

**아래쪽 — 직접 입력.** `— Or type IP manually —` 구분선 아래에 있습니다.

- 숫자패드(`1`~`9`, `0`, `.`, `⌫`, 최대 15자)로 PC IP를 입력하고 `Connect`를 누릅니다.
- 비어 있으면 `Enter the host PC's IP address`, 형식이 잘못되면 `Invalid IP address`가 뜨고 IP 글자가 빨갛게 바뀝니다.
- 직접 입력으로 연결할 때는 항상 14044 포트를 사용합니다.

> 연결할 때마다 IP가 저장되어 다음 실행 시 그대로 채워집니다. 한 번도 연결한 적 없는 헤드셋에서는 `192.168.1.100`이 기본값으로 표시됩니다.

### 연결 상태 화면

맨 윗줄에 선택한 대상이 `ip:포트`로 표시됩니다.

| 상태 | 의미 |
|------|------|
| `○ Waiting...` (회색) | 대기 |
| `⟳ Connecting...` (파란색, 밝기가 천천히 오르내림) | 패킷을 보내는 중, 아직 응답 없음 (최대 5초) |
| `● Connected` (녹색) | Studio가 응답 중 — 마지막 응답이 1.5초 이내 |
| `⚠ No response from Studio` (노란색, 밝기가 천천히 오르내림) | 응답이 1.5초 넘게 끊겼거나, Connect 후 5초 안에 한 번도 오지 않음 |

상태 문구 아래 줄은 상태에 따라 내용이 달라집니다.

- **연결된 상태**에서는 손 추적 상태가 `● L   ● R`로 표시됩니다 (● = 헤드셋 카메라가 그 손을 보고 있음, ○ = 안 보임). 손이 안 보여도 앱은 keepalive 패킷을 계속 보내기 때문에, `● Connected`인데 `○ L   ○ R`인 상태는 정상입니다.
- **연결되지 않은 상태**에서는 대신 `disc 1 · sent 72/s · ack 0`처럼 진단 카운터가 표시되고, 그 아래 한 줄의 안내 문구가 붙습니다. 카운터와 문구의 의미는 9절에 정리했습니다.

화면 하단의 빨간 `Disconnect` 버튼을 누르면 송신을 멈추고 첫 화면으로 돌아가 다른 PC를 고를 수 있습니다.

### 패널 위치

패널은 눈높이보다 약간 아래, 약 1 m 앞에 떠 있습니다. 머리에 완전히 고정된 것은 **아니고**, 고개를 약 20° 이내로 돌리는 동안에는 제자리에 머물러 있다가 그보다 크게 돌리면 천천히 정면으로 따라와 다시 마주 보는 순간 멈춥니다.

---

## 7. 네트워크 설정

- Quest와 Studio PC를 **같은 Wi-Fi**에 연결해주세요.
- PC에서 처음 MOVIN Studio를 실행하면 Windows 보안에서 네트워크 사용 안내 창이 뜰 수 있습니다. `액세스 허용`을 눌러주세요.
- 회사나 공용 Wi-Fi 중에는 같은 네트워크 안의 기기끼리 서로 통신을 막아 두는 곳이 있습니다. 이 경우 별도 라우터에 Studio PC와 Quest만 연결해서 사용해주세요.

앱이 사용하는 포트입니다. 필요하면 방화벽에서 열어주세요.

| 포트 | 방향 | 용도 |
|---|---|---|
| UDP 14044 | Quest → PC | 손 데이터. Studio의 응답도 같은 소켓으로 되돌아옵니다. |
| UDP 14045 | Quest → 브로드캐스트 | 자동 검색 요청 |
| UDP 14046 | PC → Quest | 자동 검색 응답 |

14045/14046이 막히면 자동 호스트 목록만 안 뜨고, IP를 직접 입력하면 연결됩니다. 14044가 막히면 연결 자체가 안 됩니다.

---

## 8. 동작 확인

1. PC에서 MOVIN Studio 실행 → mocap 씬 진입.
2. `Hand Tracking` 토글 ON, `Hand Type` 드롭다운에서 `Quest` 선택.
3. Quest 헤드셋 착용 후 컴패니언 앱 실행.
4. 검색된 호스트 목록에서 PC를 고르거나 IP를 직접 입력해 연결.
5. 상태가 `● Connected` + `● L   ● R`로 표시되는지, PC 화면에서 캐릭터 손이 따라오는지 확인.

자세한 사용 흐름은 GitBook의 **MOVIN Studio 사용 가이드 → Hands → Quest**를 참고해주세요.

---

## 9. 트러블슈팅

### `adb devices`에 기기가 표시되지 않음

| 증상 | 해결 |
|---|---|
| 목록이 비어 있음 | 케이블이 데이터 전송용인지 확인 / 다른 USB 포트 시도 |
| `unauthorized` | 케이블을 다시 연결해 대화상자를 띄운 뒤 Quest 안에서 USB 디버깅을 다시 허용 |
| (Windows) 노란색 느낌표 | [Oculus ADB Driver](https://developers.meta.com/horizon/downloads/package/oculus-adb-drivers/) 설치 |
| (Linux) `no permissions` | udev 규칙 누락 — 2절의 Linux 항목 참고 후 케이블 재연결 |

### 설치 실패 (`INSTALL_FAILED_*`)

| 코드 | 해결 |
|---|---|
| `ALREADY_EXISTS` | `-r` 옵션을 붙여 재설치 |
| `UPDATE_INCOMPATIBLE` | `adb uninstall com.movin.questcompanion` 후 다시 설치 |
| `INSUFFICIENT_STORAGE` | Quest 저장공간 정리 (설정 → 저장공간) |

### 앱이 Studio에 연결되지 않을 때

연결이 정상이 아닐 때 상태 화면에는 진단 카운터 한 줄과 안내 문구 한 줄이 표시됩니다. 카운터의 의미는 다음과 같습니다.

| 카운터 | 의미 |
|---|---|
| `disc` | 현재 검색된 Studio PC 수 |
| `sent` | 초당 송신 패킷 수 |
| `ack` | Connect 이후 Studio가 보내온 응답 수 |

그 아래 표시된 안내 문구를 다음 표에서 찾아 조치해주세요.

| 헤드셋에 표시된 문구 | 의미 | 조치 |
|---|---|---|
| `Not sending — socket/target issue.` | 패킷이 송신되지 않는 상태 | `Disconnect` 후 다시 연결, 입력한 IP 재확인 |
| `No reply → same Wi-Fi? firewall? Studio in the mocap scene?` | 검색도 되지 않고 응답도 없음 | 두 기기를 같은 SSID에 연결, 기기 간 통신 차단 여부 확인, Studio가 mocap 씬에 들어가 있는지 확인 |
| `PC found but silent → on Studio enable hand-tracking + Quest, or firewall on UDP 14044.` | PC는 찾았지만 데이터 응답이 없음 | Studio에서 `Hand Tracking` ON + `Hand Type`을 `Quest`로 설정, PC 방화벽에서 UDP 14044 허용 |
| `Reply intermittent — Wi-Fi/latency.` | 응답이 간헐적으로 끊김 | Wi-Fi 신호 문제 — 공유기에 가까이 가거나 5 GHz 대역 사용 |

Windows 보안 안내에서 `차단`을 누른 적이 있으면, `제어판 → 시스템 및 보안 → Windows Defender 방화벽 → 앱이 방화벽을 통해 통신하도록 허용`에서 MOVIN Studio를 다시 허용해주세요.

### 그 밖의 앱 문제

| 증상 | 해결 |
|---|---|
| 호스트 목록에 아무것도 표시되지 않음 | 자동 검색(UDP 14045/14046)이 막혔거나 두 기기가 다른 서브넷에 있음 — IP를 직접 입력해서 연결 |
| `⚠ Hand tracking permission required` 문구가 계속 표시됨 | 헤드셋에서 `설정 → 움직임 추적 → 핸드 트래킹`을 켠 뒤 앱 재실행 |
| 연결되었으나 `○ L   ○ R` | 손이 헤드셋 카메라 화각 밖이거나 헤드셋 손 추적이 꺼진 상태 — 네트워크 연결 자체는 정상 |
| 광선이 표시되지 않음 | 컨트롤러: 컨트롤러를 손에 들어야 헤드셋이 사용 중으로 인식합니다. 맨손: 손이 추적되고 있어야 하고, 컨트롤러를 들고 있지 않은 손에만 광선이 나옵니다 |

---

## 10. 제거

```bash
adb uninstall com.movin.questcompanion
```
