# VS Code SFTP 익스텐션 사용 가이드

> 대상 익스텐션: **[SFTP](https://marketplace.visualstudio.com/items?itemName=Natizyskunk.sftp)** by *Natizyskunk*
> (liximomo가 만든 원본 `vscode-sftp`의 유지보수 포크. 원본은 더 이상 관리되지 않으므로 이 포크를 사용한다.)
>
> 원문 위키: <https://github.com/Natizyskunk/vscode-sftp/wiki>
> 이 문서는 위키의 모든 하위 페이지(Home / Setting / Configuration / Common / SFTP-only / FTP(s)-only / Commands / FAQ)를 정리한 한글 요약본이다.

---

## 목차

1. [개요 & 주요 기능](#1-개요--주요-기능)
2. [시작하기 (설치 & 최초 설정)](#2-시작하기-설치--최초-설정)
3. [설정 파일 `sftp.json`](#3-설정-파일-sftpjson)
   - [공통 설정 (Common)](#31-공통-설정-common)
   - [SFTP 전용 설정](#32-sftp-전용-설정)
   - [FTP(s) 전용 설정](#33-ftps-전용-설정)
   - [프로필 (Profiles) — 다중 환경 전환](#34-프로필-profiles--다중-환경-전환)
4. [VS Code 익스텐션 설정 (Settings)](#4-vs-code-익스텐션-설정-settings)
5. [명령어 (Commands)](#5-명령어-commands)
6. [실전 예제 모음](#6-실전-예제-모음)
7. [FAQ / 트러블슈팅](#7-faq--트러블슈팅)
8. [보안 주의사항](#8-보안-주의사항)

---

## 1. 개요 & 주요 기능

VS Code 안에서 로컬 작업 디렉터리와 원격 서버(SFTP/FTP/FTPS)를 동기화해 주는 익스텐션이다. 저장할 때 자동 업로드, 원격 파일 탐색, 양방향 동기화 등을 지원한다.

**주요 기능**

- **Remote Explorer** — 원격 서버 파일을 트리로 탐색
- **로컬 ↔ 원격 파일 비교(diff)**
- **디렉터리 동기화** — 로컬→원격 / 원격→로컬 / 양방향
- **업로드 / 다운로드**
- **저장 시 자동 업로드** (`uploadOnSave`)
- **파일 감시(watcher)** — VS Code 외부에서 바뀐 파일도 감지해 업로드/삭제
- **다중 설정(context 별) & 전환 가능한 프로필(profiles)**
- **임시 파일 업로드(`useTempFile`)** — 업로드 도중 불완전한 파일이 노출되는 것을 방지

---

## 2. 시작하기 (설치 & 최초 설정)

1. 확장 패널 열기 — `Ctrl+Shift+X` (Windows/Linux) / `Cmd+Shift+X` (Mac)
2. **liximomo의 옛 SFTP 익스텐션이 깔려 있다면 제거**한다 (충돌 방지).
3. 마켓플레이스에서 **Natizyskunk의 SFTP**를 설치한다.
4. 명령 팔레트(`Ctrl/Cmd + Shift + P`)에서 **`SFTP: Config`** 실행 → 프로젝트 루트에 `.vscode/sftp.json`이 생성된다.
5. 생성된 `sftp.json`에 서버 접속 정보를 입력한다.
6. 필요하면 **`SFTP: Download Project`**로 원격 디렉터리를 로컬로 내려받는다.
7. 이후 로컬에서 파일을 저장하면(설정에 따라) 자동으로 원격에 반영된다.

**최소 설정 예시**

```json
{
  "host": "host",
  "username": "username",
  "remotePath": "/remote/workspace"
}
```

---

## 3. 설정 파일 `sftp.json`

- 위치: 프로젝트 루트의 **`.vscode/sftp.json`**
- 생성/열기: 명령 팔레트 → **`SFTP: Config`**

### 3.1 공통 설정 (Common)

모든 프로토콜에 공통으로 적용되는 옵션.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `name` | string | — | 설정을 식별하는 이름 |
| `context` | string | 워크스페이스 루트 | 워크스페이스 루트 기준 상대 경로. 이 경로 하위만 동기화 대상이 된다 |
| `protocol` | `sftp` \| `ftp` | `sftp` | 접속 프로토콜 |
| `host` | string | — | 서버 호스트명 또는 IP |
| `port` | integer | — | 서버 포트 (SFTP 보통 22, FTP 21) |
| `username` | string | — | 접속 계정 |
| `password` | string | — | 비밀번호 ⚠️ **평문 저장됨** |
| `remotePath` | string | `/` | 원격 서버의 절대 경로 |
| `filePerm` | number(8진수) | `false` | 새로 만드는 파일의 8진수 권한 (예: `0644`) |
| `dirPerm` | number(8진수) | `false` | 새로 만드는 디렉터리의 8진수 권한 (예: `0755`) |
| `uploadOnSave` | boolean | `false` | VS Code에서 저장할 때마다 업로드 |
| `useTempFile` | boolean | `false` | 저장 시 임시 파일로 먼저 올려, 불완전한 파일 접근을 방지 |
| `openSsh` | boolean | `false` | 원자적(atomic) 업로드 활성화 (**OpenSSH 서버 전용**, `useTempFile`도 `true`여야 함) |
| `downloadOnOpen` | boolean | `false` | 파일을 열 때마다 원격에서 다운로드 |
| `syncOption` | object | `{}` | Sync 명령 동작 설정 (아래 참조) |
| `syncOption.delete` | boolean | — | 목적지에만 있는 불필요한 파일 삭제 |
| `syncOption.skipCreate` | boolean | — | 목적지에 새 파일 생성 건너뜀 |
| `syncOption.ignoreExisting` | boolean | — | 목적지에 이미 있는 파일은 갱신하지 않음 |
| `syncOption.update` | boolean | — | 소스가 더 최신일 때만 목적지 갱신 |
| `ignore` | string[] | `[]` | 동기화에서 제외할 파일/폴더 (와일드카드 패턴) |
| `ignoreFile` | string | — | ignore 규칙이 담긴 파일 경로 (`.gitignore` 형식) |
| `watcher` | object | `{}` | 외부 변경 감시 설정 (아래 참조) |
| `watcher.files` | string | — | 감시할 glob 패턴 (VS Code 외부 편집도 감지) |
| `watcher.autoUpload` | boolean | — | 파일 변경 시 자동 업로드 |
| `watcher.autoDelete` | boolean | — | 파일 삭제 시 원격에서도 삭제 |
| `remoteTimeOffsetInHours` | number | `0` | 로컬-원격 서버 간 시간 차(시간 단위). 동기화 비교에 사용 |
| `remoteExplorer` | object | `{}` | Remote Explorer 동작 설정 |
| `remoteExplorer.filesExclude` | string[] | — | Remote Explorer에서 숨길 파일/폴더 패턴 |
| `remoteExplorer.order` | number | — | Remote Explorer 표시 순서 |
| `concurrency` | number | `4` | 동시 처리 최대 개수. 연결이 불안정하면 낮춰라 |
| `connectTimeout` | number(ms) | `10000` | 최대 연결 대기 시간(밀리초) |
| `limitOpenFilesOnRemote` | number \| boolean | `false` | 원격 서버에서 동시에 여는 파일 개수 제한. 파일 디스크립터 부족(`Error: Failure`) 대응용 (주의해서 사용) |

> **`ignore` vs `remoteExplorer.filesExclude`**
> `ignore`는 업로드/다운로드/동기화 대상에서 제외한다. `remoteExplorer.filesExclude`는 Remote Explorer **화면에서만** 숨긴다.

### 3.2 SFTP 전용 설정

`protocol`이 `sftp`일 때 사용하는 SSH 인증/전송 옵션.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `agent` | string | — | ssh-agent UNIX 소켓 경로. Windows는 `"pageant"` 또는 Cygwin 소켓 경로 |
| `privateKeyPath` | string | — | 개인키(private key) 파일의 전체 경로 |
| `passphrase` | string \| boolean | — | 개인키 복호화 문구. `true`로 두면 접속 시 입력창이 뜬다(평문 저장 회피) |
| `interactiveAuth` | boolean \| string[] | `false` | 키보드 인터랙티브 인증 활성화(2FA 등). 서버가 지원해야 함. 문자열 배열로 미리 응답을 넣어 프롬프트를 생략할 수도 있음 |
| `algorithms` | object | — | 전송 계층 알고리즘 override. `kex` / `cipher` / `serverHostKey` / `hmac` 배열 지정 |
| `sshConfigPath` | string | `~/.ssh/config` | SSH config 파일 경로 |
| `sshCustomParams` | string | — | `SFTP: Open SSH in Terminal` 실행 시 SSH 명령에 덧붙일 파라미터 (예: `"-g"`) |

**개인키 인증 예시**

```json
{
  "name": "My Server",
  "host": "example.com",
  "protocol": "sftp",
  "port": 22,
  "username": "user",
  "privateKeyPath": "/Users/me/.ssh/id_rsa",
  "passphrase": true,
  "remotePath": "/var/www/html"
}
```

**레거시 서버 알고리즘 override 예시** (아래 FAQ의 "Connection closed" 참고)

```json
{
  "algorithms": {
    "kex": [
      "diffie-hellman-group1-sha1",
      "diffie-hellman-group14-sha1"
    ],
    "cipher": ["aes128-ctr", "aes192-ctr", "aes256-ctr"],
    "serverHostKey": ["ssh-rsa", "ssh-dss"],
    "hmac": ["hmac-sha2-256", "hmac-sha2-512", "hmac-sha1"]
  }
}
```

### 3.3 FTP(s) 전용 설정

`protocol`이 `ftp`일 때 사용하는 옵션.

| 옵션 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `secure` | boolean \| string | `false` | `true`=제어+데이터 연결 모두 암호화, `"control"`=제어 연결만 암호화, `"implicit"`=암묵적 암호화(구식, 보통 포트 990) |
| `secureOptions` | object | — | Node.js `tls.connect()`에 전달할 추가 옵션 |

**예시**

```json
{
  "protocol": "ftp",
  "secure": "control",
  "secureOptions": {
    "enableTrace": true
  }
}
```

> `tls.connect` 옵션 참고: <https://nodejs.org/api/tls.html#tls_tls_connect_options_callback>

### 3.4 프로필 (Profiles) — 다중 환경 전환

`profiles`로 dev / prod 등 여러 환경을 하나의 `sftp.json`에 정의하고, **`SFTP: Set Profile`** 명령으로 전환한다. 최상위에 쓴 값이 공통 기본값이 되고, 각 프로필 값이 이를 덮어쓴다.

```json
{
  "username": "username",
  "password": "password",
  "remotePath": "/remote/workspace/a",
  "watcher": {
    "files": "dist/*.{js,css}",
    "autoUpload": false,
    "autoDelete": false
  },
  "profiles": {
    "dev": {
      "host": "dev-host",
      "remotePath": "/dev",
      "uploadOnSave": true
    },
    "prod": {
      "host": "prod-host",
      "remotePath": "/prod"
    }
  },
  "defaultProfile": "dev"
}
```

- `defaultProfile` — 시작 시 기본으로 선택되는 프로필
- 전환: 명령 팔레트 → **`SFTP: Set Profile`**

---

## 4. VS Code 익스텐션 설정 (Settings)

`sftp.json`이 아니라 VS Code 전역/워크스페이스 **Settings**에서 조정하는 항목. (File/Code → Preferences → Settings)

| 설정 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `sftp.debug` | boolean | `false` | SFTP 출력 패널(View → Output → SFTP)에 디버그 로그 출력. **변경 후 VS Code 재시작 필요** |
| `sftp.downloadWhenOpenInRemoteExplorer` | boolean | `false` | Remote Explorer에서 파일을 열 때 기본 동작을 "View Content" 대신 **"Edit in Local"**로 바꿈 |

---

## 5. 명령어 (Commands)

명령 팔레트(`Ctrl/Cmd + Shift + P`)에서 실행한다.

**기본 명령**

| 명령 | 설명 |
| --- | --- |
| `SFTP: Config` | 프로젝트에 `sftp.json` 설정 파일 생성 |
| `SFTP: Set Profile` | 현재 프로필 전환 (프로필 이름 입력) |
| `SFTP: Upload Active File` | 현재 열린 파일을 원격에 업로드 |
| `SFTP: Upload Changed Files` | 마지막 Git 커밋 이후 변경/생성된 파일 전부 업로드 (`Ctrl+Alt+U`) |
| `SFTP: Upload Active Folder` | 현재 파일이 속한 폴더 전체 업로드 |
| `SFTP: Download Active File` | 원격 버전을 내려받아 로컬 파일 덮어쓰기 |
| `SFTP: Download Active Folder` | 현재 파일이 속한 폴더 전체 다운로드 |
| `SFTP: Sync Local -> Remote` | 로컬 → 원격 동기화 (타임스탬프 다른/로컬 전용 파일 업로드) |
| `SFTP: Sync Remote -> Local` | 원격 → 로컬 동기화 (반대 방향) |
| `SFTP: Sync Both Directions` | 양방향 비교 후, 양쪽 모두 최신 파일이 존재하도록 동기화 |
| `SFTP: List Active Folder` | 현재 폴더의 원격 내용 목록 조회 |
| `SFTP: Cancel All Transfers` | 진행 중인 모든 업로드/다운로드 중단 |
| `SFTP: Open SSH in Terminal` | VS Code 터미널에서 설정 서버로 자동 SSH 접속 |

**키바인딩용 명령** (`keybindings.json`에서 인자와 함께 사용)

| 명령 | 설명 |
| --- | --- |
| `sftp.upload` | 지정한 파일/폴더 업로드 |
| `sftp.download` | 지정한 파일/폴더 다운로드 |

**우클릭 메뉴의 Alt 명령**

| 명령 | 설명 |
| --- | --- |
| `Force Download` | ignore 규칙을 무시하고 다운로드 |
| `Force Upload` | ignore 규칙을 무시하고 업로드 |

---

## 6. 실전 예제 모음

### 6.1 저장 시 자동 업로드 + 외부 변경 감시

```json
{
  "name": "My Server",
  "host": "<host_ip_address>",
  "protocol": "sftp",
  "port": 22,
  "username": "user1",
  "remotePath": "/var/www/html",
  "uploadOnSave": true,
  "ignore": [".vscode", ".git", ".DS_Store", "node_modules"],
  "watcher": {
    "files": "**/*",
    "autoUpload": true,
    "autoDelete": false
  }
}
```

### 6.2 폴더 "내용물"만 올리고 폴더 자체는 안 올리기

`context`를 올리려는 폴더로 지정하면, 그 하위 내용이 `remotePath` 아래로 바로 복사된다.
(예: `./build`의 js/html이 `/folder1/folder2/folder3`로)

```json
{
  "name": "My Server",
  "host": "<host_ip_address>",
  "protocol": "sftp",
  "port": 22,
  "username": "user1",
  "remotePath": "/folder1/folder2/folder3",
  "context": "./build",
  "uploadOnSave": false,
  "watcher": {
    "files": "*.{js,html}",
    "autoUpload": true,
    "autoDelete": false
  }
}
```

### 6.3 사용자 개입 없이 양방향 자동 동기화

> Git과 함께 쓰면, 브랜치 체크아웃/커밋 되돌리기 시 서버도 함께 갱신된다.

```json
{
  "name": "My Server",
  "host": "<host_ip_address>",
  "protocol": "sftp",
  "port": 22,
  "username": "user1",
  "remotePath": "/folder1/folder2/folder3",
  "uploadOnSave": false,
  "watcher": {
    "files": "**/*",
    "autoUpload": true,
    "autoDelete": true
  },
  "syncOption": {
    "delete": true
  }
}
```

---

## 7. FAQ / 트러블슈팅

### `Error: Failure`
원격 측이 보내는 일반적인 실패 메시지(syscall 실패 등). SFTP 서버의 디버그 로그를 켜고 재현하면 원인을 볼 수 있다.
- **해결 1** — `remotePath`가 심볼릭 링크라면 실제 경로로 바꾼다.
- **해결 2** — 서버의 파일 디스크립터가 부족한 경우. 한도를 늘리거나, 권한이 없으면 설정에서 `limitOpenFilesOnRemote`를 지정한다.

### `Error: Connection closed`
오래된/레거시 시스템에서 연결이 계속 끊기는 경우. 문제를 일으키는 신형 알고리즘 `diffie-hellman-group-exchange-sha256`을 `kex` 목록에서 빼도록 `algorithms`를 명시적으로 override한다. ([3.2 알고리즘 예시](#32-sftp-전용-설정) 참고)

### `Upload Changed Files`가 동작하지 않음
[vscode-sftp issue #854](https://github.com/liximomo/vscode-sftp/issues/854) 참고. 이 포크에서는 해당 명령이 보이도록 수정되었고 기본 단축키(`Ctrl+Alt+U`)가 추가되었다.

### `ENFILE: file table overflow ...` (macOS)
macOS는 열 수 있는 파일 수 제한이 낮다. 아래로 한도를 올린다.

```sh
echo kern.maxfiles=65536 | sudo tee -a /etc/sysctl.conf
echo kern.maxfilesperproc=65536 | sudo tee -a /etc/sysctl.conf
sudo sysctl -w kern.maxfiles=65536
sudo sysctl -w kern.maxfilesperproc=65536
ulimit -n 65536
```

### root 권한으로 업로드하기
확실한 방법은 아니지만, `sftp.json`에 다음 워크어라운드를 넣어볼 수 있다. ([issue #559](https://github.com/liximomo/vscode-sftp/issues/559))

```json
"sshCustomParams": "sudo su -;"
```

### Remote Explorer에 dotfiles/숨김 파일 표시 (proftpd)
`proftpd.conf`에서 `ListOptions`를 `"-l"` → `"-la"`로 변경한다.
(파일 위치 예: `/etc/proftpd.conf`, `/etc/proftpd/proftpd.conf`, `/usr/local/etc/proftpd.conf` 등)

```conf
<Global>
    ListOptions        "-la"
</Global>
```

---

## 8. 보안 주의사항

- **`password`와 `passphrase`(문자열)는 `sftp.json`에 평문으로 저장된다.**
  - 가능하면 `password`를 생략하고 접속 시 입력하도록 하거나, **SSH 키 인증(`privateKeyPath`)**을 사용한다.
  - `passphrase`는 문자열 대신 `true`로 두어 입력 프롬프트를 띄우면 평문 저장을 피할 수 있다.
- **`.vscode/sftp.json`을 `.gitignore`에 추가**해 접속 정보가 저장소에 커밋되지 않도록 한다.

---

### 참고 링크

- 위키 홈: <https://github.com/Natizyskunk/vscode-sftp/wiki>
- FAQ 원문: <https://github.com/Natizyskunk/vscode-sftp/blob/master/FAQ.md>
- 마켓플레이스: <https://marketplace.visualstudio.com/items?itemName=Natizyskunk.sftp>
