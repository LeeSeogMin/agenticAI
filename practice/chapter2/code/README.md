# 제2장: 로컬 개발 환경과 재현 가능한 실행 - 실습 코드

## 실습 환경 설정

### 1. 가상환경 생성 및 활성화

```bash
# macOS/Linux
cd practice/chapter2
python3 -m venv venv
source venv/bin/activate

# Windows
cd practice\chapter2
python -m venv venv
venv\Scripts\activate
```

### 2. 의존성 설치

```bash
pip install -r code/requirements.txt
```

> 참고: 이 장의 실습은 Python 표준 라이브러리만 사용하므로 추가 패키지가 없습니다.

### 3. 실습 코드 실행

```bash
python3 code/2-5-template.py
```

## 폴더 구조

```
practice/chapter2/
├── code/
│   ├── 2-5-template.py      # 표준 템플릿 스크립트
│   ├── requirements.txt     # 의존성 목록
│   └── README.md            # 이 파일
└── data/
    ├── input/               # 입력 데이터
    └── output/              # 산출물 저장
```

## 예상 출력

실행 시 다음과 같은 출력이 생성됩니다:

1. 콘솔 출력: 환경 검증, 경로 설정, 작업 실행, 결과 검증 정보
2. JSON 파일: `data/output/result_YYYYMMDD_HHMMSS.json`

## 학습 포인트

- `Path(__file__).parent`: 스크립트 위치 기준 상대 경로
- `path / "subdir"`: 크로스 플랫폼 경로 조합
- `sys.prefix != sys.base_prefix`: 가상환경 확인
- 타임스탬프 포함 파일명으로 실행 이력 관리
