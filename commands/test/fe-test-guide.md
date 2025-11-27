# FE 테스트 작성 가이드 (fe-test-implement)

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

## 1. 기본 원칙 (구루 스타일 요약)

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

---

## 2. 테스트 작성 절차

테스트를 만들 때는 항상 아래 순서를 따릅니다.

### 2-1. 대상 컴포넌트/기능 분석

1. 컴포넌트가 하는 역할을 한두 줄로 정리합니다.
2. 외부에서 주어지는 입력/props를 파악합니다.
3. 사용자가 할 수 있는 행동(타이핑, 클릭, 선택 등)을 나열합니다.
4. 그 행동에 따른 **눈에 보이는 결과**를 정리합니다.

이 정리를 바탕으로 **“테스트 시나리오 목록”**을 만듭니다.

### 2-2. 테스트 시나리오 작성 기준

시나리오를 만들 때는 다음 패턴을 따릅니다.

- "어떤 상황에서" (**given**)…
- "어떤 행동을 했을 때" (**when**)…
- "어떤 결과가 나타나야 한다" (**then**)…

예시 시나리오:

- `등록된 todo가 하나도 없으면, "등록된 todo가 없습니다." 라고 표시된다`
- `등록된 todo 개수만큼 "todo-item" role을 가진 요소가 있어야 한다`
- `"done" 버튼을 클릭하면, onRemoveTodo 콜백이 해당 todo의 ID와 함께 호출된다`

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

## 7. 이 커맨드를 사용할 때, 사용자가 줄 수 있는 입력 예시

- "이 컴포넌트에 대한 테스트 파일을 만들어줘"
- "이 훅에 대한 단위 테스트를 Vitest + RTL 스타일로 작성해줘"
- "위의 TodoTddFeature랑 비슷한 패턴으로, 새로운 Todo 컴포넌트 테스트를 작성해줘"

사용자가 컴포넌트/훅 코드 또는 링크된 코드 일부를 제공하면:

1. 기능 분석 → 시나리오 정리
2. 위 가이드라인을 따르는 테스트 코드 생성
3. 필요한 경우, "이 부분은 실제 코드 구조를 조금 바꾸면 테스트가 쉬워진다"는 식의 코멘트까지 함께 제공합니다.