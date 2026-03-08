# 제2장: 로컬 개발 환경과 재현 가능한 실행

## 학습목표

1. Python 가상환경을 생성하고 의존성을 고정하여 실행 재현성을 확보할 수 있다
2. 크로스 플랫폼에서 동작하는 경로 처리와 스크립트 구조를 설계할 수 있다
3. "실행-검증-산출물 저장" 패턴의 표준 템플릿을 활용할 수 있다
4. AI가 생성한 코드의 실행 환경 문제를 진단하고 해결할 수 있다

---

## 2.1 왜 재현 가능한 환경이 중요한가

"내 컴퓨터에서는 되는데요..." 이 말을 들어본 적이 있다면, 재현 가능한 환경이 왜 중요한지 이미 경험한 셈이다. AI와 협업하여 코드를 작성할 때 이 문제는 더욱 빈번해진다. AI가 제안하는 코드가 특정 라이브러리 버전이나 운영체제에 의존하는 경우, 다른 환경에서 실행하면 예상치 못한 오류가 발생하기 때문이다.

### 2.1.1 AI 코드 생성의 환경 의존성 문제

AI 코딩 어시스턴트가 생성하는 코드에는 보이지 않는 가정이 숨어 있다. 예를 들어, GitHub Copilot이 pandas를 사용하는 코드를 제안할 때, 어떤 버전의 pandas를 기준으로 작성했는지 명시하지 않는다. pandas 1.x에서 자주 사용하던 `df.append()` 메서드는 2.0 버전에서 제거되었다. AI가 학습한 시점의 코드 패턴이 현재 설치된 라이브러리 버전과 맞지 않으면, 동일한 코드도 전혀 다른 결과를 낳는다.

운영체제 차이도 문제가 된다. Windows에서는 경로 구분자로 백슬래시(`\`)를 사용하고, macOS와 Linux에서는 슬래시(`/`)를 사용한다. AI가 특정 플랫폼을 기준으로 경로를 하드코딩한 코드를 생성하면, 다른 플랫폼에서는 "파일을 찾을 수 없습니다"라는 오류가 발생한다.

### 2.1.2 재현 가능한 실행의 핵심 요소

재현 가능한 실행 환경을 구축하려면 네 가지 요소를 갖춰야 한다. 첫째, 격리된 실행 환경이다. Python 가상환경을 사용하면 프로젝트마다 독립적인 패키지 집합을 유지할 수 있다. 둘째, 고정된 의존성이다. `requirements.txt` 파일에 필요한 패키지와 정확한 버전을 기록해 두면, 누구든 동일한 환경을 재현할 수 있다. 셋째, 일관된 경로 처리다. `pathlib`을 사용하면 운영체제에 관계없이 동일하게 동작하는 경로 코드를 작성할 수 있다. 넷째, 환경 설정의 분리다. 민감한 정보나 환경별로 달라지는 설정을 코드에서 분리하면 보안과 유연성을 모두 확보할 수 있다.

**사례: pandas 버전 불일치 문제**

한 프로젝트에서 AI가 제안한 코드에 `df.append(row)`가 포함되어 있었다. pandas 1.x를 사용하던 개발자의 환경에서는 정상 동작했지만, pandas 2.0으로 업그레이드한 동료의 환경에서는 `AttributeError`가 발생했다. `requirements.txt`에 `pandas==1.5.3`을 명시하고 나서야 모든 팀원이 동일한 결과를 얻을 수 있었다.

---

## 2.2 Python 가상환경 완전 정복

### 2.2.1 가상환경이란 무엇인가

Python 가상환경은 프로젝트별로 독립적인 Python 실행 환경을 제공하는 메커니즘이다. 시스템에 설치된 Python(시스템 Python)과 별도로, 각 프로젝트에 필요한 패키지만 설치된 격리된 공간을 만들 수 있다. 

왜 격리가 필요할까? 프로젝트 A에서는 requests 2.25를, 프로젝트 B에서는 requests 2.31을 사용한다고 하자. 시스템 Python에 두 버전을 동시에 설치할 수는 없다. 가상환경을 사용하면 프로젝트 A의 가상환경에는 2.25를, 프로젝트 B의 가상환경에는 2.31을 각각 설치하여 충돌 없이 사용할 수 있다.

가상환경이 활성화되어 있는지 확인하는 방법도 알아두면 좋다. Python 내부적으로 `sys.prefix`와 `sys.base_prefix` 값이 다르면 가상환경이 활성화된 상태다. 아래 코드로 간단히 확인할 수 있다.

```python
import sys
in_venv = sys.prefix != sys.base_prefix
print(f"가상환경 활성: {in_venv}")
```

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

### 2.2.2 venv로 가상환경 만들기

Python 3.3부터 표준 라이브러리에 포함된 `venv` 모듈을 사용하면 별도의 설치 없이 가상환경을 생성할 수 있다. 가상환경 생성과 활성화 명령은 운영체제에 따라 조금 다르다.

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

가상환경이 활성화되면 터미널 프롬프트 앞에 `(venv)`가 표시된다. 이 상태에서 `pip install`로 설치하는 패키지는 모두 이 가상환경에만 설치된다. 작업을 마치고 가상환경을 비활성화하려면 `deactivate` 명령을 입력한다.

가상환경 디렉토리(보통 `venv/`)는 소스 컨트롤에 포함하지 않는다. 대신 `requirements.txt`를 커밋하여, 다른 사람이 같은 환경을 재현할 수 있도록 한다. `.gitignore` 파일에 `venv/`를 추가해 두는 것이 좋다.

### 2.2.3 conda vs venv: 언제 무엇을 선택할까

**표 2.1** venv와 conda 비교

| 기준 | venv | conda |
|:---|:---|:---|
| 설치 | Python 표준 라이브러리 (별도 설치 불필요) | Anaconda/Miniconda 설치 필요 |
| 패키지 관리 | pip만 사용 | conda + pip 혼용 가능 |
| 비Python 의존성 | 지원 안 함 | CUDA, C 라이브러리 등 지원 |
| 환경 복제 | requirements.txt | environment.yml (더 완전한 복제) |
| 용도 | 순수 Python 프로젝트 | 데이터 과학, ML (GPU 활용 시) |

대부분의 프로젝트에서는 venv로 충분하다. CUDA를 사용하는 딥러닝 프로젝트나 NumPy/SciPy의 특정 빌드가 필요한 데이터 과학 프로젝트에서는 conda가 더 편리하다. 이 책의 실습에서는 모든 환경에서 동일하게 사용할 수 있는 venv를 기준으로 설명한다.

---

## 2.3 의존성 관리와 버전 고정

### 2.3.1 requirements.txt 작성법

`requirements.txt`는 프로젝트에 필요한 Python 패키지 목록을 담는 파일이다. 가장 간단한 형태는 패키지 이름만 나열하는 것이지만, 재현 가능한 환경을 위해서는 버전까지 명시해야 한다.

버전 지정 방식에는 세 가지가 있다. 첫째, `==`는 정확한 버전을 지정한다. `pandas==1.5.3`은 반드시 1.5.3 버전만 설치한다. 둘째, `>=`는 최소 버전을 지정한다. `requests>=2.25.0`은 2.25.0 이상이면 어떤 버전이든 설치한다. 셋째, `~=`는 호환 버전을 지정한다. `numpy~=1.24.0`은 1.24.x 범위 내의 최신 버전을 설치한다.

`pip freeze > requirements.txt` 명령으로 현재 환경의 모든 패키지를 내보낼 수 있지만, 이 방법은 권장하지 않는다. 직접 설치하지 않은 의존성 패키지까지 모두 포함되어 파일이 불필요하게 길어지고, 나중에 어떤 패키지가 실제로 필요한 것인지 파악하기 어려워진다. 직접 사용하는 패키지만 수동으로 관리하는 것이 좋다.

### 2.3.2 의존성 버전 고정 전략

개발 단계와 배포 단계에서 버전 지정 전략을 다르게 가져가면 유연성과 안정성을 모두 확보할 수 있다.

개발 시에는 유연한 버전을 사용한다. `pandas>=1.5`처럼 최소 버전만 지정하면, 새로운 기능이나 버그 수정이 포함된 최신 버전을 자연스럽게 받아볼 수 있다. 배포 시에는 엄격한 버전을 사용한다. `pandas==1.5.3`처럼 정확한 버전을 고정하면, 운영 환경에서 예기치 않은 변경으로 인한 장애를 방지할 수 있다.

보다 완전한 재현성이 필요하다면 `pip-tools`나 `poetry` 같은 도구를 사용할 수 있다. 이들 도구는 직접 의존성뿐 아니라 전이 의존성(의존성의 의존성)까지 lock 파일에 기록하여, 동일한 패키지 구성을 완벽하게 재현할 수 있도록 해준다.

### 2.3.3 의존성 충돌 해결

두 패키지가 서로 다른 버전의 동일한 패키지에 의존할 때 충돌이 발생한다. 예를 들어, 패키지 A가 `requests>=2.28`을, 패키지 B가 `requests<2.27`을 요구하면 둘을 동시에 만족하는 requests 버전이 없어 설치에 실패한다.

충돌이 발생하면 pip가 오류 메시지로 어떤 패키지들이 충돌하는지 알려준다. 해결 방법으로는 충돌을 일으키는 패키지 중 하나의 버전을 변경하거나, 두 패키지가 모두 만족하는 중간 버전을 찾거나, 필요하다면 충돌하는 패키지 중 하나를 포기해야 할 수도 있다. `pip install --dry-run` 옵션을 사용하면 실제 설치 전에 충돌 여부를 미리 확인할 수 있다.

---

## 2.4 크로스 플랫폼 경로 처리

### 2.4.1 플랫폼별 경로 차이

Windows와 Unix 계열(macOS, Linux) 운영체제는 파일 경로를 표현하는 방식이 다르다. 

**표 2.2** 플랫폼별 경로 특징

| 요소 | Windows | macOS/Linux |
|:---|:---|:---|
| 경로 구분자 | `\` (백슬래시) | `/` (슬래시) |
| 루트 표현 | `C:\` | `/` |
| 홈 디렉토리 | `C:\Users\username` | `/home/username` 또는 `/Users/username` |

코드에 `"C:\\Users\\hong\\data\\file.csv"`처럼 Windows 경로를 하드코딩하면, macOS에서는 실행할 수 없다. 반대로 `"/home/hong/data/file.csv"`를 하드코딩하면 Windows에서 문제가 된다. AI가 생성한 코드에 플랫폼 특정 경로가 포함되어 있다면, 반드시 크로스 플랫폼 방식으로 수정해야 한다.

### 2.4.2 pathlib으로 경로 다루기

Python 3.4부터 표준 라이브러리에 포함된 `pathlib` 모듈을 사용하면 운영체제에 관계없이 동일하게 동작하는 경로 코드를 작성할 수 있다. 가장 중요한 패턴은 스크립트 파일 위치를 기준으로 상대 경로를 구성하는 것이다.

```python
from pathlib import Path

base_dir = Path(__file__).parent.resolve()
data_dir = base_dir / "data" / "output"
data_dir.mkdir(parents=True, exist_ok=True)
```

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

`Path(__file__).parent`는 현재 스크립트가 위치한 디렉토리를 반환한다. `.resolve()`를 호출하면 심볼릭 링크를 해석하고 절대 경로로 변환한다. `/` 연산자를 사용하여 경로를 조합하면, 운영체제에 맞는 경로 구분자가 자동으로 사용된다.

`mkdir(parents=True, exist_ok=True)`는 디렉토리 생성 시 자주 사용하는 옵션이다. `parents=True`는 상위 디렉토리가 없으면 함께 생성하고, `exist_ok=True`는 디렉토리가 이미 존재해도 오류를 발생시키지 않는다.

### 2.4.3 os.path vs pathlib 비교

레거시 코드에서는 `os.path` 모듈을 사용한 경로 처리를 자주 볼 수 있다. `os.path`도 크로스 플랫폼을 지원하지만, `pathlib`이 더 읽기 쉽고 현대적인 방식이다.

**표 2.3** os.path와 pathlib 비교

| os.path | pathlib |
|:---|:---|
| `os.path.dirname(path)` | `path.parent` |
| `os.path.basename(path)` | `path.name` |
| `os.path.join(a, b)` | `a / b` |
| `os.path.exists(path)` | `path.exists()` |
| `os.path.isfile(path)` | `path.is_file()` |
| `os.path.isdir(path)` | `path.is_dir()` |

새로운 코드를 작성할 때는 `pathlib`을 사용하고, 기존 `os.path` 코드를 유지보수할 때는 점진적으로 `pathlib`으로 마이그레이션하는 것을 권장한다. AI 코딩 어시스턴트에게 코드를 요청할 때도 "pathlib을 사용해서"라고 명시하면 크로스 플랫폼 호환 코드를 받을 가능성이 높아진다.

---

## 2.5 실습: "실행-검증-산출물 저장" 템플릿 만들기

이 절에서는 앞서 배운 가상환경, 의존성 관리, 크로스 플랫폼 경로 처리를 통합한 표준 스크립트 템플릿을 만들어 본다. 이 템플릿은 AI와 협업하여 코드를 작성할 때 기본 골격으로 활용할 수 있다.

### 2.5.1 프로젝트 구조

실습 프로젝트는 다음 구조로 구성된다.

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

`code/` 디렉토리에는 실행 가능한 스크립트와 의존성 파일을 둔다. `data/` 디렉토리는 입력 데이터와 실행 결과를 분리하여 관리한다. 이 구조를 따르면 코드와 데이터가 명확히 구분되어 유지보수가 용이해진다.

### 2.5.2 표준 템플릿의 네 가지 구성요소

표준 템플릿은 네 가지 단계로 구성된다. 첫째, 환경 검증이다. Python 버전이 최소 요구사항을 충족하는지, 필요한 패키지가 설치되어 있는지 확인한다. 둘째, 경로 설정이다. 스크립트 위치를 기준으로 입출력 경로를 자동 구성한다. 셋째, 실행 및 검증이다. 핵심 로직을 실행하고 결과가 기대치를 충족하는지 확인한다. 넷째, 산출물 저장이다. 실행 결과를 타임스탬프가 포함된 파일명으로 저장하여 실행 이력을 추적한다.

환경 검증 함수의 핵심 코드는 다음과 같다.

```python
def verify_python_version(min_version=(3, 9)):
    current = sys.version_info[:2]
    return current >= min_version
```

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

### 2.5.3 실습 실행

터미널에서 다음 명령으로 실습을 실행한다.

```bash
cd practice/chapter2
python3 -m venv venv
source venv/bin/activate  # macOS/Linux
# venv\Scripts\activate   # Windows
pip install -r code/requirements.txt
python3 code/2-5-template.py
```

실행하면 다음과 같은 출력을 확인할 수 있다.

```
============================================================
실행-검증-산출물 저장 표준 템플릿
============================================================
[환경 검증] Python 버전 확인
  현재 버전: 3.13.7 (main, Aug 14 2025, 11:12:11) [Clang 16.0.0 (clang-1600.0.26.6)]
  최소 요구: 3.9
  결과: ✓ 충족

[경로 설정] 크로스 플랫폼 경로 구성
  script_dir: /Users/.../practice/chapter2/code [✓]
  base_dir: /Users/.../practice/chapter2 [✓]
  output_dir: /Users/.../practice/chapter2/data/output [✓]

[산출물 저장] JSON 파일 생성
  파일 경로: /Users/.../practice/chapter2/data/output/result_20260102_062738.json
  파일 크기: 1717 bytes
```

### 2.5.4 산출물 확인

실행 결과는 `practice/chapter2/data/output/result_20260102_062738.json` 파일로 저장된다. 파일에는 실행 환경 정보, 경로 검증 결과, 크로스 플랫폼 테스트 결과가 JSON 형식으로 기록된다.

**표 2.4** 실행 결과 요약

| 항목 | 값 |
|:---|:---|
| Python 버전 | 3.13.7 |
| 플랫폼 | macOS-14.3.1-arm64-arm-64bit-Mach-O |
| 산출물 파일 | result_20260102_062738.json |
| 파일 크기 | 1717 bytes |

산출물 JSON 파일의 주요 내용은 다음과 같다. 모든 경로가 절대 경로로 기록되어 있으며, `exists: true`로 디렉토리가 정상 생성되었음을 확인할 수 있다. `cross_platform_test` 섹션의 `as_posix` 필드는 `pathlib`이 어떻게 플랫폼 독립적인 경로를 제공하는지 보여준다.

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

## 참고문헌

Python Software Foundation. (2024). venv — Creation of virtual environments. Python 3.13 Documentation. https://docs.python.org/3/library/venv.html

Python Software Foundation. (2024). pathlib — Object-oriented filesystem paths. Python 3.13 Documentation. https://docs.python.org/3/library/pathlib.html

Python Packaging Authority. (2024). Installing packages using pip and virtual environments. Python Packaging User Guide. https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/

Van Rossum, G., & Warsaw, B. (2013). PEP 405 – Python Virtual Environments. Python Enhancement Proposals. https://peps.python.org/pep-0405/
