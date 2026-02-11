# 제2장: 로컬 개발 환경과 재현 가능한 실행

## 학습목표

1. Python 가상환경을 생성하고 의존성을 고정하여 실행 재현성을 확보할 수 있다
2. 크로스 플랫폼에서 동작하는 경로 처리와 스크립트 구조를 설계할 수 있다
3. "실행-검증-산출물 저장" 패턴의 표준 템플릿을 활용할 수 있다
4. AI가 생성한 코드의 실행 환경 문제를 진단하고 해결할 수 있다

---

## 2.1 왜 재현 가능한 환경이 중요한가

"내 컴퓨터에서는 되는데요..." 이 말을 들어본 적이 있다면, 재현 가능한 환경이 왜 중요한지 이미 경험한 셈이다. AI와 협업하여 코드를 작성할 때 이 문제는 더욱 빈번해진다. AI가 제안하는 코드가 특정 라이브러리 버전이나 운영체제에 의존하는 경우, 다른 환경에서 실행하면 예상치 못한 오류가 발생하기 때문이다.

소프트웨어 개발에서 재현성(reproducibility)이란 동일한 코드를 다른 환경에서 실행했을 때 동일한 결과를 얻을 수 있는 성질을 말한다. 이는 단순히 "코드가 실행된다"는 수준을 넘어서, 정확히 같은 버전의 라이브러리, 같은 운영체제 조건, 같은 데이터 형식에서 동일한 출력을 보장하는 것을 의미한다. 특히 AI 기반 개발 도구가 보편화되면서, 사람이 직접 작성한 코드보다 환경 의존성이 숨겨져 있는 경우가 많아 재현성 확보가 더욱 중요해졌다.

### 2.1.1 AI 코드 생성의 환경 의존성 문제

AI 코딩 어시스턴트가 생성하는 코드에는 보이지 않는 가정이 숨어 있다. 예를 들어, GitHub Copilot이 pandas를 사용하는 코드를 제안할 때, 어떤 버전의 pandas를 기준으로 작성했는지 명시하지 않는다. pandas 1.x에서 자주 사용하던 `df.append()` 메서드는 1.4.0 버전에서 deprecated 되었고, 2.0 버전에서 완전히 제거되었다. AI가 학습한 시점의 코드 패턴이 현재 설치된 라이브러리 버전과 맞지 않으면, 동일한 코드도 전혀 다른 결과를 낳는다.

AI 모델은 학습 데이터의 시점에 고정된 API 사용 패턴을 학습한다. 대규모 언어 모델은 학습 데이터 수집 시점(knowledge cutoff) 이후의 변경사항을 반영하지 못하며, GitHub Copilot 역시 공개 코드를 학습했지만 최신 라이브러리 변경사항을 실시간으로 반영하지는 못한다. 따라서 AI가 제안한 코드가 "옛날 방식"일 가능성이 있으며, 이는 라이브러리 버전이 업그레이드될수록 문제가 된다.

운영체제 차이도 문제가 된다. Windows에서는 경로 구분자로 백슬래시(`\`)를 사용하고, macOS와 Linux에서는 슬래시(`/`)를 사용한다. AI가 특정 플랫폼을 기준으로 경로를 하드코딩한 코드를 생성하면, 다른 플랫폼에서는 "파일을 찾을 수 없습니다"라는 오류가 발생한다. AI는 학습 데이터에서 특정 플랫폼의 코드를 더 많이 접했을 수 있으므로, 크로스 플랫폼 호환성을 자동으로 보장하지 않는다.

파일 인코딩 문제도 빈번하다. Windows는 기본 인코딩으로 CP949를 사용하고, macOS와 Linux는 UTF-8을 사용한다. AI가 생성한 파일 읽기 코드에서 인코딩을 명시하지 않으면, 한글이 포함된 파일을 읽을 때 플랫폼에 따라 `UnicodeDecodeError`가 발생할 수 있다. 실제로 한 프로젝트에서 AI가 제안한 `open('data.csv')`를 Windows에서 실행했더니 한글이 깨지는 문제가 있었고, `open('data.csv', encoding='utf-8')`로 수정해야 했다.

라이브러리 하위 호환성(backward compatibility) 문제도 중요하다. 많은 Python 라이브러리가 메이저 버전 업그레이드 시 API를 변경한다. scikit-learn 1.0에서는 일부 메서드의 매개변수 기본값이 변경되었고, TensorFlow 2.x는 1.x와 완전히 다른 API 구조를 갖는다. AI가 구버전 API로 코드를 생성하면, 최신 버전에서는 `DeprecationWarning`이나 오류가 발생한다.

환경 변수 의존성도 문제가 된다. AI가 생성한 코드에서 API 키나 데이터베이스 연결 정보를 하드코딩하는 경우가 있다. 이는 보안 문제일 뿐 아니라, 환경마다 다른 설정을 사용해야 하는 경우 재현성을 해친다. 개발 환경에서는 테스트 데이터베이스를, 운영 환경에서는 실제 데이터베이스를 사용해야 하는데, 코드에 하드코딩되어 있으면 배포할 때마다 코드를 수정해야 한다.

### 2.1.2 재현 가능한 실행의 핵심 요소

재현 가능한 실행 환경을 구축하려면 다섯 가지 핵심 요소를 갖춰야 한다.

**첫째, 격리된 실행 환경이다.** Python 가상환경을 사용하면 프로젝트마다 독립적인 패키지 집합을 유지할 수 있다. 시스템 전역에 설치된 패키지는 다른 프로젝트의 요구사항과 충돌할 수 있다. 프로젝트 A는 Django 3.2를, 프로젝트 B는 Django 4.1을 사용한다면, 시스템 Python에는 둘 중 하나만 설치할 수 있다. 가상환경을 사용하면 각 프로젝트가 자신만의 Django 버전을 가질 수 있다.

가상환경의 이점은 단순히 버전 분리만이 아니다. 개발 과정에서 여러 패키지를 설치하고 제거하다 보면 시스템 Python이 "오염"될 수 있다. 사용하지 않는 패키지가 남아있거나, 패키지 간 의존성이 꼬이는 문제가 발생한다. 가상환경을 사용하면 언제든지 깨끗한 환경에서 다시 시작할 수 있다. 가상환경 디렉토리를 삭제하고 다시 생성하면, 처음부터 새로 설치한 것과 동일한 상태가 된다.

**둘째, 고정된 의존성이다.** `requirements.txt` 파일에 필요한 패키지와 정확한 버전을 기록해 두면, 누구든 동일한 환경을 재현할 수 있다. 의존성 고정은 두 가지 수준에서 이루어진다. 직접 의존성(direct dependency)은 프로젝트가 직접 사용하는 패키지다. 예를 들어 Django 웹 프레임워크를 사용한다면 Django가 직접 의존성이다. 전이 의존성(transitive dependency)은 직접 의존성이 의존하는 패키지다. Django는 내부적으로 sqlparse, pytz 등 여러 패키지를 사용하는데, 이들이 전이 의존성이다.

일반적인 `requirements.txt`는 직접 의존성만 명시한다. 하지만 완전한 재현성을 위해서는 전이 의존성까지 고정해야 한다. `pip freeze`는 현재 설치된 모든 패키지(전이 의존성 포함)를 출력하지만, 이 방법은 어떤 패키지가 직접 의존성인지 구별하기 어렵다. `pip-tools`나 `poetry` 같은 도구는 `requirements.in`(직접 의존성만)과 `requirements.txt`(전체 의존성 고정)를 별도로 관리하여, 의존성 구조를 명확히 한다.

**셋째, 일관된 경로 처리다.** `pathlib`을 사용하면 운영체제에 관계없이 동일하게 동작하는 경로 코드를 작성할 수 있다. 경로 문제는 단순히 구분자(`/` vs `\`) 차이만이 아니다. Windows는 절대 경로가 `C:\Users\...` 형태로 드라이브 문자를 포함하지만, macOS와 Linux는 `/home/...` 또는 `/Users/...` 형태로 루트부터 시작한다. 상대 경로 해석도 운영체제마다 미묘하게 다를 수 있다.

경로 처리의 핵심 원칙은 "스크립트 위치 기준 상대 경로"를 사용하는 것이다. `Path(__file__).parent`로 현재 스크립트의 디렉토리를 얻고, 거기서 상대적으로 데이터 디렉토리를 구성한다. 이렇게 하면 프로젝트를 어디에 복사하든, 스크립트를 어느 디렉토리에서 실행하든 동일한 결과를 얻는다. 하드코딩된 절대 경로(`/Users/hong/project/data`)는 다른 사용자 환경에서 작동하지 않는다.

**넷째, 환경 설정의 분리다.** 민감한 정보나 환경별로 달라지는 설정을 코드에서 분리하면 보안과 유연성을 모두 확보할 수 있다. API 키, 데이터베이스 비밀번호, 서버 주소 등은 코드에 하드코딩하지 않고 환경 변수나 별도의 설정 파일(`.env`)에 저장한다. `.env` 파일은 `.gitignore`에 추가하여 버전 관리에서 제외한다. 대신 `.env.example` 파일에 필요한 변수 목록과 예시값을 제공하여, 다른 개발자가 어떤 설정이 필요한지 알 수 있도록 한다.

환경 설정 분리는 보안뿐 아니라 배포 전략에도 중요하다. 개발 환경에서는 디버그 모드를 켜고 로그를 자세히 출력하지만, 운영 환경에서는 디버그 모드를 끄고 로그를 최소화한다. 이런 차이를 환경 변수로 관리하면, 동일한 코드를 다른 환경에 배포할 수 있다. `if os.getenv('ENV') == 'production'`처럼 환경을 구분하는 코드를 작성하면, 환경별로 다른 동작을 구현할 수 있다.

**다섯째, 버전 관리 통합이다.** Git과 같은 버전 관리 시스템을 사용하여 코드뿐 아니라 환경 설정 파일(`requirements.txt`, `.env.example`, `pyproject.toml` 등)도 함께 관리한다. 코드와 의존성은 밀접하게 연결되어 있으므로, 특정 커밋의 코드는 그 시점의 의존성 파일과 함께 기록되어야 한다. 이렇게 하면 과거 시점의 코드를 체크아웃할 때, 그 당시의 의존성도 함께 복원할 수 있다.

`.gitignore`에는 가상환경 디렉토리(`venv/`, `.venv/`), 임시 파일(`*.pyc`, `__pycache__/`), 환경 설정 파일(`.env`), 산출물(`*.log`, `output/`) 등을 추가하여 버전 관리에서 제외한다. 이들은 각 개발자의 로컬 환경에서 생성되거나, 민감한 정보를 포함하므로 공유하지 않는다. 대신 의존성 파일과 환경 설정 예시 파일을 공유하여, 다른 개발자가 동일한 환경을 재현할 수 있도록 한다.

### 2.1.3 재현 불가능한 코드의 비용

재현 불가능한 코드가 초래하는 비용은 생각보다 크다. 디버깅 시간 증가가 첫 번째 비용이다. "내 컴퓨터에서는 되는데"라는 말은 곧 "다른 환경에서는 재현되지 않는다"는 뜻이다. 문제를 해결하려면 환경 차이를 하나하나 확인해야 한다. Python 버전이 다른가? 패키지 버전이 다른가? 운영체제가 다른가? 이런 확인 작업만으로도 몇 시간씩 소요될 수 있다.

협업 어려움이 두 번째 비용이다. 팀원이 작성한 코드를 실행하려고 할 때, 환경을 맞추는 데 하루가 걸릴 수도 있다. "이 패키지 버전으로 설치해야 해요", "이 환경 변수를 설정해야 해요"라는 구두 설명에 의존하면, 정보가 누락되거나 잘못 전달될 가능성이 크다. 새로운 팀원이 합류할 때마다 같은 설명을 반복해야 한다.

배포 실패가 세 번째 비용이다. 개발 환경에서 잘 작동하던 코드가 운영 서버에서는 오류를 일으킬 수 있다. 운영 서버의 Python 버전이 다르거나, 시스템 라이브러리가 다르거나, 파일 경로 구조가 다를 수 있다. 배포 후에 문제를 발견하면, 서비스 중단이나 데이터 손실로 이어질 수 있다. 재현 가능한 환경을 미리 구축했다면, 배포 전에 스테이징 환경에서 충분히 테스트할 수 있다.

지식 손실이 네 번째 비용이다. 프로젝트를 6개월 뒤에 다시 실행하려고 할 때, 어떤 패키지를 어떤 버전으로 설치했는지 기억나지 않을 수 있다. 의존성을 기록해 두지 않으면, 과거 코드를 재현할 수 없게 된다. 이는 연구 프로젝트에서 특히 치명적이다. 논문을 제출한 후 리뷰어가 결과를 재현하려고 할 때, 환경을 복원할 수 없으면 논문이 거부될 수 있다.

**사례: pandas 버전 불일치 문제**

한 프로젝트에서 AI가 제안한 코드에 `df.append(row)`가 포함되어 있었다. pandas 1.x를 사용하던 개발자의 환경에서는 정상 동작했지만, pandas 2.0으로 업그레이드한 동료의 환경에서는 `AttributeError`가 발생했다. `append()` 메서드는 pandas 1.4.0에서 deprecated 되었고, 2.0에서 완전히 제거된 것이다. 팀원들은 하루 동안 "왜 내 컴퓨터에서는 안 되지?"라는 질문에 시달렸다. 결국 pandas 버전 차이가 원인임을 발견하고, `requirements.txt`에 `pandas==1.5.3`을 명시한 후에야 모든 팀원이 동일한 결과를 얻을 수 있었다.

이 사례는 여러 교훈을 준다. 첫째, AI가 생성한 코드는 버전 의존성을 명시하지 않으므로, 개발자가 직접 확인해야 한다. 둘째, 프로젝트 초기에 `requirements.txt`를 작성하고 버전을 고정해야 한다. 셋째, 팀원 모두가 동일한 환경을 사용하도록 가상환경과 의존성 관리를 습관화해야 한다. 이런 원칙을 따르면, 환경 문제로 인한 시간 낭비를 크게 줄일 수 있다.

---

## 2.2 Python 가상환경 완전 정복

### 2.2.1 가상환경이란 무엇인가

Python 가상환경은 프로젝트별로 독립적인 Python 실행 환경을 제공하는 메커니즘이다. 시스템에 설치된 Python(시스템 Python)과 별도로, 각 프로젝트에 필요한 패키지만 설치된 격리된 공간을 만들 수 있다. 

왜 격리가 필요할까? 프로젝트 A에서는 requests 2.25를, 프로젝트 B에서는 requests 2.31을 사용한다고 하자. 시스템 Python에 두 버전을 동시에 설치할 수는 없다. 가상환경을 사용하면 프로젝트 A의 가상환경에는 2.25를, 프로젝트 B의 가상환경에는 2.31을 각각 설치하여 충돌 없이 사용할 수 있다.

가상환경의 동작 원리를 이해하면 더 효과적으로 활용할 수 있다. 가상환경을 생성하면 프로젝트 디렉토리 안에 독립적인 Python 인터프리터 복사본과 패키지 설치 디렉토리가 만들어진다. 활성화하면 시스템 PATH 환경 변수가 수정되어, `python` 명령이 시스템 Python이 아닌 가상환경의 Python을 가리키게 된다. 이후 `pip install`로 설치하는 패키지는 시스템이 아닌 가상환경의 `site-packages` 디렉토리에 저장된다.

가상환경이 활성화되어 있는지 확인하는 방법도 알아두면 좋다. Python 내부적으로 `sys.prefix`와 `sys.base_prefix` 값이 다르면 가상환경이 활성화된 상태다. 아래 코드로 간단히 확인할 수 있다.

```python
import sys
in_venv = sys.prefix != sys.base_prefix
print(f"가상환경 활성: {in_venv}")
print(f"sys.prefix: {sys.prefix}")
print(f"sys.base_prefix: {sys.base_prefix}")
```

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

가상환경이 활성화되어 있으면 `sys.prefix`는 가상환경 디렉토리 경로(예: `/path/to/venv`)를, `sys.base_prefix`는 시스템 Python 경로(예: `/usr/local/python`)를 가리킨다. 두 값이 같으면 시스템 Python을 직접 사용하는 것이다.

가상환경 디렉토리 구조도 살펴보자. `venv/` 디렉토리 안에는 다음과 같은 하위 디렉토리가 있다. `bin/`(Windows에서는 `Scripts/`)에는 Python 실행 파일, pip, activate 스크립트 등이 들어있다. `lib/python3.x/site-packages/`에는 설치된 패키지가 저장된다. `include/`에는 C 확장을 컴파일할 때 필요한 헤더 파일이 들어간다. `pyvenv.cfg` 파일에는 가상환경 설정(Python 버전, 시스템 패키지 사용 여부 등)이 기록된다.

가상환경을 사용할 때 주의할 점도 있다. 첫째, 가상환경은 이식 불가능(non-portable)하다. 가상환경 디렉토리를 다른 위치로 복사하면 작동하지 않는다. 가상환경 내부의 스크립트와 설정 파일에 절대 경로가 하드코딩되어 있기 때문이다. 환경을 공유하려면 `requirements.txt`를 사용하여 다른 곳에서 재생성해야 한다.

둘째, 가상환경은 Python 버전을 격리하지 않는다. 가상환경은 생성 시점의 Python 버전을 그대로 사용한다. Python 3.9로 만든 가상환경에서 Python 3.11을 사용할 수는 없다. Python 버전까지 격리하려면 pyenv나 conda를 사용해야 한다.

셋째, 가상환경을 활성화하지 않고 실행하면 시스템 Python이 사용된다. `source venv/bin/activate`를 실행하지 않고 `python script.py`를 실행하면, 가상환경의 패키지가 아닌 시스템 패키지를 사용하게 된다. 실수로 시스템 Python을 사용하면 "패키지가 설치되어 있는데 왜 ImportError가 나지?"라는 혼란에 빠질 수 있다.

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

`python3 -m venv venv` 명령의 의미를 자세히 살펴보자. `-m venv`는 venv 모듈을 실행하라는 뜻이고, 마지막 `venv`는 생성할 가상환경 디렉토리 이름이다. 디렉토리 이름은 자유롭게 정할 수 있지만, 관례상 `venv`, `.venv`, `env` 등을 사용한다. `.venv`처럼 점으로 시작하면 Unix 계열에서 숨김 디렉토리로 취급된다.

가상환경이 활성화되면 터미널 프롬프트 앞에 `(venv)`가 표시된다. 이 상태에서 `pip install`로 설치하는 패키지는 모두 이 가상환경에만 설치된다. 작업을 마치고 가상환경을 비활성화하려면 `deactivate` 명령을 입력한다. 비활성화 후에는 프롬프트에서 `(venv)`가 사라지고, 다시 시스템 Python을 사용하게 된다.

가상환경 활성화 스크립트는 여러 셸을 지원한다. bash/zsh에서는 `source venv/bin/activate`를, fish 셸에서는 `source venv/bin/activate.fish`를, csh/tcsh에서는 `source venv/bin/activate.csh`를 사용한다. Windows PowerShell에서는 `venv\Scripts\Activate.ps1`을 실행하지만, 기본 보안 정책으로 인해 스크립트 실행이 차단될 수 있다. 그럴 때는 `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` 명령으로 실행 정책을 변경해야 한다.

가상환경 디렉토리(보통 `venv/`)는 소스 컨트롤에 포함하지 않는다. 대신 `requirements.txt`를 커밋하여, 다른 사람이 같은 환경을 재현할 수 있도록 한다. `.gitignore` 파일에 다음 항목을 추가해 두는 것이 좋다.

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

가상환경을 사용하는 표준 워크플로우는 다음과 같다. 프로젝트를 시작할 때 `python3 -m venv venv`로 가상환경을 생성한다. 작업을 시작할 때마다 `source venv/bin/activate`로 활성화한다. 필요한 패키지를 `pip install package_name`으로 설치한다. 작업을 마치면 `pip freeze > requirements.txt`로 의존성을 기록한다. 다른 환경에서는 `pip install -r requirements.txt`로 동일한 패키지를 설치한다.

가상환경 관련 유용한 명령어도 알아두자. `which python`(Windows에서는 `where python`)은 현재 사용 중인 Python 실행 파일의 경로를 보여준다. 가상환경이 활성화되어 있으면 `venv/bin/python` 경로가 출력된다. `pip list`는 현재 환경에 설치된 모든 패키지를 나열한다. `pip show package_name`은 특정 패키지의 상세 정보(버전, 의존성, 설치 위치 등)를 보여준다.

### 2.2.3 conda vs venv: 언제 무엇을 선택할까

Python 가상환경 도구로는 venv 외에도 conda, virtualenv, pipenv, poetry 등이 있다. 이 중 가장 널리 사용되는 venv와 conda를 비교하여, 상황에 맞는 선택을 할 수 있도록 한다.

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

**venv를 선택해야 하는 경우:**

첫째, 순수 Python 프로젝트다. 웹 개발(Django, Flask), API 서버, 크롤러 등 Python 패키지만 사용하는 프로젝트는 venv로 충분하다. PyPI의 방대한 패키지 생태계를 활용할 수 있고, 가볍고 빠르다.

둘째, 표준 도구를 선호한다. venv는 Python 공식 문서에서 권장하는 표준 도구다. 별도 설치가 필요 없고, 모든 Python 설치에 기본 포함되어 있다. 팀원 모두가 특별한 설정 없이 동일한 환경을 사용할 수 있다.

셋째, CI/CD 파이프라인에서 사용한다. GitHub Actions, GitLab CI 등 자동화 환경에서는 venv가 더 적합하다. 설치가 빠르고, 디스크 공간을 적게 차지하며, 스크립트로 쉽게 자동화할 수 있다.

**conda를 선택해야 하는 경우:**

첫째, 데이터 과학 및 머신러닝 프로젝트다. NumPy, SciPy, pandas, scikit-learn, TensorFlow, PyTorch 등은 C/C++ 확장을 포함하며, 컴파일된 바이너리에 의존한다. conda는 이런 패키지를 미리 컴파일된 바이너리로 제공하여, 설치가 간편하고 버전 호환성이 보장된다.

둘째, GPU 가속이 필요하다. CUDA, cuDNN 같은 GPU 라이브러리는 Python 패키지가 아니므로 pip로 설치할 수 없다. conda는 이런 비Python 의존성도 함께 관리하여, 복잡한 환경 설정 없이 GPU 환경을 구축할 수 있다. `conda install pytorch cudatoolkit=11.8` 한 줄로 PyTorch와 CUDA를 함께 설치할 수 있다.

셋째, Python 버전을 환경마다 다르게 사용해야 한다. venv는 시스템 Python 버전에 고정되지만, conda는 환경마다 다른 Python 버전을 사용할 수 있다. Python 2.7 레거시 프로젝트와 Python 3.11 신규 프로젝트를 동시에 관리해야 한다면 conda가 적합하다.

넷째, 완전한 환경 재현이 중요하다. conda의 `environment.yml`은 Python 버전, 모든 패키지, 비Python 의존성까지 포함한다. 이를 사용하면 다른 컴퓨터나 서버에서 정확히 동일한 환경을 재현할 수 있다. 연구 프로젝트에서 결과 재현성이 중요할 때 유용하다.

**두 도구를 혼용할 때 주의사항:**

conda 환경에서 pip를 사용할 수는 있지만, 권장하지 않는다. conda와 pip는 의존성 관리 방식이 다르므로, 혼용하면 패키지 충돌이 발생할 수 있다. 꼭 필요하다면 conda로 먼저 주요 패키지를 설치하고, conda에 없는 패키지만 pip로 설치한다. `conda install pip`로 conda가 관리하는 pip를 사용하는 것이 안전하다.

이 책의 실습에서는 모든 환경에서 동일하게 사용할 수 있는 venv를 기준으로 설명한다. GPU 가속이나 고급 데이터 과학 라이브러리가 필요한 경우, 별도로 conda 사용법을 안내한다. venv를 먼저 익히고, 필요에 따라 conda로 전환하는 것이 좋은 학습 경로다.

---

## 2.3 의존성 관리와 버전 고정

### 2.3.1 requirements.txt 작성법

`requirements.txt`는 프로젝트에 필요한 Python 패키지 목록을 담는 파일이다. 가장 간단한 형태는 패키지 이름만 나열하는 것이지만, 재현 가능한 환경을 위해서는 버전까지 명시해야 한다.

버전 지정 방식에는 여러 가지가 있으며, 각각의 용도가 다르다. 첫째, `==`는 정확한 버전을 지정한다. `pandas==1.5.3`은 반드시 1.5.3 버전만 설치한다. 이는 가장 엄격한 방식으로, 재현성을 최대한 보장한다. 둘째, `>=`는 최소 버전을 지정한다. `requests>=2.25.0`은 2.25.0 이상이면 어떤 버전이든 설치한다. 새로운 버그 수정이나 기능을 자동으로 받을 수 있지만, 하위 호환성이 깨질 위험이 있다.

셋째, `~=`는 호환 버전을 지정한다. `numpy~=1.24.0`은 1.24.x 범위 내의 최신 버전을 설치한다. 마이너 버전 업데이트는 하위 호환성을 유지한다는 시맨틱 버저닝(semantic versioning) 원칙을 따른다고 가정하는 것이다. `numpy~=1.24.0`은 1.24.1, 1.24.2는 설치하지만 1.25.0은 설치하지 않는다.

넷째, 범위 지정도 가능하다. `django>=3.2,<4.0`처럼 쉼표로 여러 조건을 결합할 수 있다. 이는 "3.2 이상이면서 4.0 미만"을 의미한다. 메이저 버전 업그레이드는 하위 호환성을 깨뜨릴 수 있으므로, 이런 범위 지정으로 안전한 업데이트만 허용한다.

다섯째, 특정 버전을 제외할 수도 있다. `requests>=2.28.0,!=2.28.1`은 2.28.1 버전에 알려진 버그가 있을 때 유용하다. 2.28.0 이상이지만 2.28.1은 제외하고 설치한다.

**표 2.2** requirements.txt 버전 지정 방식

| 방식 | 예시 | 의미 | 사용 시기 |
|:---|:---|:---|:---|
| `==` | `pandas==1.5.3` | 정확히 1.5.3 | 완전한 재현성 필요 |
| `>=` | `requests>=2.25.0` | 2.25.0 이상 | 최소 버전만 보장 |
| `~=` | `numpy~=1.24.0` | 1.24.x 범위 | 호환 업데이트 허용 |
| `>=,<` | `django>=3.2,<4.0` | 3.2 이상 4.0 미만 | 메이저 버전 고정 |
| `!=` | `requests>=2.28,!=2.28.1` | 2.28.1 제외 | 특정 버전 제외 |

`pip freeze > requirements.txt` 명령으로 현재 환경의 모든 패키지를 내보낼 수 있지만, 이 방법은 권장하지 않는다. 직접 설치하지 않은 의존성 패키지까지 모두 포함되어 파일이 불필요하게 길어지고, 나중에 어떤 패키지가 실제로 필요한 것인지 파악하기 어려워진다. 

예를 들어, Django를 설치하면 sqlparse, asgiref, pytz 등 10개 이상의 전이 의존성이 함께 설치된다. `pip freeze`는 이 모든 패키지를 나열하지만, 프로젝트가 직접 사용하는 것은 Django뿐이다. `requirements.txt`에 Django만 적어도 나머지는 자동으로 설치되므로, 직접 의존성만 관리하는 것이 명확하다.

직접 사용하는 패키지만 수동으로 관리하는 것이 좋다. `requirements.txt`를 처음 만들 때는 `import` 문을 보고 실제로 사용하는 패키지만 추가한다. 주석으로 각 패키지의 용도를 적어 두면 나중에 이해하기 쉽다.

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

주석을 사용하여 패키지를 그룹화하면, 프로젝트의 의존성 구조를 한눈에 파악할 수 있다. 새로운 팀원이 합류했을 때도 "이 프로젝트가 어떤 라이브러리를 사용하는지" 쉽게 이해할 수 있다.

### 2.3.2 의존성 버전 고정 전략

개발 단계와 배포 단계에서 버전 지정 전략을 다르게 가져가면 유연성과 안정성을 모두 확보할 수 있다. 이는 "개발 의존성(development dependencies)"과 "프로덕션 의존성(production dependencies)"을 분리하는 개념과도 연결된다.

**개발 시:** 유연한 버전을 사용한다. `pandas>=1.5`처럼 최소 버전만 지정하면, 새로운 기능이나 버그 수정이 포함된 최신 버전을 자연스럽게 받아볼 수 있다. 개발 환경에서는 최신 버전을 사용하면서 새로운 기능을 탐색하고, 성능 개선을 누릴 수 있다. 문제가 발생하면 빠르게 발견하고 수정할 수 있다.

**배포 시:** 엄격한 버전을 사용한다. `pandas==1.5.3`처럼 정확한 버전을 고정하면, 운영 환경에서 예기치 않은 변경으로 인한 장애를 방지할 수 있다. 배포 환경에서는 안정성이 최우선이므로, 검증된 버전을 그대로 유지한다. 버전 업그레이드는 충분한 테스트를 거친 후 계획적으로 수행한다.

이를 위해 두 개의 의존성 파일을 사용하는 패턴이 있다. `requirements-dev.txt`에는 개발 도구(pytest, black, flake8 등)를 포함하고, `requirements.txt`에는 실행에 필요한 패키지만 포함한다. 배포 시에는 `requirements.txt`만 설치하여, 불필요한 개발 도구가 프로덕션 환경에 들어가지 않도록 한다.

```txt
# requirements.txt (프로덕션)
django==4.2.7
psycopg2-binary==2.9.9
celery==5.3.4

# requirements-dev.txt (개발)
-r requirements.txt  # 프로덕션 의존성 포함
pytest==7.4.3
black==23.11.0
flake8==6.1.0
ipython==8.18.1
```

`-r requirements.txt` 구문은 다른 requirements 파일을 포함한다는 의미다. 이렇게 하면 개발 환경에서는 `pip install -r requirements-dev.txt` 한 줄로 프로덕션 패키지와 개발 도구를 모두 설치할 수 있다.

보다 완전한 재현성이 필요하다면 `pip-tools`나 `poetry` 같은 도구를 사용할 수 있다. 이들 도구는 직접 의존성과 전이 의존성을 별도로 관리한다.

**pip-tools 사용 예시:**

`requirements.in` 파일에 직접 의존성만 작성한다.

```txt
# requirements.in
django>=4.2
pandas>=2.0
```

`pip-compile requirements.in` 명령을 실행하면, `requirements.txt`가 자동 생성된다. 이 파일에는 Django와 pandas뿐 아니라, 그들이 의존하는 모든 패키지가 정확한 버전과 함께 기록된다.

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

이렇게 하면 직접 의존성은 `requirements.in`에서 관리하고, 완전히 고정된 환경은 `requirements.txt`로 재현할 수 있다. 버전을 업그레이드하려면 `requirements.in`을 수정하고 다시 `pip-compile`을 실행한다.

**poetry 사용 예시:**

poetry는 `pyproject.toml` 파일로 의존성을 관리한다.

```toml
[tool.poetry.dependencies]
python = "^3.9"
django = "^4.2"
pandas = "^2.0"

[tool.poetry.dev-dependencies]
pytest = "^7.4"
black = "^23.11"
```

`poetry install` 명령을 실행하면, `poetry.lock` 파일이 자동 생성되어 모든 의존성을 정확한 버전으로 고정한다. `poetry add package_name` 명령으로 새 패키지를 추가하면, `pyproject.toml`과 `poetry.lock`이 자동으로 업데이트된다. poetry는 의존성 해결, 가상환경 관리, 패키지 빌드까지 통합적으로 처리하여, 현대적인 Python 프로젝트 관리 도구로 주목받고 있다.

### 2.3.3 의존성 충돌 해결

두 패키지가 서로 다른 버전의 동일한 패키지에 의존할 때 충돌이 발생한다. 예를 들어, 패키지 A가 `requests>=2.28`을, 패키지 B가 `requests<2.27`을 요구하면 둘을 동시에 만족하는 requests 버전이 없어 설치에 실패한다.

pip의 의존성 해결기(dependency resolver)는 이런 충돌을 감지하고 오류 메시지를 출력한다. Python 3.9+와 pip 20.3+에서는 새로운 의존성 해결기가 도입되어, 충돌을 더 명확하게 보고한다.

```
ERROR: Cannot install package-a and package-b because these package versions have conflicting dependencies.

The conflict is caused by:
    package-a 1.0.0 depends on requests>=2.28
    package-b 2.0.0 depends on requests<2.27
```

충돌이 발생하면 다음과 같은 해결 방법을 시도한다.

**첫째, 패키지 버전 조정이다.** 충돌하는 패키지 중 하나의 버전을 변경하여, 둘 다 만족하는 중간 버전을 찾는다. package-a의 구버전이 `requests>=2.25`를 요구한다면, package-a 버전을 낮추는 것으로 해결할 수 있다. 각 패키지의 변경 이력(CHANGELOG)을 확인하여, 필요한 기능이 구버전에도 있는지 확인한다.

**둘째, 대체 패키지 탐색이다.** 충돌하는 패키지 중 하나를 기능이 유사한 다른 패키지로 대체한다. 예를 들어 requests 대신 httpx를 사용하거나, pandas 대신 polars를 사용하는 식이다. 이는 근본적인 해결책이지만, 코드 수정이 필요하다.

**셋째, 의존성 재협상이다.** 패키지 개발자에게 의존성 요구사항을 완화해 달라고 요청한다. 많은 패키지가 의존성 버전 범위를 너무 좁게 설정하는 경우가 있다. GitHub 이슈로 요청하면, 다음 릴리스에서 의존성 범위를 넓혀줄 수 있다.

**넷째, 격리된 환경 사용이다.** 근본적으로 해결할 수 없다면, 두 패키지를 다른 가상환경에서 사용한다. 예를 들어 package-a 전용 환경과 package-b 전용 환경을 별도로 만든다. 이는 불편하지만, 충돌을 완전히 회피할 수 있다.

`pip install --dry-run -r requirements.txt` 옵션을 사용하면 실제 설치 전에 충돌 여부를 미리 확인할 수 있다. 이 명령은 설치를 시뮬레이션하고 충돌이 있으면 오류를 출력하지만, 실제로 패키지를 설치하지는 않는다. 새로운 패키지를 추가하기 전에 충돌 여부를 미리 확인하는 습관을 들이면, 나중에 큰 문제를 예방할 수 있다.

pip의 `pipdeptree` 도구를 사용하면 의존성 트리를 시각화할 수 있다.

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

이 트리를 보면 어떤 패키지가 어떤 의존성을 요구하는지 한눈에 파악할 수 있다. 충돌이 발생했을 때, 어떤 경로로 문제의 패키지가 설치되었는지 추적할 수 있다.

---

## 2.4 크로스 플랫폼 경로 처리

### 2.4.1 플랫폼별 경로 차이

Windows와 Unix 계열(macOS, Linux) 운영체제는 파일 경로를 표현하는 방식이 근본적으로 다르다. 이 차이는 단순히 구분자만의 문제가 아니라, 파일 시스템의 구조와 철학의 차이에서 비롯된다.

**표 2.3** 플랫폼별 경로 특징

| 요소 | Windows | macOS/Linux |
|:---|:---|:---|
| 경로 구분자 | `\` (백슬래시) | `/` (슬래시) |
| 루트 표현 | `C:\` (드라이브 문자) | `/` (단일 루트) |
| 홈 디렉토리 | `C:\Users\username` | `/home/username` (Linux) 또는 `/Users/username` (macOS) |
| 대소문자 구분 | 불구분 (기본값) | 구분 (Linux), 불구분 (macOS 기본) |
| 경로 최대 길이 | 260자 (제한) | 4096자 (Linux), 1024자 (macOS) |
| 금지 문자 | `< > : " / \ | ? *` | `/`, `\0` (null) |

코드에 `"C:\\Users\\hong\\data\\file.csv"`처럼 Windows 경로를 하드코딩하면, macOS에서는 실행할 수 없다. 반대로 `"/home/hong/data/file.csv"`를 하드코딩하면 Windows에서 문제가 된다. AI가 생성한 코드에 플랫폼 특정 경로가 포함되어 있다면, 반드시 크로스 플랫폼 방식으로 수정해야 한다.

경로 문제는 코드뿐 아니라 데이터 파일에도 영향을 미친다. CSV 파일에 파일 경로가 기록되어 있다면, 플랫폼을 옮길 때 모든 경로를 변환해야 한다. SQLite 데이터베이스에 파일 경로를 저장했다면, 경로 변환 스크립트를 별도로 작성해야 할 수도 있다.

대소문자 구분 문제도 주의해야 한다. Linux는 `file.txt`와 `File.txt`를 다른 파일로 인식하지만, Windows와 macOS(기본 설정)는 같은 파일로 취급한다. Linux에서 개발한 코드에서 `import MyModule`과 `import mymodule`을 혼용하면, Linux에서는 오류가 나지 않지만 Windows에서는 `ModuleNotFoundError`가 발생할 수 있다.

경로 길이 제한도 문제가 될 수 있다. Windows의 260자 제한(MAX_PATH)은 깊은 디렉토리 구조에서 문제를 일으킨다. `C:\Users\username\Projects\long-project-name\src\modules\submodules\deep\nested\path\file.py`처럼 경로가 길어지면, Windows에서 파일을 생성하거나 열 수 없다. Python 3.6+와 Windows 10에서는 긴 경로 지원이 개선되었지만, 여전히 주의가 필요하다.

### 2.4.2 pathlib으로 경로 다루기

Python 3.4부터 표준 라이브러리에 포함된 `pathlib` 모듈을 사용하면 운영체제에 관계없이 동일하게 동작하는 경로 코드를 작성할 수 있다. `pathlib`는 객체 지향 방식으로 경로를 다루며, 직관적이고 가독성이 좋다.

가장 중요한 패턴은 스크립트 파일 위치를 기준으로 상대 경로를 구성하는 것이다.

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

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

`Path(__file__).parent`는 현재 스크립트가 위치한 디렉토리를 반환한다. `__file__`은 현재 실행 중인 스크립트의 경로를 담고 있는 특수 변수다. `.resolve()`를 호출하면 심볼릭 링크를 해석하고 절대 경로로 변환한다. 이렇게 하면 스크립트를 어느 디렉토리에서 실행하든, 항상 스크립트 자신의 위치를 기준으로 경로를 계산할 수 있다.

`/` 연산자를 사용하여 경로를 조합하는 것이 `pathlib`의 가장 큰 장점이다. `base_dir / "data" / "output"`은 운영체제에 맞는 경로 구분자가 자동으로 사용된다. Windows에서는 `C:\project\data\output`으로, Linux에서는 `/home/user/project/data/output`으로 변환된다.

`mkdir(parents=True, exist_ok=True)`는 디렉토리 생성 시 자주 사용하는 옵션이다. `parents=True`는 상위 디렉토리가 없으면 함께 생성한다. `data/output/results/`를 만들 때 `data/`와 `output/`이 없어도 자동으로 생성된다. `exist_ok=True`는 디렉토리가 이미 존재해도 오류를 발생시키지 않는다. 이 두 옵션을 함께 사용하면, 경로 존재 여부를 미리 확인하지 않고 안전하게 디렉토리를 생성할 수 있다.

**pathlib의 주요 메서드와 속성:**

```python
from pathlib import Path

# 경로 생성
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

# 글로빙
p.glob("*.txt")      # 현재 디렉토리의 txt 파일
p.glob("**/*.txt")   # 하위 모든 디렉토리의 txt 파일 (재귀)
p.rglob("*.txt")     # glob("**/*.txt")와 동일
```

`glob()` 패턴 매칭은 파일 목록을 가져올 때 유용하다. `*`는 임의의 문자열, `**`는 임의의 깊이의 디렉토리를 의미한다. `data_dir.glob("*.csv")`는 data_dir의 모든 CSV 파일을, `data_dir.rglob("*.csv")`는 하위 디렉토리까지 재귀적으로 모든 CSV 파일을 찾는다.

**실무 예시: 데이터 파일 일괄 처리**

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

이 코드는 Windows, macOS, Linux에서 동일하게 작동한다. 경로 조합, 파일 목록 가져오기, 파일명 추출이 모두 `pathlib`로 크로스 플랫폼 방식으로 구현되었다.

### 2.4.3 os.path vs pathlib 비교

레거시 코드에서는 `os.path` 모듈을 사용한 경로 처리를 자주 볼 수 있다. `os.path`도 크로스 플랫폼을 지원하지만, `pathlib`이 더 읽기 쉽고 현대적인 방식이다.

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

`os.path`는 문자열 기반으로 동작한다. 경로를 문자열로 받아서 문자열로 반환한다. 반면 `pathlib.Path`는 객체 지향 방식으로, 경로를 객체로 다룬다. 객체에 메서드가 있어 더 직관적이고, 메서드 체이닝이 가능하다.

**os.path 예시 (구버전 스타일):**

```python
import os

# 경로 조합
data_file = os.path.join("data", "input", "file.csv")

# 디렉토리 생성
output_dir = os.path.join("data", "output")
if not os.path.exists(output_dir):
    os.makedirs(output_dir)

# 파일 목록
for filename in os.listdir("data"):
    if filename.endswith(".csv"):
        filepath = os.path.join("data", filename)
        # 처리...
```

**pathlib 예시 (현대적 스타일):**

```python
from pathlib import Path

# 경로 조합
data_file = Path("data") / "input" / "file.csv"

# 디렉토리 생성
output_dir = Path("data") / "output"
output_dir.mkdir(parents=True, exist_ok=True)

# 파일 목록
for filepath in Path("data").glob("*.csv"):
    # 처리...
```

`pathlib` 코드가 더 간결하고 읽기 쉽다. `os.path.join()`은 여러 인자를 받지만, `/` 연산자는 시각적으로 더 명확하다. `os.makedirs(exist_ok=True)`는 별도 호출이지만, `mkdir(parents=True, exist_ok=True)`는 한 줄로 해결된다.

새로운 코드를 작성할 때는 `pathlib`을 사용하고, 기존 `os.path` 코드를 유지보수할 때는 점진적으로 `pathlib`으로 마이그레이션하는 것을 권장한다. AI 코딩 어시스턴트에게 코드를 요청할 때도 "pathlib을 사용해서"라고 명시하면 크로스 플랫폼 호환 코드를 받을 가능성이 높아진다.

**혼용 시 주의사항:**

`pathlib.Path` 객체는 대부분의 표준 라이브러리 함수와 호환된다. Python 3.6+에서는 `open()`, `os.rename()` 등이 `Path` 객체를 직접 받는다.

```python
from pathlib import Path

p = Path("data") / "file.txt"

# Python 3.6+에서 모두 작동
with open(p, "r") as f:
    content = f.read()

# 문자열로 변환 필요 (구버전 호환)
with open(str(p), "r") as f:
    content = f.read()
```

하지만 일부 서드파티 라이브러리는 아직 `Path` 객체를 지원하지 않을 수 있다. 그럴 때는 `str(path)`로 문자열로 변환하여 전달한다.

### 2.4.4 환경 변수와 홈 디렉토리

크로스 플랫폼 코드에서는 사용자별로 다른 경로를 처리해야 할 때가 있다. 예를 들어 설정 파일을 사용자의 홈 디렉토리에 저장하거나, 환경 변수로 데이터 디렉토리 위치를 지정할 수 있다.

**홈 디렉토리 접근:**

```python
from pathlib import Path

# 크로스 플랫폼 홈 디렉토리
home = Path.home()
# Windows: C:\Users\username
# macOS: /Users/username
# Linux: /home/username

# 설정 파일 경로
config_file = home / ".myapp" / "config.json"
config_file.parent.mkdir(parents=True, exist_ok=True)
```

`Path.home()`은 현재 사용자의 홈 디렉토리를 반환한다. 운영체제에 관계없이 동일한 코드로 홈 디렉토리를 얻을 수 있다.

**환경 변수 사용:**

```python
import os
from pathlib import Path

# 환경 변수로 데이터 디렉토리 지정
data_dir = os.getenv("DATA_DIR", "data")
data_path = Path(data_dir) / "input"

# 환경 변수가 설정되지 않으면 기본값 "data" 사용
```

환경 변수를 사용하면 코드 수정 없이 실행 환경에서 경로를 설정할 수 있다. 개발 환경에서는 로컬 디렉토리를, 운영 환경에서는 공유 스토리지를 지정하는 식으로 유연하게 구성할 수 있다.

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

표준 템플릿(`2-5-template.py`, 302줄)은 네 가지 단계로 구성된다. 각 단계는 독립적인 함수로 구현되어 있어, 재사용과 테스트가 용이하다.

**첫째, 환경 검증(verify_environment)**

Python 버전이 최소 요구사항을 충족하는지, 필요한 패키지가 설치되어 있는지 확인한다. 이 단계를 통과하지 못하면 스크립트는 실행을 중단하고 명확한 오류 메시지를 출력한다.

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

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

`sys.version_info`는 Python 버전 정보를 담은 튜플이다. `sys.version_info[:2]`는 메이저와 마이너 버전만 추출한다 (예: `(3, 11)`). 튜플 비교는 왼쪽부터 순서대로 수행되므로, `(3, 9) < (3, 11)`은 `True`가 된다.

환경 검증은 문제를 조기에 발견하는 데 중요하다. Python 3.9 이상을 요구하는 코드가 Python 3.8에서 실행되면, `match-case` 문법 같은 최신 기능에서 오류가 발생할 수 있다. 환경 검증 단계에서 미리 차단하면, 실행 중간에 실패하는 것보다 문제 파악이 빠르다.

**둘째, 경로 설정(setup_paths)**

스크립트 위치를 기준으로 입출력 경로를 자동 구성한다. 이 함수는 어느 디렉토리에서 스크립트를 실행하든 동일한 상대 경로를 유지한다.

```python
from pathlib import Path

def setup_paths():
    """크로스 플랫폼 경로 설정"""
    script_dir = Path(__file__).parent.resolve()
    output_dir = script_dir.parent / "data" / "output"
    output_dir.mkdir(parents=True, exist_ok=True)
    return {"script_dir": script_dir, "output_dir": output_dir}
```

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

`Path(__file__).parent.resolve()`는 현재 스크립트의 절대 경로를 얻는다. `.parent`는 상위 디렉토리를, `.parent.parent`는 그 상위 디렉토리를 반환한다. 이 방식은 심볼릭 링크가 있어도 실제 경로로 해석된다.

경로 딕셔너리를 반환하면, 실행 함수에서 `paths["output_dir"]`처럼 명확히 참조할 수 있다. 문자열로 경로를 직접 쓰는 것보다 오타 가능성이 낮다.

경로 검증 루프는 모든 경로가 올바르게 설정되었는지 확인한다. `output_dir.mkdir()`이 성공했는지 `path.exists()`로 검증한다. 만약 디스크 공간이 부족하거나 권한 문제로 디렉토리 생성에 실패했다면, 이 단계에서 발견된다.

**셋째, 실행 및 검증(run_task)**

핵심 로직을 실행하고 결과가 기대치를 충족하는지 확인한다. 템플릿에서는 경로 처리 데모를 실행하고, 크로스 플랫폼 호환성을 검증한다.

```python
def run_task(paths):
    """실제 작업 실행"""
    test_path = paths["output_dir"] / "subdir" / "file.txt"
    posix_path = test_path.as_posix()  # 항상 / 사용
    return {"posix": posix_path, "parts": list(test_path.parts)}
```

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

`as_posix()`는 Windows에서도 `/`를 사용하는 경로를 반환한다. 이는 JSON 파일에 경로를 저장할 때 유용하다. Windows 경로 `C:\Users\hong\data`를 그대로 JSON에 넣으면 백슬래시가 이스케이프 문자로 해석될 수 있지만, `as_posix()`를 사용하면 `C:/Users/hong/data`처럼 안전하게 저장된다.

`test_path.parts`는 경로를 구성요소로 분해한다. `/Users/hong/data/file.txt`는 `('/', 'Users', 'hong', 'data', 'file.txt')`로 분해된다. Windows에서는 `('C:\\', 'Users', 'hong', 'data', 'file.txt')`처럼 드라이브 문자가 포함된다.

실행 단계에서는 의미 있는 작업을 수행하고 그 결과를 반환해야 한다. 템플릿에서는 경로 처리를 시연했지만, 실제 프로젝트에서는 데이터 분석, 모델 학습, API 호출 등의 작업이 들어간다.

**넷째, 산출물 저장(save_output)**

실행 결과를 타임스탬프가 포함된 파일명으로 저장하여 실행 이력을 추적한다. JSON 형식은 사람이 읽기 쉽고, 프로그램으로 파싱하기도 쉽다.

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

_전체 코드는 practice/chapter2/code/2-5-template.py 참고_

`datetime.now().strftime("%Y%m%d_%H%M%S")`는 현재 시각을 `20250101_143022` 형식으로 포맷팅한다. 이 타임스탬프를 파일 이름에 포함하면, 같은 스크립트를 여러 번 실행해도 결과 파일이 덮어씌워지지 않는다.

`ensure_ascii=False`는 한글 등 유니코드 문자를 그대로 저장한다. 기본값 `ensure_ascii=True`는 유니코드를 `\uXXXX` 형태로 이스케이프하여 가독성이 떨어진다.

`indent=2`는 JSON을 들여쓰기하여 사람이 읽기 쉽게 만든다. 산출물 파일을 텍스트 에디터로 열었을 때 구조를 한눈에 파악할 수 있다.

경로 딕셔너리를 JSON에 저장할 때 `{k: str(v) for k, v in paths.items()}`로 `Path` 객체를 문자열로 변환한다. JSON은 `pathlib.Path` 타입을 직접 지원하지 않기 때문이다.

환경 정보(`python_version`, `platform`)를 함께 저장하면, 나중에 결과를 재현할 때 필요한 조건을 알 수 있다. 몇 개월 후 같은 결과를 얻으려면, 동일한 Python 버전과 플랫폼에서 실행해야 한다는 것을 알 수 있다.

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

`requirements.txt`는 이 실습에서 표준 라이브러리만 사용하므로 비어 있을 수 있다. 하지만 파일 자체는 존재해야 하며, 나중에 서드파티 패키지를 추가할 때 활용한다.

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

[실행] 크로스 플랫폼 경로 처리 테스트
  POSIX 경로: .../practice/chapter2/data/output/subdir/file.txt
  Native 경로: .../practice/chapter2/data/output/subdir/file.txt
  경로 구성요소: ('/', 'Users', ..., 'practice', 'chapter2', 'data', 'output', 'subdir', 'file.txt')

[산출물 저장] JSON 파일 생성
  파일 경로: /Users/.../practice/chapter2/data/output/result_20260102_062738.json
  파일 크기: 1717 bytes
```

macOS/Linux에서는 POSIX 경로와 Native 경로가 동일하게 `/`를 사용한다. Windows에서는 Native 경로가 `C:\...` 형태로 출력되지만, POSIX 경로는 `C:/.../` 형태로 변환된다.

출력에서 `[✓]`는 경로가 존재함을 의미한다. 만약 `[✗]`가 나타난다면, 디렉토리 생성에 실패했거나 권한 문제가 있음을 나타낸다.

### 2.5.4 산출물 확인

실행 결과는 `practice/chapter2/data/output/result_20260102_062738.json` 파일로 저장된다. 파일을 열어보면 다음과 같은 구조를 볼 수 있다.

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
    "parts": ["/" , "Users", ..., "practice", "chapter2", "data", "output", "subdir", "file.txt"]
  }
}
```

`timestamp` 필드는 ISO 8601 형식(`2026-01-02T06:27:38.123456`)으로 저장되어, 언제 실행되었는지 정확히 알 수 있다. 마이크로초 단위까지 기록되므로, 1초 이내에 여러 번 실행해도 타임스탬프로 구분할 수 있다.

`platform` 필드는 운영체제 정보를 담는다. `darwin`은 macOS, `linux`는 Linux, `win32`는 Windows를 의미한다. 이 정보로 어떤 환경에서 실행되었는지 추적할 수 있다.

`paths` 딕셔너리는 모든 경로가 절대 경로로 기록되어 있다. 나중에 이 산출물을 다른 팀원이 보더라도, 어느 디렉토리에서 실행되었는지 명확히 알 수 있다.

**표 2.5** 실행 결과 요약

| 항목 | 값 |
|:---|:---|
| Python 버전 | 3.13.7 |
| 플랫폼 | darwin (macOS) |
| 산출물 파일 | result_20260102_062738.json |
| 파일 크기 | 1717 bytes |

파일 크기가 1717 bytes로 작은 것은, 템플릿이 간단한 경로 테스트만 수행했기 때문이다. 실제 프로젝트에서 데이터 분석 결과나 모델 메트릭을 저장하면 파일 크기는 더 커진다.

### 2.5.5 템플릿 커스터마이징

새로운 실습 스크립트를 만들 때는 `2-5-template.py`를 복사하고 `run_task()` 함수만 수정하면 된다. 환경 검증과 경로 설정은 그대로 재사용할 수 있다.

**예시 1: 데이터 생성 및 CSV 저장**

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

이 예시에서는 pandas를 사용하므로, `requirements.txt`에 `pandas`를 추가해야 한다. `encoding="utf-8-sig"`는 Excel에서 한글이 깨지지 않도록 BOM(Byte Order Mark)을 포함한다.

**예시 2: 웹 API 호출 및 결과 저장**

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

이 예시에서는 `requests` 라이브러리가 필요하므로, `requirements.txt`에 추가한다. `response.raise_for_status()`는 HTTP 오류 발생 시 예외를 발생시킨다.

### 2.5.6 템플릿의 장점과 실전 활용

표준 템플릿을 사용하면 다음과 같은 이점이 있다:

1. **일관성**: 모든 실습 스크립트가 동일한 구조를 따라 이해하기 쉽다
2. **재현성**: 환경 정보와 실행 조건이 자동으로 기록되어 나중에 재현할 수 있다
3. **디버깅**: 경로와 환경이 명시적으로 검증되어 문제 발생 시 원인 파악이 빠르다
4. **협업**: 팀원이 같은 템플릿을 사용하면 코드 리뷰와 유지보수가 용이하다
5. **AI 협업**: AI 코딩 어시스턴트에게 "2-5-template.py처럼 작성해줘"라고 요청하면 표준을 따르는 코드를 받을 수 있다

실전에서는 템플릿을 프로젝트의 `scripts/` 또는 `templates/` 디렉토리에 두고, 새로운 스크립트를 만들 때 복사하여 시작한다. 버전 관리 시스템(Git)에 템플릿을 커밋하면, 팀 전체가 동일한 표준을 공유할 수 있다.

AI 코딩 어시스턴트와 협업할 때는 다음과 같이 요청하면 효과적이다:

> "2-5-template.py와 같은 구조로 iris 데이터셋을 불러와서 간단한 분류 모델을 학습하고 결과를 JSON으로 저장하는 스크립트를 작성해줘. pathlib을 사용하고, 환경 검증과 산출물 저장 부분은 템플릿과 동일하게 해줘."

이렇게 구체적으로 요청하면, AI가 템플릿 구조를 따르면서 새로운 기능을 추가한 코드를 생성할 가능성이 높아진다.

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
