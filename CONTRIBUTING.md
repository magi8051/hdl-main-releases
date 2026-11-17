# DWA_HDL 프로그램 등록 및 배포 안내

이 문서는 DWA_HDL에 연결된 프로그램을 업데이트하거나 새 프로그램을 추가하는 담당자를 위한 안내입니다. 배포 저장소는 [HDL-main-releases](https://github.com/magi8051/HDL-main-releases)입니다. 프로그램 소스와 DWA_HDL 소스는 각각 별도로 관리합니다. 이 저장소에 소스 프로젝트가 모두 들어 있는 것은 아닙니다.

## 먼저 구분하기

| 작업 | 해야 할 일 | DWA_HDL 새 버전 필요 |
|---|---|---|
| 이미 등록된 앱의 새 버전 | 해당 앱의 새 GitHub Release에 호환 패키지와 `release.json` 게시 | 보통 불필요. 단, 패키지 형식·설치 계약이 바뀌면 수정과 검증 필요 |
| 처음 등록하는 앱 | DWA_HDL 소스의 제품 정보, 표시 자료, 설치·검증 계약을 추가하고 런처를 배포 | 필요 |
| DWA_HDL 자체 변경 | DWA_HDL을 빌드·검증한 후 런처 Release 게시 | 필요 |

**설치 파일을 저장소에 올리기만 해서는 새 프로그램이 목록에 생기지 않습니다.** 기존 앱도 `release.json`과 파일이 일치하고 런처의 설치 방식과 호환되어야 합니다. 기존 Release를 지우면 해당 온라인 버전의 재설치·복구가 불가능할 수 있으므로 보존합니다.

## 사전 준비

1. 배포 저장소의 쓰기 권한을 확보합니다. GitHub 계정 또는 토큰을 고객용 파일에 넣지 않습니다.
2. 담당 제품의 소스와 실제 빌드 결과, 설치 방식, 버전, 사용자 데이터 위치를 확인합니다. DWA_HDL 소스가 필요하면 런처 관리자에게 별도로 받습니다.
3. 새 설치, 기존 설치에서 업데이트, 실행, 실패 시 복구, 사용자 데이터 보존을 제품별로 시험합니다. 하드웨어나 실제 고객 PC를 시험하지 못했다면 따로 기록합니다.
4. 공개 저장소이므로 소스, 비밀키, 고객별 인증 정보, 개인 정보와 내부 전용 자료를 배포 자산에 포함하지 않습니다. 설치 파일의 배포 허용 범위를 먼저 확인합니다.

## 1. 기존 프로그램 업데이트

### 1-1. 제품별 태그

| 제품 | 제품 ID | Release 태그 예시 |
|---|---|---|
| Haptic Studio | `haptic-studio` | `haptic-studio-v67` |
| HapTune Studio | `haptune-studio` | `haptune-studio-v0.42.0` |
| HDL Wiki | `hdl-wiki` | `hdl-wiki-v0.0.10` |
| DWA A2V Studio | `dwa-a2v-studio` | `dwa-a2v-studio-v1.0.22` |
| DWA_HDL 런처 | 별도 자체 업데이트 | `dwa-hdl-v0.5.5` |

예시의 버전은 태그 형식을 보여주기 위한 것입니다. 새 릴리스에는 해당 제품의 **새 버전과 새 태그**를 사용합니다. 다른 제품과 태그 접두어를 섞지 않습니다. 일반적인 앱 버전은 `major.minor.patch` 세 숫자를 사용하며, 기존 Haptic Studio의 `v67` 같은 정수 버전도 지원합니다. 게시된 파일을 같은 태그와 이름으로 교체하지 않습니다.

### 1-2. 앱 배포 파일과 메타데이터

각 앱 Release의 **Assets**에 호환 설치 파일과 정확히 `release.json`이라는 메타데이터 파일을 함께 올립니다. 단순 URL을 Release 설명에 적거나 Git 저장소 폴더에 EXE만 넣는 방식은 현재 공용 Release 계약이 아닙니다.

`release.json`의 주요 값은 다음과 같습니다.

| 항목 | 작성 내용 |
|---|---|
| `productId` | 위 표의 제품 ID와 정확히 일치 |
| `version` | Release 태그의 버전과 일치. 예: 태그 `haptune-studio-v0.42.0`, 버전 `0.42.0` |
| `revision` | 정수 버전은 그 정수. `major.minor.patch`는 `major × 1000000000000 + minor × 1000000 + patch` |
| `installer.source` | 같은 Release의 설치 파일 Asset 다운로드 URL |
| `installer.size`, `installer.sha256` | 최종 설치 파일의 바이트 크기와 SHA-256 |
| `installer.kind`, `requiresElevation` | 확인된 설치 방식과 관리자 권한 필요 여부 |
| `executableRelativePath`, `receiptRelativePath` | 해당 버전의 관리 설치 실행 파일과 receipt 경로 |
| `sha512` | Electron 원본에 제공된 SHA-512가 있으면 함께 확인 |
| `update`, `updateFromRevisions` | 검증된 차등 업데이트가 있을 때만 사용. 없으면 전체 설치 패키지 사용 |

실제로 게시된 예시는 [HapTune Release의 release.json](https://github.com/magi8051/HDL-main-releases/releases/download/haptune-studio-v0.42.0/release.json)과 [Haptic Studio Release의 release.json](https://github.com/magi8051/HDL-main-releases/releases/download/haptic-studio-v67/release.json)을 참고합니다. 예시의 버전·URL·크기·해시는 새 파일에 맞게 **모두 다시 계산**해야 합니다. 버전이 `0.42.0`일 때 revision은 `42000000`입니다.

Windows PowerShell에서 최종 설치 파일의 크기와 SHA-256 확인:

```powershell
$package = Get-Item -LiteralPath 'C:\path\to\installer.exe'
$package.Length
(Get-FileHash -LiteralPath $package.FullName -Algorithm SHA256).Hash
```

코드 서명하는 경우 서명 **후의 최종 파일**로 크기와 해시를 계산합니다. 해시는 파일 일치 확인이며 제작자의 서명을 대신하지 않습니다.

### 1-3. 설치 방식 확인

- **Haptic Studio 및 A2V 등 Native 앱:** HDL 관리 설치 규약에 맞는 패키지를 사용합니다. 지정된 버전 폴더, 실행 파일, 전체 파일 해시가 들어 있는 `hub-receipt.json`, 무인 설치와 복구 동작을 확인해야 합니다. 일반 설치 EXE라고 해서 자동으로 호환되지는 않습니다.
- **HapTune / HDL Wiki 등 Electron 앱:** 현재는 확인된 `electron-nsis` 패키지를 검증하고 내용을 관리 버전 폴더에 배치합니다. 설치 EXE의 버전, Electron 패키지 이름과 버전, 내부 구조, 자체 업데이트 정책이 기존 계약과 일치해야 합니다. 파일 형식이나 자체 자동 업데이트가 바뀌면 런처 호환성부터 다시 확인합니다.
- **별도 설치한 A2V Studio:** HDL 관리 설치본과 독립 설치본의 업데이트 계약은 다릅니다. 독립 설치 업데이트는 지원 기반 버전과 별도 메타데이터·공식 업데이트 파일을 확인해야 합니다. 성공한 독립 설치 업데이트의 이전 버전 자동 복원은 지원하지 않습니다.
- Haptic Studio를 포함한 새 버전이 아직 HDL 관리 설치 규약으로 준비되지 않았다면, 기존의 다른 프로그램 패키지나 예전 빌드 스크립트를 그대로 사용해 배포하지 마세요. 제품별 패키지와 검증을 먼저 준비해야 합니다.

### 1-4. 게시 및 확인

1. GitHub Releases에서 새 제품 태그로 **Draft**를 만들고 설치 파일과 `release.json`을 Assets에 올립니다. 시험판이면 **pre-release**로 표시합니다.
2. 업로드된 파일명·크기·SHA-256과 메타데이터의 제품 ID·태그·버전·URL·경로를 대조합니다.
3. 정식 또는 시험판 상태를 확인하고 게시합니다. 시험판은 일반적인 최신 정식 자동 업데이트로 취급하지 않습니다.
4. 격리된 시험 환경에서 HDL의 업데이트 조회, 설치, 실행, 기존 버전 선택·복구, 오프라인 실행을 제품에 맞게 확인합니다. 사용자가 직접 선택한 이전 버전은 해당 Release가 유지되어야 온라인에서 재설치할 수 있습니다.
5. 검증 범위와 확인하지 못한 항목, 담당자, 빌드 출처를 배포 이력에 남깁니다. GUI와 보드 FW 배포·검증은 각각 별개입니다.

기존 앱은 제품별 태그에서 최신 정식 버전을 찾습니다. 새 Release를 게시해도 기존에 설치된 다른 앱을 지우거나 일괄 설치하지 않습니다. 인터넷 연결이 없으면 새 업데이트 조회·다운로드는 할 수 없지만, 각 프로그램 자체가 오프라인 실행을 지원하는 범위에서 이미 설치된 프로그램은 실행할 수 있습니다.

## 2. 새 프로그램 등록

1. 제품 담당자가 고유 제품 ID·이름·태그 접두어, 버전 규칙, 실제 실행 파일, 설치·업데이트·복구 방식, 사용자 데이터 위치를 정합니다.
2. DWA_HDL 소스의 `catalog/external-products.json`에 제품 정보와 검증된 기준 Release를 추가합니다. 기존 Haptic Studio의 기준 패키지는 현재 `tools/Build.ps1`에서도 구성하므로 다른 앱과 구성 위치가 다릅니다. 빌드가 생성하는 `catalog/products.json`만 직접 고쳐 배포하지 않습니다.
3. `catalog/presentation.json`에 같은 제품 ID로 아이콘과 지원 보드를 등록합니다. 실제 프로그램 GUI 캡처·아이콘과 검증된 보드 정보를 준비하고 배포 빌드에서 이미지가 포함되도록 확인합니다. 확인하지 않은 보드 지원이나 연결 정보를 만들지 않습니다.
4. C#/Native와 Electron 중 실제 패키지에 맞게 설치·검증 코드를 검토합니다. 현재 지원 형식과 다르면 런처 코드를 추가로 구현해야 합니다. 기존에 독립 설치된 앱을 찾거나 직접 업데이트하는 기능도 제품별 식별과 별도 검증이 필요합니다.
5. 제품 Release를 위 1절에 따라 준비하고, 새 제품이 포함된 DWA_HDL을 빌드·검증·배포합니다. 각 프로그램의 소스나 설치 파일을 고객 PC에 한꺼번에 설치하는 구조가 아닙니다.

## 3. DWA_HDL 자체 업데이트

DWA_HDL 소스와 빌드 도구, 필요한 Studio 패키지 또는 소스가 준비된 환경에서 프로젝트의 `README.md`와 `tools/build-inputs.json`을 확인합니다. 이 파일에는 원본 PC 기준 입력 경로가 있을 수 있어 다른 PC에서 그대로 유효하다고 가정하면 안 됩니다.

런처 버전과 설치파일 버전을 일치시키고, 빌드 → 기능 및 설치본 화면 밖 검증 → 검증된 로컬 결과 반영 → 공용 Release 게시 순서로 진행합니다. 프로젝트에 포함된 `tools/Build.ps1`, `tools/Verify.ps1`, `tools/Publish-Local.ps1`, `tools/Publish-SharedHub.ps1`을 사용하며 실행 전 각 스크립트의 입력과 권한을 확인합니다. `Build.ps1 -ReuseStudioPackages`는 이미 검증된 로컬 Studio 패키지가 있을 때만 사용할 수 있습니다. 배포 스크립트는 일치하는 검증 결과가 필요합니다.

런처 Release의 태그는 `dwa-hdl-v<버전>`이고 Assets는 `DWA_HDL_Setup.exe`와 `hub-update.json`입니다. 런처의 업데이트 조회·다운로드·적용 및 실패 복구를 검증하고, 게시 후 공개 조회 버전과 다운로드 파일 해시도 확인합니다. 다른 앱의 `release.json`과 런처의 `hub-update.json`은 서로 다른 계약입니다.

## 문의할 때 전달할 내용

제품명과 담당자, 작업 목적(새 앱/기존 앱/런처), 소스 위치, 버전과 태그, 설치 파일과 해시, 설치·업데이트 방식, 보드 및 사용자 데이터 위치, 완료한 검증과 미확인 사항을 런처 관리자에게 전달합니다. 비밀번호나 토큰은 문서·Release·메시지에 적지 않습니다.
