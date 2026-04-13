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
  # 커스텀 에이전트 프로필 (개별 마운트)
  - ../az-agent-config/agents/reviewer:/a0/agents/reviewer:ro
  - ../az-agent-config/agents/devops:/a0/agents/devops:ro

  # 지식베이스
  - ../az-agent-config/knowledge:/a0/knowledge/custom/team:ro

  # 프로젝트 템플릿
  - ../az-agent-config/templates:/a0/work_dir/templates:ro

  # 커스텀 인스트루먼트
  - ../az-agent-config/instruments:/a0/instruments/custom:ro
```

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

## 관련 문서

- [하네스 킷 저장소](https://github.com/devyoon91/az-cliproxy-docker)
- [전체 구축 가이드](https://github.com/devyoon91/az-cliproxy-docker/blob/main/GUIDE.md)
- [개인화 저장소 분리 가이드](https://github.com/devyoon91/az-cliproxy-docker/blob/main/GUIDE.md#16-팁-개인화-저장소-분리)
- [에이전트 프로필 가이드](https://github.com/devyoon91/az-cliproxy-docker/blob/main/docs/agent-profiles.md)
