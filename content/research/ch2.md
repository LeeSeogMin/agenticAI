# 제2장: 로컬 개발 환경과 재현 가능한 실행 — 자료 조사

## 조사 일자
2026-01-02

---

## 1. Python 가상환경 (venv)

### 1.1 핵심 개념
- **가상환경**: 프로젝트별로 독립적인 Python 환경을 생성하여 의존성 충돌 방지
- **venv 모듈**: Python 3.3+에서 표준 라이브러리로 포함된 가상환경 생성 도구
- **격리 원리**: `sys.prefix`와 `sys.base_prefix`가 다른 값을 가질 때 가상환경 활성 상태

### 1.2 가상환경 생성 및 활성화

**생성**:
```bash
python -m venv /path/to/new/virtual/environment
```

**활성화 (플랫폼별)**:
| 플랫폼 | 셸 | 명령어 |
|--------|-----|--------|
| POSIX | bash/zsh | `source <venv>/bin/activate` |
| POSIX | fish | `source <venv>/bin/activate.fish` |
| POSIX | csh/tcsh | `source <venv>/bin/activate.csh` |
| Windows | cmd.exe | `<venv>\Scripts\activate.bat` |
| Windows | PowerShell | `<venv>\Scripts\Activate.ps1` |

**비활성화**: `deactivate`

### 1.3 주요 옵션
- `--system-site-packages`: 시스템 site-packages 접근 허용
- `--clear`: 기존 환경 삭제 후 재생성
- `--upgrade`: Python 업그레이드 시 환경 갱신
- `--without-pip`: pip 설치 생략

### 1.4 중요 참고사항
- 가상환경은 이식 불가(non-portable) - 절대 경로가 포함되어 있음
- 항상 `requirements.txt`로 재현 가능해야 함
- 소스 컨트롤에 포함하지 않음

**출처**: [Python 공식 문서 - venv](https://docs.python.org/3/library/venv.html)

---

## 2. pathlib - 객체지향 경로 처리

### 2.1 핵심 개념
- **목적**: 객체지향적 파일시스템 경로 처리
- **장점**: 플랫폼 독립적, 가독성 좋음, 연산자 오버로딩(`/`)
- **Python 버전**: 3.4+에서 표준 라이브러리

### 2.2 클래스 구조
```
PurePath
├── PurePosixPath  (비Windows)
├── PureWindowsPath (Windows)
└── Path
    ├── PosixPath
    └── WindowsPath
```

### 2.3 주요 사용법

**기본 사용**:
```python
from pathlib import Path

# 현재 파일 기준 상대 경로
base_path = Path(__file__).parent

# 경로 조합 (/ 연산자)
data_path = base_path / "data" / "input"

# 경로 존재 확인
if data_path.exists():
    ...

# 디렉토리 생성
data_path.mkdir(parents=True, exist_ok=True)

# 파일 읽기/쓰기
content = Path("file.txt").read_text()
Path("output.txt").write_text("내용")
```

**유용한 메서드**:
- `Path.cwd()`: 현재 작업 디렉토리
- `Path.home()`: 사용자 홈 디렉토리
- `path.resolve()`: 절대 경로로 변환
- `path.parent`: 상위 디렉토리
- `path.name`: 파일명
- `path.stem`: 확장자 제외 파일명
- `path.suffix`: 확장자
- `path.glob("*.py")`: 패턴 매칭

### 2.4 os.path vs pathlib 비교

| os.path | pathlib |
|---------|---------|
| `os.path.dirname()` | `path.parent` |
| `os.path.basename()` | `path.name` |
| `os.path.join()` | `path / "child"` |
| `os.path.exists()` | `path.exists()` |
| `os.path.isfile()` | `path.is_file()` |
| `os.path.isdir()` | `path.is_dir()` |

**출처**: [Python 공식 문서 - pathlib](https://docs.python.org/3/library/pathlib.html)

---

## 3. 의존성 관리 (requirements.txt)

### 3.1 버전 지정 방식
```
package==1.0.0    # 정확한 버전
package>=1.0.0    # 최소 버전
package~=1.0.0    # 호환 버전 (1.0.x)
package>=1.0,<2.0 # 범위 지정
```

### 3.2 requirements.txt 생성
```bash
# 현재 환경의 모든 패키지 (비권장 - 불필요한 패키지 포함)
pip freeze > requirements.txt

# 권장: 직접 필요한 패키지만 수동 관리
```

### 3.3 설치
```bash
pip install -r requirements.txt
```

### 3.4 버전 고정 전략
- **개발 시**: 유연한 버전 (`>=1.0`)
- **배포 시**: 엄격한 버전 (`==1.0.0`)
- **lock 파일**: pip-tools, poetry 등으로 전이 의존성까지 고정

---

## 4. 크로스 플랫폼 호환성

### 4.1 경로 차이
| 요소 | Windows | macOS/Linux |
|------|---------|-------------|
| 구분자 | `\` (백슬래시) | `/` (슬래시) |
| 루트 | `C:\` | `/` |
| 홈 | `C:\Users\name` | `/home/name` 또는 `/Users/name` |

### 4.2 안전한 코드 작성

**권장**:
```python
from pathlib import Path

# 스크립트 위치 기준 상대 경로
BASE_DIR = Path(__file__).parent.resolve()
DATA_DIR = BASE_DIR / "data"
```

**비권장**:
```python
# 하드코딩된 경로 - 크로스 플랫폼 불가
path = "C:\\Users\\name\\project\\data"
path = "/Users/name/project/data"
```

### 4.3 환경 변수 활용
```python
import os
from pathlib import Path

# 홈 디렉토리
home = Path.home()

# 환경 변수
data_dir = os.environ.get("DATA_DIR", "data")
```

---

## 5. 실패 사례 및 대응

### 5.1 의존성 버전 불일치
**사례**: pandas 1.x에서 작성된 코드가 pandas 2.x에서 경고/오류 발생
```python
# pandas 1.x
df.append(row)  # deprecated in 2.0

# pandas 2.x
df = pd.concat([df, row])
```

**대응**:
- requirements.txt에 버전 명시: `pandas==1.5.3`
- 업그레이드 시 변경사항 확인

### 5.2 경로 문제
**사례**: Windows에서 작성된 `path = "data\\file.csv"`가 Linux에서 실패

**대응**:
```python
from pathlib import Path
path = Path("data") / "file.csv"
```

### 5.3 Python 버전 불일치
**사례**: f-string, walrus operator 등 신규 문법 사용 시 구버전에서 오류

**대응**:
- pyproject.toml에 `requires-python = ">=3.9"` 명시
- 실행 전 버전 확인 로직 추가

---

## 6. 핵심 참고문헌

1. **Python 공식 문서 - venv**
   - URL: https://docs.python.org/3/library/venv.html
   - 내용: 가상환경 생성 및 관리 공식 가이드

2. **Python 공식 문서 - pathlib**
   - URL: https://docs.python.org/3/library/pathlib.html
   - 내용: 객체지향 경로 처리 API 레퍼런스

3. **Python Packaging User Guide**
   - URL: https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/
   - 내용: 가상환경 및 pip 사용 가이드

4. **PEP 405 - Python Virtual Environments**
   - URL: https://peps.python.org/pep-0405/
   - 내용: 가상환경 설계 철학 및 구현 원리

---

## 7. 실습 설계 참고

### 표준 템플릿 구조
```python
#!/usr/bin/env python3
"""실행-검증-산출물 저장 표준 템플릿"""
import sys
from pathlib import Path
from datetime import datetime
import json

# 1. 환경 검증
def verify_environment():
    """Python 버전 및 의존성 확인"""
    pass

# 2. 경로 설정
def setup_paths():
    """입출력 경로 구성"""
    pass

# 3. 실행 및 검증
def run_and_verify():
    """핵심 로직 실행 및 결과 검증"""
    pass

# 4. 산출물 저장
def save_output(result: dict, output_dir: Path):
    """타임스탬프 포함 파일명으로 저장"""
    pass

if __name__ == "__main__":
    main()
```
