# 🚀 Docker & Development Environment Setup Mission

개발 워크스테이션 구축 및 Docker 핵심 기능 실습 결과 보고서입니다. 터미널 환경 세팅부터 Docker 컨테이너 기반 웹 서버 배포, 데이터 영속성 검증까지의 전 과정을 명령어와 출력 로그로 기록했습니다.

---

## 📑 목차

1. [프로젝트 개요](#1--프로젝트-개요)
2. [실행 환경](#2--실행-환경)
3. [수행 항목 체크리스트](#3--수행-항목-체크리스트)
4. [프로젝트 디렉토리 구조](#4--프로젝트-디렉토리-구조)
5. [수행 로그 및 검증 방법](#5--수행-로그-및-검증-방법)
6. [핵심 개념 정리](#6--핵심-개념-정리)
7. [트러블슈팅](#7--트러블슈팅)
8. [Git 설정 및 GitHub 연동](#8--git-설정-및-github-연동)
9. [참고 링크](#9--참고-링크)

---

## 1. 📋 프로젝트 개요

- **미션 목표:** 개발 환경의 재현성 확보, Docker 컨테이너의 격리성 이해, 커스텀 이미지 빌드부터 볼륨 영속성 검증까지의 엔지니어링 흐름 체득
- **핵심 수행 내용:**
  1. 리눅스 CLI 환경 조작 및 파일/디렉토리 권한 제어 실습 (`chmod`)
  2. Docker 기본 운영 및 컨테이너 생명주기 관리 (`run`, `stop`, `rm`, `exec` 등)
  3. `Dockerfile`과 Nginx를 활용한 커스텀 웹 서버 이미지 빌드 및 포트 매핑 배포
  4. 바인드 마운트와 Docker Volume을 통한 "변경 사항 실시간 반영" 및 "데이터 영속성" 검증
  5. Git 사용자 설정 및 GitHub 저장소 연동

---

## 2. 💻 실행 환경

| 항목 | 내용 |
|------|------|
| OS | Windows 11 (WSL2 backend) |
| Shell | PowerShell, Git Bash(UCRT64), Ubuntu 컨테이너 내부 bash |
| Docker Version | 29.9.2 (Docker Desktop 4.99.0) |
| Git Version | 📸 `git --version` 결과를 여기에 입력 (예: git version 2.xx.x.windows.x) |

---

## 3. ✅ 수행 항목 체크리스트

- [x] 터미널 기본 조작 및 폴더 구성 (`pwd`, `ls -la`, `mkdir`, `cd`, `cp`, `mv`, `rm`, `cat`) → [5-3](#5-3-터미널-기본-조작)
- [x] 권한 변경 실습 (`chmod 600`, `chmod 700`) → [5-4](#5-4-권한-실습)
- [x] Docker 설치 및 데몬 점검 (`docker --version`, `docker info`) → [5-1](#5-1-docker-설치-및-데몬-점검)
- [x] hello-world 컨테이너 실행 → [5-2](#5-2-hello-world-실행)
- [x] 우분투 컨테이너 진입 및 제어 (`docker run -it`, `exit` vs `exec`) → [5-5](#5-5-컨테이너-생명주기-관리)
- [x] Docker 운영 명령 (`images`, `ps -a`, `logs`, `stats`) → [5-5](#5-5-컨테이너-생명주기-관리)
- [x] Dockerfile 기반 커스텀 웹 서버 이미지 빌드 및 실행 → [5-6](#5-6-커스텀-웹-서버-이미지-빌드-및-실행)
- [x] 포트 매핑 및 브라우저 접속 확인 (2회) → [5-7](#5-7-포트-매핑-및-접속-확인)
- [x] 바인드 마운트 반영 테스트 → [5-8](#5-8-바인드-마운트)
- [x] Docker Volume 생성, 연결 및 데이터 영속성 검증 → [5-9](#5-9-docker-volume-영속성-검증)
- [x] Git 설정 및 GitHub 연동 → [8장](#8--git-설정-및-github-연동)

---

## 4. 🗂️ 프로젝트 디렉토리 구조

```text
my-web-folder/            # 프로젝트 루트 (docker build / run 을 실행하는 작업 디렉토리)
├── Dockerfile            # Nginx 기반 커스텀 이미지 빌드 정의
├── index.html            # 컨테이너에 복사되어 서비스되는 웹 페이지
├── images/               # README에 사용한 증거 이미지
└── README.md             # 본 보고서
```

| 경로 | 역할 |
|------|------|
| `Dockerfile` | 베이스 이미지(`nginx`)와 복사할 파일을 정의하는 이미지 빌드 설계도 |
| `index.html` | Nginx 기본 페이지를 대체하는 사용자 정의 웹 페이지 |
| `README.md` | 수행 로그, 개념 설명, 트러블슈팅 기록 |

---

## 5. 🛠️ 수행 로그 및 검증 방법

### 5-1. Docker 설치 및 데몬 점검

```powershell
PS C:\WINDOWS\system32> docker --version
Docker version 29.9.2, build a82e528
```

```powershell
PS C:\WINDOWS\system32> docker info
Client:
 Version:    29.9.2
 Context:    desktop-linux
 Debug Mode: false
 (Plugins 목록 생략: buildx, compose, scout 등 Docker Desktop 기본 포함)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 29.9.2
 Storage Driver: overlayfs
  driver-type: io.containerd.snapshotter.v1
 Logging Driver: local
 Cgroup Driver: cgroupfs
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 nvidia runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b7a25261e029d68978bbf290df0a62fc819b3c51
 runc version: v1.5.2-0-g29dd3dc2
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.18.54.1-microsoft-standard-WSL2
 Operating System: Docker Desktop
```

`Server:` 항목이 정상 출력되므로 Docker 데몬이 동작 중임을 확인했습니다.

<img width="402" height="45" alt="docker version" src="https://github.com/user-attachments/assets/acac91fb-20f6-44bb-a6c3-e703a1c33929" />
<img width="531" height="561" alt="docker info" src="https://github.com/user-attachments/assets/3895911a-2b1c-4ffd-a73b-e3f89fa53414" />

### 5-2. hello-world 실행

```powershell
PS C:\WINDOWS\system32> docker run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

<img width="646" height="450" alt="hello-world" src="https://github.com/user-attachments/assets/c176a622-0445-4c7f-816d-2fdbc7a84aea" />

### 5-3. 터미널 기본 조작

Ubuntu 컨테이너 내부 bash에서 수행했습니다.

```bash
# 현재 위치 확인 / 목록 확인(숨김 파일 포함)
pwd
ls -la

# 생성 / 이동
mkdir -p practice
cd practice

# 파일 생성 및 내용 확인
touch empty.txt
echo hello > a.txt
cat a.txt

# 복사 / 이동(이름 변경) / 삭제
cp a.txt b.txt
mv b.txt c.txt
ls -la
rm c.txt
ls -la
```

📸 위 명령을 실행한 **터미널 출력 캡처**를 여기에 붙여넣으세요. (명령어 입력과 출력이 함께 보이게)

### 5-4. 권한 실습

```bash
root@d35aab0d3a21:/# mkdir temp
root@d35aab0d3a21:/# cd temp
root@d35aab0d3a21:/temp# touch file.txt
root@d35aab0d3a21:/temp# mkdir mydir

# ▼ 변경 전
root@d35aab0d3a21:/temp# ls -l
```

📸 **변경 전 `ls -l` 출력**을 여기에 붙여넣으세요. (기본 권한: 파일 `-rw-r--r--`, 디렉토리 `drwxr-xr-x` 형태)

```bash
# ▼ 권한 변경
root@d35aab0d3a21:/temp# chmod 600 file.txt      # 소유자 읽기/쓰기 (-rw-------)
root@d35aab0d3a21:/temp# chmod 700 mydir         # 소유자 모든 권한 (drwx------)

# ▼ 변경 후
root@d35aab0d3a21:/temp# ls -l
total 4
-rw------- 1 root root    0 Nov  9 10:37 file.txt
drwx------ 2 root root 4096 Nov  9 11:47 mydir
```

<img width="835" height="302" alt="chmod before after" src="https://github.com/user-attachments/assets/13c3faae-2d99-4810-9c8f-0eb33b188204" />
<img width="383" height="173" alt="chmod result" src="https://github.com/user-attachments/assets/78ee10cf-5b3c-40ab-86eb-b240c6c91864" />

#### 권한 숫자 표기법 요약

| 숫자 | 2진수 | rwx | 의미 |
|:---:|:---:|:---:|------|
| 0 | 000 | `---` | 권한 없음 |
| 1 | 001 | `--x` | 실행 |
| 2 | 010 | `-w-` | 쓰기 |
| 4 | 100 | `r--` | 읽기 |
| 5 | 101 | `r-x` | 읽기 + 실행 |
| 6 | 110 | `rw-` | 읽기 + 쓰기 |
| 7 | 111 | `rwx` | 읽기 + 쓰기 + 실행 |

- 세 자리는 순서대로 **소유자 / 그룹 / 기타 사용자**입니다. 각 자리는 r=4, w=2, x=1을 더한 값입니다.
- `600` = 소유자 `rw-`, 그룹 `---`, 기타 `---` → 소유자만 읽고 쓸 수 있음
- `700` = 소유자 `rwx`, 그룹 `---`, 기타 `---` → 소유자만 디렉토리에 진입/조회/생성 가능
- `755` = `rwxr-xr-x` → 소유자는 전체 권한, 그룹·기타는 읽기+실행 (실행 파일, 디렉토리의 일반적인 권한)
- `644` = `rw-r--r--` → 소유자는 읽기+쓰기, 그룹·기타는 읽기만 (일반 파일의 기본 권한)

#### 권한 변경이 시스템 동작에 미치는 영향

- `chmod 600`을 건 파일은 다른 사용자가 읽을 수 없으므로, SSH 개인키나 비밀번호 파일처럼 **민감한 파일 보호**에 사용됩니다. (실제로 SSH는 개인키 권한이 느슨하면 접속을 거부합니다.)
- 디렉토리에 `x` 권한이 없으면 `cd` 와 내부 파일 접근이 불가능하므로, `700`은 해당 디렉토리를 소유자 전용 공간으로 만듭니다.
- 소유자·그룹 변경은 `chown user:group file.txt` 로 하며, 예를 들어 웹 서버 계정(`www-data`)이 파일을 읽어야 할 때 소유자나 그룹을 맞춰 줘야 합니다. (본 실습은 `root` 단일 계정이라 `chown` 은 수행하지 않음)

#### 절대 경로 vs 상대 경로

- **절대 경로:** 루트(`/` 또는 `C:\`)부터 시작하는 전체 경로입니다. 어디서 실행해도 같은 위치를 가리킵니다.
  예: `/temp/file.txt`, `C:\Users\User\Desktop\my-web-folder`
- **상대 경로:** 현재 위치를 기준으로 한 경로입니다. 현재 위치가 바뀌면 가리키는 대상도 달라집니다.
  예: `./file.txt`(현재 폴더), `../index.html`(상위 폴더)

<img width="332" height="223" alt="absolute relative path" src="https://github.com/user-attachments/assets/c58c4004-60d2-40c1-8107-06e6b90a8d2a" />

### 5-5. 컨테이너 생명주기 관리

**ubuntu 컨테이너 실행 및 내부 명령**

```powershell
PS C:\WINDOWS\system32> docker run -it --name mybox ubuntu
root@d35aab0d3a21:/# ls
root@d35aab0d3a21:/# echo "hello ubuntu"
hello ubuntu
root@d35aab0d3a21:/# exit
```

📸 `ls` 출력 결과 캡처를 여기에 붙여넣으세요.

**컨테이너 상태 관찰**

```powershell
# 멈춘(Exited) 상태 확인
PS C:\WINDOWS\system32> docker ps -a
CONTAINER ID   IMAGE    COMMAND       CREATED       STATUS                   PORTS   NAMES
d35aab0d3a21   ubuntu   "/bin/bash"   2 hours ago   Exited (137) 14 secs ago         mybox

# 다시 시작 후 exec 접속 → 나가도 'Up' 상태 유지
PS C:\WINDOWS\system32> docker start mybox
PS C:\WINDOWS\system32> docker exec -it mybox bash
root@d35aab0d3a21:/# exit

PS C:\WINDOWS\system32> docker ps
CONTAINER ID   IMAGE    COMMAND       CREATED       STATUS          PORTS   NAMES
d35aab0d3a21   ubuntu   "/bin/bash"   2 hours ago   Up 25 seconds           mybox
```

> `Exited (137)` 은 컨테이너가 외부에서 강제 종료(SIGKILL)되었을 때의 종료 코드입니다. `exit` 로 정상 종료하면 보통 `Exited (0)` 이 표시됩니다.

**이미지 목록**

```powershell
PS C:\WINDOWS\system32> docker images
```

<img width="905" height="82" alt="docker images" src="https://github.com/user-attachments/assets/803467ad-b74f-4a8d-ad22-7c2f0ac88790" />

**로그 / 리소스 확인**

```powershell
PS C:\WINDOWS\system32> docker logs mybox
PS C:\WINDOWS\system32> docker stats --no-stream
```

📸 `docker logs`, `docker stats --no-stream` **출력 캡처**를 여기에 붙여넣으세요.

**`run -it` 와 `exec -it` 의 차이**

| 방식 | 종료(`exit`) 시 컨테이너 상태 | 이유 |
|------|------------------------------|------|
| `docker run -it ubuntu` | **Exited** | 이 bash가 컨테이너의 메인 프로세스(PID 1)라서, `exit` 하면 컨테이너도 종료됨 |
| `docker exec -it mybox bash` | **Up 유지** | 이미 실행 중인 컨테이너에 *추가* 프로세스를 붙인 것이라, 그 프로세스만 종료됨 |

### 5-6. 커스텀 웹 서버 이미지 빌드 및 실행

- **선택한 방식:** (A) 웹 서버 베이스 이미지 활용 + 정적 콘텐츠 교체
- **베이스 이미지:** `nginx` (Docker Hub 공식 이미지)

**Dockerfile**

```dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
```

| 커스텀 포인트 | 목적 |
|---|---|
| `FROM nginx` | 검증된 웹 서버 이미지를 베이스로 사용해 직접 서버를 설치하는 수고를 줄임 |
| `COPY index.html ...` | Nginx 기본 환영 페이지를 내가 만든 페이지로 교체 |

**빌드 및 실행**

```powershell
PS C:\Users\User\Desktop\my-web-folder> docker build -t my-web .
PS C:\Users\User\Desktop\my-web-folder> docker run -d -p 1205:80 --name my-web my-web
PS C:\Users\User\Desktop\my-web-folder> docker ps
```

📸 위 3개 명령의 **실제 출력 캡처**를 여기에 붙여넣으세요. (빌드 성공 메시지, 컨테이너 ID, `docker ps`의 `0.0.0.0:1205->80/tcp`가 보이게)

### 5-7. 포트 매핑 및 접속 확인

**1회차: 호스트 1205 → 컨테이너 80**

```powershell
PS C:\Users\User\Desktop\my-web-folder> curl.exe http://localhost:1205
```

📸 `curl.exe` **실제 출력**을 여기에 붙여넣으세요.

<img width="425" height="228" alt="browser 1205" src="https://github.com/user-attachments/assets/8d5fe6b0-58b6-4daf-a1f6-3c9e1232d7b7" />

**2회차: 같은 이미지로 다른 포트(1207)에 하나 더 실행**

```powershell
PS C:\Users\User\Desktop\my-web-folder> docker run -d -p 1207:80 --name my-web-2 my-web
PS C:\Users\User\Desktop\my-web-folder> curl.exe http://localhost:1207
```

📸 위 명령의 **실제 출력**과 브라우저 접속 화면(**주소창에 `localhost:1207` 보이게**)을 여기에 붙여넣으세요.

### 5-8. 바인드 마운트

호스트 폴더를 컨테이너에 연결해, 호스트에서 `index.html` 을 수정하면 **이미지를 다시 빌드하지 않아도** 즉시 반영되는지 확인했습니다.

```powershell
PS C:\Users\User\Desktop\my-web-folder> docker run -d -p 1206:80 --name my-web-bind `
    -v ${PWD}:/usr/share/nginx/html nginx

# ▼ 변경 전
PS C:\Users\User\Desktop\my-web-folder> curl.exe http://localhost:1206

# (호스트에서 index.html 내용을 수정하고 저장)

# ▼ 변경 후 (재빌드·재시작 없이)
PS C:\Users\User\Desktop\my-web-folder> curl.exe http://localhost:1206
```

📸 **변경 전 / 변경 후 `curl.exe` 출력**(또는 브라우저 화면 2장)을 여기에 붙여넣으세요. 내용이 달라져 있어야 합니다.

→ 본 실습은 재현성을 위해 `${PWD}` 를 사용했고, 프로젝트 루트에서 실행하는 것을 전제로 합니다. (PowerShell 기준이며 Git Bash/Linux에서는 `$(pwd)` 사용)

### 5-9. Docker Volume 영속성 검증

> **볼륨 `mydata` 의 목적:** 컨테이너의 `/data` 경로에 저장한 파일이 **컨테이너를 삭제해도 남는지** 검증하기 위한 데이터 저장 공간입니다. 볼륨은 Docker가 관리하는 호스트 영역에 저장되며, 컨테이너 수명과 분리됩니다. 따라서 `vol-test` 를 삭제해도 `memo.txt` 가 유지될 것으로 예상했습니다.

```powershell
# 1. 볼륨 생성
PS C:\Users\User\Desktop\my-web-folder> docker volume create mydata
mydata

# 2. 볼륨을 연결한 우분투 컨테이너 실행 후 파일 생성
PS C:\Users\User\Desktop\my-web-folder> docker run -it --name vol-test -v mydata:/data ubuntu
root@cec8f565b68b:/# echo "hello volume" > /data/memo.txt
root@cec8f565b68b:/# cat /data/memo.txt
root@cec8f565b68b:/# exit

# 3. 컨테이너 삭제 후 새 컨테이너로 볼륨 연결하여 데이터 확인
PS C:\Users\User\Desktop\my-web-folder> docker rm vol-test
vol-test
PS C:\Users\User\Desktop\my-web-folder> docker run -it --rm -v mydata:/data ubuntu cat /data/memo.txt
hello volume
```

📸 **삭제 전 `cat /data/memo.txt` 출력**(위 2번의 `cat` 결과)도 캡처해서 여기에 붙여넣으세요. 삭제 전과 삭제 후 출력이 모두 `hello volume` 이면 영속성이 증명됩니다.

#### 볼륨 백업·복원 방법

볼륨 내용을 tar 파일로 백업하고, 필요 시 새 볼륨에 복원할 수 있습니다.

```powershell
# 백업: mydata 볼륨 → 현재 폴더의 mydata-backup.tar.gz
docker run --rm -v mydata:/data -v ${PWD}:/backup ubuntu tar czf /backup/mydata-backup.tar.gz -C /data .

# 복원: 새 볼륨 mydata2 로 복원
docker volume create mydata2
docker run --rm -v mydata2:/data -v ${PWD}:/backup ubuntu tar xzf /backup/mydata-backup.tar.gz -C /data
```

📸 백업 실행 후 `dir`(또는 `ls`)로 `mydata-backup.tar.gz` 가 생성된 **캡처**를 여기에 붙여넣으세요. (이 부분은 선택 사항이므로 수행하지 않았다면 위 백업·복원 코드블록과 이 문장을 삭제하세요.)

---

## 6. 📚 핵심 개념 정리

### 6-1. 이미지(Image)와 컨테이너(Container)의 차이

| 구분 | 이미지 | 컨테이너 |
|------|--------|----------|
| 정의 | 실행에 필요한 파일·설정이 담긴 **읽기 전용 템플릿** | 이미지를 기반으로 실행된 **프로세스 인스턴스** |
| 생성 방법 | `docker build` | `docker run` |
| 변경 가능 여부 | 불변 (변경하려면 새로 빌드) | 실행 중 쓰기 가능 (삭제하면 변경분 사라짐) |
| 비유 | 클래스 / 설치 파일 | 객체 / 실행 중인 프로그램 |
| 본 실습 예 | `my-web` 이미지 (Nginx + index.html) | `my-web`, `my-web-2` 컨테이너 |

**사례:** `my-web` 이미지 하나로 `docker run -p 1205:80`, `docker run -p 1207:80` 처럼 컨테이너를 여러 개 띄울 수 있고, 한 컨테이너를 삭제해도 이미지는 그대로 남습니다.

### 6-2. 포트 포워딩이 동작하는 이유

컨테이너는 **별도의 네트워크 네임스페이스**를 가지므로 자체 IP와 포트 공간을 갖고, 호스트에서 컨테이너 내부 포트로 직접 접근할 수 없습니다. `-p 1205:80` 옵션은 Docker가 호스트의 `1205` 포트로 들어온 트래픽을 컨테이너의 `80` 포트로 전달(NAT/DNAT)하도록 규칙을 만들어 주기 때문에 `localhost:1205` 로 컨테이너 안의 Nginx에 접속할 수 있습니다. `ps` 출력의 `0.0.0.0:1205->80` 은 "모든 호스트 인터페이스의 1205 → 컨테이너 80" 이라는 뜻입니다.

### 6-3. 바인드 마운트 vs Docker Volume

| 구분 | 바인드 마운트 | Docker Volume |
|------|---------------|---------------|
| 저장 위치 | 호스트의 지정 경로 | Docker가 관리하는 영역 |
| 주 용도 | 개발 중 소스 **실시간 반영** | 데이터 **영속 저장** (DB 등) |
| 호스트 경로 의존성 | 높음 | 낮음 |
| 백업 | 일반 파일 복사 | tar 백업 등 별도 절차 ([5-9](#5-9-docker-volume-영속성-검증) 참고) |

### 6-4. Git과 GitHub의 역할 차이

- **Git:** 내 컴퓨터에서 파일 변경 이력을 기록·관리하는 **로컬 버전 관리 도구**입니다. (`git init`, `commit`, `log`)
- **GitHub:** Git 저장소를 인터넷에 올려 두고 공유·협업(push/pull, 이슈, PR)할 수 있게 해주는 **원격 협업 플랫폼**입니다.

### 6-5. 포트 충돌 진단 및 해결

`docker run -p 1205:80` 실행 시 `port is already allocated` 또는 `address already in use` 오류가 나면 해당 호스트 포트를 이미 다른 프로세스/컨테이너가 사용 중이라는 뜻입니다.

**① 사용 중인 컨테이너 확인**

```powershell
docker ps --filter "publish=1205"
```

**② 사용 중인 프로세스 확인**

```powershell
# Windows (PowerShell)
netstat -ano | findstr :1205
# 마지막 열의 PID 확인 후
tasklist | findstr <PID>
```

```bash
# Linux / macOS / WSL
sudo lsof -i :1205
sudo ss -ltnp | grep 1205
```

**③ 포트 해제**

```powershell
docker stop my-web && docker rm my-web      # 컨테이너가 원인인 경우
taskkill /PID <PID> /F                      # Windows 프로세스가 원인인 경우
```

```bash
kill <PID>                                  # Linux / macOS / WSL
```

**④ 우회:** 포트를 바꿔 실행 (예: `-p 1208:80`)

---

## 7. 🔧 트러블슈팅

### 7-1. Dockerfile 빌드 시 경로 에러 (`no such file or directory`)

- **문제 상황:** 시스템 경로(`PS C:\WINDOWS\system32>`)에서 `docker build` 를 실행했을 때 Dockerfile을 찾지 못함
- **원인 가설:** 현재 터미널의 작업 디렉토리가 Dockerfile이 있는 폴더가 아님
- **확인:** PowerShell 프롬프트의 현재 경로가 `C:\WINDOWS\system32` 였고, `dir` 로 확인하니 Dockerfile이 없었음
- **해결 방법:** `cd C:\Users\User\Desktop\my-web-folder` 로 이동한 뒤 다시 빌드하여 성공

```text
ERROR: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
```

### 7-2. 붙여넣기 시 특수문자 에러 (`$'\E[200~'`)

- **문제 상황:** 컨테이너 터미널에 명령어를 붙여넣을 때 특수문자가 섞여 명령어 인식 실패
- **원인 가설:** 클립보드 붙여넣기 모드(bracketed paste)와 터미널 간의 입력 충돌
- **확인:** 같은 명령을 직접 타이핑하니 정상 동작함 → 붙여넣기 방식이 원인임을 확인
- **해결 방법:** 명령어를 직접 타이핑하거나 텍스트를 정돈하여 입력

```text
bash: $'\E[200~mkdir': command not found
```

---

## 8. 🌐 Git 설정 및 GitHub 연동

### 8-1. 사용자 정보 설정

```bash
git config --global user.name "노 연 주"
git config --global user.email "aaaplay00001@gmail.com"
git config --global init.defaultbranch main
git config --global --list
```

```text
user.name=노 연 주
user.email=aaaplay00001@gmail.com
init.defaultbranch=main
```

<img width="238" height="42" alt="git config" src="https://github.com/user-attachments/assets/ee3b885f-9f1d-4aaf-8496-a8f560d582dd" />

### 8-2. 저장소 초기화

```bash
User@5F-105 UCRT64 ~
$ cd "C:\Users\User\Desktop\my-web-folder"

User@5F-105 UCRT64 ~/Desktop/my-web-folder
$ git init
Initialized empty Git repository in C:/Users/User/Desktop/my-web-folder/.git/
```

### 8-3. 원격 저장소 확인 (`git remote -v`)

```bash
User@5F-105 UCRT64 ~/Desktop/my-web-folder (main)
$ git remote -v
origin  https://github.com/aaaplay00001-beep/codyssey.git (fetch)
origin  https://github.com/aaaplay00001-beep/codyssey.git (push)
```

### 8-4. push 기록 (`git push`)

```bash
User@5F-105 UCRT64 ~/Desktop/my-web-folder (main)
$ git push -u origin main
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 911 bytes | 911.00 KiB/s, done.
Total 6 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/aaaplay00001-beep/codyssey.git
   fa660d8..5bab532  main -> main
branch 'main' set up to track 'origin/main'.
```

### 8-5. 커밋 이력 (`git log --oneline`)

```bash
User@5F-105 UCRT64 ~/Desktop/my-web-folder (main)
$ git log --oneline -5
5bab532 (HEAD -> main, origin/main) Merge branch 'main' of https://github.com/aaaplay00001-beep/codyssey
f2cfee8 docs: Docker 실습 README 및 웹 서버 파일 추가
fa660d8 Update Docker info and replace image in README
42b1825 Update README.md
792a294 Update README.md
```

- `HEAD -> main, origin/main` 이 같은 커밋(`5bab532`)을 가리키므로 로컬과 원격이 동기화된 상태입니다.
- 최상단 `Merge branch 'main'` 커밋은 GitHub 웹에서 먼저 수정한 README(`fa660d8`)와 로컬 커밋(`f2cfee8`)을 합친 병합 커밋입니다.

### 8-6. VSCode GitHub 로그인 및 저장소 연동

📸 VSCode에서 GitHub 계정으로 로그인된 화면(좌측 하단 계정 아이콘)과, `codyssey` 저장소가 열려 있고 소스 제어(Source Control) 탭이 보이는 **스크린샷**을 여기에 붙여넣으세요. (토큰, 인증 코드는 가리기)

### 8-7. GitHub 저장소

- **저장소 링크:** https://github.com/aaaplay00001-beep/codyssey

---

## 9. 🔗 참고 링크

- 노션 정리: https://pingmong.notion.site/codyssey
- WSL 릴리스: https://github.com/microsoft/WSL/releases/tag/3.0.3
