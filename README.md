# 개발 워크스테이션 구축

## 1. 실행 환경
- OS: Windows 11 + WSL2 (Ubuntu)
- Docker: 4.98.0
- Git: 2.56.0

## 2. 수행 체크리스트
- [x] 도구 설치 (Docker/Git/VSCode)
- [x] WSL2 엔진 설치
- [ ] Ubuntu 배포판 설치
- [ ] 터미널 기본 조작
- [ ] 권한 실습
- [ ] Docker 점검 + hello-world
- [ ] Dockerfile 빌드/실행
- [ ] 포트 매핑
- [ ] 바인드 마운트 / 볼륨 영속성
- [ ] Git + GitHub 연동

## 3. 수행 로그

### 3-0. 도구 설치 (Windows 호스트)

winget으로 Docker / Git / VSCode 설치:

```powershell
PS> winget install --id Docker.DockerDesktop -e ...
Successfully installed

PS> winget install --id Git.Git -e ...
No available upgrade found.   # 기존 설치 확인

PS> winget install --id Microsoft.VisualStudioCode -e ...
No available upgrade found.   # 기존 설치 확인
```
→ Docker Desktop 4.98.0 설치 완료, Git/VSCode 기설치 확인

### 3-1. WSL2 설치

```powershell
PS> wsl --update
Linux용 Windows 하위 시스템 3.0.3이(가) 설치되었습니다.

PS> wsl -l -v
  NAME              STATE      VERSION
* docker-desktop    Stopped    2
```
→ WSL2 엔진 설치 확인. (docker-desktop은 Docker 내부 배포판, 실습용 Ubuntu는 다음 단계에서 설치)

### 3-2. Ubuntu 설치
(Ubuntu 설치 후 여기에 이어서 작성 예정)
```

> 참고: `(괄호 메모)`는 **아직 안 한 작업 표시**예요. 작업하면 실제 로그로 교체하면 됩니다.

노션 https://pingmong.notion.site/codyssey 
<img width="402" height="45" alt="image" src="https://github.com/user-attachments/assets/acac91fb-20f6-44bb-a6c3-e703a1c33929" />
https://github.com/microsoft/WSL/releases/tag/3.0.3
