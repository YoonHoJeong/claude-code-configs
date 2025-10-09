---
name: verify-docs
description: "문서 품질, 정확성, 완전성, 사용성을 체계적으로 검증"
category: quality
complexity: advanced
mcp-servers: []
personas: [documentation-reviewer]
---

# /verify-docs - 문서 검증 및 품질 보증

## 트리거
- 문서 품질 검토 요청
- 게시 전 문서 검증 필요
- API 문서 정확성 검증
- 코드-문서 동기화 검증 요구사항
- 문서 완전성 감사

## 사용법
```
/verify-docs [문서경로|디렉토리경로] [--focus accuracy|completeness|clarity|structure|usability] [--strict] [--parallel]
```

## 오케스트레이션 모드

### 단일 문서 모드
단일 파일 경로가 제공되면 해당 문서를 직접 검증합니다.

### 다중 문서 모드 (병렬 오케스트레이션)
디렉토리 경로 또는 glob 패턴이 제공되면:
1. **탐색**: Glob을 사용하여 디렉토리 내 모든 문서 파일 찾기
2. **위임**: 각 문서마다 병렬 Task 에이전트(documentation-reviewer) 실행
3. **수집**: 모든 병렬 에이전트로부터 JSON 리포트 수집
4. **종합**: 결과를 통합된 품질 리포트로 병합, 교차 문서 인사이트 제공

**병렬화 전략:**
- 최대 5개 동시 검증 에이전트
- 각 에이전트가 독립적으로 하나의 문서 검증
- 문서 레벨 세분성으로 결과 집계
- 종합 단계에서 교차 문서 일관성 검사

## 동작 흐름
1. **탐색**: 대상 문서 식별 (단일 파일 또는 디렉토리 스캔)
2. **오케스트레이션**: 다중 문서인 경우 병렬 검증 에이전트 실행
3. **파싱**: 모든 코드 예제, API 시그니처, 타입 정의, import 경로 추출
4. **교차 참조**: 실제 소스 코드 파일과 문서 검증
5. **유효성 검사**: 코드 예제 정확성, 문법 정확성, API 시그니처 확인
6. **평가**: 완전성, 명확성, 구조, 사용성 평가
7. **종합**: 결과 집계 및 교차 문서 일관성 검사 수행
8. **리포트**: 분류된 이슈가 포함된 구조화된 JSON 검증 리포트 생성

주요 동작:
- 다중 문서 검증을 위한 **병렬 오케스트레이션** (Task 에이전트 위임)
- 효율적인 소스 코드 검증을 위한 병렬 파일 읽기
- 다차원 품질 평가 (정확성, 완전성, 명확성, 구조, 사용성)
- 심각도 기반 이슈 분류 (critical, important, suggestion)
- 실행 가능한 개선 권장사항이 포함된 품질 점수
- 의도적 누락 감지 (의사 코드, 외부 참조, TODO 섹션)
- 교차 문서 일관성 검증 (용어, API 참조, 버전 관리)

## 검증 체크리스트

### 1. 코드 정확성 검증
- [ ] 모든 코드 예제가 올바른 문법을 가지고 있음
- [ ] API 시그니처가 실제 소스 코드와 일치함
- [ ] 타입 정의가 정확하고 완전함
- [ ] Import 경로 및 의존성이 정확함

**검증 방법:**
1. 문서에서 코드 블록 추출 (언어별 파싱)
2. import 문에서 파일 경로 식별 (예: `@scope/package` → `libs/package/src/`)
3. Read 도구를 사용하여 실제 소스 파일 읽기 (병렬 실행)
4. 타입 정의, API 시그니처, 예제 코드 비교
5. 불일치 사항을 파일:라인 참조와 함께 리포트

### 2. 완전성 검증
- [ ] 모든 public API가 문서화되어 있음
- [ ] 필수 매개변수가 설명되어 있음
- [ ] 반환값과 타입이 문서화되어 있음
- [ ] 오류 조건 및 예외 처리가 설명되어 있음
- [ ] 사전 요구사항 및 의존성이 명시되어 있음

### 3. 명확성 검증
- [ ] 기술 용어가 첫 사용 시 정의됨
- [ ] 코드 예제가 실행 가능하고 현실적임
- [ ] 단계별 설명이 명확함
- [ ] 용어 사용이 전체적으로 일관됨
- [ ] 대상 독자에게 적합한 언어 사용

### 4. 구조 검증
- [ ] 정보가 논리적으로 흐름
- [ ] 헤더와 섹션이 효과적으로 구성됨
- [ ] 목차 및 내부 링크 제공됨
- [ ] 관련 문서에 대한 교차 참조 존재

### 5. 사용성 검증
- [ ] 예제를 복사하여 직접 실행 가능
- [ ] 일반적인 사용 사례가 다뤄짐
- [ ] 문제 해결 가이드 제공됨
- [ ] 성공 기준이 명확함
- [ ] 다음 단계가 제안됨

## 검증 생략 규칙

**의도적으로 건너뛰기 (검증 안 함):**
- 외부 라이브러리 코드 인용 (검증 불필요)
- 개념적 의사 코드 (의도적으로 실행 불가)
- 향후 구현 계획 섹션 (TODO)

**마킹 방법:**
```typescript
// 📝 의사 코드 - 검증 생략
function conceptualExample() { ... }
```

## 출력 형식

검증 결과는 엄격한 필드 정의와 계층적 이슈 구성을 가진 구조화된 JSON으로 반환됩니다.

```json
{
  "document_list": [
    {
      "doc_id": "string",      // 문서 식별자 또는 경로
      "purpose": "string"      // 문서 목적 또는 요약
    }
  ],
  "checklist_summary": {
    "all_checks": 12,           // 총 수행된 검사 수
    "passed": 9,                // 통과한 검사 수
    "critical": 1,              // 치명적 이슈 개수
    "important": 2,             // 중요 이슈 개수
    "suggestion": 0,            // 제안 개수
    "quality_score": 79         // 점수: 100 - 10×critical - 3×important - 1×suggestion
  },
  "critical_issues": [
    {
      "document": "string",       // 예: filename:line 또는 섹션 이름
      "problem": "string",        // 구체적 문제 설명
      "current_state": "string",  // 현재 문서 발췌
      "solution": "string",       // 수정 방법
      "reference": "string"       // 관련 소스 코드 또는 file:line
    }
  ],
  "important_issues": [
    {
      "document": "string",
      "problem": "string",
      "solution": "string"
    }
  ],
  "suggestions": [
    {
      "document": "string",
      "suggestion": "string"
    }
  ],
  "passed_checks": [
    {
      "document": "string",
      "check": "string"
    }
  ]
}
```

### 필수 출력 규칙
- 각 이슈는 모든 필수 필드를 반드시 포함해야 함: document, problem/solution/suggestion 등
- 누락되거나 불분명한 정보는 `null`로 표시
- 모든 필드 타입은 고정됨 (string/array/object/number)
- `document_list`는 `doc_id` (파일 경로)와 `purpose` (요약) 필드를 가진 배열
- `checklist_summary`는 총/통과/이슈 개수와 계산된 점수 포함
- 이슈 배열(critical_issues/important_issues/suggestions)은 비어있으면 `[]`로 출력
- `passed_checks`는 문서 및 검사 이름 필드와 함께 실제 통과 항목 제공

### 출력 예시
```json
{
  "document_list": [
    {"doc_id": "libs/ui/button/README.md", "purpose": "Button 컴포넌트 사용법 및 props 문서"}
  ],
  "checklist_summary": {"all_checks": 12, "passed": 9, "critical": 1, "important": 2, "suggestion": 0, "quality_score": 79},
  "critical_issues": [
    {"document": "libs/ui/button/src/index.ts:23", "problem": "핵심 props 누락", "current_state": "export type ButtonProps ...", "solution": "props 정의에 ... 추가", "reference": "libs/ui/button/src/types.ts"}
  ],
  "important_issues": [
    {"document": "README.md", "problem": "반환값 설명 불분명", "solution": "반환값 설명 추가"}
  ],
  "suggestions": [],
  "passed_checks": [
    {"document": "README.md", "check": "모든 public API 문서화됨"}
  ]
}
```

## 도구 조정
- **Glob**: 디렉토리 내 문서 파일 탐색
- **Task (병렬)**: 병렬 검증을 위한 다중 documentation-reviewer 에이전트 실행
- **Read**: 검증을 위한 병렬 소스 파일 읽기 (배치 작업)
- **Grep**: 코드 참조 및 API 사용 패턴 추출
- **Native Analysis**: 문서 vs 실제 코드 비교, 결과 집계

## 주요 패턴
- **단일 문서**: 문서 파싱 → import 추출 → 소스 읽기 (병렬) → 비교 → JSON 리포트
- **다중 문서 오케스트레이션**: 문서 Glob → Task 에이전트 실행 (병렬, 최대 5개) → JSON 집계 → 교차 문서 종합
- **정확성 검증**: 문서 파싱 → import 추출 → 소스 읽기 (병렬) → 비교 → 불일치 리포트
- **완전성 검사**: 문서에서 API 추출 → 소스 코드 public exports와 비교 → 격차 식별
- **품질 평가**: 다차원 점수화 → 분류된 이슈 → 실행 가능한 권장사항
- **구조화된 리포팅**: JSON 출력 → 명확한 심각도 레벨 → 파일:라인 참조
- **교차 문서 일관성**: 용어 일관성 → API 참조 검증 → 버전 정렬

## 예제

### API 문서 검증
```
/verify-docs docs/api/authentication.md --focus accuracy
# 소스 코드와 대조하여 모든 API 엔드포인트, 매개변수, 응답 타입 검증
# 구체적인 파일:라인 참조와 함께 불일치 사항 리포트
```

### 포괄적 문서 감사
```
/verify-docs README.md --strict
# 5차원 검증 전체 수행 (정확성, 완전성, 명확성, 구조, 사용성)
# 품질 점수 및 분류된 이슈가 포함된 상세 JSON 리포트 생성
```

### 코드 예제 검증
```
/verify-docs docs/guides/getting-started.md --focus accuracy
# 모든 코드 예제가 문법적으로 정확하고 실행 가능한지 검증
# 실제 소스 코드와 import 및 API 사용 교차 참조
```

### 다중 문서 검증 (병렬 오케스트레이션)
```
/verify-docs docs/ --parallel
# docs/ 디렉토리 내 모든 .md 파일 탐색
# 각 문서마다 병렬 Task 에이전트 실행 (최대 5개 동시)
# 각 에이전트가 할당된 문서를 독립적으로 검증
# 모든 JSON 리포트를 통합된 품질 평가로 집계
# 교차 문서 일관성 검사 수행 (용어, API 참조)
```

### 병렬 실행 흐름 예제
```
입력: /verify-docs docs/api/ --parallel

1. Glob "docs/api/**/*.md" → [auth.md, users.md, payments.md, webhooks.md]

2. 4개의 병렬 Task 에이전트 실행:
   - 에이전트 1: auth.md 검증 → JSON 리포트 1
   - 에이전트 2: users.md 검증 → JSON 리포트 2
   - 에이전트 3: payments.md 검증 → JSON 리포트 3
   - 에이전트 4: webhooks.md 검증 → JSON 리포트 4

3. 결과 집계:
   {
     "total_documents": 4,
     "total_quality_score": 82,
     "per_document_reports": [...],
     "cross_document_issues": [
       {"problem": "일관되지 않은 API 버전 참조", "documents": ["auth.md", "users.md"]}
     ]
   }
```

## 범위

**수행할 것:**
- 실제 소스 코드와 대조하여 문서를 체계적으로 검증
- **다중 문서에 대한 병렬 검증 에이전트 오케스트레이션**
- 효율적인 검증을 위한 병렬 파일 작업 수행
- 심각도 분류가 포함된 구조화되고 실행 가능한 품질 리포트 생성
- 치명적 오류와 개선 기회 모두 식별
- **교차 문서 일관성 검사와 함께 다중 문서 결과 집계**

**수행하지 않을 것:**
- 외부 라이브러리 문서 또는 서드파티 코드 참조 검증
- 의도적으로 표시된 의사 코드 또는 개념적 예제 검증
- 프로젝트별 문서 표준 또는 관례 무시
- 문서 자동 수정 (리포팅만 수행)
- **5개 이상의 동시 검증 에이전트 실행 (리소스 제약)**
