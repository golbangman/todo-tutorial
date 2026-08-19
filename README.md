# Todo Tutorial

[Claude Code Playbook](https://docs.claude-hunt.com) 강의의 실습용 저장소입니다. Next.js 와 shadcn/ui 로 시작한 Todo 앱을 단계별로 발전시키며 Claude Code 사용법을 익힙니다.

## 관련 링크

- 강의 본문: https://docs.claude-hunt.com
- 수강생 결과물 공유: https://claude-hunt.com

## 기능

- 할 일 추가 / 완료 토글 / 더블클릭 편집 / 삭제
- 우선순위(높음·보통·낮음), 마감일, 카테고리(업무·개인·쇼핑) 지정
- 상태별 필터(전체·진행중·완료), 카테고리 필터, 제목 검색
- 이름순 · 생성일순 · 마감일순 정렬
- `localStorage` 기반 저장(새로고침 후에도 유지)
- `d` 키로 다크 모드 토글

## 기술 스택

- Next.js 16 (App Router, Turbopack)
- React 19
- Tailwind CSS v4
- shadcn/ui (radix-mira 스타일, taupe 베이스, Phosphor 아이콘)
- TypeScript / ESLint / Prettier
- Vitest + Testing Library (컴포넌트/훅 단위 테스트)
- 패키지 매니저: bun

## 시작하기

```bash
bun install
bun dev
```

개발 서버는 기본적으로 [http://localhost:3000](http://localhost:3000) 에서 열립니다.

자주 쓰는 스크립트:

```bash
bun dev            # 개발 서버 실행
bun run build      # 프로덕션 빌드
bun run start      # 빌드 결과 실행
bun run lint       # ESLint
bun run typecheck  # tsc --noEmit
bun run format     # Prettier 포맷팅
bun run test       # Vitest 테스트 실행
bun run test:watch # Vitest 워치 모드
```

## 프로젝트 구조

```
app/            페이지, 레이아웃 (Server Component 우선)
components/     Todo 관련 컴포넌트 + components/ui (shadcn 프리미티브)
hooks/          use-todos 등 클라이언트 상태 훅
lib/            타입, 정렬/검색 유틸
```

## 컴포넌트 추가

shadcn/ui 컴포넌트는 다음과 같이 추가합니다.

```bash
bunx --bun shadcn@latest add button
```

`components/ui` 디렉토리에 컴포넌트가 추가됩니다.

## 컴포넌트 사용

```tsx
import { Button } from "@/components/ui/button";
```

## Claude Code 설정

이 저장소는 Claude Code 스킬/에이전트/훅을 함께 제공합니다.

- 스킬: `commit`(Conventional Commits 커밋), `shadcn`(shadcn 컴포넌트 작업), `ui-bug-report`(Chrome 탭 상태로 버그 리포트 작성)
- 에이전트: `test-planner`(테스트 커버리지 분석 및 테스트 계획 수립)
- 훅: 파일 저장 시 자동으로 `eslint --fix` 실행 (`.claude/hooks/lint.sh`)
- GitHub Actions에서 PR 리뷰(`claude-code-review.yml`)와 `@claude` 멘션 응답(`claude.yml`) 자동화

## Contributors

- 토이크레인 - Frontend Developer
