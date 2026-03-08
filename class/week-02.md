# Week 2. 로컬 개발 환경과 재현 가능한 실행

> 원본: docs/ch2.md

## 학습 목표

- Python 가상환경을 생성하고 의존성을 고정하여 실행 재현성을 확보할 수 있다
- 크로스 플랫폼에서 동작하는 경로 처리와 스크립트 구조를 설계할 수 있다
- "실행-검증-산출물 저장" 패턴의 표준 템플릿을 활용할 수 있다
- AI가 생성한 코드의 실행 환경 문제를 진단하고 해결할 수 있다

---

## 2.1 왜 재현 가능한 환경이 중요한가

### 2.1.1 AI 코드 생성의 환경 의존성 문제

- **"내 컴퓨터에서는 되는데요..."** 라는 말 = 재현 가능한 환경이 없다는 신호
  - AI와 협업하여 코드를 작성할 때 이 문제는 더욱 빈번하게 발생
  - 이유: AI가 제안한 코드가 특정 라이브러리 버전이나 운영체제에 의존하는 경우가 많음

- **재현성(reproducibility)** (동일한 코드를 다른 환경에서 실행해도 동일한 결과를 얻는 성질)
  - 단순히 "코드가 실행된다"는 수준이 아님
  - 정확히 같은 버전의 라이브러리, 같은 운영체제 조건, 같은 데이터 형식에서 동일한 출력을 보장해야 함
  - AI 기반 개발 도구가 보편화되면서 환경 의존성이 더 자주 숨겨져 있어 더욱 중요해짐

- **라이브러리 버전 불일치 문제**
  - AI 어시스턴트는 코드 제안 시 라이브러리 버전을 명시하지 않음
  - → 예시: pandas `df.append()` 메서드
    - pandas 1.4.0에서 deprecated(사용 중단 예고)
    - pandas 2.0에서 완전히 제거됨
    - AI가 학습한 시점에서는 정상 코드였지만, 현재 버전에서는 오류 발생
  - AI 모델의 학습 데이터 수집 시점(knowledge cutoff) 이후의 변경사항은 반영되지 않음

- **운영체제 경로 차이 문제**
  - Windows: 경로 구분자 = 백슬래시 (`\`)
  - macOS/Linux: 경로 구분자 = 슬래시 (`/`)
  - AI가 특정 플랫폼 기준으로 경로를 하드코딩(직접 고정 입력)하면 다른 플랫폼에서 오류 발생
  - → 예시: `C:\\Users\\hong\\data\\file.csv` → macOS에서 파일 없음 오류

- **파일 인코딩 문제**
  - Windows 기본 인코딩: CP949
  - macOS/Linux 기본 인코딩: UTF-8
  - 인코딩 미명시 시 한글이 포함된 파일을 읽을 때 `UnicodeDecodeError` 발생
  - → 예시: `open('data.csv')` → Windows에서 한글 깨짐 → `open('data.csv', encoding='utf-8')`로 수정 필요

- **라이브러리 하위 호환성(backward compatibility) 문제**
  - 메이저 버전 업그레이드 시 API가 변경되는 경우 많음
  - → 예시: scikit-learn 1.0 — 일부 메서드 기본값 변경 / TensorFlow 2.x — 1.x와 완전히 다른 API 구조
  - AI가 구버전 API로 코드를 생성하면 최신 버전에서 `DeprecationWarning`이나 오류 발생

- **환경 변수 하드코딩 문제**
  - AI가 API 키나 DB 연결 정보를 코드에 직접 입력하는 경우 있음
  - 보안 문제 + 환경마다 다른 설정 사용 불가능
  - 개발 환경과 운영 환경을 다른 설정으로 분리할 수 없게 됨

### 2.1.2 재현 가능한 실행의 핵심 요소

- **핵심 요소 5가지 개요**

| 요소 | 도구/방법 | 목적 |
|------|----------|------|
| ① 격리된 실행 환경 | Python 가상환경(venv, conda) | 프로젝트별 독립 패키지 관리 |
| ② 고정된 의존성 | requirements.txt, poetry.lock | 누구나 동일한 환경 재현 |
| ③ 일관된 경로 처리 | pathlib | OS 무관 동작 |
| ④ 환경 설정 분리 | .env 파일, 환경 변수 | 보안 + 배포 유연성 |
| ⑤ 버전 관리 통합 | Git + .gitignore | 코드와 의존성 함께 기록 |

- **① 격리된 실행 환경**
  - 가상환경 = 프로젝트마다 독립적인 패키지 집합을 유지하는 공간
  - → 예시: 프로젝트 A는 Django 3.2, 프로젝트 B는 Django 4.1 → 가상환경으로 충돌 없이 공존 가능
  - 추가 이점: 패키지 설치/제거를 반복해도 시스템 Python이 "오염"되지 않음
    - 가상환경 디렉토리를 삭제하고 다시 만들면 깨끗한 상태로 복원

- **② 고정된 의존성**
  - `requirements.txt`에 정확한 버전을 기록 → 누구든 동일한 환경 재현 가능
  - 직접 의존성(direct dependency): 프로젝트가 직접 사용하는 패키지 (예: Django)
  - 전이 의존성(transitive dependency): 직접 의존성이 내부적으로 사용하는 패키지 (예: sqlparse, pytz)
  - 완전한 재현성을 위해서는 전이 의존성까지 고정 필요
    - `pip-tools`, `poetry` 등을 활용하면 더 체계적으로 관리 가능

- **③ 일관된 경로 처리**
  - `pathlib` 사용 → OS에 관계없이 동일하게 동작하는 경로 코드 작성 가능
  - 핵심 원칙: **"스크립트 위치 기준 상대 경로"** 사용
    - `Path(__file__).parent`로 현재 스크립트의 디렉토리를 얻고, 거기서 상대 경로를 구성
    - → 프로젝트를 어디에 복사하든, 어느 디렉토리에서 실행하든 동일하게 작동
  - ⚠ 하드코딩된 절대 경로(`/Users/hong/project/data`)는 다른 사용자 환경에서 작동하지 않음

- **④ 환경 설정의 분리**
  - API 키, DB 비밀번호, 서버 주소 등은 코드에서 분리 → 환경 변수나 `.env` 파일에 저장
  - `.env` 파일은 `.gitignore`에 추가하여 버전 관리에서 제외
  - `.env.example` 파일로 필요한 변수 목록과 예시값을 공유 → 다른 개발자가 참고 가능
  - 배포 전략 측면 이점: 동일한 코드를 개발/운영 환경에 모두 배포 가능
    - → 예시: `if os.getenv('ENV') == 'production'` 으로 환경별 동작 분기

- **⑤ 버전 관리 통합**
  - 코드뿐 아니라 `requirements.txt`, `.env.example`, `pyproject.toml` 등도 함께 Git으로 관리
  - 특정 커밋의 코드 + 그 당시의 의존성을 함께 기록 → 과거 시점 복원 가능
  - `.gitignore`에 추가해야 할 항목:
    ```
    venv/        # 가상환경 디렉토리
    .venv/
    *.pyc        # Python 임시 파일
    __pycache__/
    .env         # 민감한 환경 설정 (절대 커밋 금지)
    *.log
    output/
    ```

### 2.1.3 재현 불가능한 코드의 비용

- **① 디버깅 시간 증가**
  - "내 컴퓨터에서는 된다" = 다른 환경에서 재현 불가
  - 환경 차이(Python 버전, 패키지 버전, OS) 하나하나 확인 → 수 시간 소요 가능

- **② 협업 어려움**
  - 팀원이 환경을 맞추는 데 하루가 걸릴 수 있음
  - 구두 설명에 의존 → 정보 누락·오전달 위험
  - 새 팀원 합류 시마다 같은 설명 반복

- **③ 배포 실패**
  - 개발 환경에서 잘 작동하던 코드가 운영 서버에서 오류 발생 가능
  - 이유: 운영 서버의 Python 버전, 시스템 라이브러리, 경로 구조가 다를 수 있음
  - ⚠ 배포 후 문제 발견 → 서비스 중단 또는 데이터 손실로 이어질 수 있음

- **④ 지식 손실**
  - 6개월 후 프로젝트 재실행 시 어떤 버전을 설치했는지 기억나지 않음
  - ⚠ 연구 프로젝트에서 치명적: 논문 제출 후 리뷰어가 결과를 재현할 수 없으면 논문 거부 가능

- **사례: pandas 버전 불일치 문제**
  - 상황: AI가 제안한 `df.append(row)` 코드
    - pandas 1.x 환경 → 정상 동작
    - pandas 2.0 환경 → `AttributeError` 발생 (메서드 완전 제거됨)
  - 결과: 팀원 모두 "왜 내 컴퓨터에서 안 되지?" 질문에 하루 소요
  - 해결: `requirements.txt`에 `pandas==1.5.3` 명시 → 모든 팀원 동일 결과
  - 교훈
    - AI 생성 코드는 버전 의존성을 명시하지 않으므로 개발자가 직접 확인 필요
    - 프로젝트 초기에 `requirements.txt`를 작성하고 버전을 고정
    - 팀원 모두 가상환경과 의존성 관리를 습관화

---

## 2.2 Python 가상환경 완전 정복

### 2.2.1 가상환경이란 무엇인가

- **가상환경** = 프로젝트별 독립적인 Python 실행 환경을 제공하는 메커니즘
  - 시스템에 설치된 Python(시스템 Python)과 별도로, 각 프로젝트에 필요한 패키지만 설치된 격리된 공간
  - → 예시: 프로젝트 A는 requests 2.25, 프로젝트 B는 requests 2.31 → 충돌 없이 각각 사용 가능

- **동작 원리**
  - 가상환경 생성 시 → 프로젝트 디렉토리 안에 독립적인 Python 인터프리터 복사본 + 패키지 설치 디렉토리 생성
  - 활성화 시 → 시스템 PATH 환경 변수 수정
    - `python` 명령이 시스템 Python이 아닌 가상환경의 Python을 가리킴
  - `pip install` → 가상환경의 `site-packages` 디렉토리에 저장

- **가상환경 활성화 확인 방법**
  ```python
  import sys
  in_venv = sys.prefix != sys.base_prefix
  print(f"가상환경 활성: {in_venv}")
  print(f"sys.prefix: {sys.prefix}")
  print(f"sys.base_prefix: {sys.base_prefix}")
  ```
  - 활성화 상태: `sys.prefix` = 가상환경 경로 (예: `/path/to/venv`)
  - 비활성화 상태: `sys.prefix` = `sys.base_prefix` (시스템 Python 경로와 동일)

- **가상환경 디렉토리 구조**
  - `bin/` (Windows: `Scripts/`): Python 실행 파일, pip, activate 스크립트
  - `lib/python3.x/site-packages/`: 설치된 패키지 저장 위치
  - `include/`: C 확장 컴파일 시 필요한 헤더 파일
  - `pyvenv.cfg`: 가상환경 설정 파일 (Python 버전, 시스템 패키지 사용 여부 등)

- **주의사항**
  - ⚠ 가상환경은 이식 불가능(non-portable)
    - 가상환경 디렉토리를 다른 위치로 복사하면 작동하지 않음
    - 이유: 내부 스크립트와 설정 파일에 절대 경로가 하드코딩되어 있음
    - 공유하려면 `requirements.txt`를 사용하여 다른 곳에서 재생성
  - ⚠ 가상환경은 Python 버전을 격리하지 않음
    - 생성 시점의 Python 버전을 그대로 사용
    - Python 버전까지 격리하려면 pyenv나 conda 사용 필요
  - ⚠ 가상환경을 활성화하지 않고 실행하면 시스템 Python이 사용됨
    - "패키지가 설치되어 있는데 왜 ImportError가 나지?" 혼란의 원인

### 2.2.2 venv로 가상환경 만들기

- **venv**: Python 3.3부터 표준 라이브러리에 포함 → 별도 설치 없이 바로 사용 가능

- **생성 및 활성화 명령**

  **macOS/Linux:**
  ```bash
  python3 -m venv venv
  source venv/bin/activate
  ```

  **Windows:**
  ```bash
  python -m venv venv
  venv\Scripts\activate
  ```

- **명령 해석**
  - `-m venv`: venv 모듈을 실행하라
  - 마지막 `venv`: 생성할 가상환경 디렉토리 이름
  - 이름은 자유롭게 설정 가능 (관례: `venv`, `.venv`, `env`)
  - `.venv`처럼 점으로 시작하면 Unix 계열에서 숨김 디렉토리로 처리

- **활성화 후 변화**
  - 터미널 프롬프트 앞에 `(venv)` 표시
  - 이후 `pip install`로 설치하는 패키지는 이 가상환경에만 설치됨
  - 비활성화: `deactivate` 명령 입력

- **셸별 활성화 스크립트**
  - bash/zsh: `source venv/bin/activate`
  - fish 셸: `source venv/bin/activate.fish`
  - csh/tcsh: `source venv/bin/activate.csh`
  - Windows PowerShell: `venv\Scripts\Activate.ps1`
    - ⚠ 기본 보안 정책으로 스크립트 실행이 차단될 수 있음
    - 해결: `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` 실행

- **`.gitignore`에 추가할 항목**
  ```
  # 가상환경
  venv/
  .venv/
  env/
  ENV/

  # Python 임시 파일
  *.pyc
  __pycache__/
  *.pyo
  *.pyd
  .Python

  # 환경 설정
  .env
  .env.local
  ```

- **표준 워크플로우**
  1. 프로젝트 시작: `python3 -m venv venv` 가상환경 생성
  2. 작업 시작할 때마다: `source venv/bin/activate` 활성화
  3. 필요한 패키지: `pip install package_name` 설치
  4. 작업 마칠 때: `pip freeze > requirements.txt` 의존성 기록
  5. 다른 환경: `pip install -r requirements.txt` 동일 패키지 설치

- **유용한 명령어**
  - `which python` (Windows: `where python`): 현재 사용 중인 Python 경로 확인
  - `pip list`: 현재 환경에 설치된 모든 패키지 목록
  - `pip show package_name`: 특정 패키지의 버전, 의존성, 설치 위치 등 상세 정보

### 2.2.3 conda vs venv: 언제 무엇을 선택할까

**표 2.1** venv와 conda 비교

| 기준 | venv | conda |
|:---|:---|:---|
| 설치 | Python 표준 라이브러리 (별도 설치 불필요) | Anaconda/Miniconda 설치 필요 (400MB+) |
| 패키지 관리 | pip만 사용 | conda + pip 혼용 가능 |
| 비Python 의존성 | 지원 안 함 | CUDA, C 라이브러리, R 패키지 등 지원 |
| 환경 복제 | requirements.txt (Python 패키지만) | environment.yml (전체 환경) |
| Python 버전 격리 | 불가 (시스템 Python 버전 사용) | 가능 (환경마다 다른 Python 버전) |
| 패키지 저장소 | PyPI (60만+ 패키지) | Anaconda Cloud (2만+ 패키지) + PyPI |
| 설치 속도 | 빠름 | 느림 (의존성 해결이 복잡) |
| 디스크 사용량 | 작음 (~50MB) | 큼 (~500MB, 패키지 포함) |
| 용도 | 순수 Python 프로젝트 | 데이터 과학, ML (GPU 활용 시) |

- **venv를 선택해야 하는 경우**
  - 순수 Python 프로젝트 (웹 개발, API 서버, 크롤러 등)
    - PyPI의 방대한 패키지 생태계 활용 가능
    - 가볍고 빠름
  - 표준 도구를 선호하는 경우
    - Python 공식 문서에서 권장하는 표준 도구
    - 별도 설치 없이 모든 Python 설치에 기본 포함
  - CI/CD 파이프라인 (GitHub Actions, GitLab CI 등) 자동화 환경
    - 설치 빠름, 디스크 공간 적게 차지, 스크립트로 쉽게 자동화

- **conda를 선택해야 하는 경우**
  - 데이터 과학 및 머신러닝 프로젝트
    - NumPy, SciPy, scikit-learn, TensorFlow, PyTorch 등은 C/C++ 확장 포함
    - conda가 미리 컴파일된 바이너리로 제공 → 설치 간편, 버전 호환성 보장
  - GPU 가속이 필요한 경우
    - CUDA, cuDNN 등 GPU 라이브러리는 pip로 설치 불가
    - → 예시: `conda install pytorch cudatoolkit=11.8` 한 줄로 PyTorch + CUDA 동시 설치
  - Python 버전을 환경마다 다르게 사용해야 하는 경우
    - → 예시: Python 2.7 레거시 프로젝트 + Python 3.11 신규 프로젝트 동시 관리
  - 완전한 환경 재현이 중요한 경우 (연구 프로젝트 등)
    - `environment.yml`이 Python 버전 + 모든 패키지 + 비Python 의존성까지 포함

- **두 도구를 혼용할 때 주의사항**
  - ⚠ conda 환경에서 pip 사용은 권장하지 않음
    - conda와 pip는 의존성 관리 방식이 달라 충돌 가능
    - 꼭 필요하다면 conda로 주요 패키지 먼저 설치 후, conda에 없는 패키지만 pip로 설치
    - `conda install pip`로 conda가 관리하는 pip를 사용하는 것이 안전

> 이 강의 실습에서는 모든 환경에서 동일하게 사용 가능한 **venv** 기준으로 진행한다.

---

## 2.3 의존성 관리와 버전 고정

### 2.3.1 requirements.txt 작성법

- **`requirements.txt`** = 프로젝트에 필요한 Python 패키지 목록을 담는 파일
  - 재현 가능한 환경을 위해 버전까지 명시하는 것이 원칙

- **버전 지정 방식**

**표 2.2** requirements.txt 버전 지정 방식

| 방식 | 예시 | 의미 | 사용 시기 |
|:---|:---|:---|:---|
| `==` | `pandas==1.5.3` | 정확히 1.5.3 | 완전한 재현성 필요 |
| `>=` | `requests>=2.25.0` | 2.25.0 이상 | 최소 버전만 보장 |
| `~=` | `numpy~=1.24.0` | 1.24.x 범위 | 호환 업데이트 허용 |
| `>=,<` | `django>=3.2,<4.0` | 3.2 이상 4.0 미만 | 메이저 버전 고정 |
| `!=` | `requests>=2.28,!=2.28.1` | 2.28.1 제외 | 특정 버그 버전 제외 |

- **방식별 설명**
  - `==` (정확한 버전): 재현성 최대 보장, 가장 엄격
  - `>=` (최소 버전): 새로운 버그 수정/기능을 자동으로 받지만 하위 호환성이 깨질 위험 있음
  - `~=` (호환 버전): 시맨틱 버저닝(semantic versioning, 버전 번호 규칙) 원칙 따름
    - → 예시: `numpy~=1.24.0`은 1.24.1, 1.24.2는 설치하지만 1.25.0은 설치하지 않음
  - 범위 지정: `django>=3.2,<4.0` = 3.2 이상이면서 4.0 미만
  - 제외: `requests>=2.28.0,!=2.28.1` = 알려진 버그 버전만 제외

- **`pip freeze > requirements.txt` 방식의 문제점**
  - ⚠ 직접 설치하지 않은 전이 의존성(transitive dependency)까지 모두 포함
  - → 예시: Django 설치 시 sqlparse, asgiref, pytz 등 10개 이상 자동 설치 → 파일이 불필요하게 길어짐
  - 어떤 패키지가 실제로 필요한 것인지 파악하기 어려워짐
  - **권장**: 직접 사용하는 패키지만 수동으로 관리 + 주석으로 용도 표시

- **주석을 활용한 `requirements.txt` 예시**
  ```txt
  # 웹 프레임워크
  django==4.2.7

  # 데이터 분석
  pandas==2.1.3
  numpy==1.26.2

  # API 클라이언트
  requests==2.31.0

  # 환경 변수 관리
  python-dotenv==1.0.0
  ```

### 2.3.2 의존성 버전 고정 전략

- **개발 vs 배포 전략 분리**

| 단계 | 전략 | 이유 |
|------|------|------|
| **개발 시** | 유연한 버전 (`pandas>=1.5`) | 최신 기능/버그 수정 자동 반영, 문제 조기 발견 |
| **배포 시** | 엄격한 버전 (`pandas==1.5.3`) | 예기치 않은 변경으로 인한 장애 방지 |

- **두 개의 의존성 파일 패턴**
  ```txt
  # requirements.txt (프로덕션, 실행에 필요한 패키지만)
  django==4.2.7
  psycopg2-binary==2.9.9
  celery==5.3.4

  # requirements-dev.txt (개발, 개발 도구 포함)
  -r requirements.txt  # 프로덕션 의존성 포함
  pytest==7.4.3
  black==23.11.0
  flake8==6.1.0
  ipython==8.18.1
  ```
  - `-r requirements.txt`: 다른 requirements 파일을 포함하는 구문
  - 개발 환경: `pip install -r requirements-dev.txt` → 프로덕션 + 개발 도구 모두 설치

- **pip-tools 활용 (더 완전한 재현성)**
  - `requirements.in`에 직접 의존성만 작성:
    ```txt
    django>=4.2
    pandas>=2.0
    ```
  - `pip-compile requirements.in` 실행 → `requirements.txt` 자동 생성 (전이 의존성 포함, 정확한 버전)
    ```txt
    # requirements.txt (자동 생성)
    django==4.2.7
    asgiref==3.7.2
    sqlparse==0.4.4
    pandas==2.1.3
    numpy==1.26.2
    python-dateutil==2.8.2
    pytz==2023.3
    six==1.16.0
    ```
  - 버전 업그레이드 시: `requirements.in` 수정 후 `pip-compile` 재실행

- **poetry 활용 (현대적인 통합 도구)**
  - `pyproject.toml` 파일로 의존성 관리:
    ```toml
    [tool.poetry.dependencies]
    python = "^3.9"
    django = "^4.2"
    pandas = "^2.0"

    [tool.poetry.dev-dependencies]
    pytest = "^7.4"
    black = "^23.11"
    ```
  - `poetry install` 실행 → `poetry.lock` 자동 생성 (모든 의존성 정확한 버전으로 고정)
  - `poetry add package_name` → `pyproject.toml`과 `poetry.lock` 자동 업데이트
  - 기능: 의존성 해결 + 가상환경 관리 + 패키지 빌드를 통합 처리

### 2.3.3 의존성 충돌 해결

- **의존성 충돌 발생 조건**
  - 두 패키지가 서로 다른 버전의 동일한 패키지에 의존할 때 발생
  - → 예시: 패키지 A가 `requests>=2.28`, 패키지 B가 `requests<2.27` 요구
    - 두 조건을 동시에 만족하는 requests 버전 없음 → 설치 실패

- **충돌 오류 메시지 예시**
  ```
  ERROR: Cannot install package-a and package-b because these package versions have conflicting dependencies.

  The conflict is caused by:
      package-a 1.0.0 depends on requests>=2.28
      package-b 2.0.0 depends on requests<2.27
  ```

- **해결 방법 4가지**
  1. **패키지 버전 조정**: 충돌하는 패키지 중 하나의 버전을 변경하여 중간 버전 탐색
     - 각 패키지의 변경 이력(CHANGELOG) 확인 후 구버전의 의존성 조건 확인
  2. **대체 패키지 탐색**: 기능이 유사한 다른 패키지로 교체
     - → 예시: requests → httpx, pandas → polars
     - 근본적 해결이지만 코드 수정 필요
  3. **의존성 재협상**: 패키지 개발자에게 의존성 요구사항 완화 요청
     - GitHub 이슈로 요청 → 다음 릴리스에서 범위를 넓혀줄 수 있음
  4. **격리된 환경 사용**: 두 패키지를 각각 다른 가상환경에서 사용
     - 불편하지만 충돌을 완전히 회피 가능

- **충돌 사전 확인 (설치 전 시뮬레이션)**
  ```bash
  pip install --dry-run -r requirements.txt
  ```
  - 실제 설치 없이 충돌 여부를 미리 확인 가능
  - 새로운 패키지 추가 전 습관적으로 실행 권장

- **의존성 트리 시각화 (`pipdeptree`)**
  ```bash
  pip install pipdeptree
  pipdeptree
  ```
  출력 예시:
  ```
  django==4.2.7
    - asgiref [required: >=3.6.0,<4, installed: 3.7.2]
    - sqlparse [required: >=0.3.1, installed: 0.4.4]
  pandas==2.1.3
    - numpy [required: >=1.22.4, installed: 1.26.2]
    - python-dateutil [required: >=2.8.2, installed: 2.8.2]
      - six [required: >=1.5, installed: 1.16.0]
    - pytz [required: >=2020.1, installed: 2023.3]
  ```
  - 어떤 패키지가 어떤 의존성을 요구하는지 한눈에 파악 가능
  - 충돌 발생 시 어떤 경로로 문제의 패키지가 설치되었는지 추적 가능

---

## 2.4 크로스 플랫폼 경로 처리

### 2.4.1 플랫폼별 경로 차이

**표 2.3** 플랫폼별 경로 특징

| 요소 | Windows | macOS/Linux |
|:---|:---|:---|
| 경로 구분자 | `\` (백슬래시) | `/` (슬래시) |
| 루트 표현 | `C:\` (드라이브 문자) | `/` (단일 루트) |
| 홈 디렉토리 | `C:\Users\username` | `/home/username` (Linux) 또는 `/Users/username` (macOS) |
| 대소문자 구분 | 불구분 (기본값) | 구분 (Linux), 불구분 (macOS 기본) |
| 경로 최대 길이 | 260자 (제한) | 4096자 (Linux), 1024자 (macOS) |
| 금지 문자 | `< > : " / \ \| ? *` | `/`, `\0` (null) |

- **하드코딩된 경로의 문제**
  - ⚠ `"C:\\Users\\hong\\data\\file.csv"` → macOS에서 실행 불가
  - ⚠ `"/home/hong/data/file.csv"` → Windows에서 문제 발생
  - AI가 생성한 코드에 플랫폼 특정 경로가 포함되어 있으면 반드시 수정 필요

- **데이터 파일 내 경로 문제**
  - CSV, SQLite 등에 파일 경로가 기록된 경우 → 플랫폼 이동 시 경로 변환 스크립트 필요

- **대소문자 구분 문제**
  - Linux: `file.txt`와 `File.txt`는 다른 파일
  - Windows/macOS(기본): 같은 파일로 취급
  - ⚠ Linux에서 개발한 코드에서 `import MyModule`과 `import mymodule` 혼용 → Windows에서 `ModuleNotFoundError`

- **경로 길이 제한 문제 (Windows)**
  - Windows 기본 MAX_PATH: 260자
  - 깊은 디렉토리 구조에서 파일 생성/열기 불가 발생 가능
  - Python 3.6+와 Windows 10에서 개선되었지만 여전히 주의 필요

### 2.4.2 pathlib으로 경로 다루기

- **`pathlib`** = Python 3.4부터 표준 라이브러리 포함, 객체 지향 방식으로 경로를 다루는 모듈
  - OS에 관계없이 동일하게 동작, 직관적, 가독성 좋음

- **핵심 패턴: 스크립트 위치 기준 상대 경로 구성**
  ```python
  from pathlib import Path

  # 현재 스크립트의 디렉토리
  script_dir = Path(__file__).parent.resolve()

  # 프로젝트 루트 (스크립트가 src/ 아래에 있다면)
  project_root = script_dir.parent

  # 데이터 디렉토리
  data_dir = project_root / "data" / "output"
  data_dir.mkdir(parents=True, exist_ok=True)
  ```
  - `Path(__file__).parent`: 현재 실행 중인 스크립트의 디렉토리
  - `.resolve()`: 심볼릭 링크(바로가기) 해석 + 절대 경로로 변환
  - 스크립트를 어느 디렉토리에서 실행하든 항상 스크립트 자신의 위치 기준으로 계산

- **`/` 연산자의 장점**
  - `base_dir / "data" / "output"` → OS에 맞는 경로 구분자를 자동으로 사용
  - Windows: `C:\project\data\output` / Linux: `/home/user/project/data/output`

- **`mkdir()` 주요 옵션**
  - `parents=True`: 상위 디렉토리가 없으면 함께 생성
  - `exist_ok=True`: 디렉토리가 이미 존재해도 오류 발생하지 않음
  - → 두 옵션을 함께 사용하면 경로 존재 여부 확인 없이 안전하게 디렉토리 생성 가능

- **pathlib의 주요 메서드와 속성**
  ```python
  from pathlib import Path

  p = Path("data") / "results" / "output.txt"

  # 경로 정보
  p.name          # "output.txt" (파일명)
  p.stem          # "output" (확장자 제외 파일명)
  p.suffix        # ".txt" (확장자)
  p.parent        # Path("data/results") (상위 디렉토리)
  p.parents[0]    # Path("data/results") (상위)
  p.parents[1]    # Path("data") (상위의 상위)

  # 경로 변환
  p.absolute()    # 절대 경로
  p.resolve()     # 심볼릭 링크 해석 + 절대 경로
  p.as_posix()    # 항상 / 구분자 사용 ("data/results/output.txt")
  p.as_uri()      # file:// URI 형식

  # 경로 존재 확인
  p.exists()      # 경로가 존재하는가?
  p.is_file()     # 파일인가?
  p.is_dir()      # 디렉토리인가?
  p.is_symlink()  # 심볼릭 링크인가?

  # 파일 작업
  p.read_text()   # 파일 내용 읽기 (텍스트)
  p.read_bytes()  # 파일 내용 읽기 (바이너리)
  p.write_text("content")  # 파일 쓰기 (텍스트)
  p.write_bytes(b"data")   # 파일 쓰기 (바이너리)

  # 디렉토리 작업
  p.mkdir(parents=True, exist_ok=True)  # 디렉토리 생성
  p.rmdir()       # 빈 디렉토리 삭제
  p.unlink()      # 파일 삭제

  # 글로빙 (패턴으로 파일 목록 검색)
  p.glob("*.txt")      # 현재 디렉토리의 txt 파일
  p.glob("**/*.txt")   # 하위 모든 디렉토리의 txt 파일 (재귀)
  p.rglob("*.txt")     # glob("**/*.txt")와 동일
  ```

- **실무 예시: 데이터 파일 일괄 처리**
  ```python
  from pathlib import Path
  import pandas as pd

  # 모든 CSV 파일 찾기
  data_dir = Path("data")
  csv_files = list(data_dir.glob("*.csv"))

  for csv_file in csv_files:
      df = pd.read_csv(csv_file)
      # 처리...

      # 결과를 results/ 디렉토리에 저장
      output_file = Path("results") / csv_file.name
      df.to_csv(output_file, index=False)
  ```
  - Windows, macOS, Linux 모두 동일하게 작동

### 2.4.3 os.path vs pathlib 비교

**표 2.4** os.path와 pathlib 비교

| 작업 | os.path | pathlib |
|:---|:---|:---|
| 디렉토리 이름 | `os.path.dirname(path)` | `path.parent` |
| 파일 이름 | `os.path.basename(path)` | `path.name` |
| 경로 조합 | `os.path.join(a, b, c)` | `a / b / c` |
| 존재 확인 | `os.path.exists(path)` | `path.exists()` |
| 파일 여부 | `os.path.isfile(path)` | `path.is_file()` |
| 디렉토리 여부 | `os.path.isdir(path)` | `path.is_dir()` |
| 절대 경로 | `os.path.abspath(path)` | `path.resolve()` |
| 파일 크기 | `os.path.getsize(path)` | `path.stat().st_size` |
| 수정 시간 | `os.path.getmtime(path)` | `path.stat().st_mtime` |

- **os.path 예시 (구버전 스타일)**
  ```python
  import os

  data_file = os.path.join("data", "input", "file.csv")
  output_dir = os.path.join("data", "output")
  if not os.path.exists(output_dir):
      os.makedirs(output_dir)

  for filename in os.listdir("data"):
      if filename.endswith(".csv"):
          filepath = os.path.join("data", filename)
  ```

- **pathlib 예시 (현대적 스타일)**
  ```python
  from pathlib import Path

  data_file = Path("data") / "input" / "file.csv"
  output_dir = Path("data") / "output"
  output_dir.mkdir(parents=True, exist_ok=True)

  for filepath in Path("data").glob("*.csv"):
      pass  # 처리...
  ```

- **비교 결론**
  - `pathlib`이 더 간결하고 읽기 쉬움
  - `/` 연산자가 `os.path.join()` 보다 시각적으로 명확
  - `mkdir(parents=True, exist_ok=True)` 한 줄로 디렉토리 생성 처리
  - **권장**: 새 코드는 `pathlib`, 기존 `os.path` 코드는 점진적으로 마이그레이션
  - AI 요청 시 **"pathlib을 사용해서"** 명시하면 크로스 플랫폼 호환 코드 받을 가능성 높아짐

- **혼용 시 주의사항**
  - Python 3.6+에서는 `open()`, `os.rename()` 등이 `Path` 객체를 직접 받음
  - 일부 서드파티 라이브러리는 `Path` 객체를 미지원할 수 있음 → `str(path)`로 변환하여 전달

### 2.4.4 환경 변수와 홈 디렉토리

- **홈 디렉토리 접근 (크로스 플랫폼)**
  ```python
  from pathlib import Path

  home = Path.home()
  # Windows: C:\Users\username
  # macOS: /Users/username
  # Linux: /home/username

  config_file = home / ".myapp" / "config.json"
  config_file.parent.mkdir(parents=True, exist_ok=True)
  ```
  - `Path.home()`: 현재 사용자의 홈 디렉토리를 OS에 관계없이 반환

- **환경 변수로 경로 지정**
  ```python
  import os
  from pathlib import Path

  data_dir = os.getenv("DATA_DIR", "data")  # 환경 변수 없으면 기본값 "data" 사용
  data_path = Path(data_dir) / "input"
  ```
  - 코드 수정 없이 실행 환경에서 경로를 설정 가능
  - → 예시: 개발 환경 = 로컬 디렉토리, 운영 환경 = 공유 스토리지

---

## 2.5 실습: "실행-검증-산출물 저장" 템플릿 만들기

### 2.5.1 프로젝트 구조

```
practice/chapter2/
├── code/
│   ├── 2-5-template.py      # 표준 템플릿
│   ├── requirements.txt     # 의존성 목록
│   └── README.md            # 실행 가이드
└── data/
    ├── input/               # 입력 데이터
    └── output/              # 산출물 저장
```

- `code/` 디렉토리: 실행 가능한 스크립트 + 의존성 파일
- `data/` 디렉토리: 입력 데이터와 실행 결과를 분리 관리
  - → 코드와 데이터가 명확히 구분되어 유지보수 용이

### 2.5.2 표준 템플릿의 네 가지 구성요소

- **템플릿 개요**
  - 파일: `2-5-template.py` (302줄)
  - 4단계로 구성, 각 단계는 독립적인 함수 → 재사용 + 테스트 용이

- **① 환경 검증 `verify_environment()`**
  - Python 버전 최소 요구사항 충족 여부 + 필요한 패키지 설치 여부 확인
  - 통과 못 하면 스크립트 실행 중단 + 명확한 오류 메시지 출력
  ```python
  import sys

  def verify_python_version(min_version=(3, 9)):
      """Python 버전 확인"""
      current = sys.version_info[:2]
      if current < min_version:
          print(f"✗ Python {min_version[0]}.{min_version[1]}+ 필요, 현재 {current[0]}.{current[1]}")
          return False
      print(f"✓ Python {current[0]}.{current[1]} 충족")
      return True
  ```
  - `sys.version_info[:2]`: 메이저.마이너 버전만 추출 (예: `(3, 11)`)
  - 튜플 비교는 왼쪽부터 순서대로: `(3, 9) < (3, 11)` → `True`
  - 이점: 실행 중간에 실패하는 것보다 조기 차단이 문제 파악에 빠름

- **② 경로 설정 `setup_paths()`**
  - 스크립트 위치 기준으로 입출력 경로를 자동 구성
  - 어느 디렉토리에서 실행하든 동일한 상대 경로 유지
  ```python
  from pathlib import Path

  def setup_paths():
      """크로스 플랫폼 경로 설정"""
      script_dir = Path(__file__).parent.resolve()
      output_dir = script_dir.parent / "data" / "output"
      output_dir.mkdir(parents=True, exist_ok=True)
      return {"script_dir": script_dir, "output_dir": output_dir}
  ```
  - 경로 딕셔너리로 반환 → `paths["output_dir"]`처럼 명확히 참조 가능 (오타 위험 감소)
  - 경로 검증 루프로 모든 경로가 올바르게 설정되었는지 확인
    - 디스크 공간 부족이나 권한 문제로 디렉토리 생성 실패 시 이 단계에서 발견

- **③ 실행 및 검증 `run_task()`**
  - 핵심 로직 실행 + 결과가 기대치를 충족하는지 확인
  ```python
  def run_task(paths):
      """실제 작업 실행"""
      test_path = paths["output_dir"] / "subdir" / "file.txt"
      posix_path = test_path.as_posix()  # 항상 / 사용
      return {"posix": posix_path, "parts": list(test_path.parts)}
  ```
  - `as_posix()`: Windows에서도 `/`를 사용하는 경로를 반환
    - JSON에 경로를 저장할 때 유용 (백슬래시 이스케이프 문제 방지)
  - `test_path.parts`: 경로를 구성요소로 분해
    - → 예시: `/Users/hong/data/file.txt` → `('/', 'Users', 'hong', 'data', 'file.txt')`

- **④ 산출물 저장 `save_output()`**
  - 실행 결과를 타임스탬프가 포함된 파일명으로 저장 → 실행 이력 추적 가능
  ```python
  import json
  from datetime import datetime

  def save_output(result, paths):
      """결과를 JSON으로 저장"""
      timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
      output_file = paths["output_dir"] / f"result_{timestamp}.json"
      with open(output_file, "w", encoding="utf-8") as f:
          json.dump(result, f, indent=2, ensure_ascii=False)
      return output_file
  ```
  - `strftime("%Y%m%d_%H%M%S")`: 예시 → `20250101_143022`
    - 같은 스크립트를 여러 번 실행해도 파일이 덮어씌워지지 않음
  - `ensure_ascii=False`: 한글 등 유니코드 문자를 그대로 저장
    - 기본값 `True`는 `\uXXXX` 형태로 이스케이프하여 가독성 저하
  - `indent=2`: JSON 들여쓰기 → 사람이 읽기 쉬운 형태
  - `Path` 객체 → `str(v)` 변환: JSON은 `pathlib.Path` 타입을 직접 지원하지 않음
  - 환경 정보(`python_version`, `platform`)를 함께 저장 → 나중에 재현 조건 확인 가능

### 2.5.3 실습 실행

```bash
cd practice/chapter2
python3 -m venv venv
source venv/bin/activate  # macOS/Linux
# venv\Scripts\activate   # Windows
pip install -r code/requirements.txt
python3 code/2-5-template.py
```

- **실행 결과 출력 예시**
  ```
  ============================================================
  실행-검증-산출물 저장 표준 템플릿
  ============================================================
  [환경 검증] Python 버전 확인
    현재 버전: 3.13.7 (main, Aug 14 2025, ...)
    최소 요구: 3.9
    결과: ✓ 충족

  [경로 설정] 크로스 플랫폼 경로 구성
    script_dir: /Users/.../practice/chapter2/code [✓]
    base_dir: /Users/.../practice/chapter2 [✓]
    output_dir: /Users/.../practice/chapter2/data/output [✓]

  [실행] 크로스 플랫폼 경로 처리 테스트
    POSIX 경로: .../practice/chapter2/data/output/subdir/file.txt
    Native 경로: .../practice/chapter2/data/output/subdir/file.txt
    경로 구성요소: ('/', 'Users', ..., 'file.txt')

  [산출물 저장] JSON 파일 생성
    파일 경로: /Users/.../practice/chapter2/data/output/result_20260102_062738.json
    파일 크기: 1717 bytes
  ```
  - macOS/Linux: POSIX 경로와 Native 경로가 동일하게 `/` 사용
  - Windows: Native 경로 = `C:\...` 형태, POSIX 경로 = `C:/.../` 형태
  - `[✓]`: 경로 존재 확인 / `[✗]`: 디렉토리 생성 실패 또는 권한 문제

### 2.5.4 산출물 확인

- **저장 파일 구조 예시**: `practice/chapter2/data/output/result_20260102_062738.json`
  ```json
  {
    "timestamp": "2026-01-02T06:27:38.123456",
    "python_version": "3.13.7",
    "platform": "darwin",
    "paths": {
      "script_dir": "/Users/.../practice/chapter2/code",
      "base_dir": "/Users/.../practice/chapter2",
      "output_dir": "/Users/.../practice/chapter2/data/output"
    },
    "result": {
      "posix": ".../practice/chapter2/data/output/subdir/file.txt",
      "native": ".../practice/chapter2/data/output/subdir/file.txt",
      "parts": ["/", "Users", ..., "file.txt"]
    }
  }
  ```
- `timestamp`: ISO 8601 형식, 마이크로초 단위까지 기록 → 동일 초에 여러 번 실행해도 구분 가능
- `platform` 필드: `darwin` = macOS / `linux` = Linux / `win32` = Windows
- `paths`: 모든 경로가 절대 경로로 기록 → 어느 디렉토리에서 실행되었는지 명확

**표 2.5** 실행 결과 요약

| 항목 | 값 |
|:---|:---|
| Python 버전 | 3.13.7 |
| 플랫폼 | darwin (macOS) |
| 산출물 파일 | result_20260102_062738.json |
| 파일 크기 | 1717 bytes |

### 2.5.5 템플릿 커스터마이징

- 새 실습 스크립트 작성 시: `2-5-template.py`를 복사 후 `run_task()` 함수만 수정
  - 환경 검증과 경로 설정은 그대로 재사용 가능

- **예시 1: 데이터 생성 및 CSV 저장**
  ```python
  def run_task(paths):
      """학생 성적 데이터 생성 및 저장"""
      import pandas as pd
      data = {
          "이름": ["홍길동", "김철수", "이영희", "박민수"],
          "국어": [85, 92, 88, 79]
      }
      df = pd.DataFrame(data)
      csv_file = paths["output_dir"] / "students.csv"
      df.to_csv(csv_file, index=False, encoding="utf-8-sig")
      return {"rows": len(df), "csv_path": str(csv_file)}
  ```
  - `encoding="utf-8-sig"`: Excel에서 한글이 깨지지 않도록 BOM(Byte Order Mark)을 포함
  - pandas 사용 시 `requirements.txt`에 `pandas` 추가 필요

- **예시 2: 웹 API 호출 및 결과 저장**
  ```python
  def run_task(paths):
      """GitHub API 호출 및 저장"""
      import requests
      url = "https://api.github.com/repos/python/cpython"
      response = requests.get(url)
      response.raise_for_status()
      data = response.json()
      return {
          "repo_name": data["name"],
          "stars": data["stargazers_count"]
      }
  ```
  - `response.raise_for_status()`: HTTP 오류 발생 시 예외 발생
  - `requests` 라이브러리 → `requirements.txt`에 추가 필요

### 2.5.6 템플릿의 장점과 실전 활용

- **이점 5가지**
  1. **일관성**: 모든 실습 스크립트가 동일한 구조를 따라 이해하기 쉬움
  2. **재현성**: 환경 정보와 실행 조건이 자동으로 기록되어 나중에 재현 가능
  3. **디버깅**: 경로와 환경이 명시적으로 검증되어 문제 발생 시 원인 파악 빠름
  4. **협업**: 팀원 모두 같은 템플릿 사용 → 코드 리뷰와 유지보수 용이
  5. **AI 협업**: "2-5-template.py처럼 작성해줘"라고 요청 → 표준을 따르는 코드 생성 가능

- **실전 사용 방법**
  - 프로젝트의 `scripts/` 또는 `templates/` 디렉토리에 템플릿을 두고, 새 스크립트 작성 시 복사하여 시작
  - Git에 템플릿 커밋 → 팀 전체가 동일한 표준 공유

- **AI에게 효과적으로 요청하는 방법**
  > "2-5-template.py와 같은 구조로 iris 데이터셋을 불러와서 간단한 분류 모델을 학습하고 결과를 JSON으로 저장하는 스크립트를 작성해줘. pathlib을 사용하고, 환경 검증과 산출물 저장 부분은 템플릿과 동일하게 해줘."
  - 구체적인 요청 = AI가 템플릿 구조를 따르면서 새로운 기능을 추가한 코드를 생성할 가능성 높아짐

---

## 핵심정리

- **가상환경**: `python3 -m venv venv`로 프로젝트별 격리된 Python 환경을 생성한다
- **의존성 고정**: `requirements.txt`에 패키지와 버전을 명시하여 재현 가능한 환경을 보장한다
- **크로스 플랫폼 경로**: `pathlib.Path`와 `/` 연산자를 사용하여 운영체제에 관계없이 동작하는 코드를 작성한다
- **표준 템플릿**: 환경 검증 → 경로 설정 → 실행/검증 → 산출물 저장의 4단계 구조를 따른다
- **AI 협업 팁**: AI에게 코드를 요청할 때 "pathlib 사용", "requirements.txt 포함" 등을 명시하면 재현 가능한 코드를 받을 수 있다

---

## 실습 파일 요약

| 파일 | 경로 | 설명 |
|:---|:---|:---|
| 표준 템플릿 | practice/chapter2/code/2-5-template.py | 실행-검증-산출물 저장 표준 구조 |
| 의존성 파일 | practice/chapter2/code/requirements.txt | 필요 패키지 목록 (표준 라이브러리만 사용) |
| 실행 가이드 | practice/chapter2/code/README.md | 실습 실행 방법 안내 |
| 산출물 예시 | practice/chapter2/data/output/result_*.json | 실행 결과 JSON 파일 |

---

## 다음 장 예고

- 3장에서는 모델 컨텍스트 프로토콜(MCP)의 개념과 도구/리소스 설계를 학습한다
- AI 에이전트가 외부 도구와 상호작용하는 표준 방법을 이해하고, MCP 서버의 핵심 구성 요소를 설계한다

---

## 참고문헌

Python Software Foundation. (2024). venv — Creation of virtual environments. Python 3.13 Documentation. https://docs.python.org/3/library/venv.html

Python Software Foundation. (2024). pathlib — Object-oriented filesystem paths. Python 3.13 Documentation. https://docs.python.org/3/library/pathlib.html

Python Packaging Authority. (2024). Installing packages using pip and virtual environments. Python Packaging User Guide. https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/

Van Rossum, G., & Warsaw, B. (2013). PEP 405 – Python Virtual Environments. Python Enhancement Proposals. https://peps.python.org/pep-0405/
