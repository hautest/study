# Journal

## 답 대기 문항 — 출제했는데 답이 안 옴. 다음 세션 원문 그대로 재제시

Unit 2. 아래 세 문항을 문구 그대로 다시 낸다.

**2-1.**
```ts
type Circle = { shape: 'circle'; r: number };
type Rect   = { shape: 'rect'; w: number; h: number };
type Shape  = Circle | Rect;

type K1 = keyof Shape;                                        // K1 = ?
type K2 = keyof Circle;                                       // K2 = ?
type K3 = (Shape extends unknown ? keyof Shape : never);      // K3 = ?
```

K3은 함정 있음 — 조건부 타입 좌변을 잘 봐라.

**2-2.** 아래 함수에서 에러 나는 줄이 있나? 있으면 몇 번째 줄, 왜?
```ts
function area(s: Shape): number {
  if (s.shape === 'circle') {
    return Math.PI * s.r ** 2;        // (1)
  }
  return s.w * s.h;                    // (2)
}

function area2(s: Shape): number {
  return s.r ? Math.PI * s.r ** 2 : s.w * s.h;   // (3)
}
```

**2-3.** `Shape`에 `Triangle = { shape: 'tri'; b: number; h: number }`를 추가했다. 아래 코드에서 컴파일 에러가 나나? 나면 왜 그게 유용한가?
```ts
function name(s: Shape): string {
  switch (s.shape) {
    case 'circle': return '원';
    case 'rect':   return '사각형';
    default: {
      const _e: never = s;
      return _e;
    }
  }
}
```

채점 기준 — 2-1은 `K1 = 'shape'`(교집합), `K2 = 'shape' | 'r'`, `K3`은 좌변이 `Shape`(구체 타입, naked 아님)라 분배되지 않으므로 `'shape'`. 2-2는 (3)만 에러 — 판별 없이 `s.r`에 접근. 2-3은 `Triangle`이 `never`에 할당 불가로 에러이며, 유니온에 멤버를 추가하면 처리 누락된 switch가 전부 컴파일 에러로 드러나는 것이 안전망이다.

## 학습자 질문 — 미해소. 해소 후 반영 여부 기록

- (없음)

## 오개념

| 날짜 | 오개념 | 교정 | 반영 |
|------|--------|------|------|
| 2026-08-18 | `ToArray<string \| number>` = `(string \| number)[]` | 좌변이 naked면 멤버별로 쪼개 실행 → `string[] \| number[]`. 분배를 끄면(`[T] extends [unknown]`) 예측이 정답이 됨 | Unit 1 |
| 2026-08-18 | `Wrap<never>` = `never[]` | 분배는 멤버마다 1회 실행. `never`는 멤버 0개라 본문이 평가되지 않고 결과 0개의 합집합 = `never`. 비유: `[].map(f)`는 콜백 미호출 | Unit 1 |
| 2026-08-18 | `extends`를 if문(같은가 판정)으로 읽음 | `A extends B`는 `A ⊆ B` 질문. 대칭이 아니고, 객체는 속성 많은 쪽이 부분집합 | Unit 0 |

## 복습 큐

- `T & {}`가 분배를 차단하는 이유 (`T & unknown`은 통과) — Unit 5에서 `{}`가 `null`·`undefined`를 배제하는 타입임을 다룰 때 회수
- 함수 파라미터 자리에서 안쪽 비교 답이 뒤집히는 현상 — Unit 6 변성에서 정식으로 다룸. Unit 1에서는 "자리에 따라 다르다"까지만 언급했고 공변/반공변 용어는 쓰지 않음

## 세션 로그

### 2026-08-18

- 진단 5문항 출제. 분배 오답 1, 나머지 4개 중 3개 "모름", 변성만 답 적중(이유 미설명). Unit 7단계 사다리 설계. 사용자가 Unit 순서 선택을 위임("전체적으로").
- Unit 1(분배)로 시작했으나 "이해가 안 됨 + `extends`가 정확히 뭐야"로 **Unit 0 신설** — `extends`를 부분집합 질문으로 재정의. 4/4 통과.
- Unit 1 재개. 1-1~1-5 전부 정답. 도중 사용자 질문 "U도 분배되나?"가 규칙 경계를 정확히 찔러 우변 비분배를 반례(`DistBoth`)로 확립.
- Unit 2(유니온 `keyof`) 출제 후 답 대기. 이후 블로그 글 작업으로 전환.
- naked 용어 검증: 실재 용어 확인(PR #21316 · TS 2.8 릴리스 노트), 현행 핸드북에는 없음. "분배는 `Exclude` 때문에 존재한다"는 타임라인으로 반증됨(`Exclude`가 6일 늦음).
- 블로그 글: Notion `블로그` DB "Typescript" 페이지에 초안. 사용자가 본문 집필, 사실 검증·윤문 담당. 사실 오류 7건 교정(extends 방향 과잉일반화, 객체 속성 호환 조건 누락, 분배 규칙의 naked 조건 누락, naked의 좌변 위치 조건 누락, 함수 파라미터 자리 과잉일반화, 객체 "포함" 방향 충돌, 분배 규칙과 never 섹션 불일치).
- 다음: Unit 2 문항 2-1~2-3 채점 → Unit 3(mapped type + key remapping).
