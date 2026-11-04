# Codyssey Pre-Course

## 1. 프로젝트 소개

본 저장소는 코디세이 사전과정에서 학습한 Linux, Docker, Git 및 GitHub 실습 내용을 정리하기 위한 저장소입니다.

## 2. 개발 환경

- OS: Windows + WSL2 Ubuntu
- Container: Docker Desktop
- Version Control: Git
- Remote Repository: GitHub

## 3. 프로젝트 구조

```text
pre-course/
├── Dockerfile
├── practice/
│   ├── hello.txt
│   └── permission.txt
├── site/
│   └── index.html
└── README.md
```

## 4. 실습 내용

### Linux 기본 명령어
- 디렉터리 및 파일 생성과 관리
- 파일 복사, 이동, 삭제
- chmod를 활용한 파일 접근 권한 변경

### Docker
- Docker 이미지와 컨테이너의 개념 학습
- Nginx 웹 서버 컨테이너 실행
- Dockerfile을 이용한 사용자 정의 이미지 빌드
- Bind Mount를 통한 실시간 파일 변경 반영
- Docker Volume을 통한 데이터 영속성 확인

### Git 및 GitHub
- 로컬 Git 저장소 생성
- 파일 스테이징 및 커밋
- GitHub 원격 저장소 연결
- 서로 다른 Git 커밋 이력 병합
- GitHub Push를 통한 코드 업로드

## 5. 트러블슈팅

### GitHub 인증 오류
- 문제: Git Push 과정에서 `Invalid username or token` 오류 발생
- 원인 분석: GitHub 비밀번호 인증 미지원 및 인증 정보 확인 필요
- 해결: Personal Access Token(Classic)을 사용해 인증 성공

### Git Push 거부
- 문제: `main -> main (fetch first)` 오류 발생
- 원인: 로컬 저장소와 GitHub 저장소에 서로 다른 초기 커밋 존재
- 해결: `git fetch`로 원격 이력을 확인하고, `git pull --allow-unrelated-histories --no-rebase`로 병합한 뒤 Push 성공

## 6. 학습 결과

Linux 명령어를 활용한 파일 관리, Docker 컨테이너 실행과 데이터 관리, Git을 이용한 버전 관리 및 GitHub 연동 과정을 실습하였습니다.
