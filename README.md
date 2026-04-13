# az-agent-config

[az-cliproxy-docker](https://github.com/devyoon91/az-cliproxy-docker) 하네스 킷의 **개인화 저장소**입니다.

## 배경

`az-cliproxy-docker`는 Agent Zero + CLIProxy 기반의 AI 에이전트 개발 환경(하네스 킷)입니다. 하네스 킷은 **인프라와 공용 설정**만 제공하며, 회사/팀/개인 고유의 에이전트 프로필, 지식베이스, 프로젝트 템플릿은 이 저장소에서 별도 관리합니다.

이렇게 분리하면:
- 하네스 킷 업데이트 시 개인화 내용과 **충돌 없음**
- 팀원 간 개인화 설정을 **비공개로 공유** 가능
- 여러 환경(PC, 서버)에서 **동일한 개인화** 적용

## 구조

```
az-agent-config/
├── agents/                     # 도메인 특화 에이전트 프로필
│   ├── reviewer/               # 코드 리뷰 전문가
│   │   ├── agent.yaml
│   │   └── prompts/
│   │       └── agent.system.main.specifics.md
│   └── devops/                 # 인프라/배포 전문가
│       ├── agent.yaml
│       └── prompts/
│           └── agent.system.main.specifics.md
│
├── knowledge/                  # 지식베이스 (에이전트가 자동 참조)
│   ├── coding-standards.md     # 코딩 표준
│   ├── api-conventions.md      # REST API 규칙
│   ├── architecture.md         # 아키텍처 패턴
│   └── git-workflow.md         # Git 워크플로우
│
├── templates/                  # 프로젝트 보일러플레이트
│
├── instruments/                # 커스텀 인스트루먼트
│
└── README.md
```

## 에이전트 프로필

Agent 0 (마스터)가 작업에 따라 전문 서브 에이전트를 호출합니다.

```
Agent 0 (마스터, Developer)
    │
    ├── call_subordinate(profile="reviewer")  → 코드 리뷰
    ├── call_subordinate(profile="devops")    → 배포 설정
    └── 추가 프로필...                         → 도메인 전문가
```

### 기본 제공 프로필

| 프로필 | 역할 | 출처 |
|--------|------|------|
| `developer` | 풀스택 개발 (마스터 기본값) | Agent Zero 내장 |
| `researcher` | 조사/연구 | Agent Zero 내장 |
| `hacker` | 보안 전문 | Agent Zero 내장 |

### 커스텀 프로필 (이 저장소)

| 프로필 | 역할 | 핵심 |
|--------|------|------|
| `reviewer` | 코드 리뷰 전문가 | 보안/성능/품질/아키텍처 4관점 체크리스트 |
| `devops` | 인프라/배포 전문가 | Docker/K8s/CI/CD/모니터링 표준 |

### 프로필 추가 예시

회사/도메인 특화 프로필을 추가할 수 있습니다:

```
agents/
  ├── dba/          ← DB 설계/최적화 전문
  ├── qa/           ← 테스트 전문
  ├── backend/      ← 회사 백엔드 스택 특화
  ├── frontend/     ← 회사 프론트 스택 특화
  └── tech-writer/  ← 기술 문서 작성 전문
```

새 프로필 생성:
1. `agents/{이름}/agent.yaml` 작성 (title, description, context)
2. `agents/{이름}/prompts/agent.system.main.specifics.md` 작성 (전문성 정의)

## 연동 방법

### 1. 디렉토리 배치

```
D:/docker/
  ├── agent-zero_cliproxy/     # 하네스 킷
  └── az-agent-config/         # 이 저장소
```

### 2. docker-compose.yml에 볼륨 추가

`az-cliproxy-docker/docker-compose.yml`의 agent-zero 서비스에 추가:

```yaml
volumes:
  # ── 에이전트 프로필 (⚠️ 반드시 개별 마운트) ──
  - ../az-agent-config/agents/reviewer:/a0/agents/reviewer:ro
  - ../az-agent-config/agents/devops:/a0/agents/devops:ro
  # 프로필 추가 시 여기에 한 줄씩 추가

  # ── 지식베이스 (⚠️ 서브 디렉토리로 마운트) ──
  - ../az-agent-config/knowledge:/a0/knowledge/custom/team:ro

  # ── 아래는 통째 마운트 가능 ──
  - ../az-agent-config/templates:/a0/work_dir/templates:ro
  - ../az-agent-config/instruments:/a0/instruments/custom:ro
```

> **⚠️ 주의: 절대 통째로 마운트하지 마세요**
> 
> | 디렉토리 | 통째 마운트 | 이유 |
> |----------|:---:|------|
> | `agents/` | ❌ | 내장 프로필(developer, researcher 등)이 사라짐 |
> | `knowledge/` | ❌ | 내장 지식(main/about/)이 사라짐 |
> | `templates/` | ✅ | work_dir 하위라 충돌 없음 |
> | `instruments/` | ✅ | 별도 경로라 충돌 없음 |
> 
> ```yaml
> # ❌ 이렇게 하면 안 됩니다
> - ../az-agent-config/agents:/a0/agents          # 내장 프로필 전부 사라짐
> - ../az-agent-config/knowledge:/a0/knowledge     # 내장 지식 전부 사라짐
> ```

### 3. 컨테이너 재시작

```bash
cd az-cliproxy-docker
docker compose up -d agent-zero --force-recreate
```

## 지식베이스

`knowledge/`에 문서를 넣으면 Agent Zero가 작업 시 **자동으로 검색하여 참조**합니다.

```
"API 만들어줘" → api-conventions.md를 참조하여 회사 표준에 맞게 구현
"이 코드 리뷰해줘" → coding-standards.md를 참조하여 표준 위반 체크
```

### 작성 팁

- **구체적으로**: "변수명은 camelCase" 보다 "Java는 camelCase, Python은 snake_case, DB 컬럼은 snake_case"
- **예시 포함**: 규칙마다 좋은 예/나쁜 예를 함께 작성
- **이유 설명**: "왜 이 규칙인지" 배경을 적으면 에이전트가 맥락을 이해
- **파일 분리**: 주제별로 파일을 나누면 에이전트가 관련 내용만 정확히 검색

## 프로젝트 템플릿

`templates/`에 프로젝트 보일러플레이트를 넣어두면 Agent Zero가 새 프로젝트 생성 시 복사하여 사용합니다.

```
templates/
  ├── springboot-api/         ← Spring Boot API 템플릿
  │   ├── src/main/java/...
  │   ├── build.gradle
  │   ├── Dockerfile
  │   └── .github/workflows/ci.yml
  ├── nextjs-app/             ← Next.js 프론트엔드 템플릿
  └── python-fastapi/         ← FastAPI 백엔드 템플릿
```

사용 예시:
> "templates/springboot-api를 복사해서 새 프로젝트 만들어줘"

### 작성 팁

- 실제로 **동작하는 최소 프로젝트** 구조
- Dockerfile, CI/CD, 테스트 구조 포함
- README.md에 사용법 작성
- 회사 표준 설정(코드 스타일, 린트, 포맷터) 포함

---

## 커스텀 인스트루먼트

`instruments/`에 실행 가능한 스크립트/프로그램을 넣으면 Agent Zero가 자동으로 인식하여 `code_execution_tool`로 실행합니다.

```
instruments/
  ├── deploy-checker/         ← 배포 상태 확인 스크립트
  │   ├── README.md           ← 인스트루먼트 설명 (에이전트가 읽음)
  │   └── check.sh
  └── db-migration/           ← DB 마이그레이션 도우미
      ├── README.md
      └── migrate.py
```

각 인스트루먼트에 `README.md`를 넣으면 에이전트가 설명을 읽고 언제 사용해야 하는지 판단합니다.

### 작성 팁

- 폴더별 하나의 기능
- `README.md`에 용도, 사용법, 파라미터 설명
- 실행 가능한 스크립트 포함 (sh, py, js 등)

---

## 관련 문서

- [하네스 킷 저장소](https://github.com/devyoon91/az-cliproxy-docker)
- [전체 구축 가이드](https://github.com/devyoon91/az-cliproxy-docker/blob/main/GUIDE.md)
- [개인화 저장소 분리 가이드](https://github.com/devyoon91/az-cliproxy-docker/blob/main/GUIDE.md#16-팁-개인화-저장소-분리)
- [에이전트 프로필 가이드](https://github.com/devyoon91/az-cliproxy-docker/blob/main/docs/agent-profiles.md)
