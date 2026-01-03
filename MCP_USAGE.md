# MCP 사용 가이드

## 개요

이 저장소는 Model Context Protocol(MCP)을 통해 “에이전트가 쓸 도구/리소스”를 표준화하는 흐름을 교재 실습에 포함한다. MCP 설정은 프로젝트 루트의 `mcp.json`에서 관리하며, 예시는 **절대 경로 없이** 작성한다.

## 1) 기본: Filesystem MCP 서버

교재 집필/실습에서는 파일 입출력이 핵심이므로, 최소 구성으로 filesystem 서버를 사용한다.

- 설정 파일: `mcp.json`
- 역할: 프로젝트 폴더(현재 디렉토리) 범위 내 파일 접근

## 2) Claude Code 연동

`mcp.json`이 있는 프로젝트에서 Claude Code를 실행하면 설정을 감지해 MCP 서버를 시작한다.

```bash
claude
```

## 3) 보안/운영 원칙

- API 키는 `.env`에만 저장하고, `mcp.json`에서는 `${ENV_VAR}` 형태로 참조한다.
- filesystem 서버는 가능한 한 프로젝트 디렉토리 범위로 제한한다.
- 허용 명령/권한 정책은 `.claude/settings.local.json`로 관리한다.

## 4) 참고 자료

- MCP 공식 문서: https://modelcontextprotocol.io/
