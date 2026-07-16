# Linux Security Automation

KISA 「주요정보통신기반시설 기술적 취약점 분석·평가 방법 상세가이드(2026)」를 기준으로 Linux 서버의 보안 점검과 선택적 조치를 자동화한 프로젝트입니다. Ansible로 원격 서버를 점검하고, Streamlit 대시보드에서 결과 확인·조치 실행·재진단·이력 관리를 수행합니다.

## 핵심 결과

| 구분 | 구현 내용 |
| --- | --- |
| 진단 기준 | KISA UNIX 서버 보안 점검 U-01~U-67 |
| 지원 환경 | Rocky Linux 9·10, Ubuntu 24.04 |
| 진단 자동화 | OS별 67개 항목을 Shell Script로 구현 |
| 조치 체계 | 48개 자동 조치, 19개 운영자 검토·수동 조치 |
| 원격 실행 | Ansible 기반 단일·다중 서버 점검 |
| 결과 관리 | 취약 항목 우선 정렬, 보안 수준 계산, Excel 저장, 이력 조회 |
| 추가 스캔 | Nuclei 템플릿 기반 웹·네트워크 취약점 점검 |

## 담당 역할

5인 팀 프로젝트에서 Linux 보안 진단·조치 스크립트 개발을 중심으로 다음 작업을 수행했습니다.

- KISA 기준 67개 항목을 Rocky Linux와 Ubuntu 환경에 맞게 점검 로직으로 구현
- 자동 조치 가능한 항목과 운영자 판단이 필요한 항목을 분리
- Ansible 플레이북을 통해 원격 진단·선택 조치·재진단 흐름 구성
- JSON 형태의 진단 결과를 Streamlit 대시보드와 연동
- OS·명령어 차이, 권한 부족, 서비스 미설치 상황을 고려한 예외 처리

## 설계 원칙

취약점이 탐지됐다는 이유만으로 모든 설정을 일괄 변경하지 않습니다. 계정 삭제, 서비스 중단, 네트워크 설정 변경처럼 운영 영향이 큰 항목은 수동 조치 대상으로 분류하고 가이드를 제공합니다. 자동 조치 항목도 사용자가 선택한 코드만 실행한 뒤 동일 항목을 다시 진단해 전후 결과를 비교합니다.

```mermaid
flowchart TD
    A["대상 서버 입력"] --> B["Ansible 원격 접속"]
    B --> C["OS 식별"]
    C --> D["U-01~U-67 진단"]
    D --> E["결과 시각화"]
    E --> F{"조치 유형"}
    F -->|자동| G["선택 항목 조치"]
    F -->|수동| H["조치 가이드 제공"]
    G --> I["재진단·전후 비교"]
```

## 주요 기능

### 보안 진단

- IP·계정 입력 기반 단일 서버 진단
- CSV 업로드 기반 다중 서버 일괄 진단
- 중요도, 양호·취약 상태, 판단 근거를 JSON으로 표준화
- 취약 항목 우선 정렬 및 중요도별 가중치를 활용한 보안 수준 표시

### 선택적 조치

- 취약 항목 중 자동 조치 대상을 사용자가 직접 선택
- Ansible Extra Variables로 U-code를 전달해 해당 함수만 실행
- 조치 완료 후 재진단하여 개선 여부와 보안 수준 변화를 비교
- 운영자 판단이 필요한 19개 항목은 PDF 가이드로 분리

### 대시보드와 결과 관리

- 진단·조치·Nuclei·이력 메뉴 제공
- 결과 테이블과 취약 항목 강조 표시
- Excel 결과 저장 및 다운로드
- Nuclei 자동 모드와 제한된 수동 명령 실행 지원

## 기술 구성

| 영역 | 기술 |
| --- | --- |
| Dashboard | Python, Streamlit, Pandas, Plotly |
| Automation | Ansible |
| Security Check | Bash, KISA UNIX U-01~U-67 |
| Vulnerability Scan | Nuclei |
| Export | OpenPyXL, python-docx, ReportLab |
| Target OS | Rocky Linux 9·10, Ubuntu 24.04 |

## 프로젝트 구조

```text
.
├── app.py
├── requirements.txt
├── README.md
└── src
    ├── guides
    │   ├── 수동_조치_가이드라인_rocky_linux.pdf
    │   └── 수동_조치_가이드라인_ubuntu.pdf
    └── dashboard_0210
        ├── ansible.cfg
        ├── check_playbook.yml
        ├── remedy_playbook.yml
        ├── scripts
        │   ├── check
        │   │   ├── rocky_check_9.sh
        │   │   ├── rocky_check_10.sh
        │   │   └── ubuntu_check.sh
        │   ├── remedy
        │   │   ├── rocky_remedy_9.sh
        │   │   ├── rocky_remedy_10.sh
        │   │   └── ubuntu_remedy.sh
        │   └── nuclei
        │       └── nuclei_check.py
        ├── templates
        ├── images
        └── styles.css
```

`nuclei-templates`, 임시 인벤토리, 진단 결과와 생성 보고서는 저장소에 포함하지 않습니다.

## 실행 방법

### 1. 환경 준비

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
sudo apt install ansible
```

### 2. Nuclei 준비

```bash
git clone --depth 1 https://github.com/projectdiscovery/nuclei-templates.git \
  src/dashboard_0210/nuclei-templates
```

Nuclei를 사용하지 않는 경우 템플릿 설치 없이 Linux 진단·조치 기능만 사용할 수 있습니다.

### 3. 대시보드 실행

```bash
streamlit run app.py
```

대상 서버에는 SSH 접속과 점검 명령 실행 권한이 필요합니다. 자동 조치는 시스템 설정을 변경하므로 운영 환경에서는 백업과 변경 승인 후 사용해야 합니다.

## 문제 해결

### OS별 명령과 설정 경로 차이

Rocky Linux와 Ubuntu는 패키지 관리자, PAM 구성, 서비스명과 설정 파일 경로가 달랐습니다. OS별 스크립트를 분리하고 배포판·주 버전을 확인한 뒤 해당 스크립트만 실행하도록 구성했습니다.

### 점검 결과 형식 불일치

항목마다 출력 형식이 달라 대시보드 파싱이 불안정한 문제가 있었습니다. 모든 점검 함수가 `code`, `item`, `severity`, `status`, `reason` 필드를 포함한 JSON 한 줄을 출력하도록 통일했습니다.

### 자동 조치의 운영 위험

일괄 조치가 계정·서비스 운영에 영향을 줄 수 있어 자동화 범위를 분리했습니다. 운영자 판단이 필요한 항목은 자동 실행에서 제외하고, 선택한 항목만 조치한 뒤 재진단하도록 설계했습니다.

## 주의사항

- 승인받은 서버에서만 사용해야 합니다.
- 자동 조치 전 설정 파일과 중요 데이터를 백업해야 합니다.
- 임시 인벤토리에는 SSH 인증정보가 포함되므로 실행 후 보관하지 않습니다.
- 프로젝트는 교육·검증 환경을 기준으로 구현했습니다.

## Team

5인 팀 프로젝트이며, 저장소에는 공동 결과물과 개인 담당 영역을 함께 포함하고 있습니다.
