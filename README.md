# PROMPT FRAME STUDIO

프롬프트를 입력하면 Flask 웹 화면이 원격 ComfyUI에 작업을 요청하고 생성된 이미지를 보여주는 실습 프로젝트입니다.

## 구성

- `app.py`: Flask 서버와 ComfyUI API 연동
- `templates/image.html`: 사진관 콘셉트의 프롬프트 입력 화면
- `static/flux2.json`: ComfyUI Flux 2 API 워크플로
- `Dockerfile`, `docker-compose.yml`: Flask 컨테이너 실행 설정
- `docs/`: Docker·VM·ComfyUI 연결 과정

## 동작 구조

```text
브라우저 → Flask 컨테이너:5000 → ComfyUI 서버:8188 → 이미지 결과
```

## 실행

```bash
docker compose up --build
```

브라우저에서 `http://localhost:5000`을 엽니다. ComfyUI 주소가 다르면 `COMFY_URL`을 지정합니다.

```bash
COMFY_URL=http://COMFYUI_IP:8188 docker compose up --build
```

## 주의

- Flask 컨테이너에서 ComfyUI의 `/system_stats` 요청이 성공해야 합니다.
- `flux2.json`의 모델 파일은 ComfyUI 환경에 설치되어 있어야 합니다.
- 모델 파일과 생성 결과 이미지는 저장소에 포함하지 않습니다.
