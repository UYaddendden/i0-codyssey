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
