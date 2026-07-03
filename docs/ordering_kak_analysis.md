# Hamiltonian term ordering과 KAK 융합: `even_then_odd` vs `even_then_odd_grouped`

**대상 코드:** `2_problem_jskmc.ipynb` 셀 8 (`interaction_terms_hint_even_then_odd`, `interaction_terms_hint_even_then_odd_grouped`)
**설정:** `LieTrotter(reps=1)`, `t=0.5`, `N=6`, periodic ring, basis `{cx, u3, u1}`

---

## 0. 한 줄 결론

`even_then_odd_grouped`의 depth 개선은 **물리(Trotter 오차)와 무관**하며, 전적으로 **transpiler의 2-qubit block 합성(KAK) 패스가 `optimization_level ≥ 2`에서 켜질 때, 같은 bond의 `RXX·RYY·RZZ` 세 회전을 하나의 2-qubit Weyl 게이트로 융합(6 CX → 3 CX)할 수 있도록 게이트 DAG 인접성을 노출하느냐**의 문제다.
`grouped`는 이 인접성을 노출하고, `even_then_odd`는 차단한다. **따라서 KAK 융합은 `grouped`에 대해서만 유효하다.**

---

## 1. 측정 결과

`LieTrotter(reps=1)`, `t=0.5`, `N=6`, basis `{cx, u3, u1}` 기준 (`depth / cx / 2Q-depth`):

| ordering | opt0 | opt1 | opt2 | opt3 |
|---|---|---|---|---|
| **even_then_odd** | 27 / 36 / 12 | 23 / 36 / 12 | 22 / **36** / 12 | 22 / **36** / 12 |
| **even_then_odd_grouped** | 27 / 36 / 12 | 25 / 36 / 12 | 13 / **18** / 6 | 13 / **18** / 6 |

- `opt0/1`: 두 ordering이 **36 CX로 동일**.
- `opt2`에서만 `grouped`가 **18 CX**로 갈라지고, `even_then_odd`는 `opt3`까지 **36 CX**에 머문다.

---

## 2. 두 함수는 같은 H, 다른 *게이트 나열 순서*를 만든다

셀 8의 두 builder는 루프 중첩 순서만 다르다.

```text
even_then_odd          : for P in [XX,YY,ZZ]:  for layer in [even,odd]:  for bond   # Pauli-major
even_then_odd_grouped  : for layer in [even,odd]:  for P in [XX,YY,ZZ]:  for bond   # layer-major
```

H 행렬은 동일하다 (`same Hamiltonian matrix = True`). `LieTrotter(reps=1)`는 연산자 리스트 순서대로
$e^{-i\theta P}$를 한 번씩 쌓을 뿐이므로, 차이는 **회로의 게이트 append 순서**에만 나타난다.

N=6 ring에서 even bonds = (0,1),(2,3),(4,5), odd bonds = (1,2),(3,4),(5,0). 각 layer는 내부적으로 서로소(disjoint).
interaction 항만 보면:

```text
even_then_odd        : XX(0,1) XX(2,3) XX(4,5) XX(1,2) XX(3,4) XX(5,0) | YY... | ZZ...
even_then_odd_grouped: XX(0,1) XX(2,3) XX(4,5) YY(0,1) YY(2,3) YY(4,5) ZZ(0,1) ZZ(2,3) ZZ(4,5) | [odd layer]...
```

---

## 3. 개선의 정체 — opt≥2에서만 켜지는 2-qubit KAK 재합성

Qiskit preset transpile 파이프라인의 opt level별 동작:

| opt level | 2-qubit block 재합성 | 비고 |
|---|---|---|
| 0 | ✗ | 각 회전을 2-CX 템플릿으로 분해만 |
| 1 | ✗ | 1-qubit 융합, 인접 역게이트 상쇄 정도 (가벼운 최적화) |
| 2, 3 | ✓ | `Collect2qBlocks` → `ConsolidateBlocks` → `UnitarySynthesis` (KAK) |

`opt2/3`에서 비로소 **같은 큐빗 쌍에 연속으로 작용하는 2-qubit 게이트들을 하나의 행렬로 합쳐 KAK(Cartan)로
재합성**한다. 따라서 "opt level에 따라 depth가 개선될 수 있다"는 것은 이 패스가 켜지는 `opt2` 이상을 가리킨다.

---

## 4. 대수적 근거 — 같은 bond의 3 회전은 한 개의 Weyl 게이트 (6 CX → 3 CX)

한 큐빗 쌍 $(a,b)$ 위에서 $X_aX_b,\ Y_aY_b,\ Z_aZ_b$ 는 **서로 모두 commute**한다
(`[XX,YY] = [YY,ZZ] = [XX,ZZ] = 0`, 직접 확인됨). 따라서

$$
R_{XX}(\alpha)\,R_{YY}(\beta)\,R_{ZZ}(\gamma)
= \exp\!\Big[-\tfrac{i}{2}\big(\alpha\,X_aX_b+\beta\,Y_aY_b+\gamma\,Z_aZ_b\big)\Big]
$$

는 **단일 canonical(Weyl-chamber) 2-qubit 게이트**다. Cartan/KAK 분해에 의해 임의의 2-qubit 유니터리는
**최대 3 CX**로, 일반적 Weyl 게이트는 **정확히 3 CX**로 구현된다. 반면 세 회전을 **따로** 합성하면 각 2 CX씩 = **6 CX**.

직접 실증:

```text
RXX·RYY·RZZ on same pair :  opt0 CX = 6  →  opt3 CX = 3
```

bond당 6 → 3 CX. 6 bond × 3 = **18 CX** (grouped), 6 bond × 6 = **36 CX** (융합 실패 시).

---

## 5. 왜 KAK가 `grouped`에만 유효한가 — 게이트 DAG 인접성

`Collect2qBlocks`가 세 회전을 한 블록으로 모으려면, $R_{XX}(a,b),R_{YY}(a,b),R_{ZZ}(a,b)$ 사이에
**큐빗 $a$ 또는 $b$를 건드리는 다른 게이트가 끼면 안 된다** (게이트 DAG에서 인접해야 함).
이것이 두 ordering의 운명을 가른다.

### 5.1 `even_then_odd_grouped` — 인접성 노출 (KAK 유효)

even layer에서 bond (0,1)을 보면:

```text
XX(0,1)  [XX(2,3) XX(4,5)]  YY(0,1)  [YY(2,3) YY(4,5)]  ZZ(0,1) ...
```

사이에 끼는 (2,3),(4,5)는 **{0,1}과 서로소**. DAG에서 다른 큐빗이라 병렬이고 (0,1) 경로를 **차단하지 않음**.
→ XX(0,1), YY(0,1), ZZ(0,1)가 (0,1) 쌍 위에서 **연속** → 한 Weyl 블록으로 융합 → **3 CX**.
6 bond 모두 동일 → **18 CX**, 2Q-depth = 2 layer × 1 = **6**.

### 5.2 `even_then_odd` — 인접성 차단 (KAK 무효)

bond (0,1)에서 XX(0,1)과 YY(0,1) 사이에 **XX block 전체**가 끼는데, 그 안에
**XX(1,2)(큐빗 1 공유)와 XX(5,0)(큐빗 0 공유)**가 있다. 이들은 {0,1}과 겹치므로 DAG에서 (0,1) 경로를 **차단**한다.

→ (0,1) 위에서 모이는 최대 블록은 RXX(0,1) **단 하나**
(다음 큐빗 0 게이트는 다른 쌍 (5,0), 큐빗 1은 (1,2)). 융합 불가 → 각 회전이 2 CX로 잔존 → **36 CX**.

---

## 6. 가장 중요한 미묘점 — transpiler가 스스로 못 고치는 이유

물리적으로는 $R_{XX}(0,1)$과 $R_{XX}(1,2)$가 사실 commute한다
(공유 큐빗에서 $X_1X_1=I$이므로 $X_0X_1\cdot X_1X_2 = X_0X_2 = X_1X_2\cdot X_0X_1$).
**그럼에도** 기본 transpile 파이프라인은 이를 활용해 재정렬하지 **않는다**.

- `Collect2qBlocks`는 **게이트 DAG 위상 기반의 greedy 수집**이지 Pauli 대수 commutation을 추론하지 않는다.
- DAG 상 두 게이트는 큐빗 1을 공유하므로 의존 간선이 생기고, 그 사이로 RYY(0,1)을 끌어올 수 없다.

→ **융합 가능성은 ordering이 미리 "노출"해 줘야 하는 성질**이며, transpiler가 사후에 복구해 주지 못한다.
이것이 `grouped`는 opt2에서 18 CX로 떨어지고 `even_then_odd`는 opt3까지도 36 CX에 머무는 근본 이유다.
(commutation-aware 라우팅 패스가 별도로 존재하지만 basis-gate transpile 기본 preset에는 포함되지 않는다.)

---

## 7. 팀원 코드(`solution (1).ipynb`)와의 naming 관계

- 팀원의 `even_odd`는 **bond-major** (`for bond: for op in XX,YY,ZZ`)라 같은 bond의 3 회전이 *애초에 인접* → 융합됨.
  그래서 팀원의 `even_odd`와 `even_odd_grouped`가 **18 CX로 동일**한 것이 맞다.
- 그러나 본인 노트북의 `even_then_odd`는 **Pauli-major across both parities** (`for op: for layer: for bond`)라
  overlapping odd bond가 gap에 끼어 융합이 깨진다 → 본인 코드에서는 두 ordering이 **동일하지 않다 (36 vs 18)**.
- 즉 **"even_odd ≡ grouped"는 팀원 정의에서만 참**이며, 본인의 `even_then_odd`에 그대로 옮기면 안 된다.

| ordering | 정의 | 같은 bond 3회전 인접? | opt2 CX |
|---|---|---|---|
| 본인 `even_then_odd` | Pauli-major (전 parity 걸쳐) | ✗ (overlapping odd bond가 차단) | 36 |
| 본인 `even_then_odd_grouped` | layer-major | ✓ (disjoint bond만 사이에 낌) | 18 |
| 팀원 `even_odd` | layer 내 bond-major | ✓ (회전이 literally 인접) | 18 |
| 팀원 `even_odd_grouped` | layer 내 op-major | ✓ | 18 |

---

## 8. 요약

1. 두 ordering은 같은 H, 다른 게이트 순서를 만든다.
2. depth/CX 개선은 Trotter 물리가 아니라 **opt≥2의 2-qubit KAK 재합성**이 만든다.
3. 한 bond의 XX/YY/ZZ는 commute → 하나의 Weyl 게이트 → KAK로 6 CX→3 CX.
4. 그 융합은 세 회전이 **DAG에서 인접**할 때만 가능하다.
5. `grouped`는 사이에 disjoint bond만 끼어 인접성을 노출 → 융합 → 18 CX.
   `even_then_odd`는 overlapping odd bond가 끼어 차단 → 융합 실패 → 36 CX.
6. transpiler는 DAG-greedy라 Pauli commutation으로 스스로 복구하지 못하므로,
   **KAK 융합은 ordering이 노출한 `grouped`에서만 유효**하다.
