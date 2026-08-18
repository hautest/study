---
goal: TypeScript 타입 레벨 프로그래밍을 자료 없이 설계하고 설명한다 — 조건부 타입과 분배, 유니온 keyof와 판별 유니온, mapped type, infer와 템플릿 리터럴, satisfies, 변성
type: topic
source: TS 5.9.3 `tsc --strict --noEmit` 실측 · microsoft/TypeScript PR #21316 · TS 2.8 릴리스 노트 (접근 2026-08-18)
updated: 2026-08-18
---

# TS 타입 레벨 프로그래밍

## 이 노트의 형식

**study 스킬 형식이 아니다.** `socratic-teaching-scaffolds`로 진행 중이라 `MAP.md`·출구 시험·근거 인덱스가 없다. 대신 Unit마다 개념 설명 → 문항 출제 → 채점으로 굴러간다. 근거는 파일:라인이 아니라 **tsc 실측**이다.

study 스킬 형식(출구 시험 선작성 → 목차 → 챕터 집필)으로 전환하려면 Phase 1부터 다시 시작해야 한다. 지금 진행분은 그 경우 진단 자료로만 쓰인다.

## 진단 (2026-08-18)

5문항 진단 결과. 타입을 **소비**하는 건 되지만 타입을 **계산**하는 축이 미개척.

| 축 | 진단 시 상태 |
|---|---|
| 조건부 타입 분배 법칙 | 오답 — `ToArray<string \| number>`를 `(string \| number)[]`로 예측 |
| `satisfies` vs 타입 주석 | 모름 |
| 변성(variance) | `x2`를 답으로 골랐으나 이유 모름 |
| `infer` + 템플릿 리터럴 + 재귀 | 모름 |
| 유니온의 `keyof` | 모름 |

시작 스캐폴딩 Level 5(Full Modeling). Unit 0에서 4/4, Unit 1에서 전 문항 정답으로 Level 4로 내림.

## 사다리

| # | Unit | 상태 | 핵심 | 진단 대응 |
|---|------|------|------|----------|
| 0 | `extends`는 부분집합 질문 | ✅ | 타입 = 값의 집합, 방향 비대칭, 객체는 속성 많은 쪽이 부분집합, `extends` 3군데가 같은 질문 | 전제 |
| 1 | 분배 법칙 | ✅ | naked type parameter, 좌변만 분배, 실행 횟수 = 멤버 수, `never`는 0회, `Exclude` 해부 | Q1 |
| 2 | 유니온의 `keyof` · 판별 유니온 | 📖 ← NEXT (문항 출제됨, 답 대기) | `keyof (A \| B)` = 키의 교집합, narrowing, exhaustive check, 분배로 전체 키 모으기 | Q5 |
| 3 | mapped type + key remapping | ⬜ | `as` 절, `Capitalize`, 수식어 추가·제거 | — |
| 4 | `infer` + 템플릿 리터럴 + 재귀 | ⬜ | 문자열에서 타입 추출, 타입 안전 라우터 params | Q4 |
| 5 | `satisfies` / 타입 주석 / `as const` | ⬜ | 리터럴 보존과 제약 검사의 분리 | Q2 |
| 6 | 변성(variance) | ⬜ | method bivariance vs 프로퍼티 반공변, 함수 파라미터 반공변·반환 공변 | Q3 |
| 7 | 응용 종합 | ⬜ | 타입 안전 라우터 또는 API 클라이언트 | — |

사용자가 Unit 순서를 위임했으므로 의존 순서대로 진행한다. Unit 7의 주제 선택은 Unit 6 종료 시 다시 묻는다.

## 진행 규칙

- **모든 타입 값은 제시 전에 tsc로 검증한다.** `npx -y -p typescript@5 tsc --strict --noEmit --target es2022` + `Eq`/`Assert` 이디엄. 검증 없이 "아마 이렇게 된다"를 쓰지 않는다.
- **개념은 한 번에 하나만.** "이해가 안 됨"이 오면 스캐폴딩을 올리고 개념을 쪼갠다. Unit 1을 시작했다가 `extends` 자체가 안 잡혀서 Unit 0을 신설한 전례가 있다.
- **확정 문항은 답이 오기 전에 바꾸지 않는다.** 재제시할 때는 원문 그대로.
- **1인칭 오답 서사를 쓰지 않는다.** "내가 ~를 틀렸는데" 형식은 설명과 산출물 양쪽에서 금지 (2026-08-18 명시 요청).
- **장황하게 쓰지 않는다.** 핵심만.

## 확립된 사실 — Unit 0·1

TS 5.9.3 `--strict` 실측. 복습·재확인용.

```ts
// extends = 부분집합 질문
'hello'         extends string           // true
string          extends 'hello'          // false
{ a: 1; b: 2 }  extends { a: 1 }         // true    왼쪽이 오른쪽 속성을 다 가짐
{ a: 1; b: 2 }  extends { a: 2 }         // false   겹치는 속성은 타입 호환 필요
never           extends string           // true    공집합
string          extends string           // true    양방향 모두 true
unknown         extends string           // false

// 분배 — 좌변이 naked 일 때만
type Naked<T>   = T extends true ? 'Y' : 'N';                  // Naked<boolean>   = 'Y' | 'N'
type Tupled<T>  = [T] extends [true] ? 'Y' : 'N';              // Tupled<boolean>  = 'N'
type Arrayed<T> = T[] extends true[] ? 'Y' : 'N';              // Arrayed<boolean> = 'N'
type Wrapped<T> = { v: T } extends { v: true } ? 'Y' : 'N';     // Wrapped<boolean> = 'N'

// naked 판정은 "정리 후에도 그 파라미터로 남는가"
(T)              // 분배 O   괄호는 타입을 만들지 않는다
Id<T>            // 분배 O   type Id<X> = X 는 풀려서 T
T & unknown      // 분배 O   unknown 교집합은 T로 줄어든다
T & {}           // 분배 X   T로 줄지 않는다
true extends T   // 분배 X   우변 자리
boolean          // 분배 X   타입 파라미터가 아니다

// 우변은 분배되지 않는다
type Check<T, U> = T extends U ? 'Y' : 'N';
Check<'a', 'a' | 'c'>                                 // 'Y'   분배됐다면 'Y' | 'N'

// 실행 횟수 = 유니온 멤버 수
Wrap<'a' | 'b'>   // 'a'[] | 'b'[]     2회
Wrap<string>      // string[]          1회
Wrap<never>       // never             0회 — 본문이 평가되지 않는다
WrapNoDist<never> // never[]           분배를 끄면 1회

// Exclude 는 분배 + never 흡수로 성립
Exclude<'a' | 'b' | 'c', 'a'>                         // 'b' | 'c'
Exclude<'a' | 'b' | 'c', 'a' | 'c'>                   // 'b'
NoDist<'a' | 'b' | 'c', 'a'>                          // 'a' | 'b' | 'c'   하나도 안 빠진다
never | string                                        // string

// 감싼 자리에 따라 안쪽 비교 답의 전달이 달라진다
(x: boolean) => void extends (x: true) => void        // 'Y'   파라미터 자리는 방향 반대
() => boolean        extends () => true               // 'N'   반환 자리는 보존

// any 특례 — 유니온이 아닌데 양쪽 분기 합집합
Naked<any>                                            // 'Y' | 'N'
Naked<unknown>                                        // 'N'
```

## 용어 출처

`naked type parameter`는 공식 용어다. 조건부 타입 도입 PR([#21316](https://github.com/microsoft/TypeScript/pull/21316), Hejlsberg, 2018-02-03)과 [TS 2.8 릴리스 노트](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html)에 나온다:

> Conditional types in which the checked type is a naked type parameter are called _distributive conditional types_.

**현행 핸드북에는 이 표현이 없다.** 인용할 때 현행 핸드북 링크를 달면 독자가 단어를 못 찾는다.

`Exclude` 등 predefined conditional types는 [PR #21847](https://github.com/microsoft/TypeScript/pull/21847)(2018-02-09)에서 **분배보다 6일 늦게** 추가됐다. 따라서 "분배는 `Exclude`를 위해 도입됐다"는 성립하지 않는다. 기록으로 뒷받침되는 건 이슈 [#12215](https://github.com/microsoft/TypeScript/issues/12215)(2016-11-14, 리터럴 타입 빼기 요구)를 #21316이 `Fixes`로 명시했다는 것까지다.

## 부산물

Unit 1 내용으로 블로그 글 작성 중. Notion `블로그` DB의 "Typescript" 페이지 (`3c0a463e83038040aacfe2f445ac6510`). 사용자가 본문을 쓰고 검증·윤문을 받는 방식. `Exclude` 섹션과 실무 버그 섹션은 사용자가 잘라냄.
