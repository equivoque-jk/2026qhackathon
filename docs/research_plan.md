# 연구계획서 v2 - Trotterized Quantum Simulation

**대상 노트북:** `2_problem_jskmc.ipynb`  
**골격 출처:** `2_problem_guide.ipynb` 우선, `1_problem_set.pdf` 보조  
**범위:** Problem 1, Problem 2, Bonus Problem 3까지 포괄한다. 단, 실제 제출 우선순위는 P0, P1, P2로 분리한다.  
**작성 목적:** 모든 synthesis 선택, 그래프 축, QPU 실행 가능성 판단을 사후 감각이 아니라 명시된 metric, hard constraint, sanity gate로 고정한다.

> 이 문서는 실행 계획이다. 표와 그림은 아래 sanity gate와 plot-scale audit을 통과한 뒤에만 확정 산출물로 인정한다.

---

## 0. 회의 결정 반영

### 0.1 제거할 주장

**ordering 변형 ablation을 "이상적 QPU에서 이론적 Trotterization error를 개선하는 창의성 포인트"로 사용하지 않는다.**

근거는 냉정하게 다음과 같다.

1. `hint_even_then_odd_grouped`의 resource 이득은 `ordering_kak_analysis.md`가 보인 것처럼, 같은 bond의 `RXX`, `RYY`, `RZZ`가 transpiler의 2-qubit KAK block으로 보이는지의 문제다.
2. 이 이득은 opt level 2 이상에서 나타나는 컴파일 이득이며, 이상적 논리 QPU의 Trotter splitting error 개선 근거로 일반화하기 어렵다.
3. ordering별 error 곡선이 일부 차이를 보이더라도, 그것을 "이론적 개선"으로 주장하려면 commutator norm 또는 rigorous product-formula error bound가 필요하다. 현재 문제 범위에서는 설득 비용이 점수 기대값보다 크다.

### 0.2 남길 내용

ordering은 완전히 삭제하지 않는다. Guide 1(a)가 ordering 설명을 요구하므로 다음 수준으로만 남긴다.

- **P0 필수 설명:** 세 hint baseline ordering은 같은 Hamiltonian을 만든다.
- **P0 컴파일 해석:** grouped ordering은 KAK 융합 가능성을 노출해 depth, CX, 2q-depth를 줄인다.
- **금지 문장:** "grouped ordering이 이상적 QPU에서 Trotter error를 본질적으로 낮춘다."
- **허용 문장:** "ordering은 본 제출에서 주로 compiled-resource metric을 개선하는 회로 배치/합성 노출 장치로 취급한다."

### 0.3 새 창의성 축

창의성 점수는 ordering ablation이 아니라 다음 세 축으로 이동한다.

1. **deployable synthesis 선택:** Trotterization error와 noisy-QPU error가 서로 반대 방향으로 움직인다는 사실을 명시적으로 모델링하고, hard constraint와 calibrated 2Q error burden 안에서 실제 QPU에 올리기 가장 적합한 synthesis option을 찾는다.
2. **QPU hard-constraint 기반 scalability:** `N=4+8r_H`, `r_H=1,2,3`에서 어떤 formula/order/reps가 `depth<100`, `2q gate<400`, `2q-depth<30`을 만족하는지 경계선을 찾는다.
3. **spectral information preservation:** `A_PF(t)`의 Fourier spectrum이 exact spectrum의 eigenenergy peak를 얼마나 보존하는지 정량화한다.
4. **symmetry leakage diagnostic:** N=12 single-excitation 문제에서 ideal dynamics는 number conserving이므로, product formula가 `⟨Nhat⟩=1` 또는 single-excitation subspace를 얼마나 벗어나는지 별도 error metric으로 기록한다.

---

## 1. 우선순위 구조

| 단계 | 목표 | 포함 범위 | 제출 역할 |
|---|---|---|---|
| **P0** | 문제 필수 요구사항을 엄밀하게 완료 | Problem 1(a-h), Problem 2(a-e), plot-scale audit, sanity gate | 본문 핵심 |
| **P1** | 창의성 및 연구성 확보 | calibrated 2Q error burden 기반 QPU 후보 선택, QPU runnability boundary, 최소 spectral distortion 진단 | 본문 일부 + appendix |
| **P2** | bonus engineering | Bonus Problem 3, backend/fake-backend opt-level comparison, Rustiq 가능성 확인 | 시간 허용 시 appendix |

### 1.1 본문-appendix 경계

과설계 리스크를 막기 위해 제출 본문과 appendix의 역할을 분리한다.

| 위치 | 반드시 포함 | 제외 또는 축약 |
|---|---|---|
| 본문 | 필수 문제 답, QPU 후보 1개 선정 근거, hard constraint pass/fail, calibrated 2Q error burden Pareto, Problem 2 exact/PF spectrum 최소 비교 | `r_H=2,3` 상세 plot, 모든 noise-model sensitivity plot, symmetry leakage 상세 유도 |
| Appendix | `r_H=1,2,3` scalability table, sensitivity sweep, symmetry leakage diagnostic, optional Rustiq/Bonus | 본문 결론을 바꾸지 않는 반복 plot |

본문의 핵심 메시지는 하나로 제한한다.

> "가장 깊고 이상적으로 정확한 회로"가 아니라, ideal Trotter error와 calibrated 2Q hardware burden 사이에서 Pareto-robust한 synthesis option을 선택한다.

### 1.2 Guide-notebook 구현 계약

최종 구현 산출물은 `2_problem_guide.ipynb`의 outline을 기준으로 만든다. 현재 guide는 29개 원본 cell로 구성되어 있으며, Setup, Problem 1(a), exact reference, Problem 1(b-c), Problem 1(d-f), Problem 1(g), challenge, Problem 1(h), Problem 2(a-e), Bonus Problem 3 순서를 가진다.

최종 notebook 작성 규칙:

1. `2_problem_guide.ipynb`의 problem-statement markdown cell 문구와 문제 순서는 보존한다.
2. 미완성 TODO code cell은 제자리에서 채워 구현한다. 빈 `if` 본문, dict 콤마 누락, placeholder return 등 skeleton의 문법/실행 오류는 해당 cell 안에서 수정할 수 있다.
3. guide cell을 채우는 것만으로 흐름이 복잡해지면 helper, plot, validation, discussion cell을 추가할 수 있다.
4. 추가 cell은 대응되는 guide 문제/TODO cell 직후 또는 같은 문제 section 안에 둔다.
5. 원본 problem statement markdown을 고치지 않는다. 단, 별도 `[Solution note]` markdown cell을 추가해 해석과 토의를 작성할 수 있다.
6. byte-level hash subsequence 보존 같은 과한 제약은 두지 않는다. 핵심은 guide의 outline을 따르며 TODO를 완성하고, 필요 시 추가 cell로 보강하는 것이다.

검증 기준:

| 항목 | 통과 조건 |
|---|---|
| guide outline preservation | guide의 문제 순서와 section 구조를 유지 |
| problem markdown preservation | 원본 problem-statement markdown 문구는 수정하지 않음 |
| TODO completion | guide의 TODO code cell은 제자리에서 실행 가능하게 완성 |
| added-cell locality | 추가 cell은 대응되는 guide 문제/TODO cell 직후 또는 해당 문제 section 내부에 위치 |
| reproducibility | Restart & Run All로 표와 그림이 재생성됨 |
| discussion separation | 서술형 답변은 원본 prompt를 고치지 않고 별도 `[Solution note]` cell 또는 지정된 discussion cell에 작성 |

### 1.3 Qiskit-native 구현 원칙

최종 구현은 가능한 한 Qiskit의 public library API와 내장 함수를 우선 사용한다. 직접 구현은 Qiskit에 안정적인 public API가 없거나, 독립 검증용으로 필요한 경우에만 허용한다.

우선 사용할 API 계층:

| 작업 | 우선 사용 | 직접 구현 허용 범위 |
|---|---|---|
| Hamiltonian 표현 | `SparsePauliOp`, `PauliList`, 필요 시 `Operator` | term 생성 검증용 보조 코드 |
| product formula 회로 | `PauliEvolutionGate`, `LieTrotter`, `SuzukiTrotter`, guide가 제공한 `ProductFormula` 구조 | Qiskit synthesis가 요구사항을 충족하지 못할 때의 wrapper |
| state evolution | `Statevector.from_label`, `Statevector.evolve`, Qiskit operator conversion | exact reference 교차검증용 NumPy/SciPy |
| transpilation | `transpile`, `generate_preset_pass_manager`, backend `Target`/`coupling_map` | pass 결과 요약 및 sanity check |
| resource count | `QuantumCircuit.depth`, `QuantumCircuit.count_ops`, `QuantumCircuit.size`, `depth(filter_function=...)` | Qiskit이 직접 제공하지 않는 custom metric, 예: edge별 native 2Q count |
| backend 정보 | backend `target`, `properties`, instruction properties, Qiskit Runtime primitives | backend 접근 불가 시 fake backend 또는 명시된 hardware-risk proxy |
| observable/expectation | Qiskit `Statevector.expectation_value`, `SparsePauliOp` 기반 계산 | 축약 표기와 plotting용 후처리 |

구현 제한:

1. Pauli string, bit ordering, tensor product, gate decomposition을 손으로 다시 만들 때는 반드시 Qiskit 결과와 cross-check한다.
2. Qiskit public API로 가능한 resource count를 별도 handwritten parser로 대체하지 않는다.
3. private/internal Qiskit API에는 의존하지 않는다.
4. NumPy/SciPy는 exact diagonalization, FFT, 통계/plot 후처리처럼 Qiskit의 핵심 회로/시뮬레이션 API 바깥 작업에 사용한다.
5. Qiskit API와 custom helper가 같은 값을 낼 수 있는 경우, 산출물에는 Qiskit API 결과를 primary로 보고하고 custom helper는 sanity check로만 둔다.

---

## 2. 문제의 under-specification과 고정할 선택

| 모호점 | 고정 선택 | 정당화 |
|---|---|---|
| "best synthesis" 기준 | Pareto front를 1차 기준, scalar score를 보조 기준 | accuracy와 resource는 단일 스칼라로 환원하면 trade-off를 왜곡한다. |
| 구현 방식 | Qiskit public API와 내장 함수 우선 | 채점 환경 호환성, 재현성, basis/ordering 오류 방지를 위해 hand-rolled 구현을 최소화한다. |
| error 집계 구간 | 단시간 `t<=2.0` 평균/최대, 전체 시간은 정성 해석 | coherent error는 장시간에 진동 및 교차하므로 전역 순위가 불안정하다. |
| resource 비교 opt level | opt=3 compiled resource | QPU 실행 비용에 가장 가까운 수치다. 단, logical error와 분리한다. |
| prompt-basis vs backend-native 2Q count | 문제 1(c)의 CX count와 QPU 선택용 `native_2q_gate_count`를 분리 | prompt basis resource와 Heron-native resource를 같은 물리량처럼 비교하면 결론이 오염된다. |
| 대표 ordering | `hint_even_then_odd_grouped` | KAK 융합으로 resource를 줄이는 baseline이다. Trotter error 개선 주장에는 쓰지 않는다. |
| QPU 실행 가능성 | `depth<100`, `2q gate<400`, `2q-depth<30` 모두 만족 | guide Hint 3의 hard constraint다. 셋 중 하나라도 실패하면 runnability 실패로 판정한다. |
| Heron noise prior | 별첨 문서의 2q error, T1/T2, crosstalk 설명은 hardware prior로만 사용 | 실제 실행일 backend calibration이 최종 근거다. 문서 수치는 threshold 정당화의 보조 설명이다. |
| calibrated 2Q burden | `Σ_e n_e p_e`와 `Π_e(1-p_e)^{n_e}`를 QPU Pareto metric으로 추가 | 같은 2Q gate count라도 어떤 backend edge를 쓰는지에 따라 hardware risk가 다르다. |
| backend calibration 부재 | edge별 `p_e`가 없으면 `calibrated`가 아니라 `hardware-risk proxy`로 명명 | 실제 calibration 없이 calibrated fidelity를 주장하면 방법론 과장이 된다. |
| Pareto 차원 수 | 본문은 2D/3D 압축 Pareto와 후보표, 4축 Pareto는 내부 검증 또는 appendix | 차원이 높을수록 연구적으로는 풍부하지만 본문 결론 전달력은 급격히 떨어진다. |
| 그래프 축과 grid | 모든 plot에 scale decision을 기록 | grid 한 칸이 어떤 수치 변화 또는 배율 변화를 뜻하는지 명시해야 시각적 해석이 과장되지 않는다. |
| noisy-QPU balance | 단일 수식이 아니라 hard constraint, 보수적 upper bound, multiplicative attenuation, empirical backend evidence를 함께 사용 | 별첨 PDF도 단순 덧셈의 한계를 지적한다. 하나의 formula로 synthesis를 결정하면 calibration과 noise-model misspecification에 취약하다. |

---

## 3. 전역 Plot-Scale Audit

모든 figure는 생성 직후 `plot_scale_audit` 표를 남긴다.

필수 컬럼:

| 컬럼 | 의미 |
|---|---|
| `figure_id` | figure 이름 |
| `x_variable`, `y_variable` | 축 변수와 단위 |
| `x_transform`, `y_transform` | `linear`, `log10`, `symlog`, `categorical` 중 하나 |
| `major_tick_rule` | major grid 간격 결정 규칙 |
| `minor_tick_rule` | minor grid 사용 여부와 이유 |
| `zero_handling` | log 축에서 0 또는 음수 처리 방식 |
| `dynamic_range` | max/min 또는 max-min |
| `scale_rationale` | 왜 그 스케일이 타당한지 |

### 3.1 축 선택 원칙

1. **time axis:** 원래 time grid가 `Δt=0.05`이므로 선형 축을 사용한다. major tick은 전체 구간에서는 1.0 또는 2.0, 단시간 확대에서는 0.25 또는 0.5처럼 grid의 정수배만 허용한다.
2. **error, infidelity, spectral error:** 항상 nonnegative이고 여러 order of magnitude를 가질 수 있으므로 기본은 `log10`이다. 0은 `floor=max(1e-16, min_positive/10)`으로 클리핑하고, `t=0`의 정확한 0은 별도 marker 또는 표로 보고한다.
3. **expectation value:** `⟨Z⟩`처럼 bounded signed quantity는 선형 축을 쓴다. `[-1,1]` 범위에서 major tick은 0.25 또는 0.5로 고정한다.
4. **resource axis:** depth, CX, 2q-depth는 양의 정수다. 한 figure 안에서 max/min이 5배 이상이면 `log10`, 그보다 작고 hard constraint 근처를 보는 경우에는 선형 축을 쓴다.
5. **categorical/discrete axis:** reps, order, optimization_level, `r_H`는 categorical 또는 integer tick만 쓴다. 임의 보간선을 그리지 않는다.
6. **energy spectrum axis:** energy는 선형 축을 쓴다. FFT grid resolution `ΔE=2π/Tmax`를 계산하고, peak matching tolerance와 major tick 간격을 `ΔE`의 정수배로 둔다.

### 3.1.1 Transform 정당성 gate

log, symlog, normalization, clipping 같은 plot 변형은 다음 질문을 통과할 때만 사용한다.

1. 변수가 양수 또는 nonnegative인가. 0이 있으면 floor 처리의 수치와 이유가 명시되어 있는가.
2. 변형 후 grid 한 칸이 해석 가능한 의미를 갖는가. 예: `log10` 한 칸은 10배 변화다.
3. 변형이 결론을 바꾸는가. 바뀐다면 원 스케일 plot 또는 sensitivity table을 함께 제시한다.
4. 변형이 outlier를 숨기는가. 숨긴다면 inset, annotation, 또는 raw-value table을 둔다.
5. 축 변환이 물리 단위가 다른 두 축을 부당하게 같은 척도처럼 보이게 만들지 않는가.

따라서 "보기 좋아서 log를 취한다"는 허용하지 않는다. log는 multiplicative scaling, power-law slope, 또는 multi-order dynamic range가 연구 질문일 때만 정당하다.

### 3.2 accuracy-resource plot의 엄밀화

Problem 1(h) 및 HTML report의 accuracy-resource trade-off plot은 다음 규칙을 적용한다.

1. y축 error는 기본적으로 `log10(mean_state_infidelity)`를 사용한다.
2. x축 resource는 dynamic range가 5배 이상이면 `log10(resource)`를 사용한다. 그렇지 않으면 hard constraint와의 거리 해석을 위해 선형 축을 유지한다.
3. log-log plot에서 한 grid 칸은 "동일한 배율 변화"를 뜻한다. 예를 들어 x축 한 major tick이 depth 10배, y축 한 major tick이 infidelity 10배라면 기울기는 resource 증가율 대비 error 감소율의 elasticity로 읽는다.
4. 사용자가 지적한 "infidelity 간격과 depth 간격이 서로 비슷한 error 변화 의미를 가져야 한다"는 요구는 물리 단위가 다른 두 축을 억지로 같은 절대 간격으로 맞추는 방식으로 처리하지 않는다. 대신 `log10(error)`와 `log10(resource)`를 사용해 양쪽 모두 multiplicative change로 통일하고, Pareto 후보에 대해 `Δlog10(error)/Δlog10(resource)`를 함께 보고한다.
5. 보조 plot으로 normalized sensitivity plot을 추가할 수 있다. 이때 `x_norm=(log10 resource-min)/(max-min)`, `y_norm=(log10 error-min)/(max-min)`를 사용하고, 원래 tick 값은 annotation으로 병기한다. 이 plot은 시각 비교용이며 1차 결론은 원 단위 log plot에서 낸다.

### 3.3 다른 plot으로의 확장

| Plot | 기본 scale | grid 정당화 |
|---|---|---|
| 1(c) resource vs setting | resource dynamic range에 따라 linear 또는 log10 | gate count가 배율로 증가하면 log, hard constraint 근처면 linear |
| 1(d-f) error vs time | x linear, y log10 | time은 실험 grid, error는 order scaling 검증 대상 |
| 1(f) short-time scaling | log-log | 이론 기울기 LT 4, ST2 6, ST4 10 검증 |
| 1(h) accuracy-resource | log y, x는 rule 기반 | Pareto 및 elasticity 해석 |
| 1(g) QPU budget plot | resource linear, constraint lines 표시 | hard cutoff와 margin을 직접 읽어야 함 |
| 1(g) 2Q error burden Pareto | x: ideal error log, y: calibrated burden linear 또는 log | burden의 range가 5배 이상이면 log, 아니면 raw expected error scale 유지 |
| 2(a) A(t) | x linear, y linear | real, imag, abs의 bounded oscillation |
| 2(c) autocorrelation error | x linear, y log10 | error growth와 saturation 분리 |
| 2(d) spectrum | energy linear, magnitude linear 또는 log magnitude | peak location은 선형 energy, 작은 leakage peak는 log magnitude 보조 |
| Bonus 3 opt-level sweep | x categorical, y resource rule 기반 | opt level은 순서형 범주이므로 보간 금지 |

---

## 4. Problem 1(a-c): Hamiltonian, ordering, circuits, resource

### 4.1 Hamiltonian construction

- N=6, `Jx=1.0`, `Jy=0.7`, `Jz=1.2`, `hx=0.4`.
- Pauli term 수: interaction `3N=18`, field `N=6`, total `4N=24`.
- periodic boundary term: `(5,0)`에서 `X_5X_0`, `Y_5Y_0`, `Z_5Z_0`.
- Qiskit label convention을 명시한다. 예: `X_5X_0`은 `XIIIIX`.

### 4.2 Ordering 처리

필수 baseline ordering 3개만 유지한다.

| ordering | 연구 내 역할 |
|---|---|
| `hint_naive` | hint baseline |
| `hint_even_then_odd` | even/odd layer baseline |
| `hint_even_then_odd_grouped` | default compiled-resource baseline |

보고 문장:

- 세 ordering은 동일한 `SparsePauliOp` matrix를 만든다.
- 차이는 product-formula gate append order에만 있다.
- grouped의 이득은 KAK block exposure에 의한 compiled-resource 이득이다.
- ordering을 P1 error-improvement ablation으로 확장하지 않는다.

### 4.3 Product-formula circuits

필수 synthesis set:

| family | Qiskit synthesis | reps |
|---|---|---|
| LT | `LieTrotter(reps=k)` | `k=1,2,3` |
| ST2 | `SuzukiTrotter(order=2, reps=k)` | `k=1,2,3` |
| ST4 | `SuzukiTrotter(order=4, reps=k)` | `k=1,2,3` |

모든 회로는 `PauliEvolutionGate` 경로로 생성한다. exact exponential을 회로 경로에 섞지 않는다.

### 4.4 Resource table

기본 resource table은 `t=0.5`, basis `{cx,u3,u1}`, opt=3로 산출한다.

필수 컬럼:

- `ordering`
- `setting`
- `formula`
- `order`
- `reps`
- `depth`
- `total_gates`
- `cx_count`
- `one_qubit_gates`
- `two_qubit_gates`
- `two_qubit_depth`

resource는 회전 각도에 의존하지 않는 구조적 수치이므로 `t` 불변성을 sanity check로 확인한다.

### 4.5 Optimization-level sweep

`optimization_level` sweep은 컴파일 ablation이다.

- 바뀌는 것: `depth`, `total_gates`, `cx_count`, `two_qubit_depth`.
- 바뀌지 않는 것: logical product-formula unitary, ideal statevector error.
- 증명: opt 0,1,2,3 transpiled circuit의 statevector가 opt0와 `<=1e-9` 일치해야 한다.

---

## 5. Problem 1(d-h): accuracy, trade-off, discussion

### 5.1 Error metrics

필수 metric:

$$
\epsilon_{\mathrm{state}}(t)=1-|\langle\psi_{\mathrm{exact}}(t)|\psi_{\mathrm{PF}}(t)\rangle|^2
$$

$$
\epsilon_O(t)=|\langle O\rangle_{\mathrm{exact}}(t)-\langle O\rangle_{\mathrm{PF}}(t)|
$$

where `O in {Z_1, Z_0Z_1}`.

### 5.2 Time-window summary

표준 window:

- `t<=0.5`: very-short-time, scaling 확인용
- `t<=1.0`: QPU time `t=1.0`과 직접 연결
- `t<=2.0`: primary ranking window
- full `t<=15.0`: qualitative dynamics only

각 window마다 mean, max, final error를 모두 기록하되, synthesis ranking은 primary window `t<=2.0` 기준으로만 낸다.

### 5.3 Short-time scaling 검증

state infidelity의 예상 기울기:

| formula | expected slope in `log epsilon` vs `log t` |
|---|---:|
| LT | 4 |
| ST2 | 6 |
| ST4 | 10 |

reps scaling의 예상 기울기:

| formula | expected slope in `log epsilon` vs `log reps` |
|---|---:|
| LT | -2 |
| ST2 | -4 |
| ST4 | -8 |

실측 slope는 linear regression으로 추정하고, 사용한 time window와 floor value를 표에 기록한다.

### 5.4 Accuracy-resource trade-off

Primary plot:

- x: opt=3 compiled `depth` 또는 `two_qubit_depth`
- y: `mean_state_infidelity` over `t<=2.0`
- scale: §3.2 규칙에 따라 log y, x는 range 기반 결정

결론 방식:

1. same reps에서는 formula order가 높을수록 accuracy가 좋아지는지 본다.
2. similar depth에서는 Pareto front가 어떤 synthesis를 선택하는지 본다.
3. scalar score는 보조로만 사용한다.

보조 scalar:

$$
S_\lambda = \log_{10}(\overline{\epsilon}_{\mathrm{state}}+\epsilon_{\mathrm{floor}})
+\lambda\log_{10}(R/R_{\min})
$$

where `R` is depth, CX, or 2q-depth. `λ`는 conclusion을 바꾸는지 sensitivity sweep한다.

### 5.5 Ideal accuracy와 deployable accuracy 분리

깊은 회로는 일반적으로 Trotterization error를 줄이지만, noisy QPU에서는 gate error와 decoherence가 커진다. 따라서 synthesis option은 두 층위에서 따로 평가한다.

| 층위 | 질문 | metric |
|---|---|---|
| ideal-PF | 이상적 statevector에서 더 정확한가 | `mean_state_infidelity`, `epsilon_O`, `epsilon_A` |
| deployable-QPU | Heron-like noise와 hard constraint를 고려해 실제 QPU에 올릴 가치가 있는가 | hard pass/fail, hardware fidelity prior, attenuated observable error, empirical backend result |

최종 QPU 후보는 ideal-PF Pareto 후보 중에서만 고르지 않는다. 오히려 다음 조건을 모두 만족해야 한다.

1. hard constraints를 통과한다.
2. ideal error가 Pareto-dominated가 아니다.
3. calibrated 2Q error burden과 hardware fidelity prior를 곱했을 때 deployable error가 최소권이다.
4. calibration sensitivity에서 후보가 쉽게 바뀌지 않는다.

---

## 6. Problem 1(g): IBM Heron backend and runnability boundary

### 6.1 기본 실행 문제

Guide 기준:

- `N=12`
- `Jx=Jy=1.0`, `Jz=1.2`, `hx=0`
- initial state: `|1000 0000 0000>`
- observable: `Z_0`
- time: `t=1.0`
- Runtime mitigation: `resilience_level=2`

Exact baseline은 `utils.single_one_evolution_meas_Z0`를 사용한다. 이유는 `Jx=Jy`, `hx=0`에서 number conservation이 성립하고, single-excitation subspace dimension이 12로 줄어들기 때문이다.

### 6.2 Hard constraints

QPU 실행 가능성은 다음 세 조건의 conjunction이다.

| constraint | threshold | 판정 |
|---|---:|---|
| compiled depth | `<100` | fail if `>=100` |
| two-qubit gate count | `<400` | fail if `>=400` |
| two-qubit depth | `<30` | fail if `>=30` |

pass margin도 함께 기록한다.

$$
m_{\mathrm{depth}}=100-\mathrm{depth}
$$

$$
m_{\mathrm{2qgate}}=400-\mathrm{2qgate}
$$

$$
m_{\mathrm{2qdepth}}=30-\mathrm{2qdepth}
$$

최소 margin `min_margin`이 작을수록 calibration 변동에 취약한 설정으로 간주한다.

### 6.3 Noisy-QPU fidelity model

별첨 PDF의 핵심 관계식은 다음 두 가지다.

1. 2-qubit gate count 중심 생존 확률:

$$
\mathcal{F}_{2q}(g)\approx(1-e_{2q})^g
$$

여기서 `g`는 2-qubit gate count, `e_2q`는 backend 2-qubit gate error prior다.

backend calibration을 사용할 수 있으면 평균 `e_2q` 대신 edge별 error를 사용한다.

$$
\mathcal{F}_{2q,\mathrm{edge}}
\approx \prod_e (1-p_e)^{n_e}
$$

$$
B_{2q}
=1-\mathcal{F}_{2q,\mathrm{edge}}
$$

작은 error에서는 다음 1차 근사가 유효하다.

$$
B_{2q}
\approx \sum_e n_e p_e
$$

여기서 `e`는 transpiled circuit이 실제로 사용한 backend coupling edge, `p_e`는 해당 edge의 calibrated 2Q gate error, `n_e`는 그 edge에서 사용된 native 2Q gate 횟수다. 이 값은 raw 2Q gate count보다 QPU deployability에 더 가깝다.

2. observable attenuation model:

$$
\langle O\rangle_{\mathrm{actual}}\approx
\lambda(d)\langle O\rangle_{\mathrm{PF}}
$$

$$
\lambda(d)=\exp(-\gamma d)
$$

여기서 `d`는 compiled depth 또는 2q-depth다. 더 일반적으로는 다음처럼 쓴다.

$$
\lambda_{\mathrm{hw}}
=\exp(-\gamma_d d-\gamma_{2q}g-\gamma_{1q}g_{1q})
$$

그리고 deployable observable error를 다음처럼 정의한다.

$$
\epsilon_{O,\mathrm{deploy}}
=|\langle O\rangle_{\mathrm{exact}}
-\lambda_{\mathrm{hw}}\langle O\rangle_{\mathrm{PF}}|
$$

이 metric은 Trotter error 감소와 hardware signal attenuation 증가 사이의 sweet spot을 찾기 위한 보조 목적함수다.

### 6.4 모델 사용 방식: 단일 수식 매몰 방지

PDF의 관계식은 유용하지만, 하나의 수식만으로 synthesis option을 결정하지 않는다. 이유는 다음과 같다.

1. `e_2q`, `gamma_d`, `gamma_2q`는 backend, qubit subset, calibration time, routing에 따라 바뀐다.
2. 실제 noise는 순수 depolarizing attenuation만이 아니라 coherent error, readout error, crosstalk residual, leakage, idle error를 포함한다.
3. Qiskit Runtime의 resilience pipeline은 Pauli twirling, ZNE 등으로 raw hardware error를 바꾸므로, raw fidelity formula와 mitigated observable error가 일치하지 않을 수 있다.
4. `epsilon_trotter + epsilon_hardware`는 보수적 upper bound로는 쓸 수 있지만 실제 observable error 모델로는 과도하게 단순하다.

따라서 synthesis 선택은 복수의 방법을 병렬 적용해 robust decision으로 낸다. 아래 표에서 `F_hw`는 보수적 하드웨어 생존 인자이며, 실험 조건에 따라 `F_2q` 또는 `lambda_hw`의 보수적 추정값을 사용한다.

| 방법 | 수식/기준 | 역할 | 한계 |
|---|---|---|---|
| Hard constraint | `depth<100`, `2q gate<400`, `2q-depth<30` | 실행 불가능한 후보 제거 | accuracy를 직접 반영하지 않음 |
| Calibrated 2Q burden | `B_2q≈Σ_e n_e p_e` or `1-Π_e(1-p_e)^{n_e}` | edge quality까지 반영한 QPU risk Pareto 축 | backend calibration과 layout에 의존 |
| Conservative upper bound | `epsilon_total_bound = epsilon_trotter + (1-F_hw)` | worst-case safety check | 실제 observable attenuation을 과장할 수 있음 |
| Multiplicative attenuation | `epsilon_deploy = |O_exact - lambda_hw O_PF|` | noisy expectation sweet spot 탐색 | noise model misspecification에 취약 |
| Empirical/QEM route | raw, mitigated backend result 비교 | 최종 실험 근거 | shot budget과 queue access 필요 |

최종 선택 규칙:

1. hard constraint fail 후보는 제거한다.
2. 남은 후보에서 ideal-PF Pareto와 calibrated 2Q burden Pareto를 각각 만든다.
3. 본문 figure는 `ideal error` 대 `B_2q` 또는 `hardware-risk proxy`의 2D Pareto를 기본으로 하고, `2q-depth`, `depth`, `native_2q_gate_count`는 후보표 컬럼으로 보고한다.
4. 가능하면 `B_2q`, `2q-depth`, `depth`, `epsilon_Z0`의 4축 Pareto를 내부 검증 또는 appendix로 만든다. 본문 결론은 이 고차원 plot 하나에 의존하지 않는다.
5. `e_2q`, edge calibration, `gamma`를 plausible range에서 sweep한다.
6. 여러 모델에서 반복적으로 선택되는 후보를 primary QPU synthesis option으로 둔다.
7. 모델마다 선택이 갈리면 "unique optimum"을 주장하지 않고, conservative candidate와 accuracy-oriented candidate를 둘 다 보고한다.

최종 QPU Pareto 표준 축:

| 축 | 의미 | 본문 사용 여부 |
|---|---|---|
| `epsilon_Z0` 또는 `mean_state_infidelity` | ideal Trotter error | 본문 |
| `depth` | decoherence/runtime pressure | 본문 |
| `two_qubit_depth` | entangling-layer pressure | 본문 |
| `native_2q_gate_count` | structural 2Q burden | 본문 |
| `B_2q≈Σ_e n_e p_e` | calibrated hardware error burden. edge calibration이 없으면 `hardware-risk proxy`로 격하 | 본문 핵심 |
| `symmetry_leakage` | logical symmetry break diagnostic | appendix 또는 후보표 보조 컬럼 |

### 6.5 r_H-boundary 탐색

제안사항을 P1 핵심으로 승격한다.

N을 다음처럼 확장한다.

$$
N=4+8r_H,\quad r_H=1,2,3
$$

즉:

| `r_H` | N |
|---:|---:|
| 1 | 12 |
| 2 | 20 |
| 3 | 28 |

각 `r_H`에 대해 9개 synthesis setting과 default grouped ordering으로 회로를 만들고, Heron coupling map 또는 fake/target backend에 transpile한다. 가능한 경우 길이 N cycle 또는 near-cycle embedding을 찾아 SWAP 증가를 최소화한다.

필수 산출:

1. `r_H, N, setting`별 resource table.
2. hard-constraint pass/fail table.
3. setting별 `r_H,max`, 즉 세 constraint를 모두 만족하는 최대 `r_H`.
4. first-failure reason: depth, 2q gate, 2q-depth 중 어떤 제약이 먼저 깨지는지.
5. QPU 실행 후보: `r_H=1`에서 pass하고, `min_margin`이 가장 크며, calibrated 2Q burden과 ideal error에서 Pareto-dominated가 아닌 setting.

### 6.6 Heron hardware prior

별첨 Word/PDF 문서의 내용은 다음처럼 사용한다.

- Heron의 tunable coupler는 crosstalk를 줄이는 구조적 이점이 있다.
- 2-qubit gate error가 회로 성공 확률을 지배하므로, 2q gate count와 2q-depth를 depth와 동등하게 본다.
- 문서의 2q error, 1q error, T1/T2 수치는 calibration-day backend properties로 재검증해야 하며, final report에서는 backend properties가 있으면 그것을 우선한다.
- `depth<100`은 decoherence 및 idle error 누적을 제한하기 위한 operational budget으로 해석한다.

### 6.7 Hardware result comparison

실제 또는 mock execution 결과는 다음 항목으로 비교한다.

- ideal exact `⟨Z_0⟩`
- noiseless PF `⟨Z_0⟩`
- calibrated 2Q burden `B_2q`
- fidelity-prior attenuated prediction `λ_hw ⟨Z_0⟩_PF`
- backend estimate `⟨Z_0⟩`
- shot confidence interval
- readout mitigation 여부
- resilience level
- discrepancy decomposition: Trotter error, shot noise, readout error, 2q error, decoherence, routing overhead

---

## 7. Problem 2: autocorrelation and spectral information

### 7.1 Exact autocorrelation

$$
A(t)=\langle\psi_0|e^{-iHt}|\psi_0\rangle
=\sum_n|\langle E_n|\psi_0\rangle|^2e^{-iE_nt}
$$

필수 sanity:

- `A(0)=1`
- `|A(t)|<=1`
- statevector overlap 방식과 eigenbasis sum 방식이 `<=1e-12` 일치

### 7.2 Trotterized autocorrelation

At least one product-formula setting을 사용한다. 기본은 Problem 1에서 Pareto 후보로 선택된 setting이다.

`A_PF(t)` 계산 경로:

1. statevector direct overlap
2. Hadamard-test circuit simulation

두 경로는 verification points에서 `<=1e-9` 일치해야 한다.

### 7.3 Autocorrelation error

$$
\epsilon_A(t)=|A_{\mathrm{exact}}(t)-A_{\mathrm{PF}}(t)|
$$

scale:

- x: time linear
- y: log10 with floor

summary:

- mean and max over `t<=2.0`
- full-time max over `t<=15.0`
- final error at `t=15.0`

### 7.4 Fourier spectrum

Convention:

```python
spectrum = fftshift(fft(A * window))
energy_axis = -2*pi*fftshift(fftfreq(len(A), d=DT_GRID))
```

Resolution:

$$
\Delta E \approx \frac{2\pi}{T_{\max}}
$$

Peak matching은 `|E_peak-E_n|<=ΔE`를 기본 tolerance로 한다. Hann window를 기본으로 쓰되, rect window를 appendix에 비교한다.

### 7.5 Spectral distortion ablation

ordering 대신 이 축을 창의성 본체로 둔다.

비교 대상:

- exact `A(t)` spectrum
- `A_PF(t)` spectrum for Pareto candidate settings
- QPU-feasible settings from §6.3

metrics:

| metric | 정의 |
|---|---|
| peak location error | `min_n |E_peak_PF - E_n|` |
| peak amplitude error | matched peak magnitude difference |
| top-k peak recall | exact top-k eigenenergy peak 중 PF spectrum이 `ΔE` 내 재현한 비율 |
| spectral leakage | dominant exact peaks 주변 window 밖의 normalized spectral mass |
| spectrum distance | normalized L1 distance after common windowing and normalization |

plot scale:

- energy axis linear, tick interval tied to `ΔE`.
- magnitude는 주 plot linear, 작은 leakage를 보일 때 보조 log-magnitude plot을 사용한다.

---

## 8. Symmetry and leakage diagnostics

N=12 QPU model에서 ideal Hamiltonian은 number conserving이다.

$$
\hat N=\sum_i\frac{1-Z_i}{2}
$$

필수 diagnostic:

1. exact single-excitation baseline: `⟨Nhat⟩=1`.
2. noiseless PF state에서 `|⟨Nhat⟩-1|`.
3. 가능하면 probability outside single-excitation subspace.
4. `⟨Z_0⟩` error와 leakage metric의 상관.

해석 원칙:

- leakage는 hardware noise가 아니라 product-formula decomposition 자체가 symmetry를 얼마나 깨는지 보는 logical diagnostic이다.
- QPU 결과에서 additional leakage-like behavior가 보이면 noise, readout, routing overhead와 구분해서 서술한다.

Optional creative extension:

- 표준 Pauli-term PF와 symmetry-aware block split을 비교할 수 있다.
- 단, 이는 guide의 필수 synthesis option을 대체하지 않는다. Appendix method로만 둔다.

---

## 9. Bonus Problem 3

Bonus는 다음 순서로 수행한다.

1. representative circuit: 기본 `ST2_k3`, `t=0.5`.
2. backend 또는 fake backend transpilation.
3. opt level 0,1,2,3 비교.
4. resource metric: depth, total gate count, two-qubit gate count, two-qubit depth.
5. Rustiq availability check.

해석은 Problem 1(c)의 opt-level sweep과 동일한 원칙을 따른다.

- opt level 변화는 resource 변화를 만든다.
- ideal logical error는 보존되어야 한다.
- Rustiq가 없으면 "environment did not provide Rustiq support"라고 명시하고 끝낸다.

---

## 10. 통합 Sanity Gate

| ID | 항목 | 통과 조건 |
|---|---|---|
| S1 | Hamiltonian | Hermitian, term count `4N`, periodic 3 terms, 모든 baseline ordering matrix 동일 |
| S2 | convention | `BITSTRING_QISKIT == BITSTRING_Q0_FIRST[::-1]`, observable label 일치 |
| S3 | exact state | norm 1, `exact_state(0)==psi0`, time reversal consistency |
| S4 | PF circuit | synthesized circuit has nonzero 2q gates, reps 증가 시 short-time error 감소 |
| S5 | opt preservation | opt 0-3 statevector equivalence `<=1e-9` |
| S6 | resource consistency | table, circuit diagram, plot이 같은 compiled circuit에서 나온다 |
| S7 | resource time-invariance | representative times에서 resource 동일 |
| S8 | Problem 2 exact | `A(0)=1`, `|A|<=1`, eigenbasis expression 일치 |
| S9 | spectrum | peak와 nonzero-overlap eigenenergy가 `ΔE` 내 일치 |
| S10 | plot scale | 모든 figure에 scale decision, tick rule, zero handling 기록 |
| S11 | QPU constraints | depth, 2q gate, 2q-depth pass/fail 및 margin 계산 |
| S12 | calibrated burden | transpiled native 2Q edge counts와 edge error를 연결해 `B_2q` 산출 |
| S13 | noisy balance | hard constraint, calibrated burden, conservative bound, attenuation model의 선택 결과를 비교 |
| S14 | scope control | 본문 산출물과 appendix 산출물을 분리 |
| S15 | N/r_H separation | N=6 baseline, N=12 QPU, N=20/28 boundary 결과를 혼합하지 않는다 |
| S16 | guide skeleton implementation | `2_problem_guide.ipynb`의 문제 순서와 problem markdown을 보존하고, TODO code cell은 제자리에서 완성 |
| S17 | metric naming discipline | backend edge calibration이 없으면 `calibrated 2Q burden`이 아니라 `hardware-risk proxy`로 보고 |
| S18 | basis separation | prompt-basis CX count와 backend-native 2Q count를 같은 축/동일 물리량으로 섞지 않는다 |
| S19 | Qiskit-native implementation | 회로 합성, transpilation, state evolution, resource count는 Qiskit public API 결과를 primary로 사용 |

---

## 11. 산출물

### P0 산출물

- guide skeleton completion report.
- Qiskit-native API usage and fallback note.
- Hamiltonian summary table.
- baseline ordering explanation table.
- opt=3 resource table.
- opt-level resource sweep with scale audit.
- state infidelity and local observable error plots.
- short-time scaling slope table.
- accuracy-resource Pareto plot with log/linear scale decision.
- Problem 1(g) N=12 QPU candidate budget table.
- calibrated 2Q error burden Pareto table.
- noisy-QPU deployable synthesis selection table with exactly one primary candidate and one fallback candidate.
- exact and PF autocorrelation plots.
- DFT spectrum and eigenenergy peak table.
- full sanity gate report.

### P1 산출물

- `r_H=1,2,3` QPU runnability boundary table.
- setting별 `r_H,max` and first-failure reason.
- minimal spectral distortion table for selected QPU candidate.
- symmetry leakage diagnostic table as appendix-only evidence unless it changes candidate choice.

### P2 산출물

- Bonus Problem 3 opt-level backend/fake-backend comparison.
- Rustiq availability statement and comparison if available.

---

## 12. 최종 결론 작성 규칙

1. "best"라는 단어는 단독으로 쓰지 않는다. 항상 "best under metric X and constraint Y"로 쓴다.
2. accuracy 결론은 `t<=2.0` 기준, full-time plot은 qualitative로만 쓴다.
3. QPU 실행 결론은 hard constraint pass/fail 및 margin이 먼저다. accuracy가 좋아도 constraint를 실패하면 QPU candidate가 아니다.
4. noisy-QPU balance 결론은 단일 수식 하나로 내지 않는다. upper bound, attenuation model, empirical/QEM evidence가 일치할 때만 강한 결론으로 쓴다.
5. calibrated 2Q error burden은 본문 Pareto 기준으로 사용하되, 그 자체를 실제 fidelity의 완전한 예측값이라고 주장하지 않는다.
6. plot caption에는 scale과 grid rule을 적는다.
7. ordering은 resource-oriented baseline 설명으로만 둔다.
8. 창의성은 deployable synthesis selection을 1순위로 주장하고, spectral preservation과 QPU runnability boundary는 보조 근거로 둔다. symmetry leakage는 appendix diagnostic으로 둔다.
9. 최종 notebook은 `2_problem_guide.ipynb`의 problem-statement markdown과 문제 순서를 보존한다. TODO code cell은 제자리에서 완성하고, 필요한 helper/plot/validation은 추가 cell로 보강한다.
10. backend calibration이 없는 경우 `B_2q`는 calibrated fidelity estimate가 아니라 hardware-risk proxy라고 쓴다.
11. prompt-basis CX count, compiled 2q gate count, backend-native 2q gate count를 구분해서 표기한다.
12. Qiskit public API로 가능한 작업은 Qiskit 결과를 primary로 보고한다. 직접 구현은 Qiskit에 해당 public API가 없거나 독립 검증이 필요한 경우에만 사용한다.
