# PROMPT FRAME STUDIO

프롬프트를 입력하면 원격 ComfyUI에서 이미지를 생성하고 결과를 보여주는 Flask 웹 애플리케이션입니다.

![Prompt Frame Studio 화면](prompt-frame-studio.png)

## 프로젝트 개요

- 사진관 콘셉트의 이미지 생성 웹 화면
- Flask 컨테이너와 ComfyUI 서버를 IP 주소로 연결
- Flux 2 워크플로 JSON을 ComfyUI API에 전달
- 생성 완료 후 결과 이미지 주소를 받아 화면에 표시
- 입력한 프롬프트와 제외 프롬프트를 워크플로에 반영

## 사용 기술

- Python 3.12
- Flask
- Requests
- Docker / Docker Compose
- ComfyUI API
- Flux 2 워크플로
- HTML / CSS / JavaScript

## 동작 구조

```text
브라우저
   ↓
Flask 컨테이너 :5000
   ↓  /prompt, /history, /view
ComfyUI 서버 :8188
   ↓
생성 이미지
```

## 폴더 구조

```text
promptframe-studio/
├─ app.py
├─ requirements.txt
├─ Dockerfile
├─ docker-compose.yml
├─ static/
│  └─ flux2.json
├─ templates/
│  └─ image.html
├─ docs/
│  └─ 20261112-docker-flux2.md
└─ prompt-frame-studio.png
```

## 사전 준비

- Docker가 설치되어 있어야 합니다.
- ComfyUI가 실행 중이어야 합니다.
- ComfyUI 서버의 IP 주소와 포트가 필요합니다.
- Flux 2 워크플로에 지정된 모델 파일이 ComfyUI에 설치되어 있어야 합니다.

## 실행 방법

### 1. ComfyUI 주소 설정

기본 주소는 `http://192.168.24.135:8188`입니다.

다른 주소를 사용할 경우 환경 변수로 지정합니다.

```bash
COMFY_URL=http://COMFYUI_IP:8188 docker compose up --build
```

Windows PowerShell에서는 다음처럼 실행합니다.

```powershell
$env:COMFY_URL = "http://COMFYUI_IP:8188"
docker compose up --build
```

### 2. 컨테이너 실행

```bash
docker compose up --build
```

### 3. 웹 화면 접속

브라우저에서 다음 주소를 엽니다.

```text
http://localhost:5000
```

## 이미지 생성 과정

1. 사용자가 Photo Prompt와 Exclude를 입력합니다.
2. Flask가 `static/flux2.json`을 읽습니다.
3. Flux 2 워크플로의 프롬프트와 시드 값을 변경합니다.
4. Flask가 ComfyUI의 `/prompt` API를 호출합니다.
5. `/history/{prompt_id}`를 확인하며 생성 완료를 기다립니다.
6. ComfyUI의 이미지 주소를 화면에 표시합니다.

## Flux 2 워크플로 참고

- 워크플로 파일 위치: `static/flux2.json`
- 양성 프롬프트 노드: `76`
- 음성 프롬프트 노드: `75:67`
- 랜덤 시드 노드: `75:73`
- 이미지 저장 노드: `9`
- 모델 파일명은 워크플로의 `UNETLoader`, `CLIPLoader`, `VAELoader` 설정과 일치해야 합니다.

기존 테스트 워크플로의 노드 번호인 `6`, `7`, `13`을 그대로 사용하면 Flux 2 JSON에서는 프롬프트와 시드가 변경되지 않거나 오류가 발생할 수 있습니다.

## 연결 확인

Flask 컨테이너에서 ComfyUI가 연결되는지 확인합니다.

```bash
docker exec flaskserver python -c "import requests; print(requests.get('http://COMFYUI_IP:8188/system_stats').status_code)"
```

응답이 `200`이면 기본 연결이 된 상태입니다.

## 문제 해결

- `Workflow not found`: 컨테이너 내부의 `static/flux2.json` 경로를 확인합니다.
- `Connection refused`: ComfyUI 실행 여부, IP 주소, 포트 `8188`을 확인합니다.
- `POST / 500`: Flask 로그에서 워크플로 노드 번호와 JSON 구조를 확인합니다.
- 이미지가 나오지 않음: Flux 2 모델 파일과 ComfyUI의 출력 경로를 확인합니다.
- 브라우저 접속 불가: Flask 컨테이너의 포트 `5000:5000` 매핑을 확인합니다.

## 문서

- [Docker·VM·ComfyUI 연결 실습](docs/20261112-docker-flux2.md)

## 주의사항

- 모델 파일은 용량이 크므로 저장소에 포함하지 않습니다.
- 생성 결과 이미지와 비밀번호·접속 토큰은 저장소에 올리지 않습니다.
- Flask 개발 서버는 실습용입니다.
