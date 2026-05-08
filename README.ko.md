# MOVIN Quest Companion APK

[English](README.md) | **한국어**

Meta Quest 헤드셋의 손 추적 데이터를 같은 Wi-Fi에 있는 PC의 MOVIN Studio로 보내주는 Quest 측 컴패니언 앱입니다. 정식 스토어 빌드가 아니라서 한 번은 PC에서 USB로 직접 설치해야 합니다.

---

## 1. 사전 준비

### Meta 계정 / 개발자 모드

> Meta 스토어 외부에서 설치한 앱을 Quest에서 실행하려면 **개발자 모드(Developer Mode)** 를 켜야 합니다.

1. https://developer.oculus.com 에서 Meta 개발자 계정 생성 (무료, 조직 등록 1회 필요).
2. 모바일에 **Meta Horizon 앱**을 설치하고 로그인.
3. 앱 → `메뉴` → `디바이스` → 사용 중인 Quest 선택 → `개발자 모드` → **ON**.
4. Quest 헤드셋을 한 번 재부팅.

### USB-C 케이블

데이터 전송이 가능한 USB-C 케이블이 필요합니다 (충전 전용은 안 됨). Quest 박스 동봉 케이블이나 Quest Link용 공식 케이블을 그대로 쓰시면 됩니다.

### APK 파일

이 README와 같은 위치에 있는 `MOVINQuestCompanion.apk` 를 사용해주세요. GitHub 웹에서 파일을 직접 다운로드하거나, 저장소를 clone 한 뒤 해당 경로의 파일을 그대로 쓰시면 됩니다.

---

## 2. ADB 설치

ADB(Android Debug Bridge)는 PC에서 Quest로 APK를 설치하기 위한 도구입니다.

### Windows

**옵션 A — winget (가장 간단)**

```powershell
winget install --id=Google.PlatformTools -e
```

**옵션 B — 직접 다운로드**

1. https://developer.android.com/tools/releases/platform-tools 에서 zip 다운로드.
2. 압축을 `C:\platform-tools` 등 한글/공백 없는 경로에 풀기.
3. 환경변수 PATH에 `C:\platform-tools` 추가 (`Win` → `시스템 환경 변수 편집` → `Path` → `편집` → `새로 만들기`).

설치 후 새 PowerShell에서 `adb version` 실행해 동작 확인.

### Linux

```bash
sudo apt install android-tools-adb       # Ubuntu / Debian
sudo dnf install android-tools           # Fedora
sudo pacman -S android-tools             # Arch
```

---

## 3. Quest 헤드셋을 PC에 연결

1. USB-C 케이블로 Quest와 PC 연결.
2. **Quest를 착용한 상태에서** 화면 안의 "USB 디버깅 허용?" 다이얼로그에서 **`이 컴퓨터에서 항상 허용`** 체크 후 **`허용`**.
3. PC에서 인식 확인:
   ```bash
   adb devices
   ```
   ```
   List of devices attached
   1WMHHxxxxxxxxx    device
   ```

`unauthorized` 로 나오면 헤드셋 안에서 허용 다이얼로그를 다시 받아주세요 (케이블 재연결).

---

## 4. APK 설치

APK 파일이 있는 폴더로 이동한 뒤:

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
2. `앱 라이브러리` → 우측 상단 카테고리 드롭다운에서 **`Unknown Sources`** 선택.
3. **`MOVIN Quest Companion`** 실행.
4. 첫 실행 시 표시되는 **Hand Tracking** / **Local Network** 권한 요청 모두 허용.

> 헤드셋 본체에서 `설정` → `움직임 추적` → `핸드 트래킹` 이 켜져 있는지 확인해주세요. 권한이 활성화되지 않은 경우 앱 화면 상단에 `⚠ Hand tracking permission required` 안내가 표시됩니다.

---

## 6. 앱 사용법

UI는 **Quest 컨트롤러로만** 조작됩니다. mocap 중 손이 움직이면서 UI가 잘못 눌리는 것을 막기 위해 손 추적 입력은 사용하지 않아요. 연결 설정이 끝나면 컨트롤러를 내려놓으면 됩니다 — 그때부터 mocap용 손 추적이 활성화됩니다.

### 컨트롤러 조작

- 컨트롤러를 손에 들면 정면 방향(40° 아래로 기울어진)으로 광선이 표시됩니다 — 평소 **cyan**, 트리거를 당기는 동안 **green**.
- 광선을 버튼/숫자 위에 맞추고 **트리거**를 당기면 클릭됩니다.
- 컨트롤러를 내려놓으면 광선은 사라집니다.

### 첫 화면 — 연결할 PC 선택

앱이 켜지면 같은 Wi-Fi 안에서 MOVIN Studio가 실행 중인 PC를 자동으로 찾습니다.

- **자동으로 찾았을 때** — 화면 상단에 `Found N hosts:` 헤더와 함께 PC 이름 + `IP:포트` 버튼이 최대 5개까지 표시됩니다. 원하는 항목을 광선으로 가리키고 트리거를 당기면 곧바로 연결이 진행됩니다.
- **자동 발견이 안 될 때** — 가운데 숫자패드(1~9 / 0 / `.` / `⌫`)로 PC IP(예: `192.168.1.100`)를 입력하고 아래 **Connect** 버튼을 눌러주세요.

> 한 번 입력한 IP는 자동으로 저장되어 다음 실행 시 그대로 표시됩니다.

### 연결 상태 화면

Connect 후에는 상태 화면으로 전환됩니다.

| 상태 | 의미 |
|------|------|
| `○ Waiting...` (회색) | 대기 |
| `⟳ Connecting...` (파란색 깜빡임) | 패킷을 보내며 응답 대기 중 (최대 5초) |
| `● Connected` (녹색) | PC 응답 수신, 손 데이터 송신 중 |
| `⚠ No response from Studio` (노란색 깜빡임) | 응답이 1.5초 이상 끊긴 상태 |

> Connect 후 5초 안에 PC 응답이 한 번도 도착하지 않으면 같은 `⚠ No response` 상태로 전환되며, `No response. Check the IP and that the PC is on the same Wi-Fi.` 안내가 함께 표시됩니다.

상태 문구 아래에는 좌/우 손이 헤드셋 카메라에 추적되고 있는지가 **● L   ● R** 형태로 표시됩니다 (●=추적 중, ○=미추적).

다른 PC에 연결하려면 화면 하단의 빨간 **Disconnect** 버튼으로 첫 화면으로 돌아갈 수 있습니다.

### 패널 위치

패널은 머리에 고정되어 어디로 돌아보거나 자리를 옮겨도 항상 정면에 보입니다. mocap 중에는 헤드셋을 목에 거는 사용 방식이라 패널이 늘 따라와도 시야를 가릴 일은 없습니다.

---

## 7. 네트워크 설정

- Quest와 Studio PC를 **같은 Wi-Fi**에 연결해주세요.
- PC에서 처음 MOVIN Studio를 실행하면 Windows 보안에서 네트워크 사용 안내 창이 뜰 수 있습니다. **'액세스 허용'**을 눌러주세요.
- 회사나 공용 Wi-Fi 중에는 같은 네트워크 안의 기기끼리 서로 통신을 막아 두는 곳이 있습니다. 이 경우 별도 라우터에 Studio PC와 Quest만 연결해서 사용해주세요.

---

## 8. 동작 확인

1. PC에서 MOVIN Studio 실행 → mocap 씬 진입.
2. `Hand Tracking` 토글 ON, `Hand Type` 드롭다운에서 **Quest** 선택.
3. Quest 헤드셋 착용 후 컴패니언 앱 실행.
4. PC 화면에서 캐릭터 손이 따라오는지 확인.

자세한 사용 흐름은 GitBook 가이드 — **MOVIN Studio 사용 가이드 → Hands → Quest** — 를 참고해주세요.

---

## 9. 트러블슈팅

### `adb devices` 에 기기가 안 뜸

| 증상 | 해결 |
|---|---|
| 비어있음 | 케이블이 데이터 전송용인지 확인 / 다른 USB 포트 시도 |
| `unauthorized` | Quest 안에서 허용 다이얼로그를 다시 받기 (케이블 재연결) |
| (Windows) 노란 ! 마크 | [Oculus ADB Driver](https://developer.oculus.com/downloads/package/oculus-adb-drivers/) 설치 |

### 설치 실패 (`INSTALL_FAILED_*`)

| 코드 | 해결 |
|---|---|
| `ALREADY_EXISTS` | `-r` 옵션을 붙여 재설치 |
| `UPDATE_INCOMPATIBLE` | `adb uninstall com.movin.questcompanion` 후 다시 설치 |
| `INSUFFICIENT_STORAGE` | Quest 저장공간 정리 (설정 → 저장공간) |

### Studio가 Quest를 인식하지 못할 때

1. 두 기기가 정말 같은 Wi-Fi 이름(SSID)에 연결되어 있는지 확인.
2. Windows 보안 안내에서 '차단'을 누른 적이 있으면, `제어판 → 시스템 및 보안 → Windows Defender 방화벽 → 앱이 방화벽을 통해 통신하도록 허용`에서 MOVIN Studio를 다시 허용.
3. 회사/공용 Wi-Fi인 경우 별도 라우터에 두 기기만 연결해 시도.
4. Quest의 Companion 앱을 한 번 종료 후 재실행.

---

## 10. 제거

```bash
adb uninstall com.movin.questcompanion
```
