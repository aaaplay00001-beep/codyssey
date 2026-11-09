# 🚀 Docker & Development Environment Setup Mission

개발 워크스테이션 구축 및 Docker 핵심 기능 실습 결과 보고서입니다. 본문과 로그를 통해 터미널 환경 세팅부터 Docker 컨테이너 기반 웹 서버 배포, 데이터 영속성 검증까지의 전 과정을 기록하였습니다.

---

## 1. 📋 프로젝트 개요
* **미션 목표:** 개발 환경의 재현성 확보, Docker 컨테이너의 격리성 이해, 그리고 커스텀 이미지 빌드부터 볼륨 영속성 검증까지의 엔지니어링 흐름 직접 체득
* **핵심 수행 내용:**
  1. 리눅스 CLI 환경 조작 및 파일/디렉토리 권한 제어 실습 (`chmod`)
  2. Docker 기본 운영 및 컨테이너 생명주기 관리 (`run`, `stop`, `rm`, `exec` 등)
  3. `Dockerfile`과 Nginx를 활용한 커스텀 웹 서버 이미지 빌드 및 포트 매핑 배포
  4. 바인드 마운트와 Docker Volume을 통한 "변경 사항 실시간 반영" 및 "데이터 영속성" 검증

---

## 2. 💻 실행 환경
* **OS:** Windows 11 (PowerShell / WSL2 backend)
* **Shell:** PowerShell / Bash (Docker Container 내우분투)
* **Docker Version:** 27.x 이상 (Docker Desktop for Windows)
* **Git Version:** 최신 버전

---

## 3. ✅ 수행 항목 체크리스트
- [x] 터미널 기본 조작 및 폴더 구성 (`mkdir`, `cd`, `ls`)
- [x] 권한 변경 실습 (`chmod 600`, `chmod 700`)
- [x] Docker 설치 및 데몬 점검 (`docker --version`, `docker info`)
- [x] 우분투 컨테이너 진입 및 제어 (`docker run -it`, `exit` vs `exec`)
- [x] Dockerfile 기반 커스텀 웹 서버 이미지 빌드 및 실행 (`docker build`, `docker run -d -p`)
- [x] 포트 매핑 및 브라우저 접속 확인 (`localhost:1205`)
- [x] 바인드 마운트 반영 테스트 (`-v ${PWD}:...`)
- [x] Docker Volume 생성, 연결 및 데이터 영속성 검증 (`docker volume create`)
- [x] Git 설정 및 VSCode 연동

---

## 4. 🛠️ 수행 로그 및 검증 방법

### ① 터미널 및 권한 실습 로그
우분투 컨테이너 내부에서 디렉토리를 생성하고 파일 권한을 제어한 결과입니다.

```bash
root@d35aab0d3a21:/# mkdir temp
root@d35aab0d3a21:/# cd temp
root@d35aab0d3a21:/temp# touch file.txt
root@d35aab0d3a21:/temp# mkdir mydir

# 권한 변경 전후 비교 (ls -l)
root@d35aab0d3a21:/temp# chmod 600 file.txt      # 소유자 읽기/쓰기 (-rw-------)
root@d35aab0d3a21:/temp# chmod 700 mydir         # 소유자 모든 권한 (drwx------)
root@d35aab0d3a21:/temp# ls -l
total 4
-rw------- 1 root root    0 Nov  9 10:37 file.txt
drwx------ 2 root root 4096 Nov  9 11:47 mydir

# exit로 나간 후 멈춘(Exited) 상태 확인
PS C:\WINDOWS\system32> docker ps -a
CONTAINER ID   IMAGE         COMMAND         CREATED      STATUS                  PORTS   NAMES
d35aab0d3a21   ubuntu        "/bin/bash"     2 hours ago  Exited (137) 14 secs ago        mybox

# 다시 시작 및 exec 접속 후 나갔을 때 'Up' 상태 유지 확인
PS C:\WINDOWS\system32> docker start mybox
PS C:\WINDOWS\system32> docker exec -it mybox bash
root@d35aab0d3a21:/# exit

PS C:\WINDOWS\system32> docker ps
CONTAINER ID   IMAGE   COMMAND       CREATED      STATUS         PORTS   NAMES
d35aab0d3a21   ubuntu  "/bin/bash"   2 hours ago  Up 25 seconds          mybox

FROM nginx
COPY index.html /usr/share/nginx/html/index.html

PS C:\Users\User\Desktop\my-web-folder> docker build -t my-web .
[+] Building 0.1s (1/1) FINISHED ...
Successfully built ...

PS C:\Users\User\Desktop\my-web-folder> docker run -d -p 1205:80 --name my-web my-web
PS C:\Users\User\Desktop\my-web-folder> docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                    NAMES
8282eaa89860   my-web    "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   0.0.0.0:1205->80         my-web

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

---

## 5. 🛠️ 트러블슈팅 (Troubleshooting)

### 1) Dockerfile 빌드 시 경로 에러 (`no such file or directory`)
* **문제 상황:** 시스템 루트 경로(`PS C:\WINDOWS\system32>`)에서 `docker build`를 실행했을 때 Dockerfile을 찾지 못하는 에러 발생.
* **원인 가설:** 현재 터미널의 작업 디렉토리가 소스코드가 위치한 폴더가 아님.
* **해결 방법:** `cd C:\Users\User\Desktop\my-web-folder` 명령어로 프로젝트 폴더 경로로 이동한 뒤 다시 빌드하여 해결.

### 2) 컨테이너 안에서 복사·붙여넣기 시 특수문자 에러 (`$'\E[200~'`)
* **문제 상황:** PowerShell에서 명령어를 복사·붙여넣기 할 때 터미널에 브래킷드 페이스트 기호가 섞여 명령어 인식 실패.
* **원인 가설:** 클립보드 붙여넣기 모드와 터미널 간의 입력 충돌.
* **해결 방법:** 명령어를 직접 타이핑하거나 텍스트를 정돈하여 입력함으로써 정상 실행 완료.

---

## 6. 🌐 Git 설정 및 GitHub 연동
* **Git 사용자 정보 설정:**
  ```bash
  git config --global user.name "본인 이름"
  git config --global user.email "본인 이메일"



노션
https://pingmong.notion.site/codyssey


<img width="402" height="45" alt="image" src="https://github.com/user-attachments/assets/acac91fb-20f6-44bb-a6c3-e703a1c33929" />


https://github.com/microsoft/WSL/releases/tag/3.0.3
<img width="646" height="450" alt="image" src="https://github.com/user-attachments/assets/c176a622-0445-4c7f-816d-2fdbc7a84aea" />
캡쳐6
<img width="835" height="302" alt="image" src="https://github.com/user-attachments/assets/13c3faae-2d99-4810-9c8f-0eb33b188204" />
ls -l                   # 변경 전 캡처
<img width="383" height="173" alt="image" src="https://github.com/user-attachments/assets/78ee10cf-5b3c-40ab-86eb-b240c6c91864" />
변경 후 캡쳐
<img width="905" height="82" alt="image" src="https://github.com/user-attachments/assets/803467ad-b74f-4a8d-ad22-7c2f0ac88790" />
미션5


<img width="425" height="228" alt="image" src="https://github.com/user-attachments/assets/8d5fe6b0-58b6-4daf-a1f6-3c9e1232d7b7" />
홍길동
<img width="332" height="223" alt="image" src="https://github.com/user-attachments/assets/c58c4004-60d2-40c1-8107-06e6b90a8d2a" />
바인드마운


<img width="816" height="83" alt="image" src="https://github.com/user-attachments/assets/0f187a83-83ef-4a17-b7a1-d4c6258fe890" />
블룸
<img width="238" height="42" alt="image" src="https://github.com/user-attachments/assets/ee3b885f-9f1d-4aaf-8496-a8f560d582dd" />
깃허브 캡
