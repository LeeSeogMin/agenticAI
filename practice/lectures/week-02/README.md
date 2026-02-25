# 2주차 실습: Flask Todo 웹앱

이 실습은 Copilot Agent Mode를 사용하여 처음부터 웹앱을 생성하는 과제이다.
아래 절차를 따라 빈 프로젝트에서 시작한다.

## 시작 방법

```bash
mkdir flask-todo && cd flask-todo
python3 -m venv venv && source venv/bin/activate
git init
git add -A && git commit -m "빈 프로젝트 시작"
```

## Agent Mode 프롬프트 예시

VS Code에서 Copilot Chat을 열고 Agent 모드를 선택한 뒤:

> "Flask와 SQLite를 사용하는 Todo 웹 애플리케이션을 만들어 줘.
> 할 일 추가, 완료 표시, 삭제 기능이 필요하고,
> HTML 템플릿은 templates/ 폴더에 분리해 줘."

## 검증 체크리스트

- [ ] `http://localhost:5000` 접속 가능
- [ ] 할 일 추가 동작
- [ ] 할 일 완료 표시 동작
- [ ] 할 일 삭제 동작
- [ ] 빈 문자열 입력 검증
- [ ] SQLite 데이터 재시작 후 유지
- [ ] HTML 출력 XSS 방지 확인
