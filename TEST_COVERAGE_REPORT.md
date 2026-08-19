# 테스트 커버리지 분석 보고서

작성일: 2026-08-19

## 1. 요약

| 구분 | 수치 |
|---|---|
| 로직/컴포넌트 소스 파일(테스트 대상) | 12개 (lib 2, hooks 1, components 9) |
| 테스트 파일 존재 | 8개 |
| 테스트 파일이 전혀 없는 소스 | 4개 (`app/page.tsx`, `app/layout.tsx`, `components/theme-provider.tsx`, `components/ui/*` 5개) |

- 파일 단위 커버리지: 핵심 로직/컴포넌트 기준 8/12 (약 67%). UI 프리미티브·레이아웃까지 포함하면 8/21 (약 38%)이지만, 이들 대부분은 shadcn 스타일 프리미티브나 순수 조립 컴포넌트라 우선순위는 낮음.
- **함수 단위로 보면 갭이 더 큼**: `hooks/use-todos.ts`는 파일 자체는 "테스트 있음"으로 분류되지만, 실제로는 `addTodo`와 로드 로직만 테스트되고 `toggleTodo`/`deleteTodo`/`editTodo`는 단 한 번도 직접 호출되지 않음.

## 2. 기존 테스트 파일과 커버 범위

| 파일 | 커버 범위 | 비고 |
|---|---|---|
| `lib/todo-utils.test.ts` | `sortTodos`(created/name/due, dueDate 없는 항목, 원본 불변) + `matchesSearch`(포함 여부, 대소문자, 빈 검색어) | 양호 |
| `hooks/use-todos.test.ts` | `addTodo` 기본값/전달값, localStorage 구버전 데이터 보정, 손상된 JSON 로드 방어 | `toggleTodo`/`deleteTodo`/`editTodo` 미검증 |
| `components/todo-item.test.tsx` | 렌더링, 체크박스→onToggle, 삭제→onDelete, 우선순위 뱃지, 완료 취소선, dueDate/category 표시, IME 조합 중 Enter 무시 | blur 커밋, Escape 취소, 빈 문자열 편집 미검증 |
| `components/todo-list.test.tsx` | 추가/빈 입력/토글/삭제/우선순위/새로고침 유지, 상태 필터, 정렬, 검색, 카테고리 태그·필터 | 더블클릭 편집(Edit) 플로우 전혀 없음 |
| `components/todo-input.test.tsx` | 우선순위/마감일/카테고리 선택 및 제출, 제출 후 초기화, 빈/공백 입력 처리 | 양호 |
| `components/todo-filter.test.tsx` | 옵션 렌더링, 선택 표시, onChange 호출 | 충분 |
| `components/todo-category-filter.test.tsx` | 옵션 렌더링, 선택 표시, onChange 호출 | 충분 |
| `components/todo-sort.test.tsx` | 옵션 렌더링, 선택 표시, onChange 호출 | 충분 |
| `components/todo-search.test.tsx` | 옵션 렌더링, 선택 표시, onChange 호출 | 충분 |

## 3. 테스트가 전혀 없는 소스 파일

- `app/layout.tsx` — 단순 조립이지만 내부 `ThemeProvider`의 `ThemeHotkey`에 조건 분기 로직(`d` 키 감지, 수정키/repeat/defaultPrevented 무시, input/textarea/contentEditable 포커스 중 무시)이 있음에도 테스트 없음.
- `app/page.tsx` — 정적 조립(Server Component), 로직 없음.
- `components/ui/aurora-text.tsx`, `button.tsx`, `card.tsx`, `checkbox.tsx`, `input.tsx` — shadcn 스타일 프리미티브. 로직 거의 없음.

## 4. 테스트는 있으나 커버리지가 부족한 부분

### High
- **`hooks/use-todos.ts`**: `toggleTodo`, `deleteTodo`, `editTodo`가 훅 단위로 직접 검증되지 않음. 특히 `editTodo`의 다음 분기는 앱 전체 어디에서도 테스트되지 않는 핵심 상태 변경 로직:
  ```ts
  if (!trimmed) { deleteTodo(id); return; }
  ```
  편집창을 빈 문자열로 저장하면 항목이 삭제되는 숨은 동작.
- **`components/todo-list.tsx`**: 실제 `useTodos` 훅과 연결된 통합 시나리오에서 "더블클릭 → 편집 → 저장" 또는 "편집 후 빈 값으로 저장 → 항목 삭제"가 전혀 검증되지 않음. 정렬/필터/검색은 매우 촘촘한 반면 편집 기능만 통째로 빠져 있음.

### Medium
- **`components/todo-item.tsx`**: blur 시 커밋(`onBlur={commit}`), Escape로 취소 후 원본 텍스트 복원, 공백/빈 값으로 지우고 커밋했을 때 `onEdit`에 전달되는 값 검증 누락.
- **`components/theme-provider.tsx`**: `isTypingTarget` 조건 분기와 단축키 토글 로직 미검증.

### Low
- **`lib/todo-utils.ts`**: 동일 `createdAt`/`dueDate`/`text` 등 동점(tie) 케이스의 정렬 안정성 미검증(현재 기능에 크리티컬하지는 않음).

## 5. 우선순위별 테스트 계획

### High
1. `hooks/use-todos.ts:editTodo` — 빈 문자열/공백만 남기고 편집 시 항목이 실제로 삭제되는지(`deleteTodo` 위임) 검증. 삭제로 이어지는 숨은 분기라 누락 시 회귀 위험 큼.
2. `hooks/use-todos.ts:toggleTodo` — 존재하지 않는 id 호출 시 무변화, 특정 id만 토글되고 나머지는 그대로인지 검증.
3. `hooks/use-todos.ts:deleteTodo` — 존재하지 않는 id 삭제 시도 시 배열 불변, 다중 항목 중 하나만 삭제되는지 검증.
4. `components/todo-list.tsx` — 통합 시나리오로 "더블클릭 → 텍스트 수정 → Enter → 목록 반영 → 새로고침 후 유지" 및 "편집 중 텍스트를 모두 지우고 저장 → 항목이 목록에서 사라짐" 추가.

### Medium
5. `components/todo-item.tsx` — Escape 취소 시 draft가 원본 `todo.text`로 복원되고 `onEdit`이 호출되지 않는지, blur 시 자동 커밋되는지 검증.
6. `components/theme-provider.tsx:ThemeHotkey` — `d` 키 입력 시 테마 토글, input/textarea/contentEditable 포커스 중 무시, Ctrl/Alt/Meta 조합 시 무시, `event.repeat`/`defaultPrevented` 시 무시하는 조건 분기 검증.
7. `lib/todo-utils.ts:sortTodos` — 동일 `dueDate`/`createdAt`을 가진 항목들의 순서 안정성(원래 배열 순서 유지 여부) 케이스 추가.

### Low
8. `components/ui/*` — 단순 스타일/variant 매핑이며 상위 컴포넌트 테스트에서 간접 검증되고 있어 별도 단위 테스트 우선순위 낮음.
9. `app/page.tsx` — 정적 조립 컴포넌트, 추가 테스트 불필요.

## 6. 가장 시급한 3가지 시나리오

1. `hooks/use-todos.test.ts`에 `editTodo`가 빈 문자열을 받으면 해당 항목이 목록에서 완전히 제거되는지 검증하는 테스트 추가 (`result.current.editTodo(id, "   ")` 호출 후 `todos`에서 해당 id 부재 확인).
2. `hooks/use-todos.test.ts`에 `toggleTodo`/`deleteTodo` 단위 테스트를 추가해 다건 목록에서 대상 id만 영향받고 나머지는 불변임을 검증.
3. `components/todo-list.test.tsx`에 "더블클릭으로 편집 모드 진입 → 텍스트 변경 → Enter로 커밋 → 화면에 반영 및 재마운트 후에도 유지" 통합 시나리오 추가.

## 관련 파일 경로

- `hooks/use-todos.ts`
- `hooks/use-todos.test.ts`
- `components/todo-item.tsx`
- `components/todo-item.test.tsx`
- `components/todo-list.tsx`
- `components/todo-list.test.tsx`
- `components/theme-provider.tsx`
- `lib/todo-utils.ts`
