# HDL-main-releases

Haptic Design Lab(HDL) 툴들의 **배포 전용 리포지토리**입니다.
HDL 위젯과, 위젯이 설치·업데이트하는 모든 애플리케이션의 배포 파일을 이 리포 하나에서 관리합니다.

> **문서 상태: 초안 (v0.1.0)** — [9. 확정이 필요한 항목](#9-확정이-필요한-항목)을 결정하면 v1.0.0으로 올립니다.

## 목차

1. [개요](#1-개요)
2. [관리 대상 앱](#2-관리-대상-앱)
3. [리포지토리 구조](#3-리포지토리-구조)
4. [규칙](#4-규칙)
5. [매니페스트](#5-매니페스트)
6. [위젯의 업데이트 동작](#6-위젯의-업데이트-동작)
7. [배포 절차](#7-배포-절차)
8. [운영 가이드](#8-운영-가이드)
9. [확정이 필요한 항목](#9-확정이-필요한-항목)
10. [문서 이력](#10-문서-이력)

---

## 1. 개요

### 1.1 이 리포의 역할

- **배포 파일 보관**: 각 앱의 설치 파일을 GitHub Releases에 올립니다.
- **최신 버전 안내**: 앱별 매니페스트 파일(`apps/*.json`)로 "지금 배포 중인 버전"을 알려줍니다.
- **소스 코드는 두지 않음**: 소스는 각 앱의 개발 리포에서 관리합니다. 이 리포는 공개 리포이므로 소스, 내부 문서, 인증 키를 올리지 않습니다.

### 1.2 동작 원리

```
[각 앱 개발 리포] --빌드--> [이 리포의 Release] <--다운로드-- [HDL 위젯]
                                                                  ^
                            [apps/<앱 ID>.json] ---최신 버전 확인---+
```

위젯은 GitHub의 "Latest release"를 쓰지 않고 **매니페스트만 읽습니다.**
한 리포에 여러 앱의 릴리스가 섞여 있어도 앱별 최신 버전을 정확히 알 수 있는 이유입니다.

### 1.3 리포 하나로 통합한 이유

| 항목 | 앱마다 리포 분리 | 이 리포로 통합 |
| --- | --- | --- |
| 앱 추가 | 리포 생성 + 권한·설정 반복 | 매니페스트 파일 1개 추가 |
| 배포 현황 확인 | 리포를 돌아다니며 확인 | `apps/` 폴더에서 한눈에 확인 |
| 배포 규칙 | 리포마다 달라지기 쉬움 | 이 문서 하나로 통일 |
| 롤백 | 릴리스 삭제 또는 재배포 | 매니페스트의 버전만 되돌림 |

---

## 2. 관리 대상 앱

| 앱 ID | 앱 | 태그 접두사 | 매니페스트 | 통합 전 리포 이름 |
| --- | --- | --- | --- | --- |
| `widget` | HDL 위젯 (런처) | `widget-v` | `apps/widget.json` | HDL-main-releases |
| `haptic-studio` | Haptic Studio | `haptic-studio-v` | `apps/haptic-studio.json` | HDL-haptic-studio-releases |
| `a2v` | A2V | `a2v-v` | `apps/a2v.json` | HDL-a2v-releases |
| `sub1` | (이름 미정) | `sub1-v` | `apps/sub1.json` | HDL-sub1-releases |
| `sub2` | (이름 미정) | `sub2-v` | `apps/sub2.json` | HDL-sub2-releases |

**앱 ID**는 태그, 파일 이름, 매니페스트에 공통으로 쓰는 고유 이름입니다.
영문 소문자, 숫자, 하이픈만 사용하고 한 번 정하면 바꾸지 않습니다.

---

## 3. 리포지토리 구조

```
HDL-main-releases/
├── README.md              # 이 문서
├── catalog.json           # 위젯이 관리하는 앱 목록
└── apps/
    ├── widget.json        # 앱별 매니페스트 (최신 버전 정보)
    ├── haptic-studio.json
    ├── a2v.json
    ├── sub1.json
    └── sub2.json
```

설치 파일 자체는 git에 커밋하지 않고 **GitHub Releases의 첨부 파일**로 올립니다.

---

## 4. 규칙

### 4.1 버전

`MAJOR.MINOR.PATCH` 형식을 씁니다 (예: `1.2.0`).

| 자리 | 올리는 경우 |
| --- | --- |
| MAJOR | 이전 버전과 호환되지 않는 변경 |
| MINOR | 기능 추가 |
| PATCH | 버그 수정 |

### 4.2 태그

```
<앱 ID>-v<버전>
```

| 예시 | 의미 |
| --- | --- |
| `widget-v1.0.0` | HDL 위젯 1.0.0 |
| `haptic-studio-v1.2.0` | Haptic Studio 1.2.0 |
| `a2v-v0.3.1` | A2V 0.3.1 |

- `v1.2.0`처럼 접두사 없는 태그는 쓰지 않습니다.
- 한 번 배포한 태그는 다시 쓰지 않습니다.

### 4.3 릴리스 제목

```
<앱 이름> <버전>        예: Haptic Studio 1.2.0
```

### 4.4 배포 파일 이름

```
<앱 ID>-<버전>-<플랫폼>.<확장자>     예: haptic-studio-1.2.0-win-x64.exe
```

| 플랫폼 값 | 대상 |
| --- | --- |
| `win-x64` | Windows 64비트 |

플랫폼이 추가되면 이 표에 값을 먼저 등록합니다.

### 4.5 GitHub의 "Latest" 표시

리포 페이지의 **Latest** 표시는 리포 전체에 하나뿐입니다.

- **위젯 릴리스만** Latest로 지정합니다. 처음 방문한 사용자가 위젯 설치 파일을 바로 받을 수 있습니다.
- **나머지 앱 릴리스**는 Latest로 지정하지 않습니다 (`--latest=false`).

---

## 5. 매니페스트

### 5.1 앱 목록 — `catalog.json`

위젯이 어떤 앱을 관리하는지 알려주는 파일입니다. 앱을 추가하거나 제거할 때만 수정합니다.

```json
{
  "schema": 1,
  "apps": ["widget", "haptic-studio", "a2v", "sub1", "sub2"]
}
```

### 5.2 앱별 매니페스트 — `apps/<앱 ID>.json`

"지금 배포 중인 버전"을 담습니다. 앱마다 파일을 나눈 이유는 두 앱을 동시에 배포해도 같은 파일을 수정하지 않게 하기 위해서입니다.

```json
{
  "schema": 1,
  "id": "haptic-studio",
  "name": "Haptic Studio",
  "version": "1.2.0",
  "tag": "haptic-studio-v1.2.0",
  "released": "2026-11-16",
  "min_widget_version": "1.0.0",
  "notes": "웨이브 편집기에 구간 반복 재생 추가",
  "files": [
    {
      "platform": "win-x64",
      "url": "https://github.com/magi8051/HDL-main-releases/releases/download/haptic-studio-v1.2.0/haptic-studio-1.2.0-win-x64.exe",
      "size": 48234496,
      "sha256": "<SHA-256 값, 16진수 64자>"
    }
  ]
}
```

### 5.3 필드 설명

| 필드 | 필수 | 설명 |
| --- | --- | --- |
| `schema` | O | 매니페스트 형식 버전. 형식이 호환되지 않게 바뀔 때만 올립니다. |
| `id` | O | 앱 ID. 파일 이름과 같아야 합니다. |
| `name` | O | 위젯 화면에 보여줄 앱 이름 |
| `version` | O | 지금 배포 중인 버전 |
| `tag` | O | 해당 릴리스의 태그 |
| `released` | O | 배포일 (`YYYY-MM-DD`) |
| `min_widget_version` | O | 이 버전을 설치하는 데 필요한 최소 위젯 버전 |
| `notes` | - | 위젯에 보여줄 변경 사항 요약 (한두 문장) |
| `files[].platform` | O | 플랫폼 값 ([4.4](#44-배포-파일-이름) 참고) |
| `files[].url` | O | 설치 파일 다운로드 주소 |
| `files[].size` | O | 파일 크기 (바이트) |
| `files[].sha256` | O | 파일의 SHA-256 체크섬 |

### 5.4 주소 형식

| 대상 | 주소 |
| --- | --- |
| 앱 목록 | `https://raw.githubusercontent.com/magi8051/HDL-main-releases/main/catalog.json` |
| 앱 매니페스트 | `https://raw.githubusercontent.com/magi8051/HDL-main-releases/main/apps/<앱 ID>.json` |
| 설치 파일 | `https://github.com/magi8051/HDL-main-releases/releases/download/<태그>/<파일 이름>` |

`raw.githubusercontent.com`은 내용을 몇 분간 캐시합니다. 매니페스트를 수정한 뒤 위젯에 반영되기까지 수 분이 걸릴 수 있습니다.

---

## 6. 위젯의 업데이트 동작

위젯은 아래 순서로 동작합니다.

1. `catalog.json`을 읽어 관리할 앱 목록을 확인합니다.
2. 앱마다 `apps/<앱 ID>.json`을 읽습니다.
3. 매니페스트의 `version`과 PC에 설치된 버전을 비교합니다.
4. 새 버전이 있으면 `min_widget_version`을 확인합니다.
   - 위젯 버전이 낮으면 위젯을 먼저 업데이트하도록 안내합니다.
5. `files`에서 현재 플랫폼에 맞는 항목을 골라 내려받습니다.
6. 내려받은 파일의 SHA-256을 계산해 `sha256` 값과 비교합니다.
   - 값이 다르면 설치하지 않고 파일을 삭제합니다.
7. 설치한 뒤, 설치된 버전을 기록합니다.

위젯 자신도 `apps/widget.json`을 같은 방식으로 확인해 스스로 업데이트합니다.

| 상황 | 위젯의 처리 |
| --- | --- |
| 매니페스트를 읽지 못함 (오프라인 등) | 업데이트 확인을 건너뛰고 설치된 앱은 그대로 실행 |
| `schema` 값이 위젯이 아는 값보다 큼 | 해당 앱의 업데이트를 건너뛰고 위젯 업데이트를 안내 |
| 체크섬 불일치 | 설치 중단, 다시 시도 안내 |

---

## 7. 배포 절차

아래 예시는 Haptic Studio 1.2.0을 배포하는 경우입니다. [GitHub CLI](https://cli.github.com/)(`gh`)가 필요합니다.

### 7.1 앱 배포

**1) 파일 준비**

빌드한 설치 파일의 이름을 규칙에 맞춥니다.

```
haptic-studio-1.2.0-win-x64.exe
```

**2) 체크섬과 크기 확인** (PowerShell)

```powershell
# SHA-256 체크섬
(Get-FileHash .\haptic-studio-1.2.0-win-x64.exe -Algorithm SHA256).Hash.ToLower()

# 파일 크기 (바이트)
(Get-Item .\haptic-studio-1.2.0-win-x64.exe).Length
```

**3) 릴리스 생성**

```powershell
gh release create haptic-studio-v1.2.0 `
  .\haptic-studio-1.2.0-win-x64.exe `
  --repo magi8051/HDL-main-releases `
  --title "Haptic Studio 1.2.0" `
  --notes "웨이브 편집기에 구간 반복 재생 추가" `
  --latest=false
```

**4) 다운로드 확인**

릴리스 페이지에서 파일을 내려받아 체크섬이 2)의 값과 같은지 확인합니다.

**5) 매니페스트 수정**

`apps/haptic-studio.json`의 `version`, `tag`, `released`, `notes`, `files`를 새 버전에 맞게 고친 뒤 `main`에 반영합니다.

```powershell
git add apps/haptic-studio.json
git commit -m "release: haptic-studio 1.2.0"
git push
```

> **순서가 중요합니다.** 반드시 릴리스를 먼저 올리고 매니페스트를 나중에 고칩니다.
> 매니페스트를 고치기 전까지는 사용자에게 새 버전이 나가지 않으므로, 4)에서 문제를 발견하면 그 상태에서 멈추면 됩니다.

### 7.2 위젯 배포

앱 배포와 같고, 3)에서 `--latest=false`만 뺍니다. 위젯 릴리스가 리포의 Latest로 표시됩니다.

```powershell
gh release create widget-v1.1.0 `
  .\widget-1.1.0-win-x64.exe `
  --repo magi8051/HDL-main-releases `
  --title "HDL Widget 1.1.0" `
  --notes "앱 목록 화면 개선"
```

### 7.3 배포 전 확인 목록

- [ ] 태그가 `<앱 ID>-v<버전>` 형식인가
- [ ] 파일 이름이 `<앱 ID>-<버전>-<플랫폼>.<확장자>` 형식인가
- [ ] 위젯이 아닌 앱에 `--latest=false`를 붙였는가
- [ ] 매니페스트의 `url`이 실제로 열리는가
- [ ] 매니페스트의 `sha256`, `size`가 올린 파일과 같은가
- [ ] `min_widget_version`이 맞는가

---

## 8. 운영 가이드

### 8.1 롤백

문제가 있는 버전을 내릴 때는 **매니페스트를 이전 버전 내용으로 되돌립니다.**

```powershell
# 직전 매니페스트 수정 커밋을 되돌림
git revert <커밋 해시>
git push
```

- 릴리스와 태그는 지우지 않습니다. 이미 받아간 사용자의 기록과 맞지 않게 됩니다.
- 이미 문제 버전을 설치한 사용자에게는 수정한 **새 PATCH 버전**을 배포합니다. 위젯은 더 높은 버전만 업데이트로 인식하기 때문입니다.

### 8.2 이미 올린 파일은 교체하지 않기

같은 태그의 파일을 바꾸면 체크섬이 달라져 위젯에서 설치가 실패합니다.
파일에 문제가 있으면 PATCH 버전을 올려 새로 배포합니다.

### 8.3 새 앱 추가

1. 앱 ID를 정합니다.
2. [2. 관리 대상 앱](#2-관리-대상-앱) 표에 등록합니다.
3. 첫 릴리스를 올립니다 ([7.1](#71-앱-배포)의 1~4단계).
4. `apps/<앱 ID>.json`을 만듭니다.
5. `catalog.json`의 `apps`에 앱 ID를 추가합니다.

5번을 마지막에 해야 위젯이 준비되지 않은 앱을 보여주지 않습니다.

### 8.4 앱 배포 중단

1. `catalog.json`의 `apps`에서 앱 ID를 뺍니다.
2. `apps/<앱 ID>.json`과 기존 릴리스는 남겨둡니다.

### 8.5 제한 사항

| 항목 | 내용 |
| --- | --- |
| 파일 크기 | 릴리스 첨부 파일은 개당 2GiB 미만 |
| 공개 범위 | 리포가 공개이므로 올린 파일은 누구나 받을 수 있음 |
| 반영 시간 | 매니페스트 수정 후 위젯 반영까지 수 분 |

### 8.6 배포 자동화 (추후)

각 앱 개발 리포의 CI에서 7.1 절차를 자동으로 수행할 수 있습니다.
이 리포에 쓰기 권한(Contents: Read and write)만 가진 토큰을 발급해 각 개발 리포의 시크릿으로 등록해야 합니다.
수동 배포 절차가 안정된 뒤에 도입합니다.

---

## 9. 확정이 필요한 항목

| # | 항목 | 현재 문서의 가정 |
| --- | --- | --- |
| 1 | 앱 ID | `widget`, `haptic-studio`, `a2v`, `sub1`, `sub2` |
| 2 | `sub1`, `sub2`의 실제 앱 이름 | 미정 |
| 3 | 지원 플랫폼 | Windows 64비트(`win-x64`)만 |
| 4 | 설치 파일 형식 | 설치 프로그램(`.exe`) |
| 5 | 매니페스트 필드 | [5.3](#53-필드-설명)의 구성 |
| 6 | 기존 4개 리포 처리 | 이 리포로 통합 후 보관(archive) |

---

## 10. 문서 이력

| 버전 | 날짜 | 내용 |
| --- | --- | --- |
| 0.1.0 | 2026-11-16 | 초안 작성 — 단일 리포 + 매니페스트 방식의 사용 방법 정리 |
