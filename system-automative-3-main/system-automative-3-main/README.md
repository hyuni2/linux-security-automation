# system-automative-3
Linux 서버 보안 점검 자동화 프로젝트 (Streamlit + Ansible + Nuclei)

## 프로젝트 목적
- 원격 Linux 서버 보안 점검을 자동화하고 결과를 대시보드에서 통합 관리합니다.
- OS 하드닝 점검(Ansible + 점검 스크립트)과 템플릿 기반 취약점 진단(Nuclei)을 한 화면에서 운영할 수 있게 구성했습니다.

## 주요 기능
- 단일 서버/다중 서버(CSV) 원격 점검
- OS별 점검 스크립트 자동 실행 (Rocky 9/10, Ubuntu 24)
- Nuclei 자동 스캔/수동 명령 실행
- 점검 결과 시각화 및 히스토리 관리
- 문서/표 형태 결과 활용 (프로젝트 코드 기반)

## 사용 툴
- `Streamlit`: 대시보드 UI/실행 제어
- `Ansible`: 원격 서버 점검 오케스트레이션
- `Nuclei`: 템플릿 기반 취약점 진단
- `bash`: OS별 점검 스크립트 구현
- `pandas`, `openpyxl`, `python-docx`, `reportlab`: 결과 가공/내보내기

## 프로젝트 구조
```text
.
├─ app.py
├─ requirements.txt
├─ src/
│  ├─ guides/
│  │  ├─ 수동_조치_가이드라인_rocky_linux.pdf
│  │  └─ 수동_조치_가이드라인_ubuntu.pdf
│  └─ dashboard_0210/
│     ├─ ansible.cfg
│     ├─ check_playbook.yml
│     ├─ remedy_playbook.yml
│     ├─ temp_inventory.ini
│     ├─ scripts/
│     │  ├─ check/
│     │  │  ├─ ubuntu_check.sh
│     │  │  ├─ rocky_check_9.sh
│     │  │  └─ rocky_check_10.sh
│     │  ├─ nuclei/
│     │  ├─ remedy/
│     │  │  ├─ ubuntu_remedy.sh
│     │  │  ├─ rocky_remedy_9.sh
│     │  │  └─ rocky_remedy_10.sh
│     │  └─ __init__.py
│     ├─ nuclei-templates/
│     ├─ reports/
│     ├─ history/
│     ├─ templates/
│     ├─ images/
│     └─ styles.css
└─ venv/ (선택)
```

## 동작 흐름
1. 대시보드에서 대상 서버 정보 입력
2. `ansible-playbook`으로 원격 점검 스크립트 실행
3. 결과를 로컬 리포트로 수집/표시
4. 필요 시 Nuclei 스캔 실행 (자동 모드/수동 모드)
5. JSON 결과를 파싱해 대시보드에서 확인

## 빠른 실행
### 1) 환경 준비
```bash
cd ~/system-automative-3
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2) Nuclei 설치 확인
```bash
which nuclei
nuclei -version
```

### 3) Nuclei 템플릿 준비
기본 사용 경로:
- `src/dashboard_0210/nuclei-templates`

템플릿이 없으면:
```bash
git clone --depth 1 https://github.com/projectdiscovery/nuclei-templates.git src/dashboard_0210/nuclei-templates
```

업데이트:
```bash
git -C src/dashboard_0210/nuclei-templates pull
```

### 4) 대시보드 실행
```bash
streamlit run app.py
```

## Nuclei 모드(대시보드)
- 웹 기본 스캔
- 웹 확장 스캔
- 네트워크 스캔
- DNS/SSL 스캔
- DAST 스캔
- 전체 템플릿 스캔

참고:
- 수동 실행은 보안상 `nuclei` 명령만 허용합니다.
- `ssh user@ip` 형식 입력은 내부적으로 대상 호스트/IP로 정규화됩니다.

## 원격 점검 실행 파일
- `src/dashboard_0210/check_playbook.yml`
- `src/dashboard_0210/scripts/check/rocky_check_9.sh`
- `src/dashboard_0210/scripts/check/rocky_check_10.sh`
- `src/dashboard_0210/scripts/check/ubuntu_check.sh`
- `src/dashboard_0210/remedy_playbook.yml`
- `src/dashboard_0210/scripts/remedy/rocky_remedy_9.sh`
- `src/dashboard_0210/scripts/remedy/rocky_remedy_10.sh`
- `src/dashboard_0210/scripts/remedy/ubuntu_remedy.sh`

## 트러블슈팅
### 1) `no templates provided for scan`
- 원인: 템플릿 경로 오타/필터 조건 불일치
- 조치:
```bash
nuclei -validate -t src/dashboard_0210/nuclei-templates -target 127.0.0.1
```

### 2) `return code 0`인데 결과 없음
- 실행은 성공, 탐지 조건 불일치(`No findings`)일 수 있습니다.
- 대상 서비스/포트와 템플릿 유형을 맞춰 확인하세요.

### 3) 원격 점검 실패(Ansible)
- SSH 계정/비밀번호, `sudo` 권한, 방화벽/네트워크 접근 확인
- 플레이북 경로/인벤토리 파일 생성 여부 확인

### 4) 쉘 스크립트 실행 에러 (`/bin/bash^M`)
- CRLF 문제
```bash
sed -i 's/\r$//' src/dashboard_0210/scripts/check/rocky_check_9.sh
sed -i 's/\r$//' src/dashboard_0210/scripts/check/rocky_check_10.sh
sed -i 's/\r$//' src/dashboard_0210/scripts/check/ubuntu_check.sh
```

## 참고
- 본 README는 현재 코드 기준 실행 흐름을 요약한 문서입니다.
- 팀 인수인계용 상세 문서는 별도 저장소 문서 정책에 맞춰 추가 관리하세요.
