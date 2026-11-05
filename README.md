# Codyssey Pre-Course — 개발 환경 구축 및 검증

Windows PC의 WSL2 Ubuntu 환경에서 Linux 명령어, Docker 컨테이너, Nginx 웹 서버, 데이터 마운트, Git/GitHub를 실습하고 명령어와 결과를 기록한 프로젝트입니다.

- GitHub: https://github.com/UYaddendden/i0-codyssey
- 주요 소스: [`Dockerfile`](Dockerfile), [`site/index.html`](site/index.html)
- 증거: [`evidence/logs/`](evidence/logs/), [`evidence/screenshots/`](evidence/screenshots/)

## 1. 개발 환경

| 항목 | 확인된 환경 |
|---|---|
| 호스트 | Windows + WSL2 (Windows 세부 버전은 미기록) |
| Linux | Ubuntu 24.04.4 LTS |
| Shell | Bash (`/bin/bash`) |
| 개발 도구 | VS Code Remote - WSL, Ubuntu 터미널 |
| Docker CLI / Engine | 29.9.2 / 29.9.2 |
| Git | 2.43.0 |
| 브랜치 | `main` |

실행 기록: [01_environment.txt](evidence/logs/01_environment.txt). Docker Desktop의 별도 앱 버전 및 Windows 세부 버전은 해당 로그에 기록하지 않았습니다.

## 2. 필수 실습 체크리스트

| 실습 | 확인 결과 | 증거 |
|---|---|---|
| Linux 기본 명령어 | 경로 조회, 목록, 파일 생성·복사·이동·삭제, 내용 확인 | [03_linux_commands.txt](evidence/logs/03_linux_commands.txt) |
| 파일/디렉터리 권한 | 파일 644→600, 디렉터리 755→700 | [02_permissions.txt](evidence/logs/02_permissions.txt) |
| Docker 버전·Engine | CLI/Engine 모두 29.9.2 | [01_environment.txt](evidence/logs/01_environment.txt) |
| 이미지·컨테이너·로그·리소스 | `images`, `ps -a`, `logs`, `stats` 확인 | [04_docker_operations.txt](evidence/logs/04_docker_operations.txt) |
| hello-world | 이전 실행 컨테이너 `Exited (0)` 확인 | [04_docker_operations.txt](evidence/logs/04_docker_operations.txt) |
| Ubuntu 컨테이너 | `ubuntu:24.04` 실행 후 내부 `pwd`, `ls`, `echo` 성공 | [05_ubuntu_container.txt](evidence/logs/05_ubuntu_container.txt) |
| 컨테이너 생명주기 | 중지→재시작→내부 명령 실행 | [06_container_lifecycle.txt](evidence/logs/06_container_lifecycle.txt) |
| Dockerfile 이미지 빌드 | `codyssey-web:submission` 빌드 성공 | [07_docker_build.txt](evidence/logs/07_docker_build.txt) |
| 포트 매핑 및 HTTP | 8080/8081/8082 모두 HTTP 200 | [08_port_mapping.txt](evidence/logs/08_port_mapping.txt) |
| Bind Mount | 호스트 파일 수정→즉시 반영→원복 | [09_bind_mount.txt](evidence/logs/09_bind_mount.txt) |
| Docker Volume | 첫 컨테이너 삭제 후 새 컨테이너에서 데이터 읽기 | [10_docker_volume.txt](evidence/logs/10_docker_volume.txt) |
| Git/GitHub | 원격 `origin`, `main`, 커밋 기록 확인 | [11_git_github.txt](evidence/logs/11_git_github.txt) |
| VS Code | WSL 프로젝트와 소스 제어 화면 캡처 | [04_vscode_git.png](evidence/screenshots/04_vscode_git.png) |
| VS Code GitHub | 계정 연동 화면 캡처(화면 내용 별도 확인 필요) | [05_vscode_github.png](evidence/screenshots/05_vscode_github.png) |

## 3. Linux CLI와 접근 권한

```bash
pwd
ls -la
mkdir -p evidence/linux-practice
cd evidence/linux-practice
touch original.txt
echo 'Hello Codyssey Linux' > original.txt
cat original.txt
cp original.txt copied.txt
mv copied.txt renamed.txt
rm renamed.txt
```

권한 비교는 `chmod 644`→`chmod 600`(파일), `chmod 755`→`chmod 700`(디렉터리) 순서로 진행했습니다. `ls -ld` 출력에서 각각 `-rw-r--r--`→`-rw-------`, `drwxr-xr-x`→`drwx------`를 확인했습니다.

- [Linux 명령어 실행 기록](evidence/logs/03_linux_commands.txt)
- [권한 변경 전후 기록](evidence/logs/02_permissions.txt)

## 4. Docker 이미지·컨테이너 운영

```bash
docker --version
docker info --format '{{.ServerVersion}}'
docker images
docker ps -a
docker logs --tail 10 codyssey-nginx
docker stats --no-stream
```

기존 `hello-world` 컨테이너의 종료 상태 `Exited (0)`을 확인했습니다. Ubuntu 컨테이너는 `docker run -dit --name codyssey-ubuntu ubuntu:24.04 bash`로 실행한 뒤 `docker exec`로 내부 파일 목록과 메시지를 확인했습니다. 이후 `docker stop`과 `docker start`로 상태 변화를 검증했습니다. 중지 로그의 `Exited (137)`은 강제 종료 신호에 해당할 수 있으나, 해당 로그만으로 정확한 발생 원인을 단정하지 않습니다.

- [운영 로그](evidence/logs/04_docker_operations.txt)
- [Ubuntu 컨테이너](evidence/logs/05_ubuntu_container.txt)
- [중지·재시작](evidence/logs/06_container_lifecycle.txt)

## 5. Dockerfile과 Nginx 웹 서버

[`Dockerfile`](Dockerfile):

```dockerfile
FROM nginx:alpine
COPY site/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

```bash
docker build -t codyssey-web:submission .
docker images codyssey-web
```

빌드 성공은 [07_docker_build.txt](evidence/logs/07_docker_build.txt)에 기록했습니다. `EXPOSE 80`은 컨테이너 내부 포트의 선언이며 실제 호스트 포트 공개는 `docker run -p`로 설정합니다.

| 컨테이너 | 이미지 | 호스트 → 컨테이너 | 응답 |
|---|---|---|---|
| `codyssey-nginx` | `nginx:alpine` | 8080 → 80 | Nginx 기본 페이지, HTTP 200 |
| `codyssey-custom` | `codyssey-web:v1` | 8081 → 80 | `Hello, Codyssey!`, HTTP 200 |
| `codyssey-bind` | `nginx:alpine` | 8082 → 80 | `Bind Mount Success!`, HTTP 200 |

> 참고: 새로 빌드한 `codyssey-web:submission`의 빌드는 검증했지만, 위 8081번 컨테이너는 이전에 만든 `codyssey-web:v1` 이미지로 실행 중입니다.

- [포트 매핑 및 HTTP 응답](evidence/logs/08_port_mapping.txt)
- [8080 브라우저 화면](evidence/screenshots/01_nginx_8080.png)
- [8081 브라우저 화면](evidence/screenshots/02_custom_8081.png)
- [8082 브라우저 화면](evidence/screenshots/03_bind_8082.png)

## 6. Bind Mount와 Volume

### Bind Mount

호스트의 `site/`를 Nginx의 `/usr/share/nginx/html`에 읽기 전용으로 연결했습니다. `site/index.html`의 내용을 `Bind Mount Success!`에서 `Bind Mount Updated!`로 바꾸자 **이미지 재빌드나 컨테이너 재시작 없이** `http://localhost:8082` 응답도 변경됐습니다. 이후 원래 파일로 복원했습니다.

- [변경 전·후·복원 로그](evidence/logs/09_bind_mount.txt)

### Docker Volume

```bash
docker volume create codyssey-data
docker run --rm -v codyssey-data:/data alpine sh -c 'echo "Hello from Docker Volume!" > /data/test.txt'
docker run --rm -v codyssey-data:/data alpine cat /data/test.txt
docker volume inspect codyssey-data
```

첫 번째 컨테이너가 `--rm`으로 삭제된 후에도 두 번째 컨테이너에서 `Hello from Docker Volume!`을 읽었습니다. 따라서 데이터가 개별 컨테이너 삭제 이후에도 유지됨을 확인했습니다.

- [Volume 영속성 검증 로그](evidence/logs/10_docker_volume.txt)

## 7. Git, GitHub 및 VS Code

Git 원격 저장소는 `https://github.com/UYaddendden/i0-codyssey.git`, 브랜치는 `main`입니다. 로컬 Git 이력에는 초기 커밋, 원격 이력 병합, README 커밋이 기록돼 있습니다.

```bash
git --version
git config user.name
git remote -v
git branch --show-current
git log --oneline -5
git status --short
```

- [Git 설정·원격·이력 로그](evidence/logs/11_git_github.txt)
- [VS Code 소스 제어 화면](evidence/screenshots/04_vscode_git.png)
- [VS Code GitHub 계정 화면](evidence/screenshots/05_vscode_github.png)

계정 비밀번호, GitHub PAT, 개인 키 등 인증 정보는 제출하지 않습니다.

## 8. 트러블슈팅

### 사례 1. GitHub HTTPS 인증 실패

- **문제:** Git Push 시 사용자명/토큰 인증 오류가 발생했습니다.
- **가설:** GitHub 비밀번호 인증 불가, PAT 입력 또는 권한 문제 가능성을 확인했습니다.
- **확인:** Classic PAT의 `repo` 권한을 확인하고 GitHub API 계정 인증을 테스트했습니다.
- **해결:** HTTPS Git 인증에 PAT를 사용해 인증을 통과했습니다. 최초 오류의 정확한 입력 원인은 단정하지 않습니다.

### 사례 2. Git Push `fetch first` 거부

- **문제:** 원격 `main`에 로컬에 없는 초기 커밋이 있어 Push가 거부됐습니다.
- **가설·확인:** `git fetch origin`, `git log --oneline --all --graph --decorate`로 서로 다른 초기 이력을 확인했습니다.
- **해결:** `git pull origin main --allow-unrelated-histories --no-rebase`로 병합하고 Push에 성공했습니다.

### 사례 3. Bind Mount 수정 결과가 처음에 반영되지 않음

- **문제:** 첫 검증 로그에서 BEFORE와 AFTER의 HTML이 같았습니다.
- **가설·확인:** 실행 로그에 실제 파일 수정 명령이 빠져 있음을 발견했습니다.
- **해결:** `printf '<h1>Bind Mount Updated!</h1>\n' > site/index.html`을 포함해 재실행한 결과, 웹 응답이 즉시 변경됐고 복원도 확인했습니다. [증거](evidence/logs/09_bind_mount.txt)

## 9. 재현 방법

Docker Desktop의 WSL 연동이 활성화된 환경에서 다음 명령어로 사용자 정의 웹 서버를 재현할 수 있습니다. 포트 8081이 사용 중이면 다른 빈 포트를 선택하세요.

```bash
git clone https://github.com/UYaddendden/i0-codyssey.git
cd i0-codyssey
docker build -t codyssey-web:local .
docker run -d --name codyssey-demo -p 8081:80 codyssey-web:local
curl -i http://localhost:8081
```

이 명령으로 실행되는 페이지는 현재 저장소의 [`site/index.html`](site/index.html) 내용이며, 과거 `codyssey-web:v1` 이미지에 포함된 `Hello, Codyssey!` 페이지와 다를 수 있습니다.

## 10. 증거 파일 구조

```text
.
├── Dockerfile
├── README.md
├── site/
│   └── index.html
├── practice/
└── evidence/
    ├── logs/
    │   ├── 01_environment.txt
    │   ├── 02_permissions.txt
    │   ├── 03_linux_commands.txt
    │   ├── 04_docker_operations.txt
    │   ├── 05_ubuntu_container.txt
    │   ├── 06_container_lifecycle.txt
    │   ├── 07_docker_build.txt
    │   ├── 08_port_mapping.txt
    │   ├── 09_bind_mount.txt
    │   ├── 10_docker_volume.txt
    │   └── 11_git_github.txt
    └── screenshots/
        ├── 01_nginx_8080.png
        ├── 02_custom_8081.png
        ├── 03_bind_8082.png
        ├── 04_vscode_git.png
        └── 05_vscode_github.png
```

---

## 11. AI 사전평가 피드백 반영: 핵심 개념과 문제 해결 절차

이 절은 2026-11-04 사전평가에서 보완을 요청받은 사항을 정리합니다. 아래에서 **실제 검증**은 기존 로그에 근거한 설명이며, **진단 예시**는 문제가 발생할 경우 사용할 절차로 실제 장애가 발생했다는 뜻은 아닙니다.

### 11.1 Linux 권한 비트와 권한 변경 의도 (#15)

Linux의 파일 권한은 `사용자(owner) / 그룹(group) / 기타 사용자(others)`의 세 묶음으로 나뉩니다. 각 묶음의 `r`(읽기)=4, `w`(쓰기)=2, `x`(실행/디렉터리 접근)=1을 더해 8진수 한 자리로 표현합니다.

| 대상 | 권한 숫자 | 문자열 표기 | 해석 |
|---|---|---|---|
| 파일 변경 전 | `644` | `-rw-r--r--` | 소유자는 읽기/쓰기, 그룹·기타 사용자는 읽기만 가능 |
| 파일 변경 후 | `600` | `-rw-------` | 소유자만 읽기/쓰기 가능 |
| 디렉터리 변경 전 | `755` | `drwxr-xr-x` | 소유자는 읽기/쓰기/접근, 그룹·기타 사용자는 목록 확인 및 접근 가능 |
| 디렉터리 변경 후 | `700` | `drwx------` | 소유자만 목록 확인·수정·접근 가능 |

예를 들어 `644 = (4+2) / 4 / 4`이고, `755 = (4+2+1) / (4+1) / (4+1)`입니다. 디렉터리의 `x`는 프로그램 실행이 아니라 경로 탐색·접근을 허용하는 권한입니다. 이번 변경 목적은 다른 사용자에게 불필요한 읽기·접근 권한을 주지 않는 **최소 권한 원칙**을 경험하는 것입니다. 실제 전후 비교는 [`02_permissions.txt`](evidence/logs/02_permissions.txt)에 있습니다.

### 11.2 컨테이너 네트워크 네임스페이스와 포트 포워딩 (#13)

Linux 컨테이너는 일반적으로 호스트와 **구분된 네트워크 네임스페이스**를 사용하므로 자체 네트워크 인터페이스, IP 주소, 라우팅 정보, 포트 공간을 가질 수 있습니다. 따라서 컨테이너 안에서 Nginx가 **80번 포트**를 듣고 있다고 해서 호스트의 `localhost:80`으로 자동 접속되는 것은 아닙니다.

Docker 실행 옵션 `-p 8081:80`은 **호스트 8081번 포트로 도착한 트래픽을 컨테이너 80번 포트로 전달**하도록 게시(publish)합니다. 일반적인 Linux Docker 브리지 네트워크에서는 Docker가 NAT/포워딩 규칙 등을 관리하고, Docker Desktop + WSL2 환경에서는 Docker Desktop의 호스트 연결 계층도 관여할 수 있습니다. 즉, 실제 네트워크 구현 세부사항은 환경에 따라 다르지만 호스트 포트와 컨테이너 포트를 연결한다는 목적은 동일합니다.

```text
Windows 브라우저 또는 WSL 터미널
  └─ http://localhost:8081
       └─ Docker가 게시한 호스트 포트 8081
            └─ 컨테이너 네트워크의 80번 포트
                 └─ Nginx → HTML 응답
```

`EXPOSE 80`은 Dockerfile에 컨테이너가 사용할 포트를 **문서화**할 뿐, 호스트 포트를 자동으로 공개하지 않습니다. 포트 게시는 `docker run -p 8081:80 ...`에서 지정합니다. 실제 환경에서는 [`08_port_mapping.txt`](evidence/logs/08_port_mapping.txt)의 `80/tcp -> 0.0.0.0:8081`과 `HTTP/1.1 200 OK`로 연결을 검증했습니다.

### 11.3 호스트 경로·컨테이너 경로·빌드 컨텍스트 선택 이유 (#14)

**Bind Mount:** 실습에서 사용한 `-v "$(pwd)/site:/usr/share/nginx/html:ro"`는 호스트 파일과 컨테이너 파일의 위치를 명시합니다.

- 호스트의 `$(pwd)/site`는 현재 프로젝트 폴더를 기준으로 **절대 경로로 확장**되어, `site`라는 상대 경로만 사용하는 것보다 어떤 폴더가 연결되는지 분명합니다. 단, 명령 실행 전 `cd`로 프로젝트 루트로 이동해야 합니다.
- 컨테이너의 `/usr/share/nginx/html`은 Nginx가 기본 웹 파일을 제공하는 **컨테이너 내부의 절대 경로**이므로 선택했습니다. 호스트와 컨테이너의 디렉터리 구조는 같을 필요가 없습니다.
- `:ro`는 **컨테이너에서 마운트한 파일을 읽기 전용**으로 취급하게 해 불필요한 수정 위험을 줄입니다. 호스트에서 원본을 수정하는 것은 가능하므로, 변경 내용이 브라우저에 바로 반영되는 실습과 모순되지 않습니다.

**Dockerfile COPY:** `docker build -t codyssey-web:submission .`의 마지막 `.`은 프로젝트 루트를 **빌드 컨텍스트**로 지정합니다. `COPY site/index.html /usr/share/nginx/html/index.html`에서 왼쪽 `site/index.html`은 빌드 컨텍스트 기준의 상대 경로이고, 오른쪽은 이미지 내부의 절대 경로입니다. 상대 소스 경로 덕분에 저장소를 다른 컴퓨터의 다른 경로에 복제해도 같은 폴더 구조라면 재빌드할 수 있습니다. 주요 파일은 `Dockerfile`, `site/index.html`, `README.md`, `practice/`, `evidence/`이며 실제 이미지 파일 복사 대상은 `site/index.html`입니다. `.dockerignore`로 불필요한 파일을 제외하면 빌드 컨텍스트를 더 작게 만들 수 있습니다.

실제 검증은 [`07_docker_build.txt`](evidence/logs/07_docker_build.txt), [`09_bind_mount.txt`](evidence/logs/09_bind_mount.txt)에 기록되어 있습니다.

### 11.4 포트 충돌 진단 및 해결 절차 (#16)

**진단 예시이며, 이번 미션에서 포트 충돌이 실제 발생했다는 의미는 아닙니다.** `8081` 포트로 새 컨테이너를 시작했을 때 `port is already allocated` 또는 `address already in use`가 나온다면 다음 순서로 확인합니다.

1. **Docker 점유 확인:** `docker ps --format 'table {{.Names}}\t{{.Ports}}'`로 실행 중인 컨테이너가 `8081`을 게시하고 있는지 확인합니다. 이번 프로젝트의 `codyssey-custom` 컨테이너는 실제로 8081번을 사용합니다.
2. **Linux/WSL 리스너 확인:** `ss -ltnp | grep ':8081'`로 Linux 측 리스너를 조회합니다. Docker Desktop이 Windows 쪽에서 포트를 게시한 경우 WSL의 `ss`만으로는 표시되지 않을 수 있습니다.
3. **Windows 측 확인(필요 시 PowerShell):** `Get-NetTCPConnection -LocalPort 8081 -ErrorAction SilentlyContinue`로 Windows 포트 점유를 확인합니다. `netstat -ano | findstr :8081`도 대안입니다.
4. **해결:** 원인이 본인이 실행한 불필요한 컨테이너라면 이름을 확인한 후 `docker stop <컨테이너명>`으로 중지합니다. **기존 실습 컨테이너를 유지해야 한다면** 삭제하거나 강제 종료하지 않고 `docker run -d --name codyssey-demo-8091 -p 8091:80 codyssey-web:submission`처럼 다른 빈 호스트 포트를 사용합니다. 포트 `8091`도 비어 있는지 먼저 확인해야 합니다.
5. **재확인:** `docker port <컨테이너명>`, `curl -v --max-time 5 http://localhost:8091`과 브라우저 주소창으로 포트 연결 및 HTTP 응답을 확인합니다. 연결 거부·시간 초과·HTTP 404는 서로 다른 종류의 문제이므로 오류 메시지를 구분해 조사합니다.

충돌이 발생하지 않았다면 임의의 장애 로그를 만들 필요는 없으며, 점검 명령과 **실제 조회 결과**만 기록합니다.

### 11.5 추가 보완: Docker 상태, 이미지·컨테이너, 진단

- **Docker 데몬/Engine:** [`01_environment.txt`](evidence/logs/01_environment.txt)의 `docker info --format '{{.ServerVersion}}' → 29.9.2`와 [`04_docker_operations.txt`](evidence/logs/04_docker_operations.txt)의 `docker ps` 결과가 정상 조회되므로 **명령 실행 시점에 Engine이 요청에 응답했다**는 사실을 확인할 수 있습니다.
- **이미지와 컨테이너:** 이미지는 컨테이너 실행에 사용할 파일·설정의 원본이며, 컨테이너는 그 이미지에서 실행되는 인스턴스입니다. 실행 중인 컨테이너 안의 파일을 수정해도 원본 이미지가 자동 변경되지는 않습니다. Dockerfile 수정 후 새 내용을 반영하려면 일반적으로 `docker build`로 새 이미지를 만들고, 새 이미지로 컨테이너를 다시 생성해야 합니다. Bind Mount는 예외적으로 호스트 파일 변경이 실행 중인 컨테이너에 바로 반영됩니다.
- **hello-world:** 기존 [`04_docker_operations.txt`](evidence/logs/04_docker_operations.txt)에는 `Exited (0)` 상태가 남아 있습니다. 화면 메시지를 직접 확인하려면 `docker run --rm hello-world`를 실행해 별도 로그로 남길 수 있습니다.
- **이미지 및 컨테이너 정리:** 실습용 임시 컨테이너를 `docker stop` → `docker rm`으로 제거하고, 이미지에 붙인 임시 태그를 `docker image rm`으로 제거할 수 있습니다. 사용 중인 `codyssey-nginx`, `codyssey-custom`, `codyssey-bind`, `codyssey-data`를 무작정 삭제하지 않습니다.
- **curl 실패 진단:** `curl -v --max-time 5 http://localhost:8081`로 연결 과정과 오류를 확인하며, `docker ps`, `docker logs <컨테이너명>`, `docker port <컨테이너명>`과 함께 판단합니다. `curl`의 HTTP 응답 코드와 TCP 연결 실패는 구별해야 합니다.

### 11.6 Volume 삭제 위험과 백업·복원 원칙

[`10_docker_volume.txt`](evidence/logs/10_docker_volume.txt)에서 확인했듯 **컨테이너 삭제는 별도로 관리되는 Docker Volume의 자동 삭제를 의미하지 않습니다.** 반면 `docker volume rm codyssey-data` 등으로 Volume 자체를 삭제하면 그 안의 데이터가 사라질 수 있으므로 원본 Volume을 바로 지우지 않습니다. 백업 시에는 컨테이너에서 Volume을 읽기 전용으로 연결해 `tar`로 파일을 묶어 호스트의 별도 백업 경로에 저장할 수 있고, 복원 테스트는 새 Volume에 압축을 풀어 원본을 보호하는 방식이 좋습니다. 중요한 데이터라면 다른 디스크나 외부 백업 저장소에도 복사해야 합니다. 이 백업·복원은 **권장 절차에 대한 설명**이며 이 미션에서 실제 복원을 검증했다는 주장은 아닙니다.

### 11.7 작업 폴더의 역할과 Git 변경 확인

| 경로 | 역할 |
|---|---|
| `Dockerfile` | Nginx 기반 이미지 빌드 정의 |
| `site/` | 웹 서버가 제공할 HTML 원본 |
| `practice/` | Linux 파일과 권한 실습 결과 |
| `evidence/logs/` | 실행한 명령어와 터미널 출력 증거 |
| `evidence/screenshots/` | 브라우저·VS Code·GitHub 화면 증거 |
| `README.md` | 재현 방법, 실습 결과, 트러블슈팅 설명 |

최종 업로드는 `git commit -m "Complete Codyssey pre-course submission"` 후 `git push origin main`으로 수행했고, 터미널에서 `8a6c669..77935cc main -> main`이 출력되었습니다. 이는 당시의 Push 성공 기록입니다. GitHub의 최신 커밋 또는 `git log --oneline -1`과 `git ls-remote origin refs/heads/main`으로 로컬·원격 커밋을 추가 대조할 수 있습니다. 인증 토큰, 비밀번호, 개인 키는 증거에 포함하지 않습니다.

보완 검증 결과는 별도 [`12_supplementary_checks.txt`](evidence/logs/12_supplementary_checks.txt)에 추가합니다.
