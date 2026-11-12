# 2026-11-12 Docker·VM·ComfyUI 실습 정리

## 1. 역할 구분

1. Windows PC: 브라우저와 작업 파일을 관리합니다.
2. VMware Ubuntu VM: SSH 접속과 Docker 명령을 실행합니다.
3. Flask 컨테이너: `app.py`를 실행하고 브라우저 요청을 받습니다.
4. ComfyUI 서버: Flux 2 워크플로를 실제로 추론합니다.

## 2. 컨테이너 생성

```bash
sudo docker run -d \
  --name flaskserver \
  -p 5000:5000 \
  -v /home/aisw/Comfyui:/workspace/comfyui \
  -w /workspace/comfyui \
  python:3.12-slim \
  sleep infinity
```

`-v`는 VM의 `/home/aisw/Comfyui`와 컨테이너의 `/workspace/comfyui`를 연결합니다.

## 3. 파일 복사

Windows CMD에서 `scp`로 복사합니다.

```cmd
scp app.py aisw@VM_IP:/home/aisw/Comfyui/app.py
scp static\flux2.json aisw@VM_IP:/home/aisw/Comfyui/static/flux2.json
scp templates\image.html aisw@VM_IP:/home/aisw/Comfyui/templates/image.html
```

처음 접속할 때 fingerprint 질문에는 `yes`, 이어서 VM 계정 비밀번호를 입력합니다.

## 4. Flux 2 워크플로 연결

```python
COMFY_URL = "http://COMFYUI_IP:8188"
WORKFLOW_PATH = "./static/flux2.json"
```

Flux 2 JSON의 실제 노드 ID는 기존 예제와 달랐습니다.

- `76`: 양의 프롬프트 문자열
- `75:67`: 음의 프롬프트
- `75:73`: 랜덤 시드

```python
graph["76"]["inputs"]["value"] = pos
graph["75:67"]["inputs"]["text"] = neg
graph["75:73"]["inputs"]["noise_seed"] = random.randint(0, 2**32 - 1)
```

기존 예제의 `6`, `7`, `13`번 노드를 그대로 사용하면 Flux 2 JSON과 맞지 않습니다.

## 5. 실행과 확인

```bash
sudo docker exec -it flaskserver bash
cd /workspace/comfyui
pip install flask requests
python app.py
```

브라우저에서는 `http://VM_IP:5000`으로 접속합니다.

## 6. 오류 원인

`Workflow not found: ./static/flux2.json`은 JSON이 컨테이너의 `/workspace/comfyui/static`에 없다는 뜻입니다.

```bash
sudo docker exec flaskserver ls -l /workspace/comfyui/static/flux2.json
```

`ERR_CONNECTION_REFUSED`는 Flask 실행 여부, 포트 매핑, 방화벽 또는 접속 IP를 확인해야 합니다.

## 7. 오늘 작업의 핵심

`test.json` 대신 `flux2.json`을 사용했기 때문에 파일만 교체하면 끝나지 않았습니다. Flux 2 JSON의 실제 노드 ID를 확인하고 `app.py`가 해당 노드에 프롬프트와 시드를 쓰도록 함께 맞춰야 했습니다.
