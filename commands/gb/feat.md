---
name: feature-dev
description: "기능 명세부터 구현까지 체계적인 검증을 통한 완전한 기능 개발 라이프사이클"
category: orchestration
complexity: advanced
mcp-servers: []
personas: [requirements-analyst, deep-research-agent, system-architect, frontend-architect, backend-architect, python-expert, quality-engineer, refactoring-expert]
---

# /feature-dev - 기능 개발 워크플로우

> **컨텍스트 프레임워크 노트**: 이 오케스트레이션 명령어는 명세, 리서치, 계획, 구현, 회고의 5단계를 통해 완전한 기능 개발 라이프사이클을 조정합니다.

## 트리거
- 완전한 라이프사이클 관리가 필요한 기능 개발 요청
- 체계적인 명세, 리서치, 구현이 필요한 복잡한 기능
- 포괄적인 문서화 및 검증 워크플로우가 필요한 프로젝트
- 컨셉에서 동작하는 코드까지 구조화된 진행이 필요한 개발 요청

## 컨텍스트 트리거 패턴
```
/feature-dev [기능명] [--spec-file 경로/명세.md] [--from-prompt] [--skip-spec] [--parallel]
```

## 오케스트레이션 아키텍처

### 컨텍스트 분리 전략
각 단계는 **전문 에이전트**로 위임되어 독립적인 컨텍스트에서 실행됩니다:
- **컨텍스트 효율성**: 각 에이전트는 해당 단계에만 집중하여 컨텍스트 사용 최적화
- **전문성**: 단계별 특화된 페르소나와 도구 사용
- **병렬성**: 독립적인 하위 작업들을 병렬로 실행 가능
- **격리성**: 단계 간 명확한 입출력으로 에러 전파 차단

### 메인 오케스트레이터 역할
1. **워크플로우 조정**: 5단계의 순차적 실행 관리
2. **상태 관리**: Serena를 통한 단계 간 상태 전달
3. **품질 게이트**: 각 단계 완료 후 검증 및 승인
4. **에러 처리**: 실패 시 복구 전략 실행
5. **진행 추적**: TodoWrite로 전체 워크플로우 가시성 제공

## 동작 흐름

### 0단계: 명세 (requirements-analyst 에이전트)
**입력**: 사용자 프롬프트 또는 기존 명세 문서
**출력**: `docs/{기능명}/spec.md`
**에이전트**: `requirements-analyst`

**에이전트 프롬프트**:
```
기능명: {기능명}
입력: {사용자_프롬프트 또는 명세_파일}

작업:
1. 요구사항 분석 및 명세 문서 작성
2. 모호한 부분 사용자에게 질문
3. 기능/비기능 요구사항 명확화
4. 성공 기준 및 제약사항 정의
5. docs/{기능명}/spec.md 생성

출력: spec.md 파일 경로 반환
```

**검증**: `/global:verify-docs docs/{기능명}/spec.md`

### 1단계: 리서치 (deep-research-agent)
**입력**: `docs/{기능명}/spec.md`
**출력**: `docs/{기능명}/research.md`
**에이전트**: `deep-research-agent`

**에이전트 프롬프트**:
```
명세 파일: docs/{기능명}/spec.md

작업:
1. 명세를 읽고 필요한 기술 스택 파악
2. 코드베이스에서 유사 패턴 검색 (Grep/Glob)
3. 프레임워크/라이브러리 문서 조사
4. 외부 모범 사례 리서치
5. 발견사항을 docs/{기능명}/research.md로 종합

출력: research.md 파일 경로 반환
```

### 2단계: 계획 (system-architect 에이전트)
**입력**: `spec.md`, `research.md`
**출력**: `docs/{기능명}/plan.md`
**에이전트**: `system-architect`

**에이전트 프롬프트**:
```
명세: docs/{기능명}/spec.md
리서치: docs/{기능명}/research.md

작업:
1. 두 문서를 읽고 구현 전략 수립
2. 모듈/컴포넌트 분해 및 파일 구조 설계
3. 의존성 그래프 및 통합 포인트 정의
4. 상세 구현 계획 작성 (파일 위치, 인터페이스, 작업 순서)
5. Git 워크플로우 및 커밋 전략
6. docs/{기능명}/plan.md 생성

출력: plan.md 파일 경로 및 예상 작업 개수 반환
```

### 3단계: 구현 (적응형 에이전트)
**입력**: `spec.md`, `research.md`, `plan.md`
**출력**: 기능 브랜치에 동작하는 코드
**에이전트**: 기술 스택에 따라 선택
- Frontend: `frontend-architect`
- Backend: `backend-architect`
- Full-stack: `python-expert` 또는 적절한 페르소나

**에이전트 프롬프트**:
```
명세: docs/{기능명}/spec.md
리서치: docs/{기능명}/research.md
계획: docs/{기능명}/plan.md

작업:
1. 세 문서를 읽고 구현 계획 이해
2. Git 브랜치 확인 및 feature/{기능명} 생성
3. 계획에 따라 TodoWrite 작업 생성 (3-15개)
4. 각 작업마다:
   - 코드 작성/편집
   - Lint 체크 (package.json 확인)
   - Type 체크 (package.json 확인)
   - Git 커밋 (설명적 메시지)
   - TodoWrite 상태 업데이트
5. 완전한 구현 (TODO/stub 없음)
6. 임시 파일 정리

출력: 브랜치명 및 커밋 개수 반환
```

**품질 게이트**:
- 모든 lint/type 체크 통과
- 실제 동작 코드 확인
- 임시 파일 정리 검증

### 4단계: 회고 (quality-engineer + refactoring-expert)
**입력**: `spec.md`, `git diff main..feature/{기능명}`
**출력**: `docs/{기능명}/reflect.md`
**에이전트**: `quality-engineer` (병렬) + `refactoring-expert` (병렬)

**에이전트 프롬프트 (병렬 실행)**:

**quality-engineer**:
```
명세: docs/{기능명}/spec.md
Diff: git diff main..feature/{기능명}

작업:
1. 명세 요구사항과 구현 비교
2. 미구현/누락 기능 식별
3. 승인 기준 충족 검증
4. 에러 핸들링 완전성 확인
5. 테스트 커버리지 평가

출력: JSON 형식 검증 리포트
```

**refactoring-expert**:
```
Diff: git diff main..feature/{기능명}

작업:
1. 코드 중복 패턴 식별
2. 불필요한/죽은 코드 발견
3. 리팩토링 기회 제안
4. 성능 개선 포인트
5. 코드 품질 점수

출력: JSON 형식 개선 리포트
```

**종합**: 두 리포트를 병합하여 `docs/{기능명}/reflect.md` 생성

## 도구 조정
- **Task (필수)**: 모든 단계를 전문 에이전트로 위임 (컨텍스트 분리)
- **Read**: 에이전트가 생성한 문서 읽기 및 검증
- **Bash**: Git 상태 확인, 브랜치 관리
- **TodoWrite**: 워크플로우 레벨 진행 추적 (5개 단계)
- **SlashCommand**: `/global:verify-docs`로 문서 품질 검증

## 에이전트 실행 패턴

### 순차 실행 (단계 간)
```
오케스트레이터
  ├─ Task(requirements-analyst) → spec.md
  │   └─ 완료 대기 + 검증 게이트
  ├─ Task(deep-research-agent) → research.md
  │   └─ 완료 대기 + 검증 게이트
  ├─ Task(system-architect) → plan.md
  │   └─ 완료 대기 + 검증 게이트
  ├─ Task(적응형-구현-에이전트) → 코드
  │   └─ 완료 대기 + 검증 게이트
  └─ Task(quality-engineer + refactoring-expert, 병렬) → reflect.md
      └─ 완료 대기 + 최종 검증
```

### 병렬 실행 (단계 내)
- **명세 단계**: 단일 에이전트 (대화형)
- **리서치 단계**: 에이전트 내부에서 코드베이스 검색 + 외부 리서치 병렬
- **계획 단계**: 단일 에이전트 (순차 분석)
- **구현 단계**: 에이전트 내부에서 독립 모듈 병렬 구현 가능
- **회고 단계**: 2개 에이전트 병렬 실행 (quality + refactoring)

## 주요 패턴

### 오케스트레이터 워크플로우
```yaml
초기화:
  - TodoWrite 생성 (5단계 작업)
  - 기능명 및 옵션 파싱
  - Git 상태 확인

각 단계:
  단계_시작:
    - TodoWrite 상태 → in_progress

  에이전트_실행:
    - Task 도구로 전문 에이전트 호출
    - 에이전트 프롬프트에 명확한 입출력 명세
    - 에이전트 완료 대기

  검증_게이트:
    - 출력 파일 존재 확인
    - 품질 검증 실행 (필요시)
    - 사용자 승인 (필요시)

  단계_완료:
    - TodoWrite 상태 → completed
    - 다음 단계로 전환

최종화:
  - 전체 워크플로우 요약
  - 생성된 문서 및 브랜치 정보
  - 다음 단계 권장사항 (PR, 테스트 등)
```

### 품질 게이트
- **명세 후**: `/global:verify-docs` + 사용자 확인
- **리서치 후**: 발견사항 완전성 자체 검증
- **계획 후**: 아키텍처 일관성 자체 검증
- **구현 후**: Lint + Type 체크 (에이전트 내부)
- **회고 후**: 병렬 리포트 병합 검증

### 세션 관리
- 중단된 워크플로우 재개 시 `--resume` 플래그 사용
- 각 주요 작업 완료 후 체크포인트 (문서 파일로 상태 유지)

## 예제

### 사용자 프롬프트로부터 (전체 워크플로우)
```
/feature-dev user-authentication --from-prompt

# 실행 흐름:
# 1. TodoWrite 생성: [명세, 리서치, 계획, 구현, 회고]
# 2. Task(requirements-analyst) → 사용자와 대화 → spec.md
# 3. /global:verify-docs spec.md → 사용자 확인
# 4. Task(deep-research-agent) → research.md
# 5. Task(system-architect) → plan.md
# 6. Task(frontend-architect) → 코드 + 커밋
# 7. Task(quality-engineer, refactoring-expert 병렬) → reflect.md
# 8. 최종 요약 및 다음 단계 안내
```

### 기존 명세로부터
```
/feature-dev payment-integration --spec-file docs/prd/payment-system.md

# 실행 흐름:
# 1. TodoWrite 생성: [명세 검증, 리서치, 계획, 구현, 회고]
# 2. /global:verify-docs 실행 → spec.md 검증
# 3. 사용자 확인 후 리서치 단계부터 진행
# 4-8. 위와 동일
```

### 명세 단계 건너뛰기
```
/feature-dev data-export --skip-spec

# 실행 흐름:
# 1. TodoWrite 생성: [리서치, 계획, 구현, 회고] (4단계)
# 2. Task(deep-research-agent) → research.md
# 3-6. 위와 동일 (명세 없이 리서치부터 시작)
```

### 재개 기능
```
/feature-dev notification-service --resume

# 실행 흐름:
# 1. docs/{기능명}/ 디렉토리 확인
# 2. 존재하는 문서 파일로 완료된 단계 파악
# 3. 마지막 완료된 단계 이후부터 재개
# 4. 예: spec.md, research.md 존재 → 계획 단계부터 시작
```

## 범위

**수행할 것:**
- **오케스트레이션**: 5단계 워크플로우를 전문 에이전트로 조정
- **컨텍스트 분리**: 각 단계를 독립 에이전트로 실행하여 효율성 극대화
- **품질 게이트**: 각 단계 완료 후 검증 및 승인 프로세스
- **상태 관리**: 문서 파일을 통한 단계 간 상태 전달 및 재개 기능
- **진행 가시성**: TodoWrite로 워크플로우 레벨 진행 추적

**수행하지 않을 것:**
- **직접 구현**: 오케스트레이터는 직접 코드 작성하지 않음 (에이전트 위임)
- **검증 생략**: 어떤 단계도 검증 게이트 없이 다음 단계로 진행 불가
- **단일 컨텍스트**: 모든 작업을 하나의 컨텍스트에서 수행 (비효율)
- **에이전트 간섭**: 에이전트 작업 중 직접 개입하지 않음 (완료 대기)
- **상태 손실**: 단계 간 전환 시 상태 정보 누락

## 워크플로우 상태 관리

### 재개 로직
```yaml
재개_판단:
  1. docs/{기능명}/ 디렉토리 확인
  2. 파일 존재 여부로 완료 단계 판단:
     - spec.md 존재 → 0단계 완료
     - research.md 존재 → 1단계 완료
     - plan.md 존재 → 2단계 완료
     - feature/{기능명} 브랜치 존재 → 3단계 완료
     - reflect.md 존재 → 4단계 완료 (전체 완료)
  3. 다음 단계부터 시작
```

## 출력 표준

### 문서 구조
```
docs/{기능명}/
├── spec.md          # 0단계: 명세
├── research.md      # 1단계: 리서치 결과
├── plan.md          # 2단계: 구현 계획
└── reflect.md       # 4단계: 회고 분석
```

### Git 워크플로우
```
feature/{기능명}
├── 초기 커밋: "feat: add {기능명} specification"
├── 구현 작업당 커밋: "feat: implement {특정-컴포넌트}"
├── 검증 커밋: "chore: fix lint/type errors for {컴포넌트}"
└── 최종 커밋: "feat: complete {기능명} implementation"
```

## 에러 핸들링

### 검증 실패
- **명세 검증 실패**: 명세 개선을 위해 사용자와 반복
- **Lint/타입 체크 실패**: 다음 작업으로 진행하기 전 에러 수정
- **Git 작업 실패**: 명확한 에러 메시지 및 해결 단계 제공
- **회고에서 격차 식별**: 개선을 위한 후속 todo 생성

### 복구 전략
- 단계 경계에서 자동 체크포인트 생성
- 충돌 복구를 위한 Serena 메모리를 통한 상태 지속성
- 작업 손실 방지를 위한 Git 브랜치 안전성
- 중요한 이슈에 대한 단계 롤백 기능