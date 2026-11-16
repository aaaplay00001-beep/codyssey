# 🚀 Docker & Development Environment Setup Mission

개발 워크스테이션 구축 및 Docker 핵심 기능 실습 결과 보고서입니다. 터미널 환경 세팅부터 Docker 컨테이너 기반 웹 서버 배포, 데이터 영속성 검증까지의 전 과정을 로그와 설명으로 기록했습니다.

> ⚠️ **작성 안내:** `✏️ 여기에 실제 출력 붙여넣기` 로 표시된 부분은 본인 PC에서 명령을 직접 실행한 뒤 **실제 출력 또는 스크린샷**으로 교체해야 합니다. 평가에서 FAIL이 난 항목은 대부분 "실행 증거"가 없었기 때문입니다.

---

## 📑 목차

1. [프로젝트 개요](#1--프로젝트-개요)
2. [실행 환경](#2--실행-환경)
3. [수행 항목 체크리스트](#3--수행-항목-체크리스트)
4. [프로젝트 디렉토리 구조](#4--프로젝트-디렉토리-구조)
5. [수행 로그 및 검증 방법](#5--수행-로그-및-검증-방법)
   - [5-1. Docker 설치 및 데몬 점검](#5-1-docker-설치-및-데몬-점검)
   - [5-2. hello-world 실행](#5-2-hello-world-실행)
   - [5-3. 터미널 및 권한 실습](#5-3-터미널-및-권한-실습)
   - [5-4. 컨테이너 생명주기 관리](#5-4-컨테이너-생명주기-관리)
   - [5-5. 커스텀 웹 서버 이미지 빌드 및 실행](#5-5-커스텀-웹-서버-이미지-빌드-및-실행)
   - [5-6. 포트 매핑 및 접속 확인](#5-6-포트-매핑-및-접속-확인)
   - [5-7. 바인드 마운트](#5-7-바인드-마운트)
   - [5-8. Docker Volume 영속성 검증](#5-8-docker-volume-영속성-검증)
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

---

## 2. 💻 실행 환경

| 항목 | 내용 |
|------|------|
| OS | Windows 11 (PowerShell / WSL2 backend) |
| Shell | PowerShell (호스트), Bash (Ubuntu 컨테이너 내부) |
| Docker Version | 27.x 이상 (Docker Desktop for Windows) |
| Git Version | 최신 버전 |

---

## 3. ✅ 수행 항목 체크리스트

- [x] 터미널 기본 조작 및 폴더 구성 (`mkdir`, `cd`, `ls`)
- [x] 권한 변경 실습 (`chmod 600`, `chmod 700`)
- [x] Docker 설치 및 데몬 점검 (`docker --version`, `docker info`)
- [x] hello-world 컨테이너 실행
- [x] 우분투 컨테이너 진입 및 제어 (`docker run -it`, `exit` vs `exec`)
- [x] Dockerfile 기반 커스텀 웹 서버 이미지 빌드 및 실행 (`docker build`, `docker run -d -p`)
- [x] 포트 매핑 및 브라우저 접속 확인 (`localhost:1205`)
- [x] 바인드 마운트 반영 테스트 (`-v ${PWD}:...`)
- [x] Docker Volume 생성, 연결 및 데이터 영속성 검증 (`docker volume create`)
- [x] Git 설정 및 GitHub 연동

---

## 4. 🗂️ 프로젝트 디렉토리 구조

```text
my-web-folder/            # 프로젝트 루트 (docker build / run 을 실행하는 작업 디렉토리)
├── Dockerfile            # Nginx 기반 커스텀 이미지 빌드 정의
├── index.html            # 컨테이너에 복사되어 서비스되는 웹 페이지
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

PS C:\WINDOWS\system32> docker info
```

<img width="402" height="45" alt="image" src="https://github.com/user-attachments/assets/acac91fb-20f6-44bb-a6c3-e703a1c33929" />

> 데몬이 꺼져 있으면 `docker info` 의 Server 항목에서 연결 오류가 나므로, 이 명령이 데몬 상태의 증거가 됩니다.

### 5-2. hello-world 실행

```powershell
PS C:\WINDOWS\system32> docker run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

<img width="646" height="450" alt="image" src="https://github.com/user-attachments/assets/c176a622-0445-4c7f-816d-2fdbc7a84aea" />

### 5-3. 터미널 및 권한 실습

```bash
root@d35aab0d3a21:/# mkdir temp
root@d35aab0d3a21:/# cd temp
root@d35aab0d3a21:/temp# touch file.txt
root@d35aab0d3a21:/temp# mkdir mydir

# ▼ 변경 전
root@d35aab0d3a21:/temp# ls -l


# ▼ 권한 변경
root@d35aab0d3a21:/temp# chmod 600 file.txt      # 소유자 읽기/쓰기 (-rw-------)
root@d35aab0d3a21:/temp# chmod 700 mydir         # 소유자 모든 권한 (drwx------)

# ▼ 변경 후
root@d35aab0d3a21:/temp# ls -l
total 4
-rw------- 1 root root    0 Nov  9 10:37 file.txt
drwx------ 2 root root 4096 Nov  9 11:47 mydir
```
<img width="835" height="302" alt="image" src="https://github.com/user-attachments/assets/13c3faae-2d99-4810-9c8f-0eb33b188204" />

<img width="383" height="173" alt="image" src="https://github.com/user-attachments/assets/78ee10cf-5b3c-40ab-86eb-b240c6c91864" />


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

- 세 자리는 순서대로 **소유자 / 그룹 / 기타 사용자**입니다.
- `600` = 소유자 `rw-`, 그룹 `---`, 기타 `---` → 소유자만 읽고 쓸 수 있음
- `700` = 소유자 `rwx`, 그룹 `---`, 기타 `---` → 소유자만 디렉토리에 진입/조회/생성 가능

#### 권한 변경이 시스템 동작에 미치는 영향

- `chmod 600`을 건 파일은 다른 사용자가 읽을 수 없으므로, SSH 개인키나 비밀번호 파일처럼 **민감한 파일 보호**에 사용됩니다. (실제로 SSH는 개인키 권한이 느슨하면 접속을 거부합니다.)
- 디렉토리에 `x` 권한이 없으면 `cd` 와 내부 파일 접근이 불가능하므로, `700`은 해당 디렉토리를 소유자 전용 공간으로 만듭니다.
- 소유자·그룹 변경은 `chown user:group file.txt` 로 하며, 예를 들어 웹 서버 계정(`www-data`)이 파일을 읽어야 할 때 소유자나 그룹을 맞춰 줘야 합니다. (본 실습은 `root` 단일 계정이라 `chown` 은 수행하지 않음)

### 5-4. 컨테이너 생명주기 관리

```powershell
# exit 로 나간 후 멈춘(Exited) 상태 확인
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

# 이미지 목록 확인
PS C:\WINDOWS\system32> docker images
<img width="905" height="82" alt="image" src="https://github.com/user-attachments/assets/803467ad-b74f-4a8d-ad22-7c2f0ac88790" />

```

| 방식 | 종료(`exit`) 시 컨테이너 상태 | 이유 |
|------|------------------------------|------|
| `docker run -it ubuntu` | **Exited** | `exit` 가 컨테이너의 메인 프로세스(PID 1, bash)를 종료시킴 |
| `docker exec -it mybox bash` | **Up 유지** | 이미 실행 중인 컨테이너에 *추가* 프로세스를 붙인 것이라, 그 프로세스만 종료됨 |

> 📝 `Exited (137)` 은 프로세스가 강제 종료(SIGKILL)되었을 때의 종료 코드입니다. 정상 `exit` 라면 보통 `Exited (0)` 이므로, 이 로그는 `docker stop` 이나 Docker Desktop 재시작 등으로 종료된 결과일 수 있습니다. 실제 수행 상황에 맞게 설명을 수정하세요.

### 5-5. 커스텀 웹 서버 이미지 빌드 및 실행

**Dockerfile**

```dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
```

**빌드 및 실행**

```powershell
PS C:\Users\User\Desktop\my-web-folder> docker build -t my-web .
[+] Building 0.1s (1/1) FINISHED ...

PS C:\Users\User\Desktop\my-web-folder> docker images my-web
REPOSITORY   TAG      IMAGE ID       CREATED          SIZE
my-web       latest   ✏️ 실제 ID     ✏️ 실제 값        ✏️ 실제 값

PS C:\Users\User\Desktop\my-web-folder> docker run -d -p 1205:80 --name my-web my-web
PS C:\Users\User\Desktop\my-web-folder> docker ps
CONTAINER ID   IMAGE    COMMAND                  CREATED         STATUS         PORTS              NAMES
8282eaa89860   my-web   "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   0.0.0.0:1205->80   my-web
```

- **이미지 태그:** `my-web:latest`
- **이미지 ID:** ✏️ `docker images` 출력의 IMAGE ID 기재

### 5-6. 포트 매핑 및 접속 확인

**curl 확인 (재현 절차의 예상 응답)**

```powershell
PS C:\Users\User\Desktop\my-web-folder> curl.exe -i http://localhost:1205
HTTP/1.1 200 OK
Server: nginx/...
Content-Type: text/html
...
<img width="905" height="82" alt="image" src="https://github.com/user-attachments/assets/803467ad-b74f-4a8d-ad22-7c2f0ac88790" />
```

<img width="425" height="228" alt="image" src="https://github.com/user-attachments/assets/8d5fe6b0-58b6-4daf-a1f6-3c9e1232d7b7" />


### 5-7. 바인드 마운트

호스트 폴더를 컨테이너에 연결해, 호스트에서 `index.html` 을 수정하면 **이미지를 다시 빌드하지 않아도** 즉시 반영되는지 확인했습니다.

```powershell
PS C:\Users\User\Desktop\my-web-folder> docker run -d -p 1206:80 --name my-web-bind `
    -v ${PWD}:/usr/share/nginx/html nginx
# → index.html 수정 후 브라우저 새로고침 시 변경 내용이 바로 반영됨
```

<img width="816" alt="바인드 마운트 반영 확인" src="https://github.com/user-attachments/assets/0f187a83-83ef-4a17-b7a1-d4c6258fe890" />

#### 절대 경로 vs 상대 경로

<img width="332" height="223" alt="image" src="https://github.com/user-attachments/assets/c58c4004-60d2-40c1-8107-06e6b90a8d2a" />

→ 본 실습은 재현성을 위해 `${PWD}` 를 사용했고, 프로젝트 루트에서 실행하는 것을 전제로 합니다.

### 5-8. Docker Volume 영속성 검증

> **볼륨 `mydata` 의 목적:** 컨테이너의 `/data` 경로에 저장한 파일이 **컨테이너를 삭제해도 남는지** 검증하기 위한 데이터 저장 공간입니다. 볼륨은 Docker가 관리하는 호스트 영역에 저장되며, 컨테이너 수명과 분리됩니다. 따라서 `vol-test` 를 삭제해도 `memo.txt` 가 유지될 것으로 예상했습니다.

```powershell
# 1. 볼륨 생성
PS C:\Users\User\Desktop\my-web-folder> docker volume create mydata
mydata

# 2. 볼륨을 연결한 우분투 컨테이너 실행 후 파일 생성 및 탈출
PS C:\Users\User\Desktop\my-web-folder> docker run -it --name vol-test -v mydata:/data ubuntu
root@cec8f565b68b:/# echo "hello volume" > /data/memo.txt
root@cec8f565b68b:/# exit

# 3. 컨테이너 완전 삭제 후 새 컨테이너로 볼륨 연결하여 데이터 확인
PS C:\Users\User\Desktop\my-web-folder> docker rm vol-test
vol-test
PS C:\Users\User\Desktop\my-web-folder> docker run -it --rm -v mydata:/data ubuntu cat /data/memo.txt
hello volume
```

<img width="816" height="83" alt="image" src="https://github.com/user-attachments/assets/0f187a83-83ef-4a17-b7a1-d4c6258fe890" />

#### 볼륨 백업·복원 방법

볼륨 내용을 tar 파일로 백업하고, 필요 시 새 볼륨에 복원할 수 있습니다.

```powershell
# 백업: mydata 볼륨 → 현재 폴더의 mydata-backup.tar.gz
docker run --rm -v mydata:/data -v ${PWD}:/backup ubuntu tar czf /backup/mydata-backup.tar.gz -C /data .

# 복원: 새 볼륨 mydata2 로 복원
docker volume create mydata2
docker run --rm -v mydata2:/data -v ${PWD}:/backup ubuntu tar xzf /backup/mydata-backup.tar.gz -C /data
```
<img width="238" height="42" alt="image" src="https://github.com/user-attachments/assets/ee3b885f-9f1d-4aaf-8496-a8f560d582dd" />

---

## 6. 📚 핵심 개념 정리

### 6-1. 이미지(Image)와 컨테이너(Container)의 차이

| 구분 | 이미지 | 컨테이너 |
|------|--------|----------|
| 정의 | 실행에 필요한 파일·설정이 담긴 **읽기 전용 템플릿** | 이미지를 기반으로 실행된 **프로세스 인스턴스** |
| 생성 방법 | `docker build` | `docker run` |
| 변경 가능 여부 | 불변 (변경하려면 새로 빌드) | 실행 중 쓰기 가능 (삭제하면 변경분 사라짐) |
| 비유 | 클래스 / 설치 파일 | 객체 / 실행 중인 프로그램 |
| 본 실습 예 | `my-web` (Nginx + index.html) | `my-web` 이름으로 실행한 컨테이너 |

**사례:** `my-web` 이미지 하나로 `docker run -p 1205:80`, `docker run -p 1207:80` 처럼 컨테이너를 여러 개 띄울 수 있고, 한 컨테이너를 삭제해도 이미지는 그대로 남습니다.

### 6-2. 포트 포워딩이 동작하는 이유

컨테이너는 **별도의 네트워크 네임스페이스**를 가지므로 자체 IP와 포트 공간을 갖고, 호스트에서 컨테이너 내부 포트로 직접 접근할 수 없습니다. `-p 1205:80` 옵션은 Docker가 호스트의 `1205` 포트로 들어온 트래픽을 컨테이너의 `80` 포트로 전달(NAT/DNAT)하도록 규칙을 만들어 주기 때문에 `localhost:1205` 로 컨테이너 안의 Nginx에 접속할 수 있습니다. `ps` 출력의 `0.0.0.0:1205->80` 은 "모든 호스트 인터페이스의 1205 → 컨테이너 80" 이라는 뜻입니다.

### 6-3. 바인드 마운트 vs Docker Volume

| 구분 | 바인드 마운트 | Docker Volume |
|------|---------------|---------------|
| 저장 위치 | 호스트의 지정 경로 | Docker가 관리하는 영역 |
| 주 용도 | 개발 중 소스 **실시간 반영** | 데이터 **영속 저장** (DB 등) |
| 호스트 경로 의존성 | 높음 | 낮음 |
| 백업 | 일반 파일 복사 | tar 백업 등 별도 절차 ([5-8](#5-8-docker-volume-영속성-검증) 참고) |

### 6-4. 포트 충돌 진단 및 해결

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

- **문제 상황:** 시스템 루트 경로(`PS C:\WINDOWS\system32>`)에서 `docker build` 를 실행했을 때 Dockerfile을 찾지 못함
- **원인 가설:** 현재 터미널의 작업 디렉토리가 Dockerfile이 있는 폴더가 아님
- **해결 방법:** `cd C:\Users\User\Desktop\my-web-folder` 로 이동한 뒤 다시 빌드

```text
ERROR: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
```

### 7-2. 붙여넣기 시 특수문자 에러 (`$'\E[200~'`)

- **문제 상황:** 컨테이너 터미널에 명령어를 붙여넣을 때 브래킷드 페이스트 기호가 섞여 명령어 인식 실패
- **원인 가설:** 클립보드 붙여넣기 모드(bracketed paste)와 터미널 간의 입력 충돌
- **해결 방법:** 명령어를 직접 타이핑하거나 텍스트를 정돈하여 입력해 정상 실행

```text
bash: $'\E[200~mkdir': command not found
```

---

## 8. 🌐 Git 설정 및 GitHub 연동

```bash
git config --global user.name "본인 이름"
git config --global user.email "본인 이메일"
```

**원격 저장소 확인 및 push 기록**
<img width="238" height="42" alt="image" src="https://github.com/user-attachments/assets/ee3b885f-9f1d-4aaf-8496-a8f560d582dd" />


---
