# 설치 가이드

이 문서는 이 저장소(대중 교재 집필 템플릿)를 로컬에서 작업하기 위한 최소 설치 절차를 정리한다. 경로는 **프로젝트 루트 기준 상대 경로**로만 안내한다.

## 시스템 요구사항

- **OS**: macOS, Linux, Windows
- **Python**: 3.10 이상 (실습 코드/자동화 스크립트)
- **Node.js**: 18+ (Word 변환 파이프라인 `ms-word/`)

## 1) Python 실습 환경

프로젝트 루트에서 다음을 실행한다.

```bash
python3 -m venv venv
source venv/bin/activate  # macOS/Linux
# venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

설치 검증은 “예상 출력”을 문서에 적지 않고, 필요한 모듈이 import 되는지만 확인한다.

```bash
python3 -c "import sys; print(sys.version)"
python3 -c "import dotenv; import pypandoc; import docx; print('OK')"
```

## 2) Word 변환(ms-word)

```bash
cd ms-word
npm install
```

## 3) 주의사항

- `venv/`는 로컬 개발 편의를 위한 산출물이며, 원고/실습 산출물로 취급하지 않는다.
- 본문에 포함되는 실행 결과/표/수치는 반드시 실제 실행으로 얻은 값만 사용한다(`docs/sample.md` 참고).
