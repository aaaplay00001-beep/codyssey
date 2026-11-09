# 개발 워크스테이션 구축

터미널, Docker, Git을 직접 세팅하며 재현 가능한 개발 환경을 구성하는 미션입니다.

## 1. 프로젝트 개요
- 리눅스 CLI, Docker, Git/GitHub를 활용해 개발 워크스테이션을 구축한다.
- "내 컴퓨터에서만 돌아가는 문제"를 줄이고, 누구나 같은 방식으로 실행 가능한 환경을 목표로 한다.

## 2. 실행 환경
- OS: Windows + WSL2 (Ubuntu)
- Shell: bash
- Docker Desktop: 4.98.0
- Git: 2.56.0
- Editor: VSCode

## 3. 수행 체크리스트
- [x] 터미널 기본 조작
- [x] 권한 변경 실습
- [x] Docker 설치 / 점검
- [x] hello-world 실행
- [x] ubuntu 컨테이너 실행 / 진입
- [x] Git 설정 + GitHub / VSCode 연동
- [x] (보너스) GitHub SSH 키 설정
- [ ] Dockerfile 빌드 / 실행
- [ ] 포트 매핑 접속
- [ ] 바인드 마운트 / 볼륨 영속성

## 4. 터미널 조작 로그

### 현재 위치 / 목록 확인
​```bash
$ pwd
(여기에 실제 출력)

$ ls -la
(여기에 실제 출력)
​```

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

<img width="238" height="42" alt="image" src="https://github.com/user-attachments/assets/ee3b885f-9f1d-4aaf-8496-a8f560d582dd" />
깃허브 캡
