# FE 테스트 작성 가이드 (fe-test-guide)

프론트엔드 코드(React 컴포넌트/훅/feature 코드)를 기반으로
**React Testing Library + Vitest/Jest 스타일 테스트 코드를 작성할 때** 사용하는 커맨드입니다.

이 커맨드를 실행하면, 주어진 코드/설명을 분석해서:

1. 어떤 테스트를 작성해야 할지 **시나리오를 정리**하고
2. 그 시나리오에 맞는 **구체적인 테스트 코드(ts/tsx)** 를 생성합니다.

---

## 0. 응답 전체 구조

항상 아래 구조를 유지해서 답변합니다.

1. **요약 (Summary)**  
   - 대상 컴포넌트/기능이 하는 일 2~3줄
   - 어떤 종류의 테스트를 만들 예정인지 한 줄로 설명

2. **테스트 시나리오 목록 (Test Plan)**  
   - `it` 설명 수준으로 **사용자/비즈니스 언어**로 시나리오 bullet 작성  
   - 예:
     - `input에 텍스트를 입력하고 "추가" 버튼을 클릭하면, todo가 리스트에 추가되고 input이 비워진다`
     - `"done" 버튼을 클릭하면, 해당 todo가 리스트에서 제거된다`

3. **테스트 코드 (Test Code)**  
   - 실제로 바로 붙여 넣어 쓸 수 있는 코드 블록
   - 기본 전제:
     - `@testing-library/react`
     - `@testing-library/user-event`
     - `vitest` (`describe`, `it`, `expect`, `vi`)

4. **추가 메모 (Optional)**  
   - 리팩터링 아이디어, 더 앞으로 추가할만한 테스트, 주의할 점 등 3줄 이내

---

## 1. 기본 원칙

테스트 코드를 생성할 때, 아래 원칙을 **항상 우선**으로 둡니다.

### 1-1. 사용자/행동 중심

- 테스트는 **내부 구현이 아니라, 사용자 행동/결과**를 검증해야 합니다.
- 테스트 이름(`it`)은 항상 **유저 스토리/비즈니스 요구사항**처럼 씁니다.

예시:

```ts
it('input에 텍스트를 입력하고 "추가" 버튼을 클릭하면, todo가 리스트에 추가되고 input이 비워진다', async () => {
  // ...
})
```

### 1-2. 구현 디테일보다 최종 결과

다음과 같은 것들을 **직접 테스트하지 않습니다.**

- 내부 state 값
- 특정 hook/함수 호출 여부
- 콜백 호출 횟수(필요 이상으로 상세하게)
- DOM 구조/클래스 이름(`nth-child`, 특정 구조 등)

대신 다음을 테스트합니다.

- 화면에 표시되는 텍스트/요소
- 버튼/입력/인터랙션 이후의 **화면 변화**
- 콜백이 **어떤 인자와 함께 호출되었는지** (정말 필요할 때만)

### 1-3. 테스트 전략: Isolation vs Integration

컴포넌트 테스트는 두 가지 전략을 상황에 맞게 선택합니다.

#### Isolation Strategy (격리 전략)

컴포넌트를 **독립적으로** 테스트합니다. 의존성을 mock하여 컴포넌트 자체의 동작에 집중합니다.

**사용 시점:**
- 컴포넌트의 핵심 로직을 빠르게 검증하고 싶을 때
- 자식 컴포넌트나 외부 서비스에 의존하지 않는 순수한 동작을 테스트할 때
- 특정 컴포넌트의 버그를 격리해서 찾고 싶을 때

**예시:**

```ts
// 자식 컴포넌트를 mock하여 부모 컴포넌트의 로직에 집중
vi.mock('./UserCard', () => ({
  default: vi.fn(({ user }) => `<div>User: ${user.name}</div>`)
}))

test('UserProfile이 로딩 상태를 표시한다', async () => {
  render(<UserProfile userId="123" />)
  expect(screen.getByText('Loading...')).toBeInTheDocument()
})
```

#### Integration Strategy (통합 전략)

여러 컴포넌트가 **함께 협업**하는 방식을 테스트합니다. 실제 데이터 흐름과 컴포넌트 간 상호작용을 검증합니다.

**사용 시점:**
- 컴포넌트 간 데이터 흐름을 검증하고 싶을 때
- 사용자 시나리오 전체를 테스트하고 싶을 때
- 실제 사용 환경과 유사한 상황을 재현하고 싶을 때

**예시:**

```ts
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

test('ProductList가 필터링과 정렬을 함께 처리한다', async () => {
  const mockProducts = [
    { id: 1, name: 'Laptop', category: 'Electronics', price: 999 },
    { id: 2, name: 'Book', category: 'Education', price: 29 }
  ]

  render(<ProductList products={mockProducts} />)
  const user = userEvent.setup()

  // 필터링 동작
  const categorySelect = screen.getByLabelText(/category/i)
  await user.selectOptions(categorySelect, 'Electronics')

  // 통합된 결과 검증
  expect(screen.getByText('Laptop')).toBeInTheDocument()
  expect(screen.queryByText('Book')).not.toBeInTheDocument()
})
```

**선택 기준:**
- 대부분의 경우 **Integration Strategy**를 우선 사용합니다. 실제 사용자 경험과 가까운 테스트가 더 가치가 높습니다.
- 성능이 중요하거나 특정 컴포넌트의 복잡한 로직을 집중적으로 테스트해야 할 때는 **Isolation Strategy**를 사용합니다.

---

## 2. 테스트 작성 절차

테스트를 만들 때는 항상 아래 순서를 따릅니다.

### 2-1. 대상 컴포넌트/기능 분석

1. 컴포넌트가 하는 역할을 한두 줄로 정리합니다.
2. 외부에서 주어지는 입력/props를 파악합니다.
3. 사용자가 할 수 있는 행동(타이핑, 클릭, 선택 등)을 나열합니다.
4. 그 행동에 따른 **눈에 보이는 결과**를 정리합니다.

이 정리를 바탕으로 **“테스트 시나리오 목록”**을 만듭니다.

### 2-2. 테스트 우선순위 (Component Testing Hierarchy)

테스트를 작성할 때는 다음 우선순위를 따릅니다:

1. **Critical User Paths** (핵심 사용자 경로)
   - 사용자가 가장 자주 사용하는 주요 기능
   - 비즈니스 가치가 높은 핵심 플로우
   - 예: `input에 텍스트를 입력하고 "추가" 버튼을 클릭하면, todo가 리스트에 추가되고 input이 비워진다`

2. **Error Handling** (에러 처리)
   - 에러 상태 표시 및 복구
   - 잘못된 입력 처리
   - 네트워크 오류, API 실패 등
   - 예: `API 호출이 실패하면, 에러 메시지가 표시되고 재시도 버튼이 나타난다`

3. **Edge Cases** (경계 케이스)
   - 빈 데이터 상태
   - 최대/최소 값 처리
   - 특수한 입력 상황
   - 예: `등록된 todo가 하나도 없으면, "등록된 todo가 없습니다." 라고 표시된다`

### 2-3. 테스트 시나리오 작성 기준

시나리오를 만들 때는 다음 패턴을 따릅니다.

- "어떤 상황에서" (**given**)…
- "어떤 행동을 했을 때" (**when**)…
- "어떤 결과가 나타나야 한다" (**then**)…

예시 시나리오:

- `등록된 todo가 하나도 없으면, "등록된 todo가 없습니다." 라고 표시된다`
- `등록된 todo 개수만큼 "todo-item" role을 가진 요소가 있어야 한다`
- `"done" 버튼을 클릭하면, onRemoveTodo 콜백이 해당 todo의 ID와 함께 호출된다`

### 2-4. 에러 상태 및 경계 테스트

에러 상황과 경계 케이스를 명시적으로 테스트합니다.

#### 에러 상태 표시 테스트

API 호출 실패, 네트워크 오류 등 에러 상황에서 사용자에게 적절한 피드백이 제공되는지 검증합니다.

```ts
test('API 호출 실패 시 에러 메시지가 표시된다', async () => {
  const fetchTodos: FetchTodosFn = vi.fn().mockRejectedValue(
    new Error('Network error')
  )
  
  render(<TodoTddFeature fetchTodos={fetchTodos} />)
  
  await waitFor(() => {
    expect(screen.getByText(/에러가 발생했습니다/i)).toBeInTheDocument()
  })
})
```

#### Error Boundary 테스트

React Error Boundary가 에러를 적절히 포착하고 fallback UI를 표시하는지 검증합니다.

```ts
function ThrowError({ shouldThrow }: { shouldThrow: boolean }) {
  if (shouldThrow) {
    throw new Error('Component error!')
  }
  return <div>Component working fine</div>
}

test('ErrorBoundary가 에러를 포착하고 fallback을 표시한다', async () => {
  const { rerender } = render(
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <ThrowError shouldThrow={false} />
    </ErrorBoundary>
  )

  // 정상 동작
  expect(screen.getByText('Component working fine')).toBeInTheDocument()

  // 에러 발생
  rerender(
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <ThrowError shouldThrow={true} />
    </ErrorBoundary>
  )

  // Error Boundary가 에러를 포착
  expect(screen.getByText('Something went wrong')).toBeInTheDocument()
})
```

#### 경계 케이스 테스트

빈 데이터, 최대/최소 값, 특수 입력 등 경계 상황을 테스트합니다.

```ts
test('빈 todo 리스트일 때 적절한 메시지가 표시된다', async () => {
  const fetchTodos: FetchTodosFn = vi.fn().mockResolvedValue([])
  render(<TodoTddFeature fetchTodos={fetchTodos} />)
  
  await waitForLoadingComplete()
  
  expect(screen.getByText(/등록된 todo가 없습니다/i)).toBeInTheDocument()
})
```

---

## 3. 테스트 파일/코드 스타일 가이드

### 3-1. 파일 및 describe 구조

기본 구조 예:

```ts
import { render, screen, within } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { TodoList } from './TodoList'
import type { Todo } from './useTodoList'

describe('TodoList', () => {
  // 헬퍼 함수, 공통 setup 등
})
```

- 상위 `describe`에는 **컴포넌트/feature 이름**을 사용합니다.
- 하위 `describe`는 **상황/행동**을 묶을 때 사용합니다.

예:

```ts
describe('"추가" 버튼을 클릭하면', () => {
  it('onAddTodo 콜백이 현재 inputValue와 함께 호출된다', () => {
    // ...
  })

  it('setInputValue가 빈 문자열과 함께 호출된다', () => {
    // ...
  })
})
```

### 3-2. AAA 패턴 (Arrange–Act–Assert)

각 테스트는 다음 구조를 명확히 구분해 작성합니다.

```ts
it('...', async () => {
  // given (Arrange)
  render(...)
  const user = userEvent.setup()

  // when (Act)
  const input = screen.getByRole('textbox')
  await user.type(input, '...')
  await user.click(...)

  // then (Assert)
  expect(...).toBeDefined()
  expect(...).toBe('')
})
```

주석 혹은 빈 줄로 구분해서 **읽었을 때 흐름이 바로 보이도록** 합니다.

---

## 4. React Testing Library 사용 규칙

### 4-0. 테스트 환경 선택: jsdom vs Browser Mode

Vitest는 두 가지 테스트 환경을 제공합니다.

**jsdom (현재 기본 설정):**
- 빠른 실행 속도
- Node.js 환경에서 실행
- 대부분의 DOM API 시뮬레이션
- CSS 렌더링이나 실제 브라우저 API는 제한적

**Browser Mode (선택적):**
- 실제 브라우저 환경에서 실행 (Playwright, WebdriverIO 등 사용)
- 정확한 CSS 렌더링 및 레이아웃 검증
- 실제 브라우저 API 사용 가능
- 실행 속도는 상대적으로 느림

**선택 기준:**
- 현재 프로젝트는 **jsdom**을 기본으로 사용하며, 대부분의 컴포넌트 테스트에 충분합니다.
- CSS 레이아웃, 실제 브라우저 API, 복잡한 이벤트 처리가 중요한 경우에만 **Browser Mode**를 고려합니다.
- Browser Mode 사용 시 `vitest-browser-react` 같은 공식 패키지를 활용할 수 있습니다.

**Browser Mode 설정 예시:**

```ts
// vite.config.ts
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    browser: {
      enabled: true,
      name: 'chromium',
      provider: 'playwright',
    },
  },
})
```

**Testing Library 통합:**
- `@testing-library/react`는 jsdom과 Browser Mode 모두에서 사용 가능합니다.
- Browser Mode에서는 `vitest-browser-react`를 사용할 수도 있지만, 기존 RTL 패턴도 그대로 사용할 수 있습니다.
- 두 방식 모두 동일한 쿼리 우선순위와 사용자 중심 테스트 원칙을 따릅니다.

### 4-1. 쿼리 우선순위

항상 아래 순서로 쿼리를 선택합니다.

1. `getByRole`
2. `getByLabelText`
3. `getByPlaceholderText`
4. `getByText`
5. `getByDisplayValue`
6. `getByTestId` (진짜 방법이 없을 때만)

예시 (Todo 코드 스타일):

```ts
const input = screen.getByRole('textbox')
const addButton = screen.getByRole('button', { name: '추가' })
const todoListElement = screen.getByRole('todo-list')
const todoItems = within(todoListElement).getAllByRole('todo-item')
```

- 실제 a11y role이 있으면 그걸 우선 사용합니다.
- 커스텀 role(`todo-list`, `todo-item`)를 쓸 때는 **프로젝트 컨벤션을 통일**합니다.

### 4-2. userEvent 사용

- 항상 `userEvent.setup()`으로 user를 생성합니다.

```ts
const user = userEvent.setup()
await user.type(input, 'new-todo')
await user.click(addButton)
```

- 단순 `fireEvent`보다 **userEvent를 우선** 사용합니다.
- 비동기 인터랙션에 대비해서, `async/await`를 일관되게 사용합니다.

---

## 5. 콜백/props 테스트 가이드

### 5-1. 상태/props 렌더링 테스트

```ts
it('inputValue prop으로 전달된 값이 input에 표시된다', () => {
  const inputValue = 'test-input'
  renderTodoList([], { inputValue })

  const input = screen.getByRole('textbox') as HTMLInputElement
  expect(input.value).toBe('test-input')
})
```

- props → 화면 반영은 **DOM 값 기준**으로 검증합니다.

### 5-2. 콜백 호출 테스트

- "무엇을 호출했는지"보다 **"어떤 인자로 호출됐는지"**를 중점적으로 검증합니다.
- 호출 횟수는 정말 필요한 경우에만 간단하게 확인합니다.

예시 (좋은 패턴):

```ts
it('"done" 버튼을 클릭하면, onRemoveTodo 콜백이 해당 todo의 ID와 함께 호출된다', async () => {
  const onRemoveTodo = vi.fn()
  const todoList: Todo[] = [
    { id: '1', text: 'todo1' },
    { id: '2', text: 'todo2' },
  ]
  renderTodoList(todoList, { onRemoveTodo })
  const user = userEvent.setup()

  const doneButtons = screen.getAllByRole('button', { name: 'done' })
  await user.click(doneButtons[0])

  expect(onRemoveTodo).toHaveBeenCalledTimes(1)
  expect(onRemoveTodo).toHaveBeenCalledWith('1')
})
```

### 5-3. 피해야 할 패턴

아래처럼 너무 디테일한 호출 시퀀스는 피합니다.

```ts
expect(setInputValue).toHaveBeenCalledTimes(4) // 't', 'e', 's', 't'
expect(setInputValue).toHaveBeenNthCalledWith(1, 't')
// ...
```

대신 이렇게 단순화합니다.

```ts
await user.type(input, 'test')

expect(setInputValue).toHaveBeenCalled()
expect(setInputValue).toHaveBeenLastCalledWith('test')
// 혹은 input.value로 검증
```

---

## 6. 테스트하기 좋은 코드에 맞춘 작성 팁

테스트 코드를 생성할 때, 프로덕션 코드 구조도 같이 고려해 코멘트/제안을 적을 수 있습니다.

- 순수 로직(함수) ↔ 상태 관리 ↔ UI를 분리할 수 있을 것 같으면,
  순수 로직은 **단위 테스트** 예시도 함께 제안합니다.
- 네트워크/시간/랜덤 등 외부 의존성이 있다면,
  - 이를 주입 가능한 형태로 나누고
  - 테스트에서는 mock/가짜 구현을 사용하는 예시 코드를 제안합니다.

예시 코멘트:

> `useTodoList` 훅 내부의 add/remove 로직을
> 순수 함수 `addTodo(list, text)`, `removeTodo(list, id)`로 분리하면
> 이 부분은 별도로 단위 테스트를 만들 수 있습니다.

---

## 7. 디버깅 기법

테스트가 실패하거나 예상대로 동작하지 않을 때 사용할 수 있는 디버깅 방법들입니다.

### 7-1. Browser Dev Tools 활용

Vitest Browser Mode를 사용하는 경우, 실제 브라우저에서 테스트가 실행되므로 브라우저 개발자 도구를 활용할 수 있습니다.

**활용 방법:**
- 테스트 실행 중 브라우저 개발자 도구 열기 (F12 또는 우클릭 → Inspect)
- 테스트 코드나 컴포넌트 코드에 breakpoint 설정
- DOM 구조를 직접 확인하여 렌더링 결과 검증
- Console 탭에서 JavaScript 에러나 경고 확인
- Network 탭에서 API 호출 상태 확인

**Browser Mode 설정 예시:**

```ts
// vite.config.ts
export default defineConfig({
  test: {
    browser: {
      enabled: true,
      name: 'chromium',
      provider: 'playwright',
    },
  },
})
```

테스트 실행 시 `headless: false` 옵션을 사용하면 브라우저 창이 열려 실시간으로 디버깅할 수 있습니다.

### 7-2. Chrome DevTools MCP 활용

Chrome DevTools MCP (Model Context Protocol)를 통해 브라우저 디버깅을 자동화하고 테스트 디버깅을 더 효율적으로 할 수 있습니다.

**활용 시나리오:**
- 테스트 실행 중 자동으로 스크린샷 캡처
- 특정 DOM 요소의 상태 확인
- 네트워크 요청 모니터링
- 콘솔 메시지 자동 수집

**예시 사용법:**

```ts
// MCP를 통한 브라우저 상태 확인
test('컴포넌트 렌더링 상태 디버깅', async () => {
  render(<ComplexComponent />)
  
  // MCP를 통해 현재 페이지 상태 확인
  // - DOM 스냅샷 캡처
  // - 특정 요소의 computed styles 확인
  // - 네트워크 요청 로그 확인
  
  const button = screen.getByRole('button', { name: /submit/i })
  // MCP를 통해 버튼의 실제 렌더링 상태 확인
})
```

**주의사항:**
- MCP는 CI/CD 환경에서는 제한적으로 사용할 수 있습니다
- 로컬 개발 환경에서 테스트 디버깅에 집중하여 활용합니다

### 7-3. 디버그 로깅 전략

전략적으로 로깅을 추가하여 테스트 실패 원인을 파악합니다.

```ts
test('폼 검증 디버깅', async () => {
  render(<ContactForm />)
  const user = userEvent.setup()

  const submitButton = screen.getByRole('button', { name: /submit/i })
  await user.click(submitButton)

  // 디버그: 요소 존재 여부 확인
  const errorElement = screen.queryByText('Email is required')
  console.log('Error element found:', errorElement !== null)
  
  // 디버그: 모든 버튼 확인
  const allButtons = screen.getAllByRole('button')
  console.log('All buttons:', allButtons.map(btn => btn.textContent))

  expect(errorElement).toBeInTheDocument()
})
```

**로깅 활용 팁:**
- `screen.debug()`를 사용하여 현재 DOM 구조 전체를 출력
- `screen.logTestingPlaygroundURL()`로 Testing Playground URL 생성하여 쿼리 확인
- 특정 요소의 속성값을 로깅하여 예상과 다른 값 확인

### 7-4. 셀렉터 검증 방법

요소를 찾지 못할 때, 셀렉터가 올바른지 확인하는 방법입니다.

**접근 가능한 이름 확인:**

```ts
test('버튼 셀렉터 디버깅', async () => {
  render(<LoginForm />)
  
  // 모든 버튼의 접근 가능한 이름 확인
  const buttons = screen.getAllByRole('button')
  for (const button of buttons) {
    const accessibleName = button.getAttribute('aria-label') || button.textContent
    console.log(`Button: "${accessibleName}"`)
  }
  
  // 특정 쿼리 전략 시도
  const submitButton = screen.getByRole('button', { name: /submit/i })
    || screen.getByTestId('submit-button')
    || screen.getByText('Submit')
})
```

**대체 쿼리 전략:**

```ts
// 여러 방법으로 같은 요소 찾기
const element = screen.getByRole('button', { name: /submit/i })
  || screen.getByLabelText(/submit/i)
  || screen.getByTestId('submit-btn')
  || screen.getByText('Submit')
```

### 7-5. 비동기 이슈 디버깅

비동기 작업이 완료되기 전에 검증을 시도하는 문제를 해결합니다.

```ts
test('비동기 컴포넌트 동작 디버깅', async () => {
  render(<AsyncUserProfile userId="123" />)

  // waitFor를 사용하여 요소가 나타날 때까지 대기
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument()
  }, {
    timeout: 3000, // 타임아웃 증가
    onTimeout: (error) => {
      // 타임아웃 시 현재 DOM 상태 출력
      screen.debug()
      throw error
    }
  })
})
```

**비동기 디버깅 팁:**
- `waitFor`의 `timeout` 옵션을 조정하여 충분한 대기 시간 확보
- `onTimeout` 콜백에서 현재 상태를 로깅하여 실패 시점의 상황 파악
- `screen.debug()`로 타임아웃 발생 시점의 DOM 상태 확인

---

## 8. 이 커맨드를 사용할 때, 사용자가 줄 수 있는 입력 예시

- "이 컴포넌트에 대한 테스트 파일을 만들어줘"
- "이 훅에 대한 단위 테스트를 Vitest + RTL 스타일로 작성해줘"
- "위의 TodoTddFeature랑 비슷한 패턴으로, 새로운 Todo 컴포넌트 테스트를 작성해줘"

사용자가 컴포넌트/훅 코드 또는 링크된 코드 일부를 제공하면:

1. 기능 분석 → 시나리오 정리
2. 위 가이드라인을 따르는 테스트 코드 생성
3. 필요한 경우, "이 부분은 실제 코드 구조를 조금 바꾸면 테스트가 쉬워진다"는 식의 코멘트까지 함께 제공합니다.