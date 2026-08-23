---
goal: TypeScript 타입 레벨 프로그래밍을 자료 없이 설계하고 설명한다 — 조건부 타입과 분배, 유니온 keyof와 판별 유니온, mapped type, infer와 템플릿 리터럴, satisfies, 변성
type: topic
source: TS 5.9.3 `tsc --strict --noEmit` 실측 · microsoft/TypeScript PR #21316 · TS 2.8 릴리스 노트 (접근 2026-08-18)
updated: 2026-08-20
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

시작 스캐폴딩 Level 5(Full Modeling). Unit 0에서 4/4, Unit 1에서 전 문항 정답으로 Level 4로 내림. Unit 2 후반에 타입을 **작성**하는 문항에서 두 번 연속 "모르겠어"가 나와 **Level 5로 되올림** — 읽기와 작성이 다른 능력이고, 채점 정답률이 작성 능력을 보증하지 않는다. 작성 문항은 조리법 + 빈칸 1개로 낸다.

## 사다리

| # | Unit | 상태 | 핵심 | 진단 대응 |
|---|------|------|------|----------|
| 0 | `extends`는 부분집합 질문 | ✅ | 타입 = 값의 집합, 방향 비대칭, 객체는 속성 많은 쪽이 부분집합, `extends` 3군데가 같은 질문 | 전제 |
| 1 | 분배 법칙 | ✅ | naked type parameter, 좌변만 분배, 실행 횟수 = 멤버 수, `never`는 0회, `Exclude` 해부 | Q1 |
| 2 | 유니온의 `keyof` · 판별 유니온 | ✅ | `keyof (A \| B)` = 키의 교집합, 값 타입은 인덱스 접근에서 합집합, narrowing 3수단, exhaustive check, 조건부 타입 작성 조리법, 분배로 전체 키 모으기, `Extract`류 자작 | Q5 |
| 3 | mapped type + key remapping | 📖 ← NEXT | `as` 절, `Capitalize`, 수식어 추가·제거 | — |
| 4 | `infer` + 템플릿 리터럴 + 재귀 | ⬜ | 문자열에서 타입 추출, 타입 안전 라우터 params | Q4 |
| 5 | `satisfies` / 타입 주석 / `as const` | ⬜ | 리터럴 보존과 제약 검사의 분리 | Q2 |
| 6 | 변성(variance) | ⬜ | method bivariance vs 프로퍼티 반공변, 함수 파라미터 반공변·반환 공변 | Q3 |
| 7 | 응용 종합 | ⬜ | 타입 안전 라우터 또는 API 클라이언트 | — |

사용자가 Unit 순서를 위임했으므로 의존 순서대로 진행한다. Unit 7의 주제 선택은 Unit 6 종료 시 다시 묻는다.

## 진행 규칙

- **모든 타입 값은 제시 전에 tsc로 검증한다.** `npx -y -p typescript@5 tsc --strict --noEmit --target es2022` + `Eq`/`Assert` 이디엄. 검증 없이 "아마 이렇게 된다"를 쓰지 않는다.
- **개념은 한 번에 하나만.** "이해가 안 됨"이 오면 스캐폴딩을 올리고 개념을 쪼갠다. Unit 1을 시작했다가 `extends` 자체가 안 잡혀서 Unit 0을 신설한 전례가 있다.
- **확정 문항은 답이 오기 전에 바꾸지 않는다.** 재제시할 때는 원문 그대로.
- **해설에는 문항을 다시 붙인다.** 채점·해설을 쓸 때 각 항목 바로 위에 해당 문항 코드를 그대로 인용한다. 학습자가 위아래로 스크롤하지 않게 한다 (2026-08-19 요청).
- **문항은 자기완결적으로 쓴다.** "위 정의 사용" 같은 참조 금지. 문항이 쓰는 타입 정의(`Circle`·`Rect`·`Shape` 등)를 그 문항 코드 블록 안에 매번 다시 적는다 (2026-08-20 요청). 재제시 때도 정의를 붙이는 것은 문항 변경이 아니다.
- **과거 문항·사실은 번호 대신 코드로 되살린다.** 힌트·해설에서 `2-4의 X3`, `Unit 1 사실 목록` 같은 번호 참조는 학습자에게 스크롤 숙제가 된다. 터미널 대화에는 되돌아볼 목차가 없다. 인용할 사실은 그 자리에 코드 한두 줄로 다시 쓴다 (2026-08-20 지적). Unit 번호는 노트 구조용이고 학습자에게 내보내는 문장에는 쓰지 않는다.
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

## 확립된 사실 — Unit 2

TS 5.9.3 `--strict` 실측. `Eq`/`Assert` 통과분.

```ts
type Circle   = { shape: 'circle'; r: number };
type Rect     = { shape: 'rect'; w: number; h: number };
type Triangle = { shape: 'tri'; b: number; h: number };
type Shape    = Circle | Rect;

// keyof 는 키 이름만 본다. 유니온이면 교집합
keyof Shape                     // 'shape'
keyof Circle                    // 'shape' | 'r'
keyof Rect                      // 'shape' | 'w' | 'h'
// 방향 규칙 — 판별자 충돌 없는 예로 확인
type P = { a: string; c: boolean };
type Q = { b: number; c: boolean };
keyof (P | Q)                   // 'c'                = keyof P ∩ keyof Q   값이 덜 확정 → 키가 줄어든다
keyof (P & Q)                   // 'a' | 'c' | 'b'    = keyof P ∪ keyof Q   값이 더 확정 → 키가 늘어난다

// 값 타입 충돌은 keyof 에 영향 없다. 인덱스 접근에서 합쳐진다
type A = { id: number; name: string };
type C = { id: string; name: string };
keyof (A | C)                   // 'id' | 'name'         이름이 같으면 남는다
(A | C)['id']                   // string | number       값은 여기서 합집합
(A | C)['name']                 // string
Shape['shape']                  // 'circle' | 'rect'     조건부 타입 없이 유니온을 훑는다

// 키가 한쪽에 없으면 인덱스 접근 자체가 막힌다
type B = { id: number; age: number };
(A | B)['name']                 // error TS2339: Property 'name' does not exist on type 'A | B'.

// 구체 타입은 분배되지 않는다 — naked 타입 파라미터가 아니기 때문
(Shape extends unknown ? keyof Shape : never)              // 'shape'
type AllKeys<T>  = T extends unknown ? keyof T : never;    // AllKeys<Shape> = 'shape'|'r'|'w'|'h'
type Wrong1<T>   = keyof T extends unknown ? keyof T : never;   // Wrong1<Shape> = 'shape'   검사 대상이 keyof T
type Wrong2<T>   = [T] extends [unknown] ? keyof T : never;     // Wrong2<Shape> = 'shape'   분배 꺼짐
// 참 분기에 keyof T 를 써도 분배가 꺼졌으면 그 T 는 유니온 통째다

// 좁히기(narrowing) 수단
s.shape === 'circle'            // 판별자 비교 — 좁힌다
'r' in s                        // in 연산자 — 좁힌다
s.r ? ... : ...                 // 값의 truthy 검사 — 좁히지 않는다 (Property 'r' does not exist on type 'Shape')

// exhaustive check 가 터지는 지점은 대입이다
const _e: never = s;            // error TS2322: Type 'Triangle' is not assignable to type 'never'.
return _e;                      // 에러 아님 — never 는 bottom type 이라 string 에 할당 가능
```

### 조건부 타입 작성 조리법

목표를 코드로 바꿀 때 묻는 것 3개.

| # | 질문 | 답이 결정하는 것 |
|---|---|---|
| ① | 유니온 멤버마다 따로 실행돼야 하나? | 예 → 좌변에 naked `T`. 아니오 → `[T] extends [...]`로 끈다 |
| ② | 조건이 실제로 걸러야 하나? | 예 → 진짜 조건. 아니오(분배만 필요) → `extends unknown` |
| ③ | 남길 것과 버릴 것은? | 남길 것을 참 분기에, **버릴 것은 `never`** |

③이 핵심이다. `never`는 "버림"이라는 뜻이고, 근거는 유니온에서 사라지는 성질이다 — `never | 'a' | never` = `'a'`. 걸러내기는 이 성질로 구현된다. 거짓 분기는 "안 맞을 때 무엇을 낼까"가 아니라 "안 맞으면 아무것도 안 낸다"를 쓰는 자리다.

참 분기에 무엇을 두느냐로 용도가 갈린다.

| 참 분기 | 하는 일 | 예 |
|---|---|---|
| `T` | 조건에 맞는 것만 남긴다 | `Extract`, `OnlyNumber` |
| `never` | 조건에 맞는 것을 버린다 | `Exclude` |
| `T`의 일부 | 조건에 맞는 것을 변환한다 | `keyof T`, `T[]` |
| 변환만 (조건 항상 참) | 걸러내기 없이 분배만 얻는다 | `AllKeys` |

```ts
type MyExclude<T, U>   = T extends U ? never : T;              // 'a'|'b'|'c', 'a'  →  'b' | 'c'
type OnlyNumber<T>     = T extends number ? T : never;         // 'a'|42|true|7     →  42 | 7
type OnlyNumberNoDist<T> = [T] extends [number] ? T : never;   // 'a'|42|true|7     →  never
// 분배가 없으면 전부 남기거나 전부 버리는 것 둘 중 하나만 된다. 걸러내기 자체가 분배에서 나온다

// 타입 파라미터는 구조 안 어디든 놓을 수 있다 — 함수 인자와 같다
type OnlyRect<T>   = T extends { shape: 'rect' } ? T : never;  // OnlyRect<Shape> = Rect
type ByShape<T, K> = T extends { shape:    K   } ? T : never;  // 하드코딩 자리에 파라미터
ByShape<Circle | Rect | Triangle, 'rect'>                      // Rect
ByShape<Circle | Rect | Triangle, 'circle' | 'tri'>            // Circle | Triangle
ByShape<Circle | Rect | Triangle, 'nope'>                      // never
Extract<Circle | Rect | Triangle, { shape: 'rect' }>           // Rect — ByShape 는 Extract 재발명

// K 는 우변 구조 안이라 쪼개지지 않는다. 실행 횟수는 T 의 멤버 수(3회)이고 K 는 매회 통째로 비교된다
Circle   extends { shape: 'circle' | 'tri' }   // true
Rect     extends { shape: 'circle' | 'tri' }   // false
Triangle extends { shape: 'circle' | 'tri' }   // true

// 거짓 분기에 never 대신 파라미터를 두면 결과가 오염된다
type Ok   = { status: 'ok';   data: string };
type Err  = { status: 'err';  code: number };
type Wait = { status: 'wait' };
type Res  = Ok | Err | Wait;
type BadBy<T, S>  = T extends { status: S } ? T : S;
type GoodBy<T, S> = T extends { status: S } ? T : never;
BadBy<Res, 'err'>            // 'err' | Err
BadBy<Res, 'ok' | 'wait'>    // 'ok' | Ok | 'wait' | Wait
GoodBy<Res, 'err'>           // Err
Res['status']                // 'ok' | 'err' | 'wait'
```

## 용어 출처

`naked type parameter`는 공식 용어다. 조건부 타입 도입 PR([#21316](https://github.com/microsoft/TypeScript/pull/21316), Hejlsberg, 2018-02-03)과 [TS 2.8 릴리스 노트](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html)에 나온다:

> Conditional types in which the checked type is a naked type parameter are called _distributive conditional types_.

**현행 핸드북에는 이 표현이 없다.** 인용할 때 현행 핸드북 링크를 달면 독자가 단어를 못 찾는다.

`Exclude` 등 predefined conditional types는 [PR #21847](https://github.com/microsoft/TypeScript/pull/21847)(2018-02-09)에서 **분배보다 6일 늦게** 추가됐다. 따라서 "분배는 `Exclude`를 위해 도입됐다"는 성립하지 않는다. 기록으로 뒷받침되는 건 이슈 [#12215](https://github.com/microsoft/TypeScript/issues/12215)(2016-11-14, 리터럴 타입 빼기 요구)를 #21316이 `Fixes`로 명시했다는 것까지다.

## 부산물

Unit 1 내용으로 블로그 글 작성 중. Notion `블로그` DB의 "Typescript" 페이지 (`3c0a463e83038040aacfe2f445ac6510`). 사용자가 본문을 쓰고 검증·윤문을 받는 방식. `Exclude` 섹션과 실무 버그 섹션은 사용자가 잘라냄.
