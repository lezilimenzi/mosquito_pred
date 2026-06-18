# 모기 비행 궤적 예측 — 설계 문서

> 🏁 **대회 종료 (2026-06-12): 최종 LB 0.7002 · 최종 순위 12등.**

> **현 채택: Rotation-gated 물리 decoder (LB 0.6996)** — 단일 MLP physics-structured decoder.
> `pred = p_last + R·[w_v·e^(−exp_v)·rodrigues(v_EMA, ω) + w_a·e^(−exp_a)·a_EMA]`, ω=(ω_hist 해석적 + ω_delta 학습)·gate.
> 핵심 새 물리 = **rotation gate**(CV 직선 base가 못 휘는 곡선 외삽). 노트북 `10_rotgate.ipynb`(구 `hr_aware_rotgate_e38.ipynb`), 제출 `submission_rotgate_e38.csv`, 캐시 `results/rotgate_e38_{oof,test}_seed{42,7,123}.npy`(둘 다 Colab/Drive 측).
> 실험 결과 전체 = EXPERIMENTS.md.

---

## 📁 저장소 노트북 매핑 (GitHub 공개본, 2026-06-11 정리)

> LB를 갱신한 핵심 노트북 10개만 `01~10`으로 재넘버링해 보존. 그 외 실험 노트북·생성기(`_gen_*.py`)·EDA/스크리닝 스크립트·중간 산출물(`results/*.npy·json`, png/html)은 저장소에서 제외(캐시는 원래 Colab/Drive 측 생성물). 본문의 옛 파일명 참조는 아래 표로 대응.

| 실험 # | 저장소 파일 | 옛 파일명 | LB |
|---|---|---|---|
| #10 | `01_v3_baseline.ipynb` | `submission_d4_v3.ipynb` | 0.6678 |
| #14 | `02_frenet_target.ipynb` | `hr_aware_t.ipynb` | 0.6878 |
| #15 | `03_frenet_multiseed.ipynb` | `hr_aware_t_ms.ipynb` | 0.6888 |
| #17 | `04_axiswise_sigma.ipynb` | `hr_aware_t_ax.ipynb` | 0.6906 |
| #29 | `05_topk_k15.ipynb` | `hr_aware_topk_e29.ipynb` | 0.6906 |
| #32 | `06_split_selector.ipynb` | `hr_aware_split_e32.ipynb` | 0.6910 |
| #33 | `07_soft_sweep.ipynb` | `hr_aware_soft_sweep_e33.ipynb` | 0.6914 |
| #35 | `08_t_grid.ipynb` | `hr_aware_t_grid_e34.ipynb`* | 0.6916 |
| #36 | `09_rnn_aug.ipynb` | `hr_aware_rnn_aug_e36.ipynb` | 0.6920 |
| #38 | `10_rotgate.ipynb` ⭐ | `hr_aware_rotgate_e38.ipynb` | 0.6996 |

> *#35의 옛 파일명은 생성 시점 라벨 `e34` 유지. 삭제된 보조 노트북(d2~d6, gru_e25, xgb_e26 등)·Colab 캐시는 EXPERIMENTS.md 각 entry의 `산출물` 기록 참조.

---

## 1. 데이터·평가

| 항목 | 내용 |
|---|---|
| 평가 | Hit Rate @ r=1cm (3D Euclidean) |
| 입력 | 11 포인트, 40ms 간격 (t = −400ms ~ 0ms) |
| 예측 | t = +80ms의 (x, y, z), sensor-local (x forward, y left, z up, m) |
| 규모 | train/test 각 10,000 sample (minority 1,575 ≈ 15.8%) |

**핵심 데이터 특성**
- 모기 평균 속도 ≈ 0.62 m/s, 80ms 이동 ≈ 50mm
- 공간 범위 **비대칭**: x ∈ [0.50, 6.88] (한쪽 벽 있는 챔버), y ∈ [−2.59, 2.59] 대칭, z ∈ [−1.68, 2.61]
- **+x 드리프트** mean v = (+0.238, +0.008, +0.035) m/s (CO₂·공기 흐름 추정)
- **중력 prior 없음** mean a_z ≈ 0 (능동 비행)
- 위치 노이즈 (cubic-detrend std): x=3.18, y=3.00, z=2.06mm — 평가 1cm의 ~1/3
- 좌표 변환 증강(yaw 회전·x-flip·재중심화) **금지** — 비대칭이 신호 (부록 A)

**가속도 구간별 HR (등속(Constant Velocity) 외삽 기준)**

| acc_mag (m/s²) | 비율 | HR@1cm |
|---|---|---|
| 0~2 | 46.8% | 0.811 |
| 2~5 | 37.4% | 0.482~0.304 |
| **5~10 (minority 진입)** | 6.1% | 0.246~0.189 |
| 10~20 | 3.8% | 0.126~0.083 |
| 20+ | 2.0% | 0.046~0.000 |

→ **acc_mag ≥ 5 m/s² (minority, 15.8%)** 가 순위를 가르는 핵심 구간. 직선 외삽이 못 따라가는 **곡률·급기동 영역**이 곧 최종 모델의 표적.

---

## 2. 파이프라인 구조 — Rotation-gated 물리 decoder (개념 + 근거)

단일 MLP가 **물리 식의 계수만 학습**하는 구조. 좌표를 직접 회귀하지 않고, CV+CA 외삽이라는 강한 prior 위에 회전·감쇠 보정만 NN이 얹는다. 이 "물리에 묶인 작은 보정"이 train-fold 과적합을 줄여 generalization 우위(§3·부록 A)로 이어진다.

### 2.1 전체 도식

```
trajectory (N, 11, 3)
        │
        ├──► extract_features → 24-dim
        │       (v_local, a_local, speed, acc_mag, θ 통계 6, p_std_local, |v_local|, jerk_l, jerk_mag)
        │       + 학습형 dir_net heading → 회전행렬 R = [fwd, right, up]
        │
        ├──► v_EMA, a_EMA   (학습형 EMA α∈[0.1,0.9] / β∈[0.8,0.999], temporal_net)
        │
        ├──► ω = (ω_hist + ω_delta) × rotation_gate
        │       ω_hist = attention(과거 3 프레임 회전벡터, validity-mask)   # 해석적
        │       ω_delta = omega_net(feat) × speed_gate                       # 학습
        │       rotation_gate = σ((θ−θ_thr)·10) × σ((speed−speed_thr)·200)   # 저속·직진 회전 차단
        │       v_rot = rodrigues(v_EMA, ω)        # ★ 곡선 외삽 (핵심 새 물리)
        │
        ├──► pred_local = w_v·e^(−exp_v)·v_rot + w_a·e^(−exp_a)·a_EMA
        │       w_v≈2 / w_a≈1 prior (2프레임 CV+CA, PriorBiasedLinear init)
        │       exp_v, exp_a = softplus(NN)·(|v|, |v|², θ, θ²) 다항식 감쇠 (dynamics_net)
        │
        └──► pred = p_last + R · pred_local
             Loss = soft-hit(1.3cm, k=408) + 129·MSE + 0.051·NLL(diffusion_net log_var)
             학습 = 등속외삽 sliding-window 증강 + theta 가중 oversampling(급회전 5×)
             검증 = honest 5-fold(stratify=minority) × 3-seed solo
             → LB 0.6996  (OOF 0.6765, OOF→LB lift +0.0231)
```

### 2.2 Local frame R — 왜 motion-aligned 좌표인가

`dir_net`이 학습한 heading으로 진행방향 `fwd`를 정하고 `right`·`up`을 직교화해 `R=[fwd,right,up]`을 구성, 모든 동역학(v, a, jerk, 예측)을 이 local 좌표에서 계산한다.

- **근거**: 모기 운동은 진행방향(속도 크기 오차)·구심방향(곡률)·평면이탈로 자연 분해된다. 절대 좌표(x,y,z)는 챔버 비대칭·드리프트와 얽혀 분포 의존이 큰 반면, motion-aligned 좌표는 운동 물리와 정렬된 축에서 보정을 학습한다. (Cart 잔차 대비 Frenet 잔차가 feature·target 정렬 R²를 크게 끌어올린 것이 cascade 라인에서 정량 확인됨 — 부록 C.)
- **학습형 heading**: `fwd`는 마지막 속도가 아니라 `dir_net`이 10개 속도 프레임에 softmax 가중을 줘 합한 방향(`Σ wᵢ·vᵢ`)으로 정한다. 고정 3-step 가중 `(3,2,1)/6`은 fallback이고, 학습 가중은 sample별로 노이즈 프레임을 죽이고 안정 프레임에 집중한다.

### 2.3 24-dim feature 구성과 서브넷 라우팅

`extract_features`가 local frame R 위에서 24-dim feature를 만들고, 각 서브넷이 필요한 부분만 입력받는다.

| idx | feature | 의미 |
|---|---|---|
| 0–5 | `v_local`(3), `a_local`(3) | local 속도·가속도 |
| 6–7 | `speed`, `acc_mag` | 속도·가속도 크기 (gate·감쇠 입력) |
| 8–13 | `θ` 통계 6 (θ, mean, std, trend, vel, acc) | 회전각 현재값 + 시계열 동향(시작/유지/종료) |
| 14–16 | `p_std_local`(3) | 궤적 위치 산포 |
| 17–19 | `v_local_abs`(3) | \|속도\| (감쇠 다항식 입력) |
| 20–23 | `jerk_l`(3), `jerk_mag` | local jerk·크기 |

**라우팅 (왜 부분 입력인가)**
- `temporal_net ← features[8:17]` (θ 통계 6 + p_std_local 3) → EMA 계수 α/β. **회전 동향·위치 산포로부터 평활 강도를 sample별로 학습** (직진/급회전에서 추종 속도를 달리).
- `dynamics_net ← 24 전체` → w_v/w_a 보정 + exp_v/exp_a 감쇠 계수. 감쇠는 `v_local_abs`(17:19)·θ에 곱해지고 softplus≥0이라 **빠르거나 급회전일수록 외삽을 줄인다**(구조적).
- `omega_net ← 24 전체` → ω_delta(회전 보정), `diffusion_net ← 24 전체` → log_var(NLL 분산).

**근거**: θ를 6개로 분해(현재·평균·표준편차·추세·1·2차차분)한 것은 회전이 *시작/유지/종료* 중 어느 국면인지가 외삽·감쇠 결정에 핵심이기 때문(모기 회전은 transient — §2.8). 속도·jerk 절댓값은 감쇠 다항식의 비선형 입력.

### 2.4 학습형 EMA (v_EMA, a_EMA)

마지막 속도/가속도를 단순히 쓰지 않고, `temporal_net`이 sample별 EMA 계수 α∈[0.1,0.9]·β∈[0.8,0.999]를 출력해 시퀀스를 평활한 v_EMA·a_EMA를 만든다.

- **근거**: 40ms 간격에서도 궤적이 빠르게 변해 긴 고정 윈도우 평활은 정보 파괴(부록 C의 K-window). 고정 recency 가중(예: [1,2,3]/6)의 일반화로, 노이즈 평활과 recency를 sample별로 타협한다.

### 2.5 Rotation gate — 핵심 새 물리 (곡선 외삽)

회전속도 `ω = (ω_hist + ω_delta) × rotation_gate`를 만들고 `v_rot = rodrigues(v_EMA, ω)`로 속도 벡터를 휘게 외삽한다.
- `ω_hist` = 과거 3 프레임 회전벡터의 attention 합 (validity-mask, 해석적)
- `ω_delta` = `omega_net(feat) × speed_gate` (학습 보정)
- `rotation_gate = σ((θ−θ_thr)·10) × σ((speed−speed_thr)·200)` — 회전각·속도 soft-AND

**왜 이게 +0.007의 출처인가**: soft-hit loss·sliding-window 증강·motion-aligned frame·HR-가중은 직전 base가 이미 보유했다. 유일하게 없던 것이 **회전(곡선 외삽)**이다. CV 직선 base가 구조적으로 못 휘는 **minority(곡률, acc≥5 15.8%, minor HR 0.30~0.36 정체)** 를 직격해 plateau(~0.692)를 돌파했다.

**gate 상수 근거** (θ_thr ≈ 1.0876 rad ≈ 62°/frame, speed_thr ≈ 0.0346 m/frame):
- 저속·직진 구간에서 gate→0 으로 회전을 끈다. 이는 base를 바꾸면 majority를 망치는 cannibalize(부록 B.1)를 **구조적으로 회피** — 직진·저속(majority)은 CV 외삽 그대로 두고, 급회전 sample에만 곡선 보정을 건다.
- 보수적 임계라 raw 발화는 소수에 그치고, 부족분은 `ω_delta` NN이 위에서 추가 학습한다.

### 2.6 물리 decoder — prior + 감쇠, 왜 일반화에 유리한가

`pred_local = w_v·e^(−exp_v)·v_rot + w_a·e^(−exp_a)·a_EMA`
- `w_v≈2 / w_a≈1` = +80ms(2프레임) **CV+CA 외삽 prior**를 `PriorBiasedLinear`로 초기화 (NN은 prior 주변 보정만).
- `exp_v, exp_a = softplus(NN)·(|v|, |v|², θ, θ²)` 다항식 — softplus≥0 이라 항상 **감쇠**(1 이하 배율)만 가능. 즉 NN이 물리 식 안에 묶여 발산할 수 없다.

- **근거**: 절대좌표 회귀가 아니라 검증된 물리 외삽의 *계수*만 학습하므로 자유도가 작다. 이 제약이 train-fold 과적합을 줄여 OOF→LB lift를 키운다(§3, +0.0231). 이것이 "제약 없는 NN < tree" 결론의 반례 — 진 것은 제약 없는 NN이지 NN 자체가 아니다.

### 2.7 Loss + 증강

**Loss** = `soft-hit(thr=1.3cm, k=408)` + `129·MSE` + `0.051·NLL(log_var)`
- soft-hit: HR@1cm을 미분가능하게 근사 (거리<thr → hit). thr를 평가 1.0cm보다 큰 1.3cm로 둬 boundary 과적합을 완화.
- 129·MSE: soft-hit은 멀리서 평탄(gradient 소실)하므로 기하 구조 유지를 위해 큰 가중의 MSE 병행.
- NLL(heteroscedastic, `diffusion_net`의 log_var): 큰 잔차(주로 minority)에 자동으로 낮은 가중 → minority 잔차 완화.

**증강** = 등속외삽 sliding-window(w∈[3,11], extended target[4..10,12]) + θ 가중 oversampling(급회전 5×)
- 한 trajectory 내부의 짧은 윈도우로 **진짜 새 (X,y) pair**를 만든다 — 좌표 불변 변환(회전/미러)이 단순 upweighting에 그치는 것과 다르다(부록 B.3). "10k는 좁은 잔차엔 충분하나 절댓값엔 부족" 제약을 완화.

### 2.8 검증 — honest 5-fold, OOF→LB lift, 절대 OOF 금지

- **leakage-free**: 5-fold StratifiedKFold(stratify=minority) **row-level split**. sliding-window 증강은 train row 내부에서만, val/test는 real 11점 → step-12 직접 입력(실제 task와 동일).
- **3-seed solo**: seed{42,7,123} fold 평균 + seed 평균. 단일 모델(selector·블렌드 없음).
- **OOF 절대비교 금지**: 증강이 train 분포를 바꿔 OOF가 낙관 편향. 따라서 채택은 **LB로 판정**, OOF는 동일 family 내 상대 비교로만 사용.

**OOF→LB lift 비교 (채택 근거)**

| 모델 | OOF | LB | lift |
|---|---|---|---|
| RNN_aug solo (직전 base) | 0.6742 | 0.6920 | +0.0178 |
| **★ Rotation-gated decoder** | **0.6765** | **0.6996** | **+0.0231** |

물리 제약 decoder의 lift가 더 크다 = train-fold 과적합이 적어 private set generalization 우위. minority는 −0.0133(곡률 overshoot로 일부 손해)이나 major +0.0053·overall 이득이 LB에서 압도 → **신 아키텍처는 overall-primary로 판정**.

**왜 *학습* 회전만 이득인가**: 고정 ω의 해석적 CTRV는 모든 gate에서 ΔHR<0(overshoot — 모기 회전은 transient)이고 base 차이가 feature에 흡수(R² 0.83~0.99)된다. NN이 감쇠를 학습한 회전만 LB 이득 — base를 그냥 바꾸면 흡수되어 죽는다는 교훈(부록 B.1)과 양립.

### 2.9 설정값·학습 recipe

loss·gate·학습 하이퍼파라미터. **대부분 채택 시점의 검증된 설정값을 그대로 고정**(자체 재튜닝 안 함) — 값 자체보다 §2.5~2.7의 *역할*이 설계 근거다. 실제 값 = 노트북 셀 12(`HyperPhysics_xy2`)·셀 2.

| 그룹 | 값 | 근거 |
|---|---|---|
| soft-hit | `thr=0.0130`(1.3cm), `k=408` | thr>평가 1cm = boundary 과적합 완화. k = 거리→hit sharpness |
| loss 가중 | `MSE×129`, `NLL×0.051` | MSE = 기하 유지(soft-hit 평탄 보완), NLL = minority 잔차 자동 완화 |
| gate | `θ_thr=1.0876`, `speed_thr=0.0346`, sharpness `θ·10`·`speed·200` | 저속·직진 회전 차단(§2.5). sharpness = soft-AND 경사 |
| EMA 범위 | α∈[0.1, 0.9], β∈[0.8, 0.999] | 속도/가속도 평활 계수 학습 범위(§2.4) |
| prior 초기화 | `w_v≈2`/`w_a≈1`(decoder), `omega_w=[0,−0.5,−1]`(ω attention), `dir/dyn` PriorBiasedLinear | CV+CA 외삽 + 최근 프레임 회전 우선 prior |
| optimizer | AdamW `lr=0.0054`, `wd=0.0057`, `GRAD_CLIP=1.0` | 작은 보정만 학습 → 보수적 lr/wd |
| 스케줄 | `BATCH=256`, `EPOCHS=12`, `StepLR(step=3, γ=0.5)`, `PATIENCE=5` | val R-hit 기준 early-stop, best state 복원 |
| 검증 | `N_FOLDS=5`, `SEEDS={42,7,123}`, `MINORITY_THR=5.0`, `HIT_RADIUS=0.01` | §2.8 |

---

## 3. 현재 모델로 온 경로 — 효과 있던 방법 (순서대로)

LB는 1일 1회 제출. 두 국면으로 나뉜다 — **(A) cascade 라인**이 데이터 이해와 LB를 ~0.692까지 끌어올렸고, **(B) rotgate**(별개 단일 MLP lineage)가 회전 물리로 plateau를 돌파했다.

```
v3 baseline (Cart×Cart, LGBM residual):                 LB 0.6678
 → #1(14) Frenet target (motion-aligned basis 회전):     0.6878  (+0.0200)
 → #2(15) multi-seed (fold seed {42,7,123} ensemble):    0.6888  (+0.0010)
 → #3(17) axis-wise W1 σ (Frenet std 비례 가중):          0.6906  (+0.0018)
 → #4(29) V3 26→15 feature (simplicity tie):             0.6906  (±0.0000)
 → #5(32) K15/GRU Soft selector (hetero backbone blend): 0.6910  (+0.0004)
 → #6(33) + T=1.5 soften:                                0.6914  (+0.0004)
 → #7(35) T grid → T=0.5:                                0.6916  (+0.0002)
 → #8(36) RNN_aug solo (별개 lineage, backbone 교체):     0.6920  (+0.0004)
 → ★#9(38) Rotation-gated 물리 decoder (별개 lineage):    0.6996  (+0.0076, plateau 돌파)
                                                          ↓
누적 (v3 → #9(38)): +0.0318
```

| 방법 | EXP | 무엇 | 왜 효과 | LB Δ |
|---|---|---|---|---:|
| **Frenet target** | #14 | 잔차를 Cart(x/y/z) 대신 motion-aligned(L/N/B) 축에서 학습 | feature·target 정렬로 학습 신호 강화. 단일 lever 최대 효과 | +0.0200 |
| **Multi-seed** | #15 | fold split seed 3개 ensemble + simplicity로 feature 축소 | OOF 분산↓, 동률이면 단순한 쪽 채택 | +0.0010 |
| **Axis-wise σ 가중** | #17 | Frenet 잔차 std 비대칭(L:N:B=1:0.81:0.62)에 비례한 boundary 가중 | 대칭 Cart에선 무효였던 가중을 비대칭이 활성화 (minor +0.0028) | +0.0018 |
| **Simplicity 축소** | #29 | feature 26→15 (gain 상위만), LB 동률 | 핵심 신호는 상위 15개로 충분 — 42% 단순화 | ±0.0000 |
| **Soft selector** | #32 | LGBM ⊕ GRU per-sample 가중 blend | 이질 backbone(corr<0.95) sample별 가치 추출 | +0.0004 |
| **T soften (→0.5)** | #33·#35 | selector 가중을 logit-space로 0.5쪽 압축 | 약한 classifier 환경에서 균등 blend 접근 → minor boundary push. T grid plateau 확정 | +0.0006 |
| **RNN_aug solo** | #36 | per-step Frenet 9ch GRU + HR-surrogate loss + 내부 sliding-window 증강 | 단일 RNN이 cascade와 LB 동급 → selector 라인 소진, backbone 교체 (tie-break) | +0.0004 |
| **★ Rotation-gated decoder** | #38 | 회전 gate + 감쇠 물리 decoder 단일 MLP (§2) | **신 SOTA.** 없던 곡선 외삽 물리로 minority 직격, OOF→LB lift +0.0231로 일반화 우위 | **+0.0076** |

> #14~#35는 cascade 라인(보조자산, 부록 C), #36·#38은 backbone 교체. rotgate는 LGBM·selector를 쓰지 않는 별개 단일 MLP다.

---

## 부록 A. 검증된 설계 판단 (항상 유지하는 제약)

| 판단 | 근거 |
|---|---|
| **direct prediction(target=절대좌표) 금지** | minority 1,575개로 0.5~6.9m 절댓값 학습 불가. 외삽 prior + 보정 구조가 필수 |
| **좌표 변환 증강 금지** | x∈[0.50,6.88] 비대칭 + +x 드리프트가 신호. 회전·평행이동·x-flip 모두 mapping을 깸 |
| **입력 smoothing(긴 윈도우) 위험** | 40ms 안에도 궤적 변화 — K-window HR: K1 0.5788 → K2 0.5267 → K5 0.3729. 평활은 정보 파괴 |
| **base는 거의 unbiased** | median \|target\|/\|2·v_last\| = 0.999 → 외삽 prior 위 보정이 0 중심으로 안정 |
| **가속도 클리핑 금지** | acc_mag와 \|residual\| 상관 0.386 — 크기는 유효 신호, 방향만 노이즈 |
| **OOF-based 가중 필수** | in-sample 가중은 leakage. no-weight OOF로 측정 후 가중 학습 |
| **simplicity tiebreak** | OOF noise zone(\|Δ\|<std×√2) overlap 시 단순한 쪽 채택 (#15·#29 LB로 정당) |
| **증강 OOF는 낙관** | 내부 증강은 OOF를 부풀림 → 채택은 LB, OOF는 family 내 상대 비교만 (§2.8) |

---

## 부록 B. 폐기 라인 핵심 교훈 (재사용 자산; 상세 수치 = EXPERIMENTS.md)

**B.1 base 변경 = 후속 모델이 흡수(cannibalize)** (#12 Kalman / #19 Latency / #34 base-diversity)
물리 base + 잔차 모델 구조에서, base를 smoothing/shift하면 그 보정이 base단에서 미리 제거되어 잔차 모델이 잡을 게 줄어든다. base 차이 `delta`가 v/a에 선형이면 feature span에 **흡수 R²≈1**(#34 CA delta_L=1.0, damped=1.0) → 구조적 dead. **평가 규칙**: base 변경 lever는 raw HR 금지, 흡수-R² + corrected Δ로 판정. — rotgate의 rotation_gate는 직진·저속을 끄는 방식으로 이 cannibalize를 *구조적으로 회피*하면서 곡선 보정만 추가한다(§2.5).

**B.2 EDA-feature 추가 = transfer fail 3연속** (#11 D6 / #13 F / #18 F-INT)
수학적 새 정보 평면(적분/곱셈)도 모델이 기존 feature 조합으로 implicit reconstruct → no-op/noise. marginal corr·univariate R²만으론 채택 금지. feature는 가중 적용 후 lift(P1→P2)까지 봐야 한다.

**B.3 augmentation = sample upweighting (invariant transform일 때)** (#16 T-AUG)
Frenet basis가 trajectory에서 계산되면 회전/미러 시 (X,y) 모두 invariant → aug = 단순 upweighting(mapping 변화 0)이라 가중과 충돌. **유효 조건** = 진짜 새 trajectory(예: rotgate의 내부 sliding-window, §2.7)·world-frame anchored feature.

**B.4 hetero-backbone ensemble 양방향 closed** (#25 NN / #26 tree-tree)
blend LB ≈ component 단독 LB의 선형 평균 → 하나라도 base보다 낮으면 blend ≤ base. **ensemble 진입 gate** = component 단독 LB ≥ base AND corr < 0.95. (per-sample selector는 별개 메커니즘 — #32 채택.)

**B.5 post-hoc lever transfer fail** (#22 calibration / #28 feature) — LB-시도 자격 3가드
OOF micro-positive(Δo < +0.0028)는 LB transfer 보장 안 함. LB 시도 자격 = (1) backbone 동질, (2) Δo>+0.0010, (3) **model parameter 변경**(post-hoc calibration/TTA/blend 제외) 셋 다 충족. HR(1cm)은 mean bias가 아니라 boundary 분포에 민감 → mean correction 단독 금지.

---

## 부록 C. 보조자산 — K15/GRU/LGBM cascade (직전 base, LB 0.6916)

rotgate와 **별개 lineage**. 보존 이유 = selector 재결합(rotgate ⊕ K15/rnn_solo) 미시도 자산. 상세 = EXPERIMENTS.md.

**도식 (요약)**
```
trajectory → physics_baseline (CV, 2점 차분, HR 0.5788)
           → Frenet frame (eL, eN, eB)
  Base 1: K15  = LGBM × 3축(res_L/N/B) × 5fold × 3seed, axis-wise σ 가중 (LB 0.6906 단독)
  Base 2: GRU  = Frenet residual GRU(64) (보존 cache)
  Lever : K15/GRU Soft selector (binary classifier p_clf) + T=0.5 soften blend
           → final = w'·GRU + (1−w')·K15            (LB 0.6916)
```

**핵심 설계 요소**
- **Physics CV base**: `base = p[-1] + (p[-1]−p[-2])/DT · DT_PRED`. 2점 차분만(K-window 근거, 부록 A). HR 0.5788 하한 보장.
- **Frenet residual target**: `label−base`를 (res_L 속도크기·res_N 곡률·res_B 평면이탈)로 분해해 LGBM 3 모델이 각각 학습. 잔차 std 9~14mm는 label std 1.19m보다 좁아 학습이 안정. 단일 lever 최대 효과(#14 +0.0200).
- **V3 15 feature**: motion magnitude 12 + Frenet jerk projection 2(j_L,j_N) + 시퀀스 std 1. gain 상위 15개로 26개 대비 LB 동률(#29).
- **W1/W5 sample weight**: boundary(10mm) 근처 Gaussian 가중(`max_w=2.5`), Frenet std 비대칭에 비례한 axis-wise σ(L4.3/N3.5/B2.7mm). OOF-based 2-phase 필수.
- **LGBM hp**: `regression_l1`(MAE, HR 정렬·outlier robust), `lr=0.05`, `num_leaves=31`, `min_data_in_leaf=5`(minority 분해), `max_bin=511`, early_stopping=50.
- **Soft selector**: K15↔GRU는 이질 backbone(corr<0.95)이라 per-sample blend로 oracle ceiling +0.0221 중 ~18% 회수. classifier는 약해도(recovery-AUC 0.5547) ensemble 효과. T soften은 0.5쪽 압축으로 균등 blend 접근.

**selector framework 요지** (재진입 시): HR 영향 sample은 recovery-area(A only hit ∪ B only hit, 2~5%) 한정 → overall accuracy/AUC는 informational, **recovery-area AUC + HR 4th guard(Δo>+0.0020) + Floor(vs 학습 0 rule)** 가 primary. backbone heterogeneity가 selector ROI를 결정(homo ~1% vs hetero ~17% 회수).

---