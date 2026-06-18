# 실험 로그

> 🏁 **대회 종료 (2026-06-12): 최종 LB 0.7002 · 최종 순위 12등.**

모든 실험 결과를 시간순으로 기록. 초기 RF/HGBM 단계는 RESEARCH.md 부록 B 참조.

평가 protocol: **#1~#9는 M1 (8000/2000 단일 holdout), #10부터 M2 (full 10000 5-fold OOF), LB는 M3**. metric convention은 RESEARCH §8 참조.

---

## 📁 저장소 노트북 매핑 (GitHub 공개본, 2026-06-11 정리)

> LB를 갱신한 핵심 노트북 10개만 `01~10`으로 재넘버링해 보존. 그 외 실험 노트북·생성기(`_gen_*.py`)·EDA/스크리닝 스크립트·중간 산출물(`results/*.npy·json`, png/html)은 저장소에서 제외(캐시는 원래 Colab/Drive 측 생성물). 아래 로그 각 entry의 옛 파일명·`산출물` 목록은 **그 시점 실제 산출물의 기록**이며, 저장소엔 이 표의 10개 노트북만 포함된다.

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

> *#35의 옛 파일명은 생성 시점 라벨 `e34` 유지. 채택 안 된 실험(#1~#9, #11~#13, #16, #18~#28, #30·#31, #34, #37)의 노트북·Colab 캐시는 저장소에 없음 — 로그 기록으로만 보존.

---

## 요약 표

| # | 날짜 | 실험 | 전체 HR | major | minor | 채택? |
|---|---|---|---|---|---|---|
| 1 | 2026-05-20 | baseline.ipynb (LightGBM residual, 28 feat) | M1 0.6485 | 0.7247 | 0.2516 | — |
| 2 | 2026-05-20 | feature v2 (31 feat: +jerk +centripetal +half_diff) | M1 0.6515 | 0.7265 | 0.2609 | ✓ |
| 3 | 2026-05-22 | feature v3 ablation (21 feat, traj-wide 통계 제거) | M1 0.6440 | 0.7223 | 0.2360 | ✗ |
| 4 | 2026-05-22 | imm_lstm (LGBM + LSTM gating) | M1 0.6580 | 0.7270 | 0.2889 | ✗ |
| 5 | 2026-05-23 | hr_aware D1 (OOF + boundary 분석) | (tr OOF 0.6430) | — | — | (D2 입력) |
| 6 | 2026-05-23 | hr_aware D2 W1 Gaussian (σ ∈ {2,3,5}mm) | M1 0.6655 (σ=3) | 0.7347 | 0.2952 | ✓ |
| 7 | 2026-05-23 | hr_aware D3 W2 sigmoid (α×τ 9-grid) | (best M1 0.6560) | 0.7228 | 0.2984 | ✗ |
| 8 | 2026-05-23 | hr_aware D4 W1 정밀 (σ×max_w 12-grid) | **M1 0.6665** (σ=3.5, max_w=2.5) | 0.7359 | 0.2952 | ✓ |
| 9 | 2026-05-23 | hr_aware D5 W4 axis-wise (V1+V2 × α) | M1 0.6665 (V2 α=0.5) | 0.7347 | 0.3016 | ✗ (D4 동률) |
| 10 | 2026-05-24 | submission_d4_v3.ipynb (full 10000 W1) | (OOF) | — | — | **LB 0.6678** ✓ |
| 11 | 2026-05-24 | hr_aware D6 (feature engineering, 4 config) | M2 0.6525 (D6.0) | 0.7223 | 0.2794 | ✗ |
| 12 | 2026-05-24 | hr_aware K (Kalman base, K-CV + K-CA) | M2 0.6494 (K1) | 0.7188 | 0.2781 | ✗ |
| 13 | 2026-05-24 | hr_aware F (feat Phase A+B, Cart) | M2 0.6475 (13 feat) | 0.7189 | 0.2658 | ✗ |
| 14 | 2026-05-25 | hr_aware T (Frenet target, V0~V5) | **M2 0.6630** (V2) | 0.7293 | 0.3086 | **LB 0.6878** ✓ |
| 15 | 2026-05-25 | hr_aware T-MS (V0/V2/V3 × 3 seed) | **M2 0.6615 ± 0.0007** (V3) | 0.7292 | 0.2995 | **LB 0.6888** ✓ |
| 16 | 2026-05-25 | hr_aware T-AUG (augmented data, V3 base) | M2 0.6504 (B best) | 0.7212 | 0.2720 | ✗ |
| 17 | 2026-05-25 | hr_aware T-AX (axis-wise W1 σ, V3 base) | **M2 0.6622 ± 0.0010** (C1) | 0.7295 | 0.3022 | **LB 0.6906** ✓ (당시 SOTA) |
| 18 | 2026-05-25 | hr_aware F-INT (internal displacement, T-AX C1 base) | M2 0.6615 (S1a) / 0.6601 (S1b) | 0.7293 / 0.7267 | 0.2988 / 0.3035 | ✗ (둘 다 fail) |
| 19 | 2026-05-26 | hr_aware Latency (L2 time_scale s grid 11개, v2 + σ-retune 옵션 C) | M2 0.6604 (v2 s=1.02) / 0.6609 (retune) | 0.7285 / 0.7289 | 0.2963 / 0.2967 | ✗ (cannibalize 재발) |
| 20 | 2026-05-26 | hr_aware Stage 2-a σ × max_w narrow grid (7 config, C1 + 6 ±10~20% variants) | M2 0.6622 ± 0.0010 (C1 sanity) / best variant 0.6619 (sB_2.3) | 0.7295 / 0.7294 | 0.3022 / 0.3012 | ✗ (6/6 G1 fail, C1 interior optimum 재확인) |
| 25 | 2026-05-27 | hr_aware GRU e25 (LGBM⊕GRU residual ensemble, V2 Frenet main + V1 Cart sanity) | M2 Mode B 0.6696 / GRU solo 0.6659 / V1 Cart 0.6508 | 0.7346 / 0.7306 / 0.7211 | 0.3219 / 0.3181 / 0.2749 | ✗ (Mode B LB 0.6888, GRU solo LB 0.6862 — 모든 blend α ≤ LGBM, NN backbone < tree backbone 정량 확인) |
| 26 | 2026-05-27 | hr_aware XGB e26 (XGBoost heterogeneous backbone, T-AX C1 protocol 비트 단위, V3 26 feat / Frenet target / W1 axis-wise σ) | M2 0.6626 ± 0.0018 | 0.7305 | 0.2997 | ✗ (XGB solo LB 0.6886, Δ −0.0020 vs LGBM C1 — corr 0.9779 → blend 자동 skip, tree-tree heterogeneity dead) |
| 22 | 2026-05-27 | hr_aware calibration e22 (Per-axis Cart calibration α/δ, T-AX C1 post-hoc) | M2 Mode C 0.6639 (Δo +0.0017 e17_candidate E) / Mode A best=identity / Mode B fail | 0.7308 / — / 0.7272 | 0.3060 / — / 0.3041 | ✗ (Mode C LB 0.6882, Δ −0.0024 vs T-AX C1 — e17_candidate flag 강화 조건 (backbone 동질 + Δo > +0.0010) 만족에도 LB transfer fail. post-hoc lever 별개 메커니즘) |
| 28 | 2026-05-28 | hr_aware v4_lite e28 (in-traj 2-step physics residual Frenet projection, 8 config feature ablation) | M2 best C5 (V3 26 + C:6=32 feat) **0.6624 ± 0.0003** / C3 full V4-lite 50 feat 0.6610 / C4 V4 standalone 24 feat 0.6261 | 0.7294 / 0.7283 / 0.6988 | 0.3041 / 0.3012 / 0.2372 | ✗ (C5 LB **0.6872, Δ −0.0034 vs T-AX C1** — OOF marginal +0.0002 (noise zone, threshold +0.0029) → LB transfer fail. univariate R² PASS (std_rL 0.0646 > V3 26 max 0.0530) → joint LGBM marginal → LB FAIL 3-stage 신호 degradation. #22 post-hoc + #28 model param 양쪽 OOF noise zone lever LB transfer fail로 **EXP #17 candidate flag 4번째 가드 (Δo > combined_std × √2 미충족 시 LB 제출 금지)** 추가. C4 standalone catastrophic = V3 26 본질 신호 확인. A 시계열 raw가 LGBM noise (C5 vs C3 +0.0014)) |
| 29 | 2026-05-28 | hr_aware topk e29 (V3 26 → top-K by gain subset 7 config, T-AX C1 protocol 비트 단위) | M2 K15 (15 feat) **0.6630 ± 0.0012** / K10 0.6625 / K20 0.6617 / K_tied_min (24) 0.6614 / K26 (=ref) 0.6622 ± 0.0010 / K5 0.6612 / K5_indep (4) 0.6579 | K15 0.7290 / K26 0.7295 (major peak) | K10 0.3105 (minor peak) / K15 0.3096 / K26 0.3022 | **✓ K15 채택 (LB 0.6906, Δ LB 0 vs K26 strict tie, Karpathy simplicity tiebreak 15 vs 26 feat)** |
| 30 | 2026-05-28 | hr_aware r2step e30 (in-traj 2-step CV residual current-frame projection, 3-way ablation K15+L/K15+N/K15+LN, V3 15 baseline) | M2 K15+L (17 feat) 0.6620 ± 0.0006 / K15+N (17) 0.6614 ± 0.0005 / K15+LN (19) 0.6608 ± 0.0004 (모두 K15_ref 0.6630 미달) | K15+L 0.7284 / K15+N 0.7291 / K15+LN 0.7281 | K15+L 0.3067 / K15+N 0.2995 / K15+LN 0.3007 | ✗ (3/3 4th guard fail, Δ_L −0.0010 / Δ_N −0.0016 / Δ_LN −0.0022, ADDITIVE 패턴 둘 다 음, 시나리오 "L− N−" → in-traj feature add 5차 fail, multi-baseline selector 즉시 전환) |
| 31 | 2026-05-28 | hr_aware split e31 (L15 Major/Minor Split Selector, K10 vs K15 LGBM clf V3 15 feat + minor→GRU hard split sanity, 3 mode ablation) | M2 Hard 0.6644 / Soft 0.6650 / Minor→GRU 0.6661 (vs K15 ref 0.6649) | 0.7309 / 0.7302 / 0.7312 | 0.3086 / 0.3162 / 0.3181 | ✗ (3/3 4th guard fail, classifier acc 0.5246 / **AUC 0.5139** V3 15 K10/K15 분기 정보 본질 부족 정량 확정. **K15/GRU 2-way oracle ceiling +0.0221 / minor +0.0533 발굴** = K10/K15 ceiling의 2배, backbone heterogeneity (LGBM vs Frenet GRU)가 same-backbone K-subset보다 selector value 2배. L15 binary K10/K15 폐기, 시나리오 3 K15/GRU selector 진입) |
| 32 | 2026-05-28 | hr_aware Split e32 (L16 K15/GRU hetero-backbone Soft per-sample selector, 6 mode) | M2 Soft 0.6686 (Δo +0.0037 vs K15) | 0.7332 | 0.3232 | **LB 0.6910** ✓ (+0.0004) |
| 33 | 2026-05-28 | hr_aware Soft Sweep e33 (L17 σT×Δ×α 27-cell full grid, T=1.5 채택) | M2 cell27 (T1.5/Δ∞/α1) 0.6687 | 0.7332 | 0.3238 | **LB 0.6914** ✓ (+0.0004) |
| 34 | 2026-05-31 | P0 baseline-diversity cheap-screen (CA β / damped / EWMA base family, 로컬 numpy + G_S5-lite, LB 미제출) | **corrected OOF CA β0.15 0.6546** (Δ −0.0043 vs CV 0.6589; raw 0.6008은 +0.0221이나 LGBM 흡수) | 0.7209 (−0.0053, G2 fail) | 0.2997 (+0.0013) | ✗ (raw 우위가 LGBM residual에 흡수. delta_L R²=1.0 (=a_tang_last) + delta_N R²=0.95 → cannibalization 측정 확정 = EXP #12 재현. baseline-diversity / L21 multi-baseline selector prior 하향) |
| 35 | 2026-05-31 | hr_aware t_grid e34 (L18 T 단일축 9-point fine grid {0.5~3.0}, T=0.5 채택) | M2 T=0.5 0.6688 (Δo +0.0039 vs K15, span 0.0003=plateau) | 0.7332 | 0.3225 | **LB 0.6916** ✓ (+0.0002) |
| 36 | 2026-06-01 | hr_aware rnn_aug e36 (L24 내부증강+L25 per-step Frenet+L22 softhit RNN→3-way selector, rnn_solo 채택) | M2 rnn_solo 0.6742 (⚠ 증강 OOF 과대평가, LB-only 판정) | 0.7335 | 0.3568 | **LB 0.6920** ✓ (+0.0004 sub-std, tie-break) |
| 37 | 2026-06-01 | hr_aware rnn_axsig e37 (L26 axis-σ RNN loss ablation, w_axis∝σ^−p p-ablation, rnn_solo #36 fork) | M2 p0=control 0.6719 / p1 0.6715 (single-seed 42) | 0.7320 / 0.7336 | 0.3505 / 0.3390 | ✗ **폐기** (Δ(p1−p0) −0.0004 ⊂ noise ±0.0014, p0=best=control → axis-σ 무효, 제출 없음) |
| 38 | 2026-06-01 | hr_aware rotgate e38 (L27 B **rotation-gated 물리 decoder**, 외부 0.699 HyperPhysics_xy2 이식, honest 5-fold OOF 3-seed) | M2 3-seed **0.6765** (Δo +0.0023 vs rnn_solo) | 0.7388 (+0.0053) | 0.3435 (−0.0133) | **✓ LB 0.6996** ⭐ (+0.0076, **신 SOTA**, 1등 0.70까지 −0.0004) |

**Cumulative LB (현 채택: Rotation-gated 물리 decoder, #38 LB 0.6996 ⭐)**:

```
v3 baseline (#10)                                       LB 0.6678
 → V2 single-seed Frenet target (#14)                   LB 0.6878  (+0.0200)
 → V3 multi-seed 3 seed (#15)                            LB 0.6888  (+0.0010)
 → T-AX C1 K26 axis-wise W1 σ (#17)                      LB 0.6906  (+0.0018)
 → V3 15 simplicity tiebreak K15 (#29)                  LB 0.6906  (±0.0000, 26→15 feat)
 → K15/GRU Soft selector hetero backbone (#32)          LB 0.6910  (+0.0004)
 → K15/GRU Soft + T=1.5 logit-space soften (#33)        LB 0.6914  (+0.0004)
 → K15/GRU Soft + T=0.5 (#35, T 9-point fine grid)      LB 0.6916  (+0.0002)
 → RNN_aug solo (#36, 별개 lineage·tie-break)            LB 0.6920  (+0.0004, sub-std)
 → ★ Rotation-gated 물리 decoder (#38, 외부 0.699 이식)   LB 0.6996  (+0.0076, plateau 돌파·신 SOTA)
   (#37 axis-σ RNN loss = 폐기·제출 없음, LB ladder 미포함)
                                                         ─────
v3 → 현재 cumulative gain                                +0.0318
1등 LB 0.70까지 남은 거리                                −0.0004 (약 0.04%)
```

폐기 EXP: #3·#4·#7·#9·#11·#12·#13·#16·#18·#19·#20·#22·#25·#26·#28·#30·#31·#34·#37. 채택 cascade(보조자산): #29 K15 → #32 Soft selector → #33/#35 T scaling → #36 rnn_solo. **현 채택 = #38 Rotation-gated decoder (별개 lineage).**

**결번 (계획만, 미실행)**: #21(L3 LGBM hyperparam) · #23(L4 Reflection TTA) · #24(V2 multi-seed ensemble) · #27(L8 boundary MLP) — 시나리오 1/2 closed로 미진입, 자세한 plan은 plan.md §3.3/§3.5/§3.6/§3.9 보존. 요약 표 번호 건너뜀은 이 4건.

**참고치**: HGBM 통합 weight없음 = 전체 0.6472 / minority 0.2705 (RESEARCH 부록 B).

⚠️ #4의 va_idx는 #1~#3과 다름 (stratify=minority 적용). #5·#6은 #4와 동일 protocol이라 직접 비교 가능.

---

## 효과 없었던 구조 변경 (✗) — 재시도 시 새 근거 필요

| 출처 | 시도 | ΔHR 전체 | ΔHR minor | 비교 기준 |
|---|---|---|---|---|
| #3 | feature 31→21 (trajectory-wide 통계 10개 제거) | −0.0075 | −0.0249 | #2 (31 feat) |
| #4 E1 | single LSTM (velocity 3D, hidden 32) | −0.0585 | −0.0699 | #4 LGBM-only |
| #4 E2 | IMM soft gating (LGBM normal + LSTM maneuver + MLP) | −0.0010 | −0.0032 | #4 LGBM-only |
| #4 E3 | hard split @ acc=5 (low LGBM / high LSTM) | −0.0275 | −0.0350 | #4 LGBM-only |
| #7 | D3 W2 sigmoid (α×τ 9-grid 전부 fail) | −0.0085~−0.003 | 일부 +Δ, gate1·2 fail | #4 LGBM-only |
| #9 | D5 W4 axis-wise weight (V1 단독 + V2 결합) | 0 (D4 동률) | +0.0064 (~1샘플) | #8 D4 |
| #11 | D6 (turn_accel + eff_change + −a_tang_last) | −0.0007~−0.0023 | −0.0038~−0.0108 | D6.0 (= v3 protocol) |
| #12 K1 | Kalman CV smoothing | −0.0031 | −0.0013 | K0 (= D6.0) |
| #12 K2 | Kalman CA | sanity fail | sanity fail | K0 |
| #13 | F Phase B lean 13 feat | −0.0019 | −0.0036 | F0 (= 31 feat no-weight) |
| #15 (post-hoc) | V2+V3 hetero ensemble | LB −0.0004 | — | V3 multi-seed LB 0.6888 |
| #16 | T-AUG aug_w=0.3 (best B) | −0.0111 | −0.0275 | V3 multi-seed |
| #18 S1a | F-INT 28 feat (V3+iL_mean+iN_mean, T-AX C1 σ) | −0.0007 | −0.0034 | T-AX C1 (V3_ref this run 0.6622) |
| #18 S1b | F-INT 11 feat (lean 9+iL_mean+iN_mean) | −0.0021 | +0.0013 (major −0.0028) | T-AX C1 |
| #19 v2 | L2 Latency best_s=1.02 (T-AX C1 σ) | −0.0018 | −0.0059 (G3 hard stop fail) | T-AX C1 (#17) |
| #19 retune | L2 Latency best_s=1.02 + σ-retune (4.35, 3.5, 2.7) | −0.0013 | −0.0055 (G3 hard stop fail) | T-AX C1 (#17) |
| #20 sL_4.0 | σ_L 4.3→4.0 (−7%), σ_N/σ_B/max_w C1 고정 | −0.0011 | −0.0006 | T-AX C1 (#17) |
| #20 sL_4.7 | σ_L 4.3→4.7 (+9%), 그 외 C1 | −0.0005 | −0.0036 | T-AX C1 (#17) |
| #20 sB_2.3 | σ_B 2.7→2.3 (−15%), 그 외 C1 | −0.0003 | −0.0011 | T-AX C1 (#17) |
| #20 sB_3.0 | σ_B 2.7→3.0 (+11%), 그 외 C1 | −0.0005 | −0.0011 | T-AX C1 (#17) |
| #20 mw_2.0 | max_w 2.5→2.0 (−20%), σ=C1 | −0.0005 | −0.0021 | T-AX C1 (#17) |
| #20 mw_3.0 | max_w 2.5→3.0 (+20%), σ=C1 | −0.0007 | −0.0013 | T-AX C1 (#17) |
| #25 Mode B blend | LGBM (T-AX C1) ⊕ GRU (V2 Frenet) axis-wise α=[0.5, 0.2, 0.4] | OOF +0.0074 / **LB −0.0018** | OOF +0.0197 / LB 추정 약화 | T-AX C1 LB 0.6906 |
| #25 GRU solo | GRU V2 Frenet 단독 (α=0, no blend) | OOF +0.0037 / **LB −0.0044** | OOF +0.0159 / LB 추정 약화 | T-AX C1 LB 0.6906 |
| #25 V1 Cart (sanity) | GRU Cart target (V2와 동일 architecture) | OOF −0.0114 vs V2 | OOF −0.0423 vs V2 | EXP #25 V2 (Frenet) |
| #26 XGB solo | XGBoost heterogeneous (T-AX C1 protocol, backbone만 LGBM→XGB, V3 26 feat / Frenet target / W1 axis-wise σ) | OOF +0.0004 / **LB −0.0020** | OOF −0.0025 / major +0.0010 | T-AX C1 LB 0.6906 |
| #26 XGB+LGBM blend | corr 0.9779 (axis 0.97~0.98) → 자동 skip. 선형 근사 LB 추정 ∈ [0.6886, 0.6906] → α=1 (LGBM 단독) 미달 보장 | — / 미제출 | — | T-AX C1 LB 0.6906 |
| #22 Mode A α grid (5³=125) | per-axis multiplicative scaling α ∈ {0.97, 0.985, 1.000, 1.015, 1.030} | OOF best α=(1,1,1) identity (no-op) | — | T-AX C1 OOF 0.6622 |
| #22 Mode B offset δ=−μ | δ = −μ_axis = (+0.498, −0.176, −0.165)mm 단일 적용 | OOF Δo −0.0016 (G1·G2 fail) | major Δ −0.0023 (G2 hard stop fail) | T-AX C1 OOF 0.6622 |
| #22 Mode C (z+0.5mm) | hybrid narrow: α=(1,1,1), δ_mm=(0, 0, +0.50), G1 strict 동률 e17_candidate flag E | OOF +0.0017 / **LB −0.0024** | OOF major +0.0013 / OOF minor +0.0038 | T-AX C1 LB 0.6906 |
| #31 Hard | LGBM clf V3 15 → K10/K15 binary, p>0.5 hard switch (selector 930 sample K10 선택) | OOF −0.0005 (G4 fail) | minor −0.0019, major −0.0003 | K15 OOF 0.6649 |
| #31 Soft | LGBM clf V3 15 → K10/K15 binary, w·K10 + (1−w)·K15 smooth blend | OOF +0.0001 (G4 fail, noise zone) | minor +0.0057, major −0.0010 | K15 OOF 0.6649 |
| #31 Minor→GRU | acc_norm_last ≥ 5 minority_mask → GRU, else K15 (학습 0, cache 재사용 hard rule) | OOF +0.0012 (G4 +0.0020 미달) | minor +0.0076, major +0.0000 | K15 OOF 0.6649 |

**공통 함의**
- 11프레임 → 31 feature 압축이 이미 효율적, 추가 sequence model 정보 추출 한계(#4). minority(acc≥5)는 어느 모델로도 25~30% = task noise floor 가능성(1cm radius). residual MAE ~6mm ≈ radius 10mm → L1로 MAE 줄여도 HR 직접 개선 안 됨 → HR-aware learning(W1) 근거. IMM류 결합은 expert가 zone 우위 명확할 때만 가치(여기선 단일 LGBM이 전 zone 우위).
- **EDA marginal stat(40.55x) ≠ predictive transfer**(D6). **수학적 새 정보 평면 ≠ LGBM 학습 가능 신호**(D6/F/F-INT 3연속, 적분 평면 P1+0.0003→P2−0.0007 W1 conflict). **Cart-optimized gain ranking은 target framework 종속**(Frenet 미전이). **invariant transform augmentation = sample upweighting**(T-AUG).
- **시퀀스 통계(v*/a*_std·path_eff·turn_*)는 majority carrier, point-in-time accel/Frenet은 minority carrier**(#18 S1b Δmajor −0.0028 vs Δminor +0.0013).
- **Base 개선 lever는 LGBM cannibalize 사전 측정 필수**(#12 K-CV/#19 Latency): base +Δ가 P1서 net non-positive로 흡수되면 폐기. **G_S는 strict `>` 대신 no-drop `≥` with tolerance**(#19 v1 버그, v2: G_S1/2/3 tol −0.001/−0.002/−0.005, G_S4 폐기).
- **W1 σ×max_w 라인 ceiling 확정**(#20): C1(4.3,3.5,2.7)·max_w=2.5 interior optimum(overall+minor), σ_L > σ_B ≈ max_w(σ_B/max_w noise floor). σ retune은 base/target 구조 변경 후에만 의미.
- **NN backbone < tree backbone on LB**(#25): GRU solo 0.6862<LGBM 0.6906(Δ −0.0044), NN gap +0.0203 vs LGBM +0.0270(28% 작음)=train fold overfit. 단순 NN lever SOTA 갱신 불가, 큰 NN(TCN/Transformer)은 ROI 더 낮음.
- **blend LB ≈ 두 단독 LB 선형평균**(#25): Mode B 0.4·0.6906+0.6·0.6862=0.6880≈실측 0.6888, 어떤 α도 두 단독 사이 → component 단독 LB ≥ base 필수(LB-OOF gap 사전 측정). **OOF→LB transfer는 backbone family 의존**: Mode B LB 0.6888=V3 multi-seed 정확 일치(axis-wise σ 효과 cancel).
- **Tree-tree heterogeneity 자체로는 diversity 부족**(#26 XGB↔LGBM corr 0.9779): ensemble lever = backbone family 달라야 + component LB ≥ base.
- **OOF micro-positive(Δ<+0.001)는 LB transfer 보장 안 함**(#26 XGB +0.0004→LB −0.0020). **Post-hoc lever는 별개 transfer 메커니즘**(#22 Mode C +0.0017, 두 가드 만족에도 LB −0.0024). 시리즈 3: #17(동질·model param)→+0.0018 success / #26(이질)→fail / #22(동질·post-hoc)→fail. **EXP #17 candidate flag 최종 3조건**: backbone family 동질 + Δo>+0.0010 + model parameter 변경(post-hoc/TTA/blend 제외).

---

## #1 — baseline.ipynb (LightGBM residual) · 2026-05-20

**설정**: physics baseline + LightGBM residual, **feature 28개**, 8000/2000 holdout (seed=42), sample_weight 없음, 축별 3 모델. params regression_l1, lr=0.05, num_leaves=31, min_data_in_leaf=5, max_bin=511. best_iter (x/y/z) = 241/316/417.

**결과 (M1)**

| 지표 | HR |
|---|---|
| 전체 | 0.6485 |
| majority | 0.7247 |
| minority | 0.2516 |

LightGBM이 HGBM(0.6472) 대역 재현. minority는 HGBM no-weight(0.2705)보다 −0.02 낮으나 RF(0.2070)보단 높음. best_iter 축별 편차 큼 (241~417) — z축이 더 깊은 학습 요구.

---

## #2 — feature v2 (31 feat) · 2026-05-20

**설정 (v1 대비)**: 28 → 31 feat. 삭제 4 (`turn_last`, `ax/ay/az_m3`), 추가 7 (`jerk_{x,y,z}_last`, `a_tang_last`, `a_cent_last`, `speed_diff_half`, `turn_mean_half_diff`). 그 외 동일. best_iter 352/145/310.

**결과 (M1)**

| 지표 | HR | Δ vs v1 |
|---|---|---|
| 전체 | 0.6515 | +0.0030 |
| majority | 0.7265 | +0.0018 |
| minority | 0.2609 | +0.0093 |

세 지표 모두 개선. minority가 가장 크게 (+0.0093, 상대 +3.7%) — `jerk`/`a_cent_last` 가설 부분 검증. y축 best_iter 316 → 145 급감 (새 feature가 y축 잔차 빠르게 설명). best_iter 모두 cap(500) 미도달 → n_estimators 증가 무의미.

---

## #3 — feature v3 ablation (21 feat) · 2026-05-22 ✗

**설정 (v2 대비)**: 31 → 21 feat. trajectory-wide 통계 10개 제거 (C 7: `v*_std`, `a*_std`, `path_eff`, E 1: `turn_mean`, H 2: `speed_diff_half`, `turn_mean_half_diff`). best_iter 197/89/298.

**결과 (M1)**

| 지표 | HR | Δ vs v2 |
|---|---|---|
| 전체 | 0.6440 | **−0.0075** |
| majority | 0.7223 | −0.0042 |
| minority | 0.2360 | **−0.0249** |

→ **21개 폐기, 31개 유지**. minority −0.0249가 결정타 — `v*_std`/`a*_std`/`path_eff`/`turn_mean`이 minority 핵심 신호. 마지막 step kinematics만으로는 "현재 급기동 중"은 잡아도 "이 궤적이 평소 얼마나 불안정한가" 컨텍스트를 못 잡음. best_iter 감소 (mean 269 → 195) — feature 정보량 감소만큼 일찍 수렴.

EDA "단기 신호가 장기 신호 압도 (K-window)"는 **외삽 윈도우** 결과였고, 트리 feature로서 통계는 외삽과 다른 차원에서 유효함.

---

## #4 — imm_lstm.ipynb (IMM: LGBM + LSTM gating) · 2026-05-22 ✗

**설정**: physics + LGBM residual + LSTM maneuver + MLP gating. holdout `stratify=minority`, 5-fold OOF, va_idx 2000은 K=5 fold 모델 평균. LSTM multi-seed (42,1,2) × 5fold = 15 model.

**va_idx 결과 (M1)**

| 모델 | 전체 HR | major | minor |
|---|---|---|---|
| physics-only | 0.5915 | — | — |
| **LGBM-only** (5fold 앙상블) | **0.6590** | **0.7276** | **0.2921** |
| E1 single LSTM (vel 3D, h=32) | 0.6005 | 0.6712 | 0.2222 |
| E2 IMM Stage 2 (gating) | 0.6515 | 0.7181 | 0.2952 |
| E2 IMM Stage 3 (fine-tune) | 0.6580 | 0.7270 | 0.2889 |
| E3 hard split @ acc=5 | 0.6315 | — | 0.2571 |

**Stage 1.5 zone 검증 (tr_idx OOF, n=8000)**

| zone | n | LGBM HR | LSTM HR | winner |
|---|---|---|---|---|
| low (acc<2.5) | 4630 | 0.7957 | 0.7773 | LGBM |
| mid (2.5~5) | 2110 | 0.5374 | 0.4521 | LGBM |
| high (≥5) | 1260 | 0.2587 | 0.2421 | **LGBM** |

cos sim 0.4911 (두 예측 다름). **plan_imm §6.3 통과 기준 fail** — "high에서 LSTM이 LGBM 보완" 핵심 전제 부정.

**Gating 학습 패턴** (acc_mag 구간별 gate_normal): [0,2) 0.535 / [2,4) 0.561 / [4,5) 0.575 / [5,8) 0.606 / [8,∞) 0.652 — high acc에서 gate_normal ↑ (가설과 정반대). LAMBDA_GATE 정규화 때문에 LSTM 비중이 0으로 못 가서 noise만 추가. **Stage 3 fine-tune = LGBM 모방으로 수렴**.

**#2와 비교 (LGBM-only 0.6515 vs 0.6590)**: stratify=None vs minority + 단일 vs 5fold 앙상블 차이. 5fold bagging (+0.005) + 다른 va_idx (+0.002~0.003) 추정.

**결정**: IMM 아키텍처 폐기. submission은 LGBM-only로. 31 hand-crafted feat가 11프레임 sequence 정보를 이미 짜내서 LSTM 추가 신호 작음. 6.3mm residual MAE = 1cm hit radius 스케일이라 MAE-기반 L1 학습은 HR 직접 개선 안 됨 → **HR-aware learning (§9) 동기**.

---

## #5 — hr_aware.ipynb (D1: boundary 분석) · 2026-05-23

**설정**: #4 LGBM-only protocol과 동일 (stratify=minority + 5-fold). 5-fold OOF로 residual 분포 분석.

**결과 (tr_idx 8000)**

| 지표 | 값 |
|---|---|
| OOF HR (single-fold) | **0.6430** |
| residual norm mean | 11.90mm |
| residual norm median | 7.28mm |
| residual norm p90 | 24.83mm |
| fold_best_iter mean | 234 (#4와 일치) |

**Boundary 분석 (band [8, 15]mm)**: n_boundary=2165 (27.1%), hits in band=805, **miss in band=1360 ← D2 W1 핵심 타겟**, n_minority_in_band=376 (17.4%, 전체 15.8%보다 ↑).

**D2 진입 3-gate 전부 통과** (OOF ∈ [0.64, 0.67], n_boundary ≥ 500, n_minority ≥ 50).

residual right-skewed (mean > median, p90 25mm) → RESEARCH §9.2 (4) "miss-only emphasis 단독 금지" 정량 근거. boundary band 안 1360 miss가 5~7mm 보정으로 hit 가능 영역.

---

## #6 — hr_aware_d2.ipynb (D2 W1 Gaussian σ ablation) · 2026-05-23 ✓

**설정 (RESEARCH §9.4 W1)**: σ ∈ {0.002, 0.003, 0.005}, `w = clip(1 + 2·exp(-(e_i - r)²/2σ²), 0.5, 3.0)`, r=0.01. weight 입력 = D1 OOF residual norm (in-sample 금지). 3 σ × 5 fold × 3 axis = 45 모델, ~1.5분.

**결과 (va_idx 2000, M1) — 3 σ 전부 §9.3 3-조건 통과**

| σ (mm) | overall | Δ vs baseline | major | minor | boundary | 3-gate |
|---|---|---|---|---|---|---|
| baseline (w=1) | 0.6590 | — | 0.7276 | 0.2921 | 0.3959 | — |
| 2 | 0.6600 | +0.0010 | 0.7282 | 0.2952 | 0.4033 | ✓ |
| **3** ⭐ | **0.6655** | **+0.0065** | **0.7347** | **0.2952** | **0.4217** | ✓ |
| 5 | 0.6640 | +0.0050 | 0.7323 | 0.2984 | 0.4180 | ✓ |

**채택: σ = 3mm**.

**핵심 관찰**
- **3 σ 전부 3-gate 통과** → RESEARCH §9 HR-aware learning 가설 검증
- σ=3mm unimodal peak (σ=2 좁고 가파른, σ=5 넓고 minority 약간 +)
- boundary HR 명확히 개선 (+0.0258, 상대 +6.5%) — W1이 의도대로 boundary push 자원 집중
- minority 망가뜨림 없음 (+0.0031) — RESEARCH §9.3 hard stop 우려 발생 안 함 (minority가 boundary 근처에 충분 분포)
- baseline 5-fold 재현 정합: 셀 9 측정 (0.6590/0.7276/0.2921) = #4 LGBM-only와 비트 단위 일치 → fold·seed 재현성 확인

---

## #7 — hr_aware_d3.ipynb (D3 W2 sigmoid) · 2026-05-23 ✗

**설정**: α ∈ {0.5, 1.0, 2.0}, τ ∈ {0.0015, 0.003, 0.005}. `w = clip(1 + α·sigmoid((e-r)/τ), 0.5, 3.0)`. weight 입력 = D1 OOF residual norm. 9 grid × 5 fold × 3 axis = 135 모델, ~5분.

**결과 (M1) — 9개 전부 §9.3 3-조건 실패, D2 천장 미달**

| α | τ (mm) | overall | Δ vs baseline | minor | g1 | g2 | g3 | pass |
|---|---|---|---|---|---|---|---|---|
| baseline | — | 0.6590 | — | 0.2921 | — | — | — | — |
| D2 best (σ=3) | — | 0.6655 | +0.0065 | 0.2952 | — | — | — | ✓ |
| 0.5 | 1.5 | 0.6560 | −0.0030 | 0.2984 | ✗ | ✗ | ✓ | ✗ |
| 0.5 | 3.0 | 0.6550 | −0.0040 | 0.2857 | ✗ | ✗ | ✗ | ✗ |
| 1.0~2.0 × 3 τ | — | 0.6505~0.6530 | −0.006~−0.009 | 일부 +Δ | ✗ | ✗ | mixed | ✗ |

**채택: 없음. D3 폐기.**

**핵심 관찰** — W1 (D2 통과) vs W2 (D3 실패) 본질적 차이:

```
W1 Gaussian (σ=3, far-miss weight → 1, 자원 절약)
W2 sigmoid    (far-miss weight → 1+α, 자원 낭비)
```

D1 측정: residual right-skewed, **p90 = 25mm** (far-miss 10%). 25mm 떨어진 sample을 1cm 안으로 끌어올 가능성 거의 없음. W2는 far-miss에도 1+α 가중 → 학습 자원이 끌어올 수 없는 sample에 낭비 → boundary push에 쓸 자원 깎임.

→ RESEARCH §9.2 (4) "miss-only emphasis 단독 금지" 정량 확인. **W3 hybrid도 W2 component를 포함하므로 같은 손해 일부 떠안음 → 미실험 폐기**.

---

## #8 — hr_aware_d4.ipynb (D4 W1 정밀 ablation) · 2026-05-23 ✓

**설정 (D3 결과 반영, D4 W3 hybrid → W1 정밀 ablation으로 재정의)**: σ × max_w 12-grid (σ ∈ {2.5, 3.0, 3.5, 4.0}mm × max_w ∈ {2.0, 2.5, 3.0}) + boundary band 3개 평가 ([8,15], [7,14], [9,16]). `w = clip(1 + (max_w-1)·exp(-(e-r)²/2σ²), 0.5, max_w)`. 180 모델, ~6분.

**결과 (M1) — 12-grid 전부 §9.3 3-조건 통과, (3.5, 2.5)가 best**

| σ (mm) | max_w | overall | major | minor | b1 [8,15] | OOF HR | >D2 |
|---|---|---|---|---|---|---|---|
| 2.5 | 2.0 | 0.6615 | 0.7300 | 0.2952 | 0.4070 | 0.6430 | ✗ |
| 3.0 | 3.0 (D2 sanity) | 0.6655 | 0.7347 | 0.2952 | 0.4217 | 0.6416 | ✗ |
| **3.5** | **2.5** ⭐ | **0.6665** | **0.7359** | 0.2952 | **0.4254** | 0.6439 | ✓ |
| 4.0 | 3.0 | 0.6660 | 0.7318 | **0.3143** | 0.4236 | 0.6451 | ✓ |

(전체 12-grid 결과는 git 이전 commit 참조; 위는 핵심 4개)

**채택: (σ, max_w) = (3.5mm, 2.5)**, overall **0.6665** (+0.0010 vs D2, +0.0075 vs baseline).

**핵심 관찰**
- D2 best (σ=3, max_w=3.0) 재현 sanity 완벽 — fold·seed·구현 재현성 확인
- (3.5, 2.5)가 3 boundary band 모두에서 best → robustness 강함
- b3 [9,16]mm 영역 개선폭 최대 (+0.0456) — W1이 boundary 직후 miss를 hit로 끌어오는 효과
- max_w 효과는 σ에 따라 비일관 — 일부는 noise 영역. 12 조합 overall 분산 0.0055에서 +0.0010은 분산 범위 내
- minority 보호 완벽 (12개 전부 baseline 능가/동등, 최고 σ=4.0/max_w=3.0의 0.3143)
- **Diminishing Return 정량 확인**: D2 +0.0065 → D4 +0.0010

---

## #9 — hr_aware_d5.ipynb (D5 W4 axis-wise) · 2026-05-23 ✗ (D4 동률)

**설정 (D5 Hinge MLP → W4 axis-wise scaled weight 재정의)**: D1 OOF에서 `ratio_axis = e_axis²/‖e‖²` (행합=1). V1 단독 `w_axis = 1 + α·ratio_axis`, V2 D4 결합 `w_axis = w_gauss(‖e‖, σ_d4, max_w_d4)·(1 + α·ratio_axis)`. α ∈ {0.5, 1.0, 2.0}, 6 조합 × 5 fold × 3 axis = 90 모델, ~3분.

**중요**: axis 모델별 다른 sample_weight (W1: 모든 axis 동일, W4: axis별 다른 ratio) — 직교 메커니즘.

**결과 (M1)**

| variant | α | overall | Δ vs baseline | major | minor | 3-gate | >D4 |
|---|---|---|---|---|---|---|---|
| baseline | — | 0.6590 | — | 0.7276 | 0.2921 | — | — |
| D4 best | — | 0.6665 | +0.0075 | 0.7359 | 0.2952 | ✓ | — |
| V1 α=0.5 | — | 0.6590 | 0.0000 | 0.7276 | 0.2921 | ✓ | ✗ |
| V1 α=1.0 | — | 0.6575 | −0.0015 | 0.7270 | 0.2857 | ✗ | ✗ |
| V1 α=2.0 | — | 0.6600 | +0.0010 | 0.7300 | 0.2857 | ✗ (minor fail) | ✗ |
| **V2** | **0.5** | **0.6665** | **+0.0075** | 0.7347 | **0.3016** | ✓ | ✗ (동률) |
| V2 α=1.0 | — | 0.6615 | +0.0025 | 0.7300 | 0.2952 | ✓ | ✗ |
| V2 α=2.0 | — | 0.6590 | 0.0000 | 0.7282 | 0.2889 | ✓ | ✗ |

**채택: D4 유지** (D5 best V2 α=0.5는 overall·boundary 완전 동률, minor/major는 ~1샘플 noise).

**핵심 관찰**
- V1 (W4 단독) 무효 — small α에서 ratio_axis term이 거의 항등 (axis 책임 분배 단독으로는 추가 signal 없음)
- V2 (W4 × W1) α 증가에 따라 overall 단조 감소 (W4 multiplicative term이 W1 효과 희석)
- **W 라인 ceiling 0.6665 확정** (M1). D2 +0.0065 → D4 +0.0010 → D5 Δ=0 — Tree+weight 한계 (Cart 한정, EXP #17에서 Frenet representation에서는 재부활)
- W4 메커니즘 한계: LGBM axis별 모델이 이미 자기 축 residual만 fit하므로 책임 비율 정보가 leaf 분기에 추가 기여 못함

→ **D 라인 종료**. 다음 lever: §8.2 Kalman 또는 §8.5 앙상블.

---

## #10 — submission_d4_v3.ipynb (full 10000 재학습) · 2026-05-24 ✓ LB 0.6678

**설정**: D4 채택 파라미터 (σ=3.5mm, max_w=2.5)로 **전체 10000 sample** 5-fold W1 학습 + test 10000 예측. Phase 1 no-weight 5-fold → OOF residual on 10000 → Phase 2 W1 5-fold + test 평균. 30 모델, ~2.5분.

**LB 결과**

| 버전 | 학습 | W1 | 5-fold | LB |
|---|---|---|---|---|
| v1 (#8 D4) | tr_idx 8000 | ✓ | ✓ | 0.664 |
| v2 (함정) | full 10000 | ✗ | ✗ | 0.664 |
| **v3** | **full 10000** | **✓** | **✓** | **0.6678** (+0.0038) |

**v1 vs v3 prediction diff** (10000 test): 동일 row 0/10000, 1cm 이상 다른 row 254 (2.5%), ‖Δpred‖ mean 2.12mm.

**채택: v3 (LB 0.6678)** — D 라인 채택 모델, 후속 lever 평가 baseline.

**핵심 관찰**
- **데이터 volume이 W 라인 sample weight tuning보다 큰 lever** — v3 +0.0038 LB vs D4 → D5 시도(0)
- **va_idx ↔ LB gap ≈ 0.003** (rounding 영역) — va_idx ablation 신뢰성 확인
- **v2 함정**: "전체 10000 재학습"이라고 했지만 W1+5-fold 누락 → LB가 우연히 v1과 같아 보임. cherry-picking 시 모델 구성 명세 확인 필수

**시리즈 갱신**: W 라인 ceiling (M1 0.6665) → 데이터 lever +0.0038 LB → 다음 lever 후보 (D6 feature / Kalman / Multi-seed).

---

## #11 — hr_aware_d6.ipynb (D6 feature engineering) · 2026-05-24 ✗

**설정**: v3 protocol 비트 단위 복제 + feature 4 config (D6.0 baseline 31 / D6.1 +turn_accel / D6.2 +eff_change / D6.3 둘 다 + a_tang_last 제거). full 10000, 5-fold W1 (σ=3.5, max_w=2.5), config간 OOF 공유 금지 (config마다 30 모델 전부 재학습 = 120 모델, ~10분).

**결과 (M2 OOF Phase 2 W1)**

| Config | feat | overall | Δ vs D6.0 | major | minor | 3-gate |
|---|---|---|---|---|---|---|
| D6.0 (= v3) | 31 | **0.6525** | 0 | 0.7223 | 0.2794 | ref |
| D6.1 (+turn_accel) | 32 | 0.6502 | −0.0023 | 0.7215 | 0.2686 | ✗ |
| D6.2 (+eff_change) | 32 | 0.6518 | −0.0007 | 0.7224 | 0.2743 | ✗ |
| D6.3 (둘 다 + −a_tang_last) | 32 | 0.6509 | −0.0016 | 0.7211 | 0.2756 | ✗ |

**채택: 없음. D6 폐기. LB 제출 skip.**

**핵심 관찰**: ① **Karpathy caveat 정량 적중** — "marginal discrimination 40.55x ≠ predictive transfer"(D6.1 −0.0023), "R² +27% = linear upper bound"(D6.2 −0.0007 noise). ② 세 config 모두 minority 손실>majority(D6.1 minor −0.0108 vs major −0.0008) = 신규 feature가 minority 판별에 noise(EDA marginal stat은 통합 분포 차이일 뿐, minority residual 검증 아님). ③ D6.0 OOF 0.6525 vs v3 LB 0.6678 갭 −0.0153 = M2(full OOF) vs M3(test) + va_idx 우호 subset(RESEARCH §8). → **Feature 라인 ceiling**, 향후 feature는 tree-conditional signal + minority residual 직접 ablation 필요.

---

## #12 — hr_aware_k.ipynb (K Kalman base) · 2026-05-24 ✗

**설정**: v3/D6 protocol 비트 단위 + `physics_baseline → kalman_baseline` 교체. K0 = D6.0 캐시 재사용. K1 (Kalman CV 6D, Q_CV=10.745), K2 (Kalman CA 9D, Q_CA=11128.431). **σ 직접 측정**: σ_a = 3.278 m/s², σ_j = 105.491 m/s³ (σ_a/dt 근사 81.95보다 +29% — naive approximation 폐기 정당화).

학습량: K0 캐시 + K1 30 모델 + K2 skip = **30 모델 (~3분)**.

**Sanity gate 4-criterion (base 단독)**

| 지표 | physics | K1 (CV 6D) | K2 (CA 9D) |
|---|---|---|---|
| overall HR | 0.5787 | **0.5981** (+0.0194 O) | 0.3109 (−0.2678 X) |
| major HR | 0.6522 | **0.6662** (+0.0140 O) | 0.3680 (−0.2842 X) |
| minor HR | 0.1854 | **0.2337** (+0.0483 O) | 0.0057 (−0.1797 X) |
| resid std | 16.99mm | 16.84mm (−0.15 O) | 33.53mm (+97% X) |
| 종합 | — | **ALL PASS** | **ALL FAIL** |

- **K2 즉시 폐기** — 11-step 가속도 state 수렴 부족 + forward 2-step의 `0.5·DT_PRED²·a` noisy acc 증폭. RESEARCH §10.7 예상 "K-CA noise 증폭" 정량 적중. Q·R retune skip.
- **K1 sanity 매우 promising** — minority +0.0483은 D 라인 전체에서 본 적 없는 폭. plan §10.5 "sanity 통과 = LGBM 결합 worst case 손실 없음" 가정상 +Δ 확실시.

**LGBM 결합 결과 (M2 Phase 2 W1) — sanity와 정반대**

| Config | sanity | overall | Δ vs K0 | major | minor | 3-gate |
|---|---|---|---|---|---|---|
| K0 (physics, = D6.0) | O | **0.6525** | 0 | 0.7223 | 0.2794 | ref |
| K1 (Kalman CV) | O | 0.6494 | **−0.0031** | 0.7188 | 0.2781 | ✗ (G1·G2 fail) |
| K2 (Kalman CA) | X | — | — | — | — | ✗ (sanity fail) |

**채택: 없음. Kalman 라인 종료. LB 제출 skip.**

**핵심 메커니즘 — Sanity-pass + LGBM-fail (cannibalize)**: LGBM net gain 분해 (gain = base+LGBM − base):

| 지표 | physics gain | K1 gain | Δ gain |
|---|---|---|---|
| overall | +0.0738 | +0.0513 | **−0.0225** |
| major | +0.0701 | +0.0526 | −0.0175 |
| minor | +0.0940 | +0.0444 | **−0.0496** |

→ **Kalman smoothing이 LGBM의 systematic 보정을 base단서 cannibalize**(minority 최대 −0.0496 = Kalman sanity 최대 향상 영역과 정확히 겹침). 근본 원인: 31 feat(v_last·a_last·jerk·acc_norm)가 physics base의 2-point 차분 noise 보정 학습 중인데 Kalman이 v_last→smoothed 교체로 그 bias를 base단서 미리 제거. residual std 거의 불변(16.84 vs 16.99) = 분포 모양 아닌 특정 보정 가능 부분만 잡고 그게 LGBM 영역.

**§10.5 sanity gate 설계 결함**: "sanity 통과 = LGBM worst case 손실 없음"은 반증(LGBM 학습 자체가 base 변경에 영향). **누락 가드 G_S5**: Kalman base no-weight LGBM 5-fold OOF > physics base OOF(0.6430). 향후 base lever 5-criterion 필수. → 단일 변수 lever(W·feature·base 3 라인) 모두 막힘 → 남은 옵션 = Multi-seed/hetero ensemble/새 모델군/**target representation(T 라인)**.

---

## #13 — hr_aware_f.ipynb (F: feature subset Phase A + B) · 2026-05-24 ✗

**설정**: v3 31-feature subset 탐색. F0 protocol: **no-weight Phase 1 5-fold OOF, 3 seed × full 10000** (M2). 9 group: A (recent v/a 8), B (a_w3 4), C (traj stats 7), D (sensor 2), E (turn 3), F (jerk 3), G (acc 분해 2), H (trend 2) = 31. Phase A 8 LOGO Δ 측정, Phase B backward greedy (B-2 criterion `Δo > −combined_std × √iter`, loss-only block, simplicity-first) + A 그룹 hard-lock.

**Phase A 결과 (LOGO)**

| config | n_feat | overall Δ vs F0 | 비고 |
|---|---|---|---|
| F0 | 31 | 0.6494 ± 0.0005 | reference (3 seed std) |
| FA−A | 23 | **−0.0172** | A 압도적 필수, Δminor −0.0059 (G3 fail) |
| FA−H | 29 | **+0.0010** (std 0.0012) | noise zone +Δ |
| FA−B | 27 | **+0.0007** (std 0.0007) | noise zone +Δ |
| FA−C/D/E/F/G | (5개) | strict 통과 0 | noise floor 내부 |

→ 3-gate strict 통과 0/8. A hard-lock + B-2 (B-1 strict로는 0개 제거 예상)로 Phase B 진입.

**Phase B 결과 (B-2 + A lock)**

| 지표 | F0 (31) | best subset (13) | Δ |
|---|---|---|---|
| overall HR | 0.6494 | 0.6475 | **−0.0019** |
| majority HR | 0.7204 | 0.7189 | −0.0016 |
| minority HR | 0.2694 | 0.2658 | **−0.0036** |
| boundary HR | 0.3864 | 0.3900 | +0.0036 |

**Best subset 13 feat**: A 8 (hard-lock) + vy_std (C) + distance_r (D) + va_dot (E) + a_tang_last (G) + speed_diff_half (H). 전멸 그룹 = **B (smoothed accel), F (jerk)**.

**제거 18개**(F0 gain 하위부터, B-2 통과): a_cent_last(1번째) → ay_std → turn_mean → acc_norm_w3 → … → turn_mean_half_diff. 전멸 그룹 = **B(smoothed accel), F(jerk)**. **Phase A↔B order-dependency**(H 그룹): backward greedy gain-순서 path-dependency(turn_mean_half_diff gain 244.4 제거 통과 vs speed_diff_half gain 254.7 늦게 시도돼 lock). **채택 미결정 → T 라인 진입으로 F 폐기**(Δo −0.0019, Δminor −0.0036).

**핵심 사후 검증 (T 라인 진입 후 EXP #14 V4 확인)**: Cartesian gain ranking은 target framework에 종속.

| feature | Phase B Cart gain (rank) | Cart target R² | Frenet target R² |
|---|---|---|---|
| a_cent_last | 161.2 (1/31, 최하위) | ~0.031 | **0.0725 (res_N 1위)** |
| jerk_x_last | 244.0 (17/31) | 0.038 | j_N = 0.070 |
| ax_w3 | 221.6 (13/31) | 0.001 | aw_N = 0.043 |

→ Phase B lean 13 feat은 Cartesian-optimized → Frenet target에 그대로 적용 시 핵심 신호 carrier 미리 제거. **EXP #14 V4 (lean × Frenet)** Phase 2 0.6589 — V0 대비 +0.0067이나 V2/V3보다 −0.004 미달, 정량 확인.

---

## #14 — hr_aware_t.ipynb (T: Frenet target, 5 변형) · 2026-05-25 ✓ LB 0.6878

**설정**: v3 protocol + **Cartesian target → Frenet target**. Frame: `eL = v_last/‖v‖`, `eN = a_perp/‖a_perp‖` (a_perp = a_sm − (a_sm·eL)eL), `eB = eL × eN`. a_sm = w·a[-3:], w=[1,2,3]/6.

**5 변형** (V0~V5, RESEARCH §4 참조). full 10000, 5-fold stratified (seed=42 single 1차), Phase 1 (no-weight) → G_S5_T → Phase 2 (D4 W1 σ=3.5mm, max_w=2.5). 7 Frenet projection: a_N, a_B, j_L, j_N, j_B, aw_L, aw_N (v_L/a_L 중복, v_N/v_B/aw_B 구조적 dead).

**Frame sanity**: inverse round-trip max abs err **1.83e-7**, orthogonality < 1e-6, hard fallback 0/10000.

**residual std (mm)**: Cart 13.79:13.31:9.40 (1:0.96:0.68) vs Frenet 14.74:11.93:9.20 (1:0.81:0.62). RESEARCH §4 진단 1 정확 일치.

**Phase 1 결과 (no-weight 5-fold OOF, seed=42) — G_S5_T**

| 변형 | n_feat | overall | major | minor | G_S5_T (≥0.6430) |
|---|---|---|---|---|---|
| V1 | 31 | 0.6548 | 0.7233 | 0.2883 | O |
| V2 | 38 | 0.6592 | 0.7269 | 0.2971 | O |
| **V3** | 26 | **0.6609** | 0.7276 | **0.3041** | **O** |
| V4 | 13 | 0.6541 | 0.7244 | 0.2781 | O |
| V5 | 20 | 0.6599 | 0.7274 | 0.2990 | O |

- 5/5 G_S5_T 통과 (모두 +0.011~+0.018 margin)
- Phase 1 ordering: V3 > V5 > V2 > V1 > V4 (사전 예측 우선순위 완벽 일치)
- 모든 변형이 EXP #13 F0 (Cart×Cart no-weight 0.6494) 초과 — Frenet target 자체가 no-weight에서 우위

**Phase 2 결과 (D4 W1 σ=3.5/max_w=2.5, seed=42) — 3-gate**

| 변형 | n_feat | overall | Δ vs V0 | major | minor | 3-gate |
|---|---|---|---|---|---|---|
| **V0** (Cart × Cart ref) | 31 | **0.6522** | 0 | 0.7224 | 0.2768 | ref |
| V1 | 31 | 0.6613 | +0.0091 | 0.7291 | 0.2984 | O |
| **V2** | 38 | **0.6630** | **+0.0108** | 0.7293 | **0.3086** | **O** |
| V3 | 26 | 0.6619 | +0.0097 | 0.7283 | 0.3067 | O |
| V4 | 13 | 0.6589 | +0.0067 | 0.7265 | 0.2971 | O |
| V5 | 20 | 0.6613 | +0.0091 | 0.7287 | 0.3010 | O |

- 5/5 3-gate strict 통과 (모든 변형 G1·G2·G3 동시)
- Phase 2 ordering: V2 > V3 > V1 = V5 > V4 (V2·V3 ordering 역전)
- **V0 0.6522 sanity**: EXP #11 D6.0 (M2) 0.6525와 ±0.0003 일치 → protocol 정합 확인. RESEARCH §9.5의 "D 라인 ceiling 0.6665"는 M1 (8k+2k va_idx), 본 V0는 M2 (full 10k OOF) reference
- V3 Phase 1 → 2 minor Δ +0.0026 (포화) — no-weight 단계에서 minority 학습 saturated

**채택: V2** — `submission_t_v2.csv` 생성. **LB 0.6878 (+0.0200 vs v3 0.6678)**.

**LB-OOF gap**: V2 +0.0248 vs V0 hypothetical +0.0156, **+0.0092 확대** — Frenet target 변환이 train→private test generalization 개선.

**시리즈 갱신**: T 라인 단일 lever LB +0.0200 = D 라인 전체 시리즈 (W1+D4+D5) 합계보다 큰 단일 step.

---

## #15 — hr_aware_t_ms.ipynb (T-MS: V0/V2/V3 multi-seed) · 2026-05-25 ✓ LB 0.6888

**설정**: T 노트북 비트 단위 + seeds loop. V0/V2/V3 × seeds {42, 7, 123}. 의사결정 규칙 사전 정의: V2 vs V3 strict 우위 (|Δ| > combined_std × √2) → 승자, overlap → V3 (simplicity tiebreak). 학습량: 3 변형 × 3 seed × 2 phase × 5 fold × 3 axis = **270 model**.

**Multi-seed OOF aggregate (mean ± std)**

| 변형 | n_feat | P1 overall | P2 overall | P2 major | P2 minor |
|---|---|---|---|---|---|
| V0 | 31 | 0.6494 ± 0.0007 | 0.6507 ± 0.0013 | 0.7207 ± 0.0015 | 0.2764 ± 0.0026 |
| V2 | 38 | 0.6601 ± 0.0008 | 0.6618 ± 0.0011 | 0.7296 ± 0.0004 | 0.2995 ± 0.0079 |
| **V3** | **26** | 0.6607 ± 0.0011 | **0.6615 ± 0.0007** | 0.7292 ± 0.0014 | 0.2995 ± 0.0065 |

**Per-seed P2 overall — seed=42 systematic 편향**

| 변형 | seed=42 | seed=7 | seed=123 | spread |
|---|---|---|---|---|
| V0 | 0.6522 | 0.6498 | 0.6502 | 0.0024 |
| V2 | 0.6630 | 0.6609 | 0.6616 | 0.0021 |
| V3 | 0.6619 | 0.6607 | 0.6620 | **0.0013** |

- **seed=42가 V0·V2 best fold split** (V3는 중간). EXP #14 single-seed V2 0.6630 vs V3 0.6619 Δ=+0.0011의 대부분이 split 우연 → multi-seed 평균하면 V2 vs V3 Δ → +0.0003로 축소
- V3 std (0.0007) < V2 std (0.0011) — feature 적은 만큼 LGBM split tie variance 감소

**3-gate strict vs V0 (Bonferroni-lite √2)**

| 변형 | Δoverall | thr | Δmajor | Δminor | G1·G2·G3 |
|---|---|---|---|---|---|
| V2 | +0.0111 | +0.0024 | +0.0089 | +0.0231 | O·O·O |
| V3 | +0.0108 | +0.0021 | +0.0085 | +0.0231 | O·O·O |

**V2 vs V3 직접 비교**: Δ = +0.0003, threshold ±0.0018 → overlap → **V3 채택** (26 vs 38 feat, simplicity tiebreak).

**LB 결과**

| Submission | OOF mean | LB | Δ |
|---|---|---|---|
| **`submission_t_v3_ms3.csv`** | 0.6615 | **0.6888** | **+0.0010** vs V2 single (#14 LB 0.6878) |
| `submission_t_v2v3_ms3.csv` (V2 3 + V3 3 평균) | — | 0.6884 | −0.0004 (동질 ensemble 폐기) |
| `submission_t_v2_ms3.csv` | 0.6618 | (미제출) | — |

- **V3 multi-seed LB 0.6888 = T 라인 신기록** — Karpathy simplicity tiebreak가 OOF뿐 아니라 LB로도 정당
- LB-OOF gap V3 +0.0273 (V2 single +0.0248 대비 +0.0025 확대) — multi-seed가 OOF noise는 줄였지만 train↔private gap은 systematic하게 유지
- **v2v3 hetero ensemble lever 폐기** — V3 ≥ V2 LB이면 균등 평균이 V3 신호 희석

---

## #16 — hr_aware_t_aug.ipynb (T-AUG: augmented data) · 2026-05-25 ✗

**설정**: V3 multi-seed (#15, LB 0.6888) baseline + mispred_aug_v2 (1250 source × 8 augs rotation/mirror_y, mirror_z/noise 제외) 추가 학습. **Augmentation-aware fold split** (source_id 기반): val 원본만, train 원본 tr + (source ∈ tr) aug → 누수 0. 2 configs × 3 seeds = 180 model (~17분).

**Configs**: A (aug_w=1.0 baseline), B (aug_w=0.3 down). W1 weight는 원본 Phase 1 OOF에서만.

**결과 (3 seed mean ± std)**

| Config | aug_w | P1 overall | P2 overall | Δ vs V3 (0.6615) | minor | Δ vs V3 (0.2995) | 3-gate |
|---|---|---|---|---|---|---|---|
| A | 1.0 | 0.6006 ± 0.0020 | **0.6269 ± 0.0007** | **−0.0346** | 0.2372 | −0.0623 | X (all) |
| B | 0.3 | 0.6006 ± 0.0020 | **0.6504 ± 0.0018** | **−0.0111** | 0.2720 | −0.0275 | X (all) |

**채택: 없음. LB skip.**

**진짜 원인 — Frenet basis rotation invariance**: "leakage 의심"은 오해(P1이 V3보다 **낮음** — leakage면 inflate). 실제: V3는 rotation-invariant target+feature(Frenet basis가 trajectory서 계산 → 회전 시 basis도 회전, norms/dot/projection·res_L/N/B 모두 invariant) → 회전/mirror 시 같은 (X,y) pair → **새 training signal 0 = 8x augmentation = 8x sample_weight**(수학: mirror_y는 a_N sign-flip 상쇄, a_B+res_B 동시 flip으로 mapping 보존).

**W1과 충돌 — over-upweighting**: hard source effective weight V3 ~2.5 → B ~5(2x) → A ~10(4x). 모델이 augmented 분포(hard 56%)에 fit → val(원본 hard 15.75%) 하락. A→B(aug_w 1.0→0.3) +0.0235 = upweighting 감소로 회복.

**교훈**: ① 데이터 lever ≠ augmentation lever(#10 +0.0038은 실제 새 trajectory). ② **invariant representation + invariant transform = no-op**(motion-aligned target/feature서 회전·mirror는 mapping 보존). ③ 유효 augmentation = non-invariant transform(noise)·실제 새 trajectory.

---

## #17 — hr_aware_t_ax.ipynb (T-AX: axis-wise W1 σ on Frenet) · 2026-05-25 ✓ LB 0.6906

**설정**: V3 multi-seed (#15, LB 0.6888) baseline + W1 σ 축별 분리. V3 26 feat / Frenet target / W1 D4 (max_w=2.5, peak r=10mm) 비트 단위 유지, **Phase 2 σ만 axis-wise**. 4 configs × 3 seeds = **360 model** (~22분).

**4 configs (Stage 1 σ scan)**

| Config | σ_L | σ_N | σ_B | scaling | 의미 |
|---|---|---|---|---|---|
| V3_ref | 3.5 | 3.5 | 3.5 | uniform | V3 multi-seed 재현 (control + sanity) |
| **C1** | **4.3** | **3.5** | **2.7** | 1:1 std-prop | Frenet std (14.74:11.93:9.20) 비례 |
| C2 | 5.0 | 3.5 | 2.0 | super-prop (~×1.4) | 더 aggressive |
| C3 | 4.0 | 3.5 | 3.0 | sub-prop (~×0.6) | 더 conservative |

**Sanity (V3_ref bit-exact 재현)**: P1 0.6607 ± 0.0011 = EXP #15 V3 ±0.0000 ✓. P2 V3_ref 0.6615 ± 0.0007 = EXP #15 V3 ±0.0000 ✓. Protocol drift 0.

**Multi-seed OOF (3 seed mean ± std)**

| Config | P2 overall | Δ vs V3_ref | P2 major | Δ | P2 minor | Δ |
|---|---|---|---|---|---|---|
| V3_ref | 0.6615 ± 0.0007 | — | 0.7292 | — | 0.2995 | — |
| **C1** | **0.6622 ± 0.0010** | **+0.0007** | 0.7295 | +0.0003 | **0.3022 ± 0.0029** | **+0.0028** |
| C2 | 0.6616 ± 0.0007 | +0.0000 | 0.7288 | −0.0004 | 0.3020 | +0.0025 |
| C3 | 0.6614 ± 0.0010 | −0.0002 | 0.7287 | −0.0006 | 0.3014 | +0.0019 |

**3-gate strict (vs V3_ref, Bonferroni √3)** — 모든 config **G1 strict fail** (Δ threshold 1/3 수준), G2/G3 모두 통과.

**Trend — C1 interior optimum**

```
overall vs V3_ref:
  C3 (sub-prop)   −0.0002  ← 약함
  C2 (super-prop) +0.0000  ← 무의미
  C1 (1:1)        +0.0007  ← 최선 (interior optimum)
```

→ 1:1 std proportional scaling 가설 directional 확정. Stage 2 boundary 확장 불필요.

**Minor HR 일관된 +Δ** — axis-wise σ가 minority에 selectively 효과 (3 config 모두 +0.0019~+0.0028). Broader σ_L이 minority의 큰 residual error 영역을 weight curve로 포착, sharper σ_B가 작은 z축 변동에 정확 매칭.

**LB 검증 — 채택**

OOF strict G1 fail이었으나 T 라인 LB-OOF gap 일관성 (V2 +0.0248, V3 +0.0273) 신뢰하여 C1 LB 제출:

| Submission | OOF | LB | gap | Δ vs V3 LB |
|---|---|---|---|---|
| V3 multi-seed (EXP #15) | 0.6615 | 0.6888 | +0.0273 | — |
| **C1 axis-wise (EXP #17)** | **0.6622** | **0.6906** | **+0.0284** | **+0.0018** |

- **LB +0.0018 = OOF Δ +0.0007의 2.6배 lift** — gap이 C1에서 살짝 더 확장
- LB-OOF gap 일관성 (+0.0273 → +0.0284) → axis-wise σ lever가 진짜 signal (OOF artifact 아님)
- **C1 strict G1 미달 (OOF)에도 LB validation 통과** — "verify before commit" 원칙 작동

**채택: C1 (당시 SOTA, LB 0.6906 — 현 채택은 #38 rotgate)**

**메커니즘 (RESEARCH §6 W5 자산)**: ① Frenet residual std 비대칭(1:0.81:0.62)이 axis-wise σ 유효의 결정 인자(Cart D5 비대칭 1:0.96:0.68 작아 D4 동률 vs Frenet C1 LB +0.0018). ② 1:1 std-proportional sweet spot(C2 1.4x flat / C3 0.6x 약함). ③ minority-selective(major ±미세, minor +0.0028). ④ T 라인 LB-OOF gap +0.027 일관 → OOF strict pass 없어도 LB 검증 cost 낮으면 시도 가치.

---

## #18 — hr_aware_f_int.ipynb (F-INT: internal displacement) · 2026-05-25 ✗

**설정**: T-AX C1 (#17, LB 0.6906) baseline 위 feature set 단일 변수 변경. C1 σ=(4.3, 3.5, 2.7)mm·max_w=2.5·peak r=10mm 고정. 3 configs × 3 seeds = **270 model** (~17분).

**가설 H_FINT**: motion-aligned axis의 **적분 평면** 신호 추가 → LGBM 학습 신호 강화. 기존 26 feat은 순간(`speed_last`) + 변동(`*_std`, `path_eff`, `speed_diff_half`) + 미분(`a_*`, `j_*`) 평면만 cover, 적분 축이 비어있음.

**Configs (3)**

| Config | n_feat | 정의 | 의미 |
|---|---|---|---|
| V3_ref | 26 | V3 26 feat 그대로 | T-AX C1 재현 (control + sanity) |
| **S1a** | 28 | V3 + `iL_mean` + `iN_mean` | 추가만 (single-variable touch) |
| **S1b** | 11 | 9 lean (RESEARCH §4 #1,2,3,14,16,17,20,25,26) + `iL_mean` + `iN_mean` | 17 제거 + 2 추가 (multi-variable touch) |

**Internal displacement 정의** (Step 1 추가 후보)

- `internal[i] = p[i+2] − p[i]`, i = 0..8 (80ms 간격, 9 vectors)
- `iL_mean = (1/9)·Σ_i internal[i] · eL_last` — window 평균 변위의 진행축 성분
- `iN_mean = (1/9)·Σ_i internal[i] · eN_last` — window 평균 변위의 회전축 성분
- 텔레스코프 정리 sanity: `iL_mean = [(p[9]+p[10])−(p[0]+p[1])]·eL_last / 9` (assert max abs err < 1e-5, 실측 1e-17)

**Sanity (V3_ref bit-exact 재현)**: P1 0.6607 ± 0.0011 = T-AX C1 reference ±0.0000 ✓. P2 V3_ref 0.6622 ± 0.0010 = T-AX C1 reference ±0.0000 ✓. Protocol drift 0.

**Multi-seed OOF (3 seed mean ± std)**

| Config | n | P1 | P2 overall | Δ vs V3_ref | P2 major | Δ | P2 minor | Δ |
|---|---|---|---|---|---|---|---|---|
| V3_ref | 26 | 0.6607 ± 0.0011 | 0.6622 ± 0.0010 | — | 0.7295 | — | 0.3022 | — |
| **S1a** | 28 | **0.6610 ± 0.0012** | 0.6615 ± 0.0013 | **−0.0007** | 0.7293 | −0.0002 | 0.2988 | −0.0034 |
| **S1b** | 11 | 0.6573 ± 0.0006 | 0.6601 ± 0.0011 | **−0.0021** | 0.7267 | −0.0028 | **0.3035** | **+0.0013** |

**3-gate strict (vs V3_ref, Bonferroni √2)**

| Config | G1 (Δ vs +0.0022 thr) | G2 (Δmajor > −0.002) | G3 (Δminor > −0.005) | pass |
|---|---|---|---|---|
| S1a | X (−0.0007) | O | O (−0.0034) | **X** |
| S1b | X (−0.0021) | **X (−0.0028)** | O (+0.0013) | **X** |

**채택: 없음. H_FINT 부정. F-INT 라인 폐기.**

**핵심 Findings + 교훈 (재사용 자산)**

1. **Phase 1→Phase 2 interaction(가장 중요)**: S1a는 P1에서 +0.0003 micro-positive인데 P2 axis-wise W1 후 −0.0007 역전(ΔP2−P1: V3_ref +0.0015 → S1a +0.0005, 1/3). `iL_mean`이 `speed_last`와 약한 redundancy → boundary(e≈10mm) weight 증폭 시 over-correction. **feature 평가는 P1 단독 부족, W1 적용 후 lift까지 봐야 함**(P1+Δ→P2−Δ 첫 사례). per-seed range(0.0025)가 |Δ|보다 큼 — noise zone.
2. **S1b major/minor 비대칭**: Δmajor **−0.0028(G2 fail)** = 시퀀스 통계가 majority 핵심 carrier, Δminor +0.0013 = minority는 lean에서 약간 이득. **EXP #3 결론 부분 수정**(majority가 시퀀스 통계에 더 의존, EXP #3는 W1 등장 전 측정). multi-touch(17 제거+2 추가)라 분리 불가.
3. **수학적 새 정보 평면 ≠ LGBM 학습 가능 신호**(#11 D6/#13 F/#18 3번째 동일 패턴): 440ms 윈도우 평균 변위는 speed_last+speed_diff_half+path_eff로 implicit reconstruct 가능, tree split 활용 conditional signal 아님. univariate stat만으론 채택 금지, single-variable 원칙 유지.

---

## #19 — hr_aware_latency_e19{,_retune}.ipynb (L2 Latency time_scale scan) · 2026-05-26 ✗

**설정**: T-AX C1 (#17, LB 0.6906) baseline 위 **base_pred 변경 단일 변수**. `base_pred(s) = p[-1] + v_last · DT_PRED · s`, s ∈ {0.85, 0.90, 0.92, 0.95, 0.98, 1.00, 1.02, 1.05, 1.08, 1.10, 1.15} (11개). 외부 PB 0.6822 솔루션 차용 (`time_scale ∈ [0.85, 1.15]` 명시).

**가설 H_L1**: LiDAR 센서 지연이 0보다 크면 best_s ≠ 1.00. T-AX C1은 s=1.00 (0ms 지연 가정).

**위험 (사전 식별)**: EXP #12 K-CV cannibalize 패턴 재발 가능. G_S 5-gate sanity 필수, 특히 G_S5 (P1 OOF ≥ 0.6430).

### v1 — G_S strict protocol 버그 발견

v1 초기 G_S threshold strict `>`:
- G_S1: HR > 0.5788 / G_S2: major > 0.6523 / G_S3: minor > 0.1854
- G_S4: resid_std ≤ phys_resid_std (16.99mm)

**문제**: s=1.00 control 자체가 fail (HR=0.5787 < 0.5788, microscopic epsilon). G_S4도 s=1.02처럼 HR 향상 + std 증가 케이스를 차단.

**G_S1~4 통과 s**: 0/11. ⚠️ 모든 s fail → 즉시 진단: protocol 버그.

→ v2 재정의 (HR-primary tolerance).

### v2 — G_S 재정의 후 재실행

G_S threshold:
- G_S1: HR ≥ **0.5778** (tolerance −0.001, 부동소수점 + 측정 noise margin)
- G_S2: major ≥ **0.6503** (3-gate G2와 통일 −0.002)
- G_S3: minor ≥ **0.1804** (3-gate G3와 통일 −0.005)
- **G_S4 폐기** (HR vs std trade-off로 false-negative)
- G_S5: P1 OOF ≥ 0.6430 (유지)

**G_S1~3 (v2) 통과 s** (3/11): {0.98, 1.00, 1.02}

| s | base HR | major | minor | resid_std (mm) | G_S1-3 |
|---|---|---|---|---|---|
| 0.85 | 0.4453 | 0.4976 | 0.1657 | 15.43 | XXX |
| 0.90 | 0.4960 | 0.5548 | 0.1816 | 15.88 | XXO |
| 0.92 | 0.5290 | 0.5941 | 0.1810 | 16.09 | XXO |
| 0.95 | 0.5622 | 0.6309 | 0.1949 | 16.43 | XXO |
| **0.98** | 0.5789 | 0.6503 | 0.1968 | 16.78 | OOO PASS |
| **1.00** | 0.5787 | 0.6522 | 0.1854 | 16.99 | OOO PASS (control) |
| **1.02** | 0.5808 | 0.6542 | 0.1879 | 17.18 | OOO PASS |
| 1.05 | 0.5662 | 0.6368 | 0.1886 | 17.42 | XXO |
| 1.08~1.15 | 0.4254~0.5294 | <0.6 | <0.18 | >17.6 | XXX |

**best_s = 1.02** (P1 OOF max, 3 candidate 중).

**Phase 2 full (3 seed × T-AX C1 axis-wise σ)**

| 지표 | E19 best_s=1.02 | T-AX C1 (s=1.0 ref) | Δ |
|---|---|---|---|
| P1 OOF | 0.6605 ± 0.0009 | 0.6607 | **−0.0002** (net non-positive) |
| P2 overall | 0.6604 ± 0.0014 | 0.6622 ± 0.0010 | **−0.0018** |
| P2 major | 0.7285 ± 0.0008 | 0.7295 | −0.0010 |
| P2 minor | 0.2963 ± 0.0047 | 0.3022 | **−0.0059 (hard stop fail)** |

**3-gate vs C1 reference (Bonferroni √2)**

| Gate | 기준 | 결과 |
|---|---|---|
| G1 overall | Δ > combined_std × √2 ≈ +0.0024 | X (Δ −0.0018) |
| G2 major | Δ > −0.002 | O (Δ −0.0010) |
| G3 minor | Δ > −0.005 | **X (Δ −0.0059, hard stop fail)** |

LB 시도 안 함.

### 옵션 C — σ-retune (s=1.02 raw std 재측정 후 새 σ)

**가설 H_L1b**: T-AX C1 σ=(4.3, 3.5, 2.7)는 s=1.00 raw residual std (L:N:B=14.74:11.93:9.20mm) 기반. s=1.02에서 std 비율 다르면 σ 미스매치 → 미스매치 해소 시 v2 결과 회복 가능?

**s=1.02 raw residual std 측정**

| axis | s=1.02 (mm) | s=1.00 (mm) | Δ |
|---|---|---|---|
| L | 14.82 | 14.74 | +0.08 |
| N | 11.93 | 11.93 | −0.00 |
| B | 9.20 | 9.20 | −0.00 |
| 비율 L:N:B | **1 : 0.805 : 0.620** | **1 : 0.809 : 0.624** | < 0.005 |

**새 σ (1:1 std-prop on s=1.02)**

| σ | retune (mm) | C1 (mm) | Δ |
|---|---|---|---|
| σ_L | 4.35 | 4.3 | +0.05 |
| σ_N | 3.50 (anchor) | 3.5 | 0 |
| σ_B | 2.70 | 2.7 | −0.00 |
| max \|Δσ\| | **0.05mm** | — | < 0.1mm threshold |

→ **σ 미스매치 가설 부정 정량 확인**. s=1.02 base는 모든 axis residual을 거의 비례 증가시킬 뿐 → σ scaling 보존.

**Phase 2 with new σ (3 seed)**

| 지표 | retune | v2 (C1 σ) | C1 ref | Δ vs v2 | Δ vs C1 |
|---|---|---|---|---|---|
| P1 OOF | 0.6605 ± 0.0009 | 0.6605 ± 0.0009 | 0.6607 | +0.0000 | −0.0002 |
| P2 overall | 0.6609 ± 0.0025 | 0.6604 ± 0.0014 | 0.6622 | **+0.0005** | −0.0013 |
| P2 major | 0.7289 ± 0.0011 | 0.7285 ± 0.0008 | 0.7295 | +0.0004 | −0.0006 |
| P2 minor | 0.2967 ± 0.0103 | 0.2963 ± 0.0047 | 0.3022 | +0.0004 | **−0.0055 (hard stop fail)** |

retune은 v2 대비 noise zone 내 변화 (+0.0005 << seed std 0.0025).

**3-gate vs C1 (retune)**: G1 X (Δ −0.0013 < +0.0038), G2 O (Δ −0.0006), G3 **X (Δ −0.0055)**. LB 시도 안 함.

### Findings + 교훈 (재사용 자산)

1. **Base 개선이 LGBM에 cannibalize 정량(#12 K-CV 재발)**: s=1.02 base 단독 HR +0.0021 → P1 OOF −0.0002(완전 흡수) → P2 −0.0018(음수 전환). V3 26의 v/a/j_last·a_tang·a_cent가 v_last×DT_PRED 외삽 bias 보정 학습 중인데 s=1.02가 base단서 미리 흡수. #12 K1(base +0.0194→net −0.0031)과 동형.
2. **σ scaling은 latency lever에 robust**: s=1.02 raw std 비율(1:0.805:0.620)≈s=1.00(1:0.809:0.624), 새 σ 0.05mm 차이(<0.1mm) → **σ 미스매치는 실패 원인 아님, cannibalize가 본질**.
3. **G_S strict `>` protocol 버그**: v1이 s=1.00 control을 microscopic epsilon(0.5787 vs 0.5788)으로 fail → v2(tolerance `≥` + G_S4 폐기)로 재정의. G_S는 strict 대신 no-drop with tolerance.
4. **Phase 1 net non-positive = 즉시 폐기 신호**: G_S5(절댓값 0.6430) 통과해도 P1 Δ −0.0002면 cannibalize → **G_S6(P1 Δ vs baseline > +tol) 추가 권장**(Phase 2 sunk cost 차단).
5. **외부 솔루션 차용 시 단일 변수 transferability 검증**: PB 0.6822는 latency를 candidate selector 입력으로 사용(다중), 단일 base 변경으로 격리하면 cannibalize → 결합 구조 전체 또는 완전 격리 둘 중 선택. (base 변경 lever는 #25 GRU 등 hetero model과 함께면 재고 가능.)

---

## #20 — hr_aware_sigma_e20.ipynb (Stage 2-a σ × max_w narrow grid) · 2026-05-26 ✗

**설정**: T-AX C1 (#17, LB 0.6906) baseline 위 W1 σ × max_w 단일 변수 좁은 grid. V3 26 feat / Frenet target / Phase 1 no-weight (3 seed 캐시 공유) 비트 단위 유지, Phase 2 W1 schedule 7 config 1-D ablation. 7 config × 3 seed × 2 phase × 5 fold × 3 axis = **315 model + Phase 1 cache 45 model (~22분)**.

**가설 H_S2**: C1 (σ_L=4.3, σ_N=3.5, σ_B=2.7, max_w=2.5)는 EXP #17 Stage 1 4-config 결과로 ±40~50% 범위 interior optimum 확인. ±10~20% narrow grid는 미시도 — Phase 2 W1 정밀화로 marginal +0.0010~+0.0020 가능.

**7 configs (1-D ablation)**

| Config | σ_L | σ_N | σ_B | max_w | perturbation |
|---|---|---|---|---|---|
| **C1** (baseline) | 4.3 | 3.5 | 2.7 | 2.5 | sanity 재현 |
| sL_4.0 | **4.0** | 3.5 | 2.7 | 2.5 | σ_L −7% |
| sL_4.7 | **4.7** | 3.5 | 2.7 | 2.5 | σ_L +9% |
| sB_2.3 | 4.3 | 3.5 | **2.3** | 2.5 | σ_B −15% |
| sB_3.0 | 4.3 | 3.5 | **3.0** | 2.5 | σ_B +11% |
| mw_2.0 | 4.3 | 3.5 | 2.7 | **2.0** | max_w −20% |
| mw_3.0 | 4.3 | 3.5 | 2.7 | **3.0** | max_w +20% |

**Sanity (C1 bit-exact 재현)**: C1 overall **0.6622 ± 0.0010** = EXP #17 T-AX C1 ±0.0000 ✓. Protocol drift 0. P1 mean 0.6605~ = EXP #17 C1 reference (0.6607) ±0.0002 (3 seed × 5 fold 공유 캐시 정합).

**Multi-seed OOF (3 seed mean ± std)**

| Config | σ (mm) | max_w | P2 overall | Δ vs C1 | P2 major | Δ | P2 minor | Δ |
|---|---|---|---|---|---|---|---|---|
| **C1** | (4.3,3.5,2.7) | 2.5 | **0.6622 ± 0.0010** | — | 0.7295 | — | 0.3022 | — |
| sL_4.0 | (4.0,3.5,2.7) | 2.5 | 0.6611 ± 0.0012 | **−0.0011** | 0.7283 | −0.0012 | 0.3016 | −0.0006 |
| sL_4.7 | (4.7,3.5,2.7) | 2.5 | 0.6617 ± 0.0005 | −0.0005 | **0.7296** | **+0.0001** | 0.2986 | **−0.0036** |
| sB_2.3 | (4.3,3.5,2.3) | 2.5 | **0.6619 ± 0.0005** | **−0.0003** | 0.7294 | −0.0001 | 0.3012 | −0.0011 |
| sB_3.0 | (4.3,3.5,3.0) | 2.5 | 0.6617 ± 0.0004 | −0.0005 | 0.7291 | −0.0004 | 0.3012 | −0.0011 |
| mw_2.0 | (4.3,3.5,2.7) | 2.0 | 0.6617 ± 0.0017 | −0.0005 | 0.7293 | −0.0002 | 0.3001 | −0.0021 |
| mw_3.0 | (4.3,3.5,2.7) | 3.0 | 0.6615 ± 0.0003 | −0.0007 | 0.7289 | −0.0006 | 0.3010 | −0.0013 |

**3-gate strict (vs C1, Bonferroni √6=2.449)** — 6 variants 전부 G1 fail, G2/G3 strict 통과, **e17_candidate 0** (Δoverall > 0 만족 0).

| Config | Δoverall | G1 thr | G1 | G2 (>−0.002) | G3 (>−0.005) | flag |
|---|---|---|---|---|---|---|
| sL_4.0 | −0.0011 | +0.0038 | X | O | O | X |
| sL_4.7 | −0.0005 | +0.0026 | X | O | O | X |
| sB_2.3 | −0.0003 | +0.0026 | X | O | O | X |
| sB_3.0 | −0.0005 | +0.0025 | X | O | O | X |
| mw_2.0 | −0.0005 | +0.0047 | X | O | O | X |
| mw_3.0 | −0.0007 | +0.0025 | X | O | O | X |

**채택: 없음. Stage 2-a 폐기. LB 제출 skip.**

### Findings

**Finding 1 — C1 interior optimum 양방향 재확인 (가설 H_S2 부정)**

EXP #17 Stage 1은 ±40~50% (C2 super-prop ×1.4, C3 sub-prop ×0.6) coarse grid에서 C1 +0.0007 우위 확인. 본 실험은 ±10~20% narrow grid에서 6 perturbation 전부 Δoverall < 0:

```
σ_L axis:   sL_4.0 (−7%)  −0.0011  ←  sL_4.7 (+9%)  −0.0005   asymmetric (down side 더 가파름)
σ_B axis:   sB_2.3 (−15%) −0.0003  ↔  sB_3.0 (+11%) −0.0005   symmetric flat
max_w axis: mw_2.0 (−20%) −0.0005  ↔  mw_3.0 (+20%) −0.0007   symmetric flat
```

→ C1 (4.3, 3.5, 2.7)·max_w=2.5는 **σ×max_w 평면 4-D smooth local minimum** (overall에서). marginal +0.0010~+0.0020 가능성 부정. RESEARCH §6 W5 (1:1 std-proportional sweet spot)에 σ_B/max_w robustness 추가.

**Finding 2~4 + 교훈 (재사용 자산)**

2. **민감도 ranking σ_L > σ_B ≈ max_w**: ±perturbation mean|Δo| σ_L 0.0008(active, asymmetric) / σ_B 0.0004 / max_w 0.0006(둘 다 noise floor, ±20%도 flat). **σ_L이 W1 dominant tuning parameter**(Frenet std L 14.74mm 최대, weight curve 폭 큰 axis에서 σ 영향 최대).
3. **Minor HR도 C1 optimum**(6/6 Δminor≤0): sL_4.7 Δminor −0.0036(σ_L 확대가 far-miss까지 weight↑→over-correction). overall·minor 동시 C1 optimum, EXP #17 "minority-selective"는 ±40~50% scale 1차 효과(±10%선 minor가 overall보다 약간 sensitive).
4. **Bonferroni √6 threshold가 noise floor 안(실험 설계 결함)**: 3 seed std 0.0005~0.0017 → √6 threshold 0.0024~0.0047, ±0.001 effect 검출 불가. 단 모든 variant Δo≤−0.0003이라 데이터 자체가 "C1=optimum" 지지.

**교훈**: ① **W1 σ×max_w 라인 ceiling 확정**(D2 σ{2,3,5}→D4 12-grid→T-AX axis-wise→E20 narrow, 4차 — C1 채택 확정). ② σ_L이 dominant(σ_B/max_w ±20% flat). ③ narrow grid는 power 사전 계산 필수(3 seed×√6은 ±0.001 검출 불가). ④ W1/feature/base 3 라인 모두 saturate → hetero model/ensemble로 전환.

**산출물**: `results/sigma_e20_*`(meta·oof·test, submission 미생성).

---

## #25 — hr_aware_gru_e25.ipynb (LGBM + GRU residual ensemble, 시나리오 2 핵심) · 2026-05-27 ✗

**설정**: T-AX C1(LB 0.6906) LGBM 캐시 + 신규 GRU branch → Cart α blend. 시나리오 2(hetero backbone) 첫 진입(#21~#24 skip). 가설: tree+sequential inductive bias 차이로 blend +0.003~0.008. Variants V2 Frenet(main, target res_L/N/B) + V1 Cart(sanity). GRU(input 11×9 + scalar V3 26, hidden=64 1L → fc 128→64 → head, tanh ±5cm cap, 34.5K params), Loss=euclid+0.3·softhit. 3 seed×5 fold×2 variant=30 model(실측 4.5분, AdamW lr5e-4 ES30 full determinism).

**Phase 1 — V2 Frenet GRU OOF** (target = residual_fren_train)

| seed | OOF overall | major | minor | mean_ep | sec |
|---|---|---|---|---|---|
| 42 | 0.6673 | 0.7322 | 0.3200 | 50 | 46s |
| 7 | 0.6646 | 0.7304 | 0.3124 | 46 | 36s |
| 123 | 0.6645 | 0.7290 | 0.3194 | 46 | 34s |
| **mean** | **0.6655 ± 0.0016** | **0.7306** | **0.3172** | 47 | — |

**Phase 1 — V1 Cart GRU OOF** (target = residual_cart_train, sanity)

| seed | OOF overall | major | minor | mean_ep |
|---|---|---|---|---|
| 42 | 0.6528 | 0.7234 | 0.2749 | 66 |
| 7 | 0.6494 | 0.7189 | 0.2775 | 72 |
| 123 | 0.6503 | 0.7209 | 0.2724 | 67 |
| **mean** | **0.6508 ± 0.0018** | **0.7211** | **0.2749** | 68 |

**ΔV2 − V1 = +0.0146 OOF** — Frenet target 우위가 GRU backbone에도 transfer 확인 (LGBM +0.0108보다 약간 큼, 같은 방향). plan §4.7 risk "Frenet × NN backbone 부적합" 부정. **Cap reach 0.00%** (60 fold 전부) — ±5cm cap 무효 검증.

**Phase 2 α blend** (T-AX C1 LGBM OOF + V2 Frenet GRU OOF Frenet→Cart inverse)

LGBM 단독 OOF (3 seed mean as single OOF) = **0.6636**, GRU 단독 OOF = **0.6659** (GRU > LGBM by +0.0023 on single OOF).

| Mode | grid | best α | OOF overall | Δ vs C1 (0.6622) | OOF major | OOF minor |
|---|---|---|---|---|---|---|
| A | 21 (scalar 0.05 step) | α=0.45 | 0.6689 | +0.0067 | 0.7333 | 0.3244 |
| **B** | 343 (axis³ 7×7×7) | α=[0.5, 0.2, 0.4] | **0.6696** | **+0.0074** | **0.7346** | 0.3219 |
| C | 1 (α=0.5 fixed) | α=0.5 | 0.6686 | +0.0064 | 0.7335 | 0.3213 |

**Phase 3 3-gate (vs T-AX C1, Bonferroni √3)** — 모든 Mode strict pass3

| Mode | Δo (thr +0.0033) | Δmaj (>−0.002) | Δmin (>−0.005) | G1·G2·G3 |
|---|---|---|---|---|
| A | +0.0067 | +0.0038 | **+0.0222** | O·O·O |
| **B** | **+0.0074** | +0.0051 | +0.0197 | O·O·O |
| C | +0.0064 | +0.0040 | +0.0191 | O·O·O |

→ **D2 W1 (#6) 이후 13 EXP만에 3-gate strict pass3** (게다가 3 mode 동시). Δminor +0.019~+0.022는 V2 → V3 전환 이후 최대 minority gain. 채택 **Mode B**.

**Phase 4 LB 실측 (3 submission)**

| 모델 | OOF | LB | LB−OOF gap | Δ vs T-AX C1 LB |
|---|---|---|---|---|
| **T-AX C1** (LGBM only, reference) | 0.6622 | **0.6906** | +0.0284 | 0 |
| Mode B blend (axis-wise α=[0.5,0.2,0.4]) | 0.6696 | 0.6888 | +0.0192 | **−0.0018** |
| **GRU solo** (α=0, V2 Frenet 15 model 평균) | 0.6659 | **0.6862** | **+0.0203** | **−0.0044** |

**채택: 없음. EXP #25 ensemble lever 폐기 확정. T-AX C1 SOTA 유지.**

### Finding 1 — Mode B LB = V3 multi-seed LB 정확 일치 (가장 강한 단일 정량)

```
EXP #15 V3 multi-seed (LGBM + Frenet target + W1 uniform σ=3.5)   LB 0.6888
EXP #25 Mode B blend (T-AX C1 + GRU axis-wise α=[0.5, 0.2, 0.4])  LB 0.6888
                                                                   ─────────
                                                                   Δ = 0.0000
```

T-AX C1 (LB 0.6906)이 V3 multi-seed 위에서 얻은 **axis-wise σ 효과 +0.0018**이 GRU blend로 정확히 cancel. GRU branch는 W1 schedule 없는 모델이라 Phase 2 axis-wise σ minority-selective 효과 (EXP #17 Finding 3) 평균 시 희석. **RESEARCH.md §6 W5 메커니즘 (Frenet std 비대칭이 axis-wise σ 효과의 근원)** 정량 재확인.

### Finding 2 — Blend LB는 두 단독 모델 LB의 선형 평균 근사

Mode B α=[0.5, 0.2, 0.4] axis별 가중치 → 평균 ≈ 0.4 (LGBM) + 0.6 (GRU). 단순 선형 예측:
```
예측 = 0.4 × LGBM_LB + 0.6 × GRU_LB = 0.4 × 0.6906 + 0.6 × 0.6862 = 0.6880
실측 Mode B LB                                                       = 0.6888
                                                                        ─────
                                                                    Δ = +0.0008 (±0.001 오차)
```

→ blend는 LB 수준에서 **두 component 단독 LB의 선형 가중 평균**과 거의 같음. axis별 비선형 효과 없음.

**함의**: GRU 단독 LB (0.6862) < LGBM 단독 LB (0.6906)이므로 **어떤 α∈[0, 1] blend도 LGBM 단독을 넘을 수 없음** (선형 평균은 항상 두 단점 사이). Mode A α=0.45 LB 예측 = 0.45·0.6906 + 0.55·0.6862 ≈ 0.6881, Mode C α=0.5 ≈ 0.6884. **EXP #25 ensemble lever 구조적으로 dead**.

### 핵심 Findings (계속) + 교훈 (재사용 자산)

3. **NN gap +0.0203 < LGBM gap +0.0270~+0.0284(28%↓)**: GRU OOF 0.6659는 LGBM급이나 test(0.6862) 못 따라감 → **단순 GRU(64)<LGBM tree** 확정(34K params train-fold sensitive + 11-frame seq 짧음 + 저자도 GRU 0.6780<<LGBM).
4. **Frenet target NN transfer +0.0146**(LGBM +0.0108보다 큼) → "Frenet×NN 부적합" 가설 부정. 향후 NN lever는 Frenet target 유지 + 더 큰 backbone(TCN/Transformer) 권장.
5. **OOF strict 3-gate pass ≠ LB transfer 보장**(강한 반례): #17 C1 G1 fail이나 gap 일관→LB +0.0018, #25 Mode B G1 pass(+0.0074)나 LB −0.0018(gap 0.0092 축소). → **backbone family 동질성 가드 추가**(component 단독 LB ≥ base 사전 측정).
6. **GRU solo OOF→LB transfer 합리적**(gap +0.0203=LGBM의 75%): NN이 완전 못 일반화가 아니라 tree보다 약간 못함 — 31 feat가 sequence 정보 이미 압축(#4 IMM 재확인), 더 강력 NN ROI 낮음. **시나리오 2 1순위 #26 tree-tree로 이동**.

**산출물**: `hr_aware_gru_e25.ipynb`, `submission_e25_lgbm_gru_blend.csv`(LB 0.6888)/`_gru_solo.csv`(0.6862), GRU OOF/test cache 12개.

---

## #26 — hr_aware_xgb_e26.ipynb (XGBoost heterogeneous backbone, 시나리오 2 차기) · 2026-05-27 ✗

**설정**: T-AX C1 (#17, LB 0.6906) protocol **비트 단위 동일** (V3 26 feat / Frenet target / Phase 1 no-weight → Phase 2 W1 axis-wise σ=(4.3, 3.5, 2.7)mm·max_w=2.5 / 3 seed × 5 fold × 3 axis), backbone만 **LightGBM → XGBoost** 교체. plan §4.8 사용자 결정 (2026-05-27): #25 NN backbone fail 후 tree-tree heterogeneity가 시나리오 2 1순위.

**가설 H_E2**: XGB (boosting + hist + depthwise) vs LGBM (boosting + GOSS + leaf-wise) — tree family이지만 알고리즘 차이로 prediction diversity 존재. **gap 동질성** (#25 NN gap +0.0203 << LGBM +0.0270 문제 회피) + **prediction diversity** → ensemble +0.0010~+0.0030 LB.

**XGB hyperparam (사용자 확정 2026-05-27, 대중적 + LGBM 일치)**: objective=`reg:absoluteerror` (L1), tree_method=`hist`, max_bin=511, lr=0.05, max_depth=6 depthwise, min_child_weight=5 (≈ LGBM min_data_in_leaf=5), subsample=0.9, colsample_bytree=0.9, reg_lambda=1.0, n_estimators=2000, early_stopping_rounds=50. xgboost 2.x API.

학습량: 2 phase × 3 seed × 5 fold × 3 axis = **90 model**, 실측 약 5분 (P1 + P2).

**Phase 1 — no-weight 5-fold OOF (G_S5 sanity)**

| seed | P1 overall | 시간 |
|---|---|---|
| 42 | 0.6596 | 43s |
| 7 | 0.6588 | 44s |
| 123 | 0.6595 | 47s |
| **mean** | **0.6593** | — |

- G_S5 (P1 ≥ 0.6430) **PASS** (margin +0.0163)
- LGBM T-AX C1 P1 reference 0.6607 → XGB Δ **−0.0014** (P1 단계 XGB 살짝 약함)

**Phase 2 — W1 axis-wise σ C1 (3 seed mean ± std)**

| seed | P2 overall | major | minor | 시간 |
|---|---|---|---|---|
| 42 | 0.6647 | 0.7306 | 0.3124 | 55s |
| 7 | 0.6619 | 0.7306 | 0.2946 | 64s |
| 123 | 0.6613 | 0.7303 | 0.2921 | 53s |
| **mean** | **0.6626 ± 0.0018** | **0.7305** | **0.2997** | — |

**3-gate strict (vs T-AX C1, Bonferroni √2)** — EXP #17 candidate flag

| Gate | 기준 | XGB | 판정 |
|---|---|---|---|
| G1 overall | Δo > +0.0029 (combined_std × √2) | Δo **+0.0004** | **X** |
| G2 major | Δmaj > −0.002 | Δmaj **+0.0010** | O |
| G3 minor | Δmin > −0.005 | Δmin **−0.0025** | O |

→ G1 fail + G2/G3 pass + Δo>0 → **EXP #17 candidate (E)** — Strict pass 아니지만 EXP #17 (C1 OOF Δ +0.0007 → LB +0.0018) 사례에 의해 LB 검증 가치 있다고 판단, XGB 단독 LB 1슬롯 측정.

**XGB ↔ LGBM Pearson correlation (OOF residual, Cart)** — Step 7 diversity 측정

| 축 | corr |
|---|---|
| x | 0.9793 |
| y | 0.9785 |
| z | 0.9738 |
| **overall (flatten)** | **0.9779** |

→ corr ≥ 0.95 threshold → **blend 자동 skip** (Mode A scalar / Mode C α=0.5 / Mode B 343-grid 모두 미시도). XGB↔LGBM tree family는 본 protocol에서 diversity 충분 못 만듦.

**Single-mean OOF (sanity, 3 seed test pred 평균)**

| Backbone | single-mean OOF HR |
|---|---|
| LGBM (T-AX C1) | 0.6636 |
| **XGB (E26)** | **0.6653** (Δ +0.0017) |

→ Single-mean OOF에선 XGB가 LGBM 대비 +0.0017 우위 (3-seed Phase 2 +0.0004보다 큼). 평균화 효과 차이.

**LB 결과**

| Submission | OOF | LB | gap | Δ vs T-AX C1 LB |
|---|---|---|---|---|
| **T-AX C1** (LGBM, reference) | 0.6622 | **0.6906** | +0.0284 | 0 |
| **XGB solo** (E26 단독, 3 seed × test mean) | 0.6626 | **0.6886** | **+0.0260** | **−0.0020** |
| Blend (XGB+LGBM α-grid) | — | 미제출 (corr skip) | — | — |

**채택: 없음. EXP #26 폐기 확정. T-AX C1 SOTA 유지.**

### Finding 1 — Tree-tree backbone heterogeneity 정량 dead (가장 강한 단일 정량)

XGB↔LGBM corr **0.9779** — EXP #25 NN backbone과 본질적으로 다른 메커니즘 (NN corr 사전 측정 없었지만 V1 Cart vs V2 Frenet 0.65~0.75 추정). 동일 V3 26 feat + Frenet target + Phase 1 no-weight → Phase 2 W1 axis-wise σ schedule + 동일 fold seed split + 동일 stratify 조건에서 **tree family 알고리즘 차이 (depthwise vs leaf-wise, GOSS vs uniform sampling, hist binning)는 1−corr² ≈ 4.4% 직교 신호밖에 못 만듦**. blend ROI 구조적으로 음수.

**예측 blend LB (시도 안 했지만 #25 Finding 2 적용)**:
```
어떤 α∈[0, 1]도 LB ∈ [XGB_LB, LGBM_LB] = [0.6886, 0.6906]
α = 1.0 (LGBM 단독) ≈ 0.6906 (최대)
α = 0.5 ≈ 0.6896 (LGBM 단독 미달 −0.0010)
비선형 diversity 효과: corr 0.9779 → 이론적 +0.0001~+0.0003, XGB deficit −0.0020 못 넘음
```

→ 강제 blend 시도해도 LGBM 단독 갱신 불가. corr 자동 skip 결정 정당.

### Finding 2~5 + 교훈 (재사용 자산)

2. **XGB gap +0.0260 < LGBM gap +0.0284**: XGB는 OOF +0.0004 우위인데 LB −0.0020(train fold 0.0024 더 overfit). tree family 안에서도 **LGBM = single-best backbone** 확정(LGBM GOSS+leaf-wise가 minority residual 분해에 유리, XGB hist depthwise는 dominant signal 균등 분배로 over-fit).
3. **single-mean OOF(+0.0017) vs 3-seed mean(+0.0004)**: OOF 측정 단위가 transfer 신뢰도에 영향(LB는 3-seed mean에 가깝게 −0.0020 발현) → 두 단위 모두 측정 권장.
4. **EXP #17 candidate flag 강화(강한 정량)**: #17 C1(+0.0007, 동질 LGBM↔LGBM)→LB +0.0018 success vs #26(+0.0004, 이질 backbone swap)→LB −0.0020 fail. 같은 flag 정반대 → **flag 단독 불신**, 가드 ①backbone family 동질 ②Δo>+0.0010.
5. **corr≥0.95 자동 skip threshold 검증**: blend ROI 사전 진단 cheap(component OOF Pearson이 first gate). 0.95 이하면 비선형 gain이 component deficit 보상 가능(#25 GRU corr 0.65~0.75는 blend 효과 있었으나 component LB 차이 커서 net 음).
6. **시나리오 2(heterogeneous backbone) 완전 closed**: #25 NN(LB 0.6862) + #26 tree-tree(corr≥0.95) 양방향 → 시나리오 1(calibration/TTA) 또는 시나리오 3(새 모델) 또는 base+hetero 동시 차용 검토.

**산출물**: `hr_aware_xgb_e26.ipynb`, `submission_xgb_e26_solo.csv`(LB 0.6886), `results/xgb_e26_{oof,test}_seed*.npy`+meta.(blend corr skip 미생성)

---

## #22 — hr_aware_calibration_e22.ipynb (Per-axis Cart Calibration α, 시나리오 1 잔여 1순위) · 2026-05-27 ✗

**설정**: T-AX C1 (#17, LB 0.6906) LGBM OOF/test 캐시 위 **post-hoc Cart calibration** — 모델 학습 불필요. plan §4.4 사용자 결정 (2026-05-27 후반): #25 NN + #26 tree-tree 시나리오 2 양방향 closed 후 시나리오 1 surgical 1순위 진입. Kalman+GRU 저자 α=(1, 0.95, 1) lever 차용 (저자 0.6756 → 0.6780, +0.0024).

**가설 H_C1**: LGBM 최종 OOF residual에 Cart 축별 systematic bias가 있다면 `final_pred · α + δ` 단순 보정으로 +0.0005~+0.0015 LB. 우리 직교성: 저자는 Cart 좌표 단순 회귀 → Cart calibration이 axis bias 첫 처리, 우리는 Frenet residual + axis-wise W1 σ → Cart calibration이 W1과 직교 가능성. backbone family 동질 (LGBM↔post-hoc shift) → EXP #26 Finding 4 강화 조건 (backbone 동질 + Δo > +0.0010) 적용 가능.

**사용자 결정 (2026-05-27 후반)**:
- Mode A α grid: **5³=125** (#25 343-grid OOF overfit lesson 축소). α_axis ∈ {0.970, 0.985, 1.000, 1.015, 1.030}
- 사전 fail gate: **|μ_axis| < 1e-4 m = 0.1mm** 모든 axis → 사전 fail로 #23 이동
- Mode B (offset δ = −μ_axis 단일 조합, free param 0) + Mode C (Mode A best ± 3³ × δ ±0.5mm = 81 config narrow hybrid)

학습량: **0 model** (post-hoc lever, OOF/test 캐시만 활용). 실행 시간 ~3분 (Drive mount + 데이터 로드 포함).

**μ_axis 측정 (OOF, T-AX C1 3-seed mean prediction)**

| axis | μ (mm) | |μ| (mm) | > 0.1mm threshold |
|---|---|---|---|
| x | **+0.4979** | 0.4979 | ✓ |
| y | −0.1759 | 0.1759 | ✓ |
| z | +0.1645 | 0.1645 | ✓ |

→ 모든 axis |μ| ≥ 0.1mm → #22 진행 정당. μ_x 가장 큼 (+0.498mm = pred가 x축에서 살짝 낮음).

**Mode A — per-axis α scaling 5³=125**

```
best: α=(1.000, 1.000, 1.000) = identity
OOF overall=0.6636, Δ vs C1 OOF 0.6622 = +0.0014
major=0.7308, minor=0.3041
```

→ **best가 identity** — 125 α 조합 중 어떤 multiplicative scaling도 no-op (α=1)을 이기지 못함. 즉 T-AX C1 prediction의 axis별 multiplicative 비율은 이미 거의 정확. bias 보정은 multiplicative (×0.97~×1.03)로는 불가능, additive offset에서만 가능 (Mode C에서 확인).

**Mode B — offset δ = −μ_axis 단일 조합 (free param 0)**

```
δ = (−0.4979, +0.1759, −0.1645) mm (μ 정확 보상)
OOF overall=0.6606, Δ vs C1 OOF = −0.0016
major=0.7272 (Δ −0.0023, G2 hard stop fail), minor=0.3041 (Δ +0.0019)
```

→ **μ correction이 HR과 정반대 방향**. mean unbiased 가정으로 δ=−μ를 적용했더니 G2 hard stop fail. **HR (1cm boundary) metric은 mean bias가 아니라 boundary 근처 sample 분포에 민감** — μ correction이 boundary push와 직교 또는 반대.

**Mode C — hybrid narrow (Mode A best 주변 3³ × δ ±0.5mm = 81 config)**

```
best: α=(1.000, 1.000, 1.000), δ_mm=(0, 0, +0.50)
OOF overall=0.6639, Δ vs C1 OOF = +0.0017 (G1 strict threshold +0.0017과 동률)
major=0.7308 (Δ +0.0013, G2 pass), minor=0.3060 (Δ +0.0038, G3 pass)
```

→ α는 identity 유지, **z axis에만 단일 +0.5mm offset**. μ_z = +0.1645mm와 반대 부호 (μ correction의 3배 크기). bias 보상이 아닌 **boundary push** 효과로 해석.

**3-gate vs T-AX C1 OOF (Bonferroni √3, threshold +0.0017)**

| Mode | Δoverall | G1 (>+0.0017) | G2 (>−0.002) | G3 (>−0.005) | flag |
|---|---|---|---|---|---|
| A (identity best) | +0.0014 | X | O (+0.0013) | O (+0.0019) | E |
| B (offset = −μ) | −0.0016 | X | **X (−0.0023)** | O (+0.0019) | **X (G2 fail)** |
| **C (z+0.5mm)** | **+0.0017** | X (동률) | O (+0.0013) | O (+0.0038) | **E** |

채택: **Mode C** (Mode A identity 회피, simplicity tiebreak 결함 수동 보정). e17_candidate flag (backbone 동질 + Δo > +0.0010 #26 강화 조건 만족) → LB 1슬롯 시도.

**LB 결과**

| Submission | OOF | LB | gap | Δ vs T-AX C1 LB |
|---|---|---|---|---|
| **T-AX C1** (reference) | 0.6622 | **0.6906** | +0.0284 | 0 |
| Mode C (z+0.5mm) | 0.6639 | **0.6882** | **+0.0243** | **−0.0024** |

**채택: 없음. EXP #22 폐기 확정. T-AX C1 SOTA 유지.**

### 핵심 Findings + 교훈 (재사용 자산)

1. **Post-hoc lever = model parameter lever와 별개 transfer 메커니즘(가장 강한 정량)**: #22 Mode C는 #26 두 가드(backbone 동질 + Δo +0.0017>+0.0010) 모두 만족했는데도 **LB −0.0024 fail**(#17 C1 동질+model param은 +0.0018 success / #26 이질 backbone은 fail). → **EXP #17 candidate flag 3번째 가드: model parameter 변경(W schedule/hyperparam/feature/target)인지 vs post-hoc shift(calibration/TTA/blend)인지**. post-hoc는 train OOF axis bias가 test서 다른 분포로 reverse 가능 → 별도 framework 필요.
2. **HR metric은 mean bias 아닌 boundary 분포에 민감**(Mode B 정량): δ=−μ 정확 적용에도 overall −0.0016, major **−0.0023(G2 fail)**, minor +0.0019. majority residual(mode 5~10mm)은 boundary 근처 몰려 mean shift 시 일부 boundary 밖 push, minority(평균 27mm)는 영향 작음 → calibration은 boundary-aware shift 필요.
3. **Mode A best=identity(multiplicative scaling dead)**: 5³=125 α 중 α=(1,1,1) 최적. base 거의 unbiased(median 0.999) + residual small(12.94mm)이라 ±3% scaling 영향 적음 → additive offset만 적합.
4. **simplicity tiebreak 로직 결함**: free-param 적은 순 tiebreak가 Mode A identity(no-op) 우선 채택 → submission이 baseline 비트 동일, LB 슬롯 낭비. **identity-skip 가드 필요**(자동화 시).
5. **Mode B G2 hard stop fail은 Karpathy guideline 부합**: G2(Δmajor>−0.002)가 minority cherry-pick(single positive Δminor) polarized lever 사전 차단.

**산출물**: `hr_aware_calibration_e22.ipynb`, `submission_calibration_e22_mode_c.csv`(LB 0.6882), `results/calibration_e22_meta.json`.

---

## #28 — hr_aware_v4_lite_e28.ipynb (V4-lite: in-traj 2-step physics residual feature) · 2026-05-28 ✗

**설정**: T-AX C1 (#17, LB 0.6906) 위 **24 V4-lite feature 추가** (5 anchor × 3 axis + slope + mean/std). plan §4.10 v3 시나리오 3 1순위. 11-trajectory 내 2-step (80ms) displacement 분포가 anchor 간 stationary, label 분포와 3D scatter 일치 (사용자 제공 HTML 시각화)라는 EDA finding 기반 self-supervised 신호 추출 가설.

**가설 H_V4**: 각 in-traj anchor t (k_a ∈ {4..8}, t = −240..−80ms)에서의 2-step physics residual `pos(t+80) − (pos(t) + 2·v_t)`이 label 2-step residual `y − base`과 **동일 메커니즘**. 5 anchor (rL, rN, rB) 시계열이 sample-specific 가속/회전 패턴 인코딩 → V3 26과 직교 정보. baseline 평가 시 univariate R² > V3 26 max R² PASS.

**사용자 결정 (2026-05-27 후반 → 2026-05-28 실행)**:
- 8 config 분할 ablation (외부 R² 분석으로 4 → 7 → 8로 확장)
- Phase 1+2 T-AX C1 동일 protocol (σ=4.3/3.5/2.7 max_w=2.5 고정, Karpathy 단일변수)
- R² gate (Step 3 사전 차단): 새 column 중 1개라도 V3 26 max R² 능가 → Step 4 진입

학습량: **8 configs × 3 seed × 5 fold × 3 axis × 2 phase = 720 model** (~176분 Colab).

### Feature 정의 상세

**V3 26** (T-AX C1 baseline, RESEARCH §4) = 19 scalar(`speed_last·acc_norm_last·acc_norm_w3·v*/a*_std·path_eff·distance_r·radial_v·turn_mean·cos_turn_last·va_dot·a_tang_last·a_cent_last·speed_diff_half·turn_mean_half_diff`) + 7 Frenet proj(`a_N·a_B·j_L·j_N·j_B·aw_L·aw_N`, t=0 기준). OOF 0.6622±0.0010 / LB 0.6906.

**V4-lite 24** (anchor t∈{−240..−80}ms, k_a∈{4..8}): 각 anchor서 `res_t = pos(t+80) − (pos(t)+2·v_t)`를 anchor-local Frenet(eL/eN/eB_t) 투영한 (rL/N/B). 그룹 **A 시계열 15**(5 anchor×3 axis) + **B slope 3** + **C aggregate 6**(mean/std). Sub-group: A_{a4..a7}(12)·A_{a6,a7}(6)·A_L(5)·C_std(3).

#### 8 Config feature 구성 매트릭스

| Config | V3 26 | A (시계열) | B (slope) | C (mean/std) | total | 가설 |
|---|:---:|---|---|---|:---:|---|
| **C1** | ✓ | A (15) | — | — | **41** | A 시계열 단독 효과 |
| **C2** | ✓ | A (15) | — | C (6) | **47** | A+C (B skip — 외부 R² slope 무기여) |
| **C3** | ✓ | A (15) | B (3) | C (6) | **50** | V4-lite full (B의 marginal 기여 empirical 확인) |
| **C4** | — | A (15) | B (3) | C (6) | **24** | V4 standalone (V3 26 의존도 측정) |
| **C5** | ✓ | — | — | C (6) | **32** | C standalone (A 무기여 검증) |
| **C6** | ✓ | A_{a4..a7} (12) | — | C_std (3) | **41** | a8 (−80ms) drop — V3 26 last-step과 중복 가설 |
| **C7** | ✓ | A_{a6,a7} (6) | — | C_std (3) | **35** | 핵심 anchor (a6=−160, a7=−120ms) 만 — minimal feature subset |
| **C8** | ✓ | A_L (5) | slope_rL (1) | mean_rL + std_rL (2) | **34** | L-only V4 (N/B 완전 제거 — v_N=v_B=0 정의상 L이 dominant) |

T-AX C1 baseline = V3 26 단독 (26 feat), OOF 0.6622 ± 0.0010 / LB 0.6906.

### Step 1~3: eda_2step_features.py (univariate R² gate)

24 column 생성 + V3 26 vs V4-lite univariate R² 비교. NaN 0, fallback frame 0.01~0.02% (1~2 sample, corner case).

**R² gate 결과 (vs label res_L/N/B)**:

| target | V3 26 max R² | V4 best R² | V4 best column | gate strict |
|---|---|---|---|---|
| res_L | 0.0530 (`acc_norm_last`) | **0.0646** | `std_rL` | **PASS** |
| res_N | 0.0898 (`a_N`) | 0.0047 | `rL_a8` | FAIL |
| res_B | 0.0051 (`a_B`) | 0.0042 | `slope_rB` | FAIL |

→ res_L에서 strict PASS, Step 4 진입.

**V4-lite top-5 (res_L)**: std_rL (0.0646) > std_rB (0.0593) > std_rN (0.0457) > mean_rL (0.0414) > rL_a6 (0.0323). **uncertainty 신호 (std_*) dominant**, individual anchor 중 `rL_a6` (−160ms) 최강. 사용자 외부 R² 분석의 "a6/a7 핵심" 일치.

### Step 4~6: 8 config OOF (3-gate vs T-AX C1)

```
Config feat  P1 mean±std       P2 overall          major               minor
C1     41   0.6582 ± 0.0008  0.6601 ± 0.0012   0.7274 ± 0.0017  0.3001 ± 0.0019
C2     47   0.6578 ± 0.0014  0.6603 ± 0.0004   0.7273 ± 0.0004  0.3020 ± 0.0038
C3     50   0.6568 ± 0.0009  0.6610 ± 0.0020   0.7283 ± 0.0013  0.3012 ± 0.0064
C4     24   0.6239 ± 0.0009  0.6261 ± 0.0007   0.6988 ± 0.0005  0.2372 ± 0.0048
C5     32   0.6597 ± 0.0015  0.6624 ± 0.0003   0.7294 ± 0.0004  0.3041 ± 0.0029
C6     41   0.6566 ± 0.0016  0.6596 ± 0.0007   0.7276 ± 0.0004  0.2961 ± 0.0031
C7     35   0.6571 ± 0.0013  0.6603 ± 0.0019   0.7275 ± 0.0014  0.3010 ± 0.0088
C8     34   0.6591 ± 0.0017  0.6617 ± 0.0008   0.7294 ± 0.0012  0.2997 ± 0.0025
```

**3-gate (T-AX C1 OOF 0.6622±0.0010, Bonferroni √8=2.83)**:
- **8 config 전부 G1 strict fail** (max Δo = C5 +0.0002 < threshold +0.0029~+0.0064)
- C4 catastrophic: Δo −0.0361, Δmajor −0.0307, Δminor −0.0650 — V4 standalone은 V3 26 없이 동작 불가
- best config = **C5** (G1 fail이지만 G2/G3 pass + max +Δ, plan rule "EXP #17 C1 사례" 적용)

**Ablation 진단**:

| 비교 | Δ | 해석 |
|---|---|---|
| C5 vs C3 (A 무기여) | **+0.0014** | A 시계열 15 feat이 LGBM에 noise. mean/std aggregate가 raw sequence보다 유용 |
| C2 vs C3 (B 무기여) | −0.0007 | B (slope 3 feat) marginal contribution (외부 R² 분석 부분 일치) |
| C6 vs C1 (a8 V3 26 중복) | −0.0005 | a8 drop이 marginal positive (V3 26 last-step과 부분 중복 가설 약한 확인) |
| C7 vs C3 (minimal subset) | −0.0007 | 9 feat (C7)가 24 feat (C3)와 동등 — simplicity tiebreak로 C7 우위 |
| C8 vs C3 (N/B 무기여) | **+0.0007** | L-only (N/B 제거)가 marginal positive — N/B 신호 약함 (외부 R² 분석 일치) |
| C4 vs T-AX C1 (V4 standalone) | −0.0361 | V3 26 본질 신호. V4 단독으론 부족 |

→ 결론: A 시계열 raw가 noise, C aggregate (mean/std)만 marginal 정보, N/B는 거의 무신호. **V4-lite의 "signal density"가 매우 낮음** — 24 feat 전체로도 baseline 못 넘김.

### Step 7~8: Phase B skip + C5 LB 실측 (transfer calibration)

best C5가 G1 fail이라 Phase B skip (plan rule). C5만 LB 1슬롯 제출 — 사용자 결정: "근거 약해도 transfer 실측" (이전 답변).

**C5 LB 결과**:

```
T-AX C1 baseline:   OOF 0.6622 ± 0.0010 → LB 0.6906  (LB−OOF gap = +0.0284)
C5 V4-lite + C:6:   OOF 0.6624 ± 0.0003 → LB 0.6872  (LB−OOF gap = +0.0248)
                                          ──────────────────────────────
                                          Δ LB = −0.0034
                                          Δ OOF = +0.0002 (noise zone)
```

→ **OOF marginal positive (+0.0002) → LB strong negative (−0.0034)** = post-hoc transfer fail (#22)과 동일 패턴이지만 mechanism 다름 (feature add = model parameter 변경).

### 4번째 가드 도출 (EXP #17 candidate flag 시리즈 4차 정량)

| EXP | Lever type | OOF Δo | LB Δ | 결론 |
|---|---|---|---|---|
| #17 | Model param (axis-wise σ) | +0.0007 | **+0.0018** | ✓ favorable transfer |
| #26 | Backbone heterogeneity | +0.0004 | −0.0020 | ✗ backbone 이질로 fail |
| #22 | Post-hoc shift (z+0.5mm) | +0.0017 | −0.0024 | ✗ post-hoc 별개 framework |
| **#28** | **Model param (feature add)** | **+0.0002** | **−0.0034** | ✗ **OOF noise zone lever transfer 못 함** |

#22(post-hoc)+#28(model param) 양쪽 OOF noise zone(Δo<combined_std×√2≈+0.0028) → LB transfer fail = **lever type 무관 OOF noise zone signal은 LB transfer 안 됨**.

**EXP #17 candidate flag 4번째 가드**(누적): ①G_S 5-criterion sanity(#12) ②backbone 동질(#26) ③model parameter 변경(post-hoc 제외, #22) ④**Δo > combined_std×√2(real signal zone, #28)**. → #17 C1 "G1 fail+G2/G3 pass→LB 시도" 규칙은 4번째 가드로 사실상 무효화.

**3-stage 신호 degradation**: univariate R² PASS(std_rL 0.0646>V3 26 0.0530) → joint LGBM MARGINAL(C5 +0.0002 noise) → LB FAIL(−0.0034). univariate R²는 단독 정보량일 뿐, LGBM이 V3 26과 redundant signal 분리 + noise zone signal은 LB-OOF gap에 묻힘 → univariate R² gate는 진입 조건이지 충분 조건 아님.

**교훈 (재사용 자산)**: ① **OOF noise zone lever LB 금지**(#22+#28 정량, Δo>combined_std×√2 필수). ② A 시계열 raw는 LGBM noise, C aggregate(mean/std)가 유용. ③ **V3 26 = 본질 신호**(C4 standalone −0.0361 catastrophic) → feature engineering은 V3 26 직교성 사전 분석 필수. ④ N/B 신호 약함(C8 vs C3 +0.0007) → axis-wise는 L축 우선. ⑤ univariate R² ≠ joint LGBM gain(cheap screening, 채택 기준 아님). ⑥ **3D 분포 stationarity ≠ feature 직교성**.

**산출물**: `hr_aware_v4_lite_e28.ipynb`, `eda_2step_features.py`, `submission_v4_lite_C5_g1fail_ms3.csv`(LB 0.6872), `results/v4_lite_*`(oof·univariate_r2·features·C1~C8 seed).

---

## #29 — hr_aware_topk_e29.ipynb (Top-K Feature Subset Ablation by gain) · 2026-05-28 ✓ LB 0.6906 (strict tie → K15 채택)

**설정**: T-AX C1 (LB 0.6906) protocol 비트 단위 + V3 26 feature subset 7 config ablation. Phase 1 no-weight Frenet target + Phase 2 axis-wise W1 (σ_L=4.3, σ_N=3.5, σ_B=2.7 mm, max_w=2.5) 고정, **input X matrix column 수만 차이**. 6 신규 config × 3 seed × 5 fold × 3 axis × 2 phase = **540 model**, Colab T4 실측 ~35분.

**가설 H_K** (사용자 제기 2026-05-28): "Feature가 지금 너무 많아서 성능이 오히려 낮아지고 있을 가능성". gain importance plateau (rank 11~26, ~880~1160) 영역의 noise split + algebraic redundancy 의심.

**근거 (사전 screening)**

| 항목 | 정량 |
|---|---|
| 5-fold mean gain ranking (`results/v3_26_gain_ranking.json`, seed=42) | Top-4 dominance (`a_tang_last` 5914 → `va_dot` 3662 → `speed_last` 2915 → `j_L` 2371) + rank 11~26 plateau (1163 → 882) |
| 1-fold sanity (`v3_26_gain_ranking_1fold.json`) | Spearman ρ vs 5-fold = 0.8892. K=5/15 100% overlap, K=10/20 80~90% overlap (plateau 경계) |
| **Algebraic dependency 발견** (실측 max err 3.8e-6) | (1) `a_tang_last = va_dot × acc_norm_last`  (2) `acc_norm_last² = a_tang_last² + a_cent_last²` (Pythagorean)  (3) `a_cent_last² = a_N² + a_B²` (Frenet 직교성) → 6 feature 자유도 3, gain 35% 차지 |
| Pairwise Pearson (a_tang_last ↔ va_dot = 0.499) | moderate corr이 곱셈/제곱 dependency를 숨김 — 선형 corr만으론 탐지 불가 |

**7 Config 정의 (nested 5 + dependency-aware 2)**

| Config | n_feat | feature set | 가설 |
|---|---:|---|---|
| K5_indep | 4 | `a_tang_last, speed_last, j_L, cos_turn_last` (K5 - va_dot) | va_dot이 a_tang_last에 흡수되는지 test |
| K5 | 5 | top-5 by gain (위 + va_dot) | nested baseline |
| K10 | 10 | + `j_N, a_cent_last, speed_diff_half, turn_mean_half_diff, acc_norm_last` | rank 6~10 추가 (시퀀스 통계 일부 진입) |
| **K15** ⭐ | **15** | + `radial_v, turn_mean, acc_norm_w3, distance_r, vy_std` | **EXP #13 Phase B best subset (13)과 인접** |
| K20 | 20 | + `path_eff, ax_std, aw_L, az_std, j_B` | bottom 6만 제거 |
| K_tied_min | 24 | V3 26 - `{va_dot, acc_norm_last}` | algebraic 잉여만 제거, 자유도 3 유지 |
| K26 (= T-AX C1 ref) | 26 | full V3 26 | baseline |

**결과 (M2 OOF, 3 seed mean ± std)**

| Config | n | P1 mean±std | P2 overall±std | P2 major | P2 minor | rL_mae | rN_mae | rB_mae |
|---|---:|---|---|---|---|---|---|---|
| K5_indep | 4 | 0.6569 ± 0.0018 | 0.6579 ± 0.0010 | 0.7257 ± 0.0013 | 0.2957 ± 0.0035 | 5.565 | 6.421 | 5.122 |
| K5 | 5 | 0.6589 ± 0.0008 | 0.6612 ± 0.0002 | 0.7271 ± 0.0002 | 0.3086 ± 0.0000 | 5.533 | 6.393 | 5.122 |
| K10 | 10 | 0.6589 ± 0.0003 | 0.6625 ± 0.0009 | 0.7283 ± 0.0008 | **0.3105 ± 0.0028** | 5.459 | 6.294 | 5.120 |
| **K15** ⭐ | **15** | 0.6597 ± 0.0009 | **0.6630 ± 0.0012** | 0.7290 ± 0.0005 | 0.3096 ± 0.0065 | 5.471 | 6.274 | 5.121 |
| K20 | 20 | 0.6596 ± 0.0018 | 0.6617 ± 0.0008 | 0.7289 ± 0.0013 | 0.3022 ± 0.0029 | 5.453 | 6.282 | 5.121 |
| K_tied_min | 24 | 0.6595 ± 0.0020 | 0.6614 ± 0.0020 | **0.7295 ± 0.0021** | 0.2971 ± 0.0050 | 5.475 | 6.289 | 5.119 |
| K26 (= ref) | 26 | 0.6607 ± 0.0011 | 0.6622 ± 0.0010 | **0.7295 ± 0.0017** | 0.3022 ± 0.0029 | 5.469 | 6.285 | 5.119 |

**Sanity** (K26 ≈ T-AX C1 reference):
- K26 P1 = 0.6607 vs ref 0.6607 (Δ +0.0000) ✓
- K26 OOF = 0.6622 vs ref 0.6622 (Δ +0.0000) ✓ — protocol drift 0

**3-gate + 4th guard (vs K26, Bonferroni √6 = 2.449, 4th guard √2 = 1.414)**

| Config | Δoverall | g1_thr | g4_thr | Δmajor | Δminor | G1 | G2 | G3 | G4 | LB eligible |
|---|---:|---:|---:|---:|---:|:---:|:---:|:---:|:---:|:---:|
| K5_indep | −0.0043 | +0.0034 | +0.0020 | −0.0038 | −0.0066 | ✗ | ✗ | ✗ | ✗ | ✗ |
| K5 | −0.0010 | +0.0024 | +0.0014 | −0.0024 | +0.0063 | ✗ | ✗ | ✓ | ✗ | ✗ |
| K10 | +0.0003 | +0.0032 | +0.0019 | −0.0012 | +0.0083 | ✗ | ✓ | ✓ | ✗ | ✗ |
| **K15** | **+0.0008** | +0.0037 | +0.0021 | −0.0005 | +0.0074 | ✗ | ✓ | ✓ | ✗ | ✗ |
| K20 | −0.0005 | +0.0031 | +0.0018 | −0.0006 | +0.0000 | ✗ | ✓ | ✓ | ✗ | ✗ |
| K_tied_min | −0.0008 | +0.0054 | +0.0031 | +0.0000 | −0.0051 | ✗ | ✓ | ✗ | ✗ | ✗ |

→ **6/6 4th guard fail**. EXP #28 룰에 따르면 LB 제출 금지. 그러나 K15는 #17 C1 사례 (OOF +0.0007 → LB +0.0018)과 거의 동일 zone (+0.0008) + 4th guard 룰 자체의 calibration 가치 + 슬롯 cost 낮음으로 **사용자 결정 LB 제출**.

**LB 결과**

| Submission | OOF | LB | Δ vs T-AX C1 K26 | Δ vs K26 LB |
|---|---|---|---|---|
| T-AX C1 K26 (EXP #17) | 0.6622 | 0.6906 | — | — |
| **K15 (EXP #29)** | 0.6630 | **0.6906** | OOF +0.0008 | **LB ±0.0000 (strict tie)** |

**채택: K15** — Karpathy **simplicity tiebreak** 적용 (CLAUDE.md §2 "Simplicity First"). LB 동률에서 feature 적은 쪽 (15 vs 26) 우위.

**Karpathy precedent**: EXP #15 V2(38) vs V3(26) Δo +0.0003 overlap → V3 채택 (LB 0.6888 → simplicity)으로 V2 우위 단독 LB 미시도. 본 결정도 동일 원칙.

**Algebraic ablation findings**

**1) K5_indep (4 feat) vs K5 (5 feat) — va_dot drop 효과**
- Δoverall = −0.0032 (combined_std 0.0010, **signal_zone=True**)
- Δmajor = −0.0014, Δminor = −0.0129
- → **K5_indep < K5**: va_dot은 algebraic dependency (`= a_tang_last / acc_norm_last`)에도 LGBM에 **추가 split utility** 가져옴. 0.499 Pearson이 숨긴 사실 정량 확인. **"algebraic redundancy = LGBM redundancy" 명제 부정**

**2) K_tied_min (24 feat) vs K26 (26 feat) — algebraic 잉여 {va_dot, acc_norm_last} 제거 효과**
- Δoverall = −0.0008 (combined_std 0.0022, signal_zone=False)
- Δmajor = +0.0000, Δminor = −0.0051
- → **K_tied_min ≈ K26**: algebraic 잉여 (2 feat) 제거가 net 0. LGBM이 redundant view에서 split convenience 추출. 사용자 가설 "algebraic redundancy가 성능 깎고 있다" **부정**

### 핵심 Findings + 교훈 (재사용 자산)

1. **Karpathy simplicity tiebreak는 LB 동률에서 더 강함**: #15(V2 38 vs V3 26, OOF overlap→V3) → #29(K26 26 vs K15 15, **LB strict tie→K15**). V3 28→26→15 누적 42% feature 감소하며 LB 동등. 향후 LB Δ≈0 lever는 적은 쪽 자동 채택.
2. **Algebraic dependency ≠ LGBM redundancy(정량 새 정보)**: V3 26 숨은 관계 `a_tang_last=va_dot×acc_norm_last`, `acc_norm_last²=a_tang²+a_cent²`(Pythagorean), `a_cent²=a_N²+a_B²`(Frenet 직교) — 6 feat 자유도 3, gain 35%. 그러나 K_tied_min vs K26≈0 & **K5_indep<K5(Δo −0.0032 signal zone)** → LGBM이 algebraic redundant feature를 다른 split depth/threshold서 활용(선형 corr 0.499가 곱/제곱 dependency 은폐). 단독 제거 가설은 학습 검증 필수.
3. **Per-metric peak 분기 = carrier 비대칭**: overall=K15(0.6630)/major=K26(0.7295)/minor=K10(0.3105). K15는 두 carrier balance point(#18 "majority=시퀀스 통계, minority=point accel-Frenet" 재확인).
4. **4th guard 시리즈 5차 갱신**: OOF noise zone lever는 success(#17)/fail(#22,#28)/**tie(#29)** 모두 가능. LB 동률도 simplicity tiebreak 자격으로 충분 → "Δo<+0.0028 시 LB 제출은 simplicity 가능성 있을 때만, success prior 매우 낮음".
5. **V3 15 새 baseline 확정**: 제거 11개(시퀀스 통계 5 + Frenet proj 5 + L축 외 std)에도 LB 무손실. 시나리오 1 잔여·시나리오 3 lever 모두 V3 15 위에서 진행.

**산출물**: `hr_aware_topk_e29.ipynb`, `submission_topk_K15_ms3.csv`(**LB 0.6906**, 채택), `results/topk_e29_*`(meta·algebraic·oof·test), `results/v3_26_gain_ranking{,_1fold}.json`.

---

## #30 — hr_aware_r2step_e30.ipynb (In-traj 2-step CV Residual Current-Frame Projection, 3-way ablation) · 2026-05-28 ✗

**설정**: T-AX C1 K15 (V3 15 feat, EXP #29 채택) protocol 비트 단위 + 4 new feat 추가, 3-way ablation으로 L vs N 효과 분리. 두 hypothesis 동시 검증:
1. **2-step CV residual 패턴 가설**: 과거 anchor t에서의 등속 2-step 외삽 오차 `res_t = p[k_a+2] − p[k_a] − 2·(p[k_a] − p[k_a−1])`이 미래 label residual `y − base`과 동일 메커니즘
2. **Lateral signed 가설** (사용자 제기 2026-05-28): V3 15에 부족한 signed lateral 신호 (rN_*_0frame) 제공

**EXP #28과 결정적 차이**: anchor-local frame (eL_t) → **current-frame (eL_last, eN_last)** sample-consistent projection.

**Feature 정의 (4 new feat)**

```
res_t(k_a) = p[k_a+2] − p[k_a] − 2·(p[k_a] − p[k_a−1])     # 3D residual mm scale

rL_-120_0frame = res_t(k_a=7) · eL_last     # signed, single anchor t=-120ms
rN_-120_0frame = res_t(k_a=7) · eN_last     # signed
std_rL_0frame  = std({res_t(k) · eL_last : k=4..8})   # ≥ 0, 5-anchor std
std_rN_0frame  = std({res_t(k) · eN_last : k=4..8})   # ≥ 0
```

**Configs (3-way ablation, K15_ref EXP #29 캐시 재사용)**

| Config | n_feat | 추가 feature | 가설 |
|---|---:|---|---|
| K15_ref | 15 | — | EXP #29 baseline, control |
| K15+L | 17 | `rL_-120_0frame, std_rL_0frame` | 2step framework L성분 단독 |
| K15+N | 17 | `rN_-120_0frame, std_rN_0frame` | Lateral signed 단독 (사용자 가설) |
| K15+LN | 19 | rL + rN 4개 모두 | L+N 결합 (synergy/additive/interference) |

**학습량**: 3 신규 config × 3 seed × 5 fold × 3 axis × 2 phase = **270 model**, Colab T4 실측 ~21분. K15_ref는 EXP #29 캐시 재사용 (0 model).

**4 new feat sanity (분포, NaN, V3 15 dependency)**

| Feature | mean (mm) | std (mm) | min | max | NaN | V3 15 max \|corr\| |
|---|---:|---:|---:|---:|---:|---|
| rL_-120_0frame | ~0 | 5~7 | (signed range) | — | 0 | < 0.95 ✓ |
| std_rL_0frame | + | 3~5 | 0 | + | 0 | < 0.95 ✓ |
| rN_-120_0frame | ~0 | 5~7 | (signed) | — | 0 | < 0.95 ✓ |
| std_rN_0frame | + | 3~5 | 0 | + | 0 | < 0.95 ✓ |

algebraic dependency 검출 안 됨 — 안전.

**결과 (M2 OOF, 3 seed mean ± std)**

| Config | n | P1 mean±std | P2 overall±std | P2 major±std | P2 minor±std |
|---|---:|---|---|---|---|
| K15_ref | 15 | 0.6597 ± 0.0009 | **0.6630 ± 0.0012** | 0.7290 ± 0.0005 | 0.3096 ± 0.0065 |
| K15+L | 17 | 0.6600 ± 0.0004 | 0.6620 ± 0.0006 | 0.7284 ± 0.0010 | 0.3067 ± 0.0061 |
| K15+N | 17 | 0.6602 ± 0.0020 | 0.6614 ± 0.0005 | 0.7291 ± 0.0007 | 0.2995 ± 0.0036 |
| K15+LN | 19 | 0.6594 ± 0.0013 | 0.6608 ± 0.0004 | 0.7281 ± 0.0001 | 0.3007 ± 0.0019 |

**Per-seed P2 overall — 9/9 K15_ref보다 낮음**

| Config | seed=42 | seed=7 | seed=123 |
|---|---|---|---|
| K15_ref | 0.6640 | 0.6632 | 0.6617 |
| K15+L | 0.6620 | 0.6625 | 0.6614 |
| K15+N | 0.6618 | 0.6616 | 0.6608 |
| K15+LN | 0.6612 | 0.6607 | 0.6605 |

→ K15_ref min 0.6617 vs K15+L max 0.6625 → 한 case만 겹침. **체계적 신호 손상** (noise zone 아님).

**3-gate + 4th guard (vs K15_ref, Bonferroni √3 = 1.732, 4th guard √2 = 1.414)**

| Config | Δo | g1_thr | g4_thr | ΔM | Δm | G1 | G2 | G3 | G4 | LB |
|---|---:|---:|---:|---:|---:|:---:|:---:|:---:|:---:|:---:|
| K15+L | −0.0010 | +0.0022 | +0.0018 | −0.0006 | −0.0030 | ✗ | ✓ | ✓ | ✗ | ✗ |
| K15+N | −0.0016 | +0.0022 | +0.0018 | +0.0000 | **−0.0102** | ✗ | ✓ | ✗ | ✗ | ✗ |
| K15+LN | −0.0022 | +0.0021 | +0.0017 | −0.0009 | −0.0089 | ✗ | ✓ | ✗ | ✗ | ✗ |

→ 3/3 G1·G4 fail, 2/3 G3 hard stop fail. **LB 제출 0 (4th guard 통과 0개)**.

**Ablation 진단 — ADDITIVE 패턴**

| 비교 | Δ | 해석 |
|---|---:|---|
| Δ_L (K15+L − K15_ref) | −0.0010 | 2step framework L성분 단독 — noise zone 음 |
| Δ_N (K15+N − K15_ref) | −0.0016 | Lateral signed 단독 — minor −0.0102 손상 가장 큼 |
| Δ_LN (K15+LN − K15_ref) | −0.0022 | L+N 결합 — 손상 누적 |
| Additive predict (Δ_L + Δ_N) | −0.0026 | — |
| Synergy score (Δ_LN − Δ_L − Δ_N) | +0.0004 | **ADDITIVE** (\|synergy\| < combined_std) |

→ L·N 효과 독립이며 두 독립 효과 모두 음. 결합 시 두 손상이 그대로 누적.

**R4 4 feat gain importance (Phase 2 mean across 3 axis × 5 fold × 3 seed)**

| Config | Feature | gain | rank |
|---|---|---:|---|
| K15+L | rL_-120_0frame | 2895 | 15/17 |
| K15+L | std_rL_0frame | 3376 | 10/17 |
| K15+N | rN_-120_0frame | 2853 | 13/17 |
| K15+N | std_rN_0frame | 2823 | 14/17 |
| K15+LN | rL_-120_0frame | 3617 | 6/19 |
| K15+LN | std_rL_0frame | 3423 | 9/19 |
| K15+LN | rN_-120_0frame | 2954 | 17/19 |
| K15+LN | std_rN_0frame | 2961 | 16/19 |

→ LGBM이 새 feature를 active하게 사용 (gain 2800~3600, 중상위 rank)하지만 OOF는 음 → **train fold에 fit하지만 OOF generalize 실패 = noise**. EXP #28 패턴 재발 정량.

### 핵심 Findings + 교훈 (재사용 자산)

1. **Lateral signed 사용자 가설 반대 방향 기각**: K15+N Δo −0.0016, **Δminor −0.0102**(3 config 중 최대 손상), major 0.0000. 고가속(|a|≥5, minority dominant)서 past anchor 등속 2-step 외삽이 가장 깨져 res_t noise↑ → eN_last 투영해도 future res_N 부호와 mechanism mismatch.
2. **Current-frame projection도 #28 fail 회피 못 함**: #28(anchor-local eL_t)+#30(current-frame eL_last) 양방향 fail → **frame 선택 문제 아니라 res_t 자체가 future residual predictor 아님**. EDA 3D 분포 stationarity(internal_2step ≈ label_0~80ms)는 marginal/joint 유사일 뿐 predictive transfer 아님(#28 Finding 6 "3D stationarity ≠ feature 직교성" 2차 확인).
3. **ADDITIVE 패턴 + LGBM "쓰이지만 해롭다"**: synergy +0.0004≈0(L·N 독립, 둘 다 음→누적). LGBM이 4 feat active 사용(gain 2800~3600, rank 중상위)에도 OOF 음 = train-fold fit / OOF generalize 실패 noise. #30은 joint LGBM 단계서 이미 음 → LB 시도 사전 차단.
4. **In-traj feature add 라인 5차 누적 fail**(#11 D6 / #13 F / #18 F-INT / #28 V4-lite / #30 r2step), frame 양방향 검증 완료 → **완전 closed**, multi-baseline selector 전환 트리거.
5. **3-way component ablation = information gain 3~5배**(2× 비용): L vs N 효과 분리·결합 패턴(synergy/additive)·사용자 가설 단독 검증. 사용자 가설이 specific direction일 때 권장.
6. **V3 15는 mosquito kinematics 본질 신호(v_last·a_last·jerk·trajectory_stat) 거의 완전 cover** — feature add 라인(in-traj residual·EDA 통계·적분 평면 5차 fail) ROI ≈0, 다음 lever는 base·loss·representation 변경(시나리오 3).

**산출물**: `hr_aware_r2step_e30.ipynb`, `results/r2step_e30_*`(meta·features·oof·test·ablation). **submission 0**(4th guard 통과 0).

---

## #31 — hr_aware Split e31 (L15 Major/Minor Split Selector + minor→GRU sanity) · 2026-05-28 후반

**설정**: plan.md §4.16. K10/K15 binary split selector + minor→GRU hard split sanity. EXP #29 K10/K15 OOF/test cache 재사용 (모델 학습 0) + LGBM classifier 학습 (3 seed × 5 fold = 15 model, V3 15 feat → K10 vs K15 binary). Phase 1 5-모델 확장 skip. EXP #25 GRU OOF cache 추가 동기화 (form 자동 판별 → Frenet residual 채택). 3 lever ablation (Hard / Soft / Minor→GRU) vs K15 OOF 0.6649 baseline.

**Sanity gates 결과**

| Gate | 기준 | 측정 | Pass? |
|---|---|---|---|
| K10/K15 cache 재현 | oracle_ceiling_l15.py 비트 일치 | K10 0.6639 / K15 0.6649 | ✓ |
| GRU cache 형태 판별 | Cart vs Frenet | Cart 0.5865 / **Frenet 0.6659** (Δ vs #25 보고 0.6659 = 0.0000) | ✓ Frenet 채택 |
| GRU minor sanity | #25 보고 0.3181 ±0.005 | 0.3181 (bit-exact) | ✓ |
| Classifier accuracy guard | 3-seed mean OOF acc > 0.55 (majority 0.5174 + 3.26pp) | **0.5246** (+0.72pp) | **✗ FAIL** |
| AUC sanity | > 0.55 | **0.5139** | ✗ FAIL |
| Fold variance | std < 0.020 | 0.0134 | ✓ |

→ V3 15가 K10/K15 분기에 거의 random — accuracy guard 미달 정량 확정.

**Agreement breakdown (10000 sample)**

| Pair | both hit | A only | B only | neither | 회수 영역 |
|---|---:|---:|---:|---:|---:|
| K10 / K15 | 6534 | 105 (K10) | 115 (K15) | 3246 | 220 (2.2%) |
| **K15 / GRU** | **6438** | **211 (K15)** | **221 (GRU)** | **3130** | **432 (4.3%)** |

→ K15/GRU 회수 영역 K10/K15의 **2배**. backbone heterogeneity (LGBM vs Frenet GRU) > same-backbone K-subset diversity.

**Oracle ceiling (per-sample best 가정)**

| Pair | overall | major | minor | Δo vs K15 | Δm vs K15 |
|---|---:|---:|---:|---:|---:|
| K15 (baseline) | 0.6649 | 0.7312 | 0.3105 | — | — |
| Oracle K10/K15 | 0.6754 | 0.7386 | 0.3371 | +0.0105 | +0.0267 |
| **Oracle K15/GRU** | **0.6870** | **0.7474** | **0.3638** | **+0.0221** | **+0.0533** |

→ K15/GRU ceiling이 K10/K15의 2.1배 (overall) / 2.0배 (minor).

**Classifier 결과 (3 seed × 5 fold, V3 15 feat → K10 vs K15 binary)**

| 지표 | 측정 | 비교 기준 | 해석 |
|---|---:|---|---|
| 3-seed mean accuracy | 0.5246 | majority baseline 0.5174 | +0.72pp — 사실상 랜덤 |
| 3-seed mean AUC | 0.5139 | random 0.5 | feature가 K10/K15 분기 거의 못 봄 |
| Fold std (mean) | 0.0134 | < 0.020 | 학습 안정성 OK (단 학습할 신호 없음) |
| K10 wins | 4826 (48.26%) | random 50% | label 거의 균형 |
| K15 wins | 5174 (51.74%) | — | majority class baseline |

**3 mode HR + 3-gate + 4th guard (vs K15 OOF 0.6649)**

| Mode | overall | major | minor | Δo | Δm | G1 (+0.0024) | G2 | G3 | G4 (+0.0020) | LB? |
|---|---:|---:|---:|---:|---:|:---:|:---:|:---:|:---:|:---:|
| Hard (p>0.5 → K10) | 0.6644 | 0.7309 | 0.3086 | −0.0005 | −0.0019 | ✗ | ✓ | ✓ | **✗** | ✗ |
| Soft (w·K10 + (1−w)·K15) | 0.6650 | 0.7302 | 0.3162 | +0.0001 | +0.0057 | ✗ | ✓ | ✓ | **✗** | ✗ |
| Minor→GRU (학습 0 rule) | 0.6661 | 0.7312 | 0.3181 | +0.0012 | +0.0076 | ✗ | ✓ | ✓ | **✗** | ✗ |

→ 3/3 4th guard fail. **LB 제출 0**.

**Hard mode recall analysis**

| 항목 | 값 |
|---|---:|
| selector가 K10 선택 | 930 sample |
| K10 only 회수 | 13 / 105 (12.4%) |
| K15 only 손실 | 18 / 115 (15.7%) |
| Net gain (hit) | **−5** (K10/K15 oracle max +105) |

→ selector 정확도가 random에 가까워 회수(13) < 손실(18). hard switch 자체가 noise injection.

**Minor→GRU hard split recall (학습 0 lever, minority subset n=1575)**

| 항목 | 값 |
|---|---:|
| GRU only 회수 (minority 내) | 84 / 221 |
| K15 only 손실 (minority 내) | 72 / 211 |
| Net gain (hit, minority subset) | **+12** |

→ classifier 없이도 minority subset에서 net +12. acc_norm_last ≥ 5 규칙이 K15 우위 영역을 정확히 분리해 GRU 보완 효과 추출.

### 핵심 Findings + 교훈 (재사용 자산)

1. **V3 15는 K10/K15 분기 정보 본질적 부재(AUC 0.5139)**: same backbone(LGBM)+same Frenet target에서 feature subset 차이(10 vs 15)만 → 잔차 차이가 V3 15 공간서 분리 불가. **K-subset selector는 어떤 subset도 fail, selector value는 backbone family heterogeneity에서 나옴.**
2. **K15/GRU oracle ceiling +0.0221(minor +0.0533) = K10/K15(+0.0105/+0.0267)의 2배**: backbone heterogeneity(LGBM Frenet ↔ GRU Frenet, corr<0.95) > feature subset diversity. blend(#25)는 LB 패배(GRU LB<K15라 선형 envelope 미달)했으나 **per-sample selector는 별개 메커니즘**(envelope 외부 sample-level value 접근) → 다음 selector 1순위 K15/GRU.
3. **Hard mode net −5 = classifier noise injection**: K10 선택 930 중 회수 13(12.4%)<손실 18(15.7%). acc<(1−K15_only/(K10+K15))≈47.7% 임계 미만이면 hard switch 항상 음(ceiling×accuracy_recovery_factor 사전 측정 가능).
4. **Soft mode = neutralize, not amplify**(Δo +0.0001): random p_clf noise 평활→K15 수렴. weak classifier서 confidence threshold variant(p>0.7)가 noise 영역 회피에 더 효과적.
5. **minor→GRU hard rule(학습 0) net +12 = minority_mask 자체가 strong selector**: acc_norm_last≥5가 K15 우위 영역 정확 분리. classifier가 이 rule 못 이기면 lever 폐기(floor 자산), 강화 시 minority_mask strata 통합.
6. **GRU OOF cache form auto-detect = cache 재사용 표준 sanity**: metadata 부재 시 두 가설 HR 비교 후 보고치(0.6659) 일치 채택(Cart 0.5865 / Frenet 0.6659) → cache 손상 자동 검출.

**산출물**: `hr_aware_split_e31.ipynb`, `results/split_e31_meta.json`+clf oof/test seed{42,7,123}. **submission 0**(3 lever 모두 4th guard fail).

---

## #32 — hr_aware Split e32 (L16 K15/GRU Heterogeneous Backbone Selector) · 2026-05-28 후반

**설정**: plan.md §4.17. K15 (LGBM Frenet, EXP #29) ⊕ GRU (Frenet residual, EXP #25) per-sample selector. EXP #29 K15 OOF/test cache + EXP #25 GRU OOF/test cache 재사용 (모델 학습 0). V3 15 LGBM classifier (3 seed × 5 fold = 15 model). GRU cache form auto-detect (#31 framework 재사용). recovery-area AUC primary sanity (overall acc 0.55 임계는 recovery-focused classifier 부당 차단). 6 mode ablation (Hard / Soft / Thr p>0.7 / Thr p>0.8 / MR-Hard / MinorGRU rule).

**Sanity gates 결과**

| Gate | 기준 | 측정 | Pass? |
|---|---|---|---|
| K15 cache 재현 | EXP #29 비트 일치 | 0.6649 (overall) / 0.7312 (major) / 0.3105 (minor) | ✓ bit-exact |
| GRU cache 형태 판별 | Cart vs Frenet | Cart 0.5865 / **Frenet 0.6659** (Δ vs #25 보고 0.6659 = 0.0000) | ✓ Frenet 채택 |
| GRU minor sanity | #25 보고 0.3181 ±0.005 | 0.3181 (bit-exact) | ✓ |
| Classifier overall AUC | > 0.50 (sanity) | 측정값 단독 미보고 (recovery-AUC primary) | — |
| **Classifier recovery-AUC** | **> 0.55 (primary)** | **0.5547 (phase1_v15)** | **✓ (margin +0.0047)** |
| Fold std | < 0.020 | (3 seed mean) 안정 | ✓ |

→ #31 overall acc 0.55 → #32 recovery-area AUC 0.55 framework 전환 정량 정당화: recovery-focused classifier overall acc 상한 0.5216 (recovery 100% + 비-recovery 50% 가정) < 0.55 → 임계 부당.

**Agreement breakdown (K15/GRU, 10000 sample)**

| 분류 | count | 비율 |
|---|---:|---:|
| both hit | 6438 | 64.4% |
| K15 only | 211 | 2.1% |
| GRU only | 221 | 2.2% |
| neither | 3130 | 31.3% |
| **Recovery mask (K15_only ∪ GRU_only)** | **432** | **4.3%** |
| Recovery minority / majority | 156 / 276 | 36% / 64% |
| GRU wins (y_sel_kg=1) | 4474 | 44.7% (majority baseline acc 0.5526) |

→ #31 K10/K15 recovery 220 (2.2%) 대비 **2.0배**. backbone family heterogeneity (LGBM ↔ GRU Frenet) > same-backbone K-subset diversity.

**Oracle ceiling (per-sample best 가정)**

| Pair | overall | major | minor | Δo vs K15 | Δm vs K15 |
|---|---:|---:|---:|---:|---:|
| K15 baseline | 0.6649 | 0.7312 | 0.3105 | — | — |
| GRU solo | 0.6659 | 0.7309 | 0.3181 | +0.0010 | +0.0076 |
| **Oracle K15/GRU** | **0.6870** | **0.7474** | **0.3638** | **+0.0221** | **+0.0533** |

→ #31 K10/K15 oracle +0.0105 / +0.0267 대비 **2.1배 / 2.0배**. #31 Finding 2 정량 재현.

**6 mode HR + 4-gate + Floor (vs K15 OOF 0.6649, G1=√6·0.0014=+0.0034, G4=√2·0.0014=+0.0020, Floor=MinorGRU +0.0012)**

| Mode | overall | major | minor | Δo | Δm | G1 | G2 | G3 | G4 | Floor | LB? |
|---|---:|---:|---:|---:|---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Hard p>0.5 | 0.6647 | 0.7309 | 0.3105 | −0.0002 | −0.0000 | ✗ | ✓ | ✓ | ✗ | ✗ | ✗ |
| **Soft (w=p_clf raw)** | **0.6686** | **0.7332** | **0.3232** | **+0.0037** | **+0.0127** | **✓** | **✓** | **✓** | **✓** | **✓** | **✓** |
| Thr p>0.7 | 0.6649 | 0.7312 | 0.3105 | 0.0000 | 0.0000 | ✗ | ✓ | ✓ | ✗ | ✗ | ✗ |
| Thr p>0.8 | 0.6649 | 0.7312 | 0.3105 | 0.0000 | 0.0000 | ✗ | ✓ | ✓ | ✗ | ✗ | ✗ |
| MR-Hard | 0.6652 | 0.7312 | 0.3124 | +0.0003 | +0.0019 | ✗ | ✓ | ✓ | ✗ | ✗ | ✗ |
| MinorGRU rule (학습 0 floor) | 0.6661 | 0.7312 | 0.3181 | +0.0012 | +0.0076 | ✗ | ✓ | ✓ | ✗ | ✓ | ✗ |

→ **Soft 단독 5-gate 전부 통과** (LB 1슬롯 자격 확정).

**Recall analysis (recovery area: K15_only 211 / GRU_only 221)**

| Mode | GRU 선택 | GRU_only 회수 | K15_only 손실 | net hit |
|---|---:|---:|---:|---:|
| Hard p>0.5 | 34 | 1/221 | 3/211 | **−2** |
| Thr p>0.7 | 0 | 0 | 0 | 0 |
| Thr p>0.8 | 0 | 0 | 0 | 0 |
| MR-Hard | 113 | 7/221 | 4/211 | +3 |
| MinorGRU rule | 1575 (minority_mask 전체) | 84/221 | 72/211 | **+12** |

→ Hard p>0.5 net −2: #31 Finding 3 ("classifier acc < K15_wins/(K15_wins+GRU_wins) 임계 = 55.26%이면 hard switch net 음") 재현. recovery-AUC 0.5547 > sanity 0.55이지만 hard threshold용 임계 미달. Thr p>0.7/0.8 0 sample = `p_clf` 분포가 [0.5−ε, 0.5+ε]에 압축 (약한 classifier).

**LB 제출 결과 (2026-05-28 후반)**

| Submission | mode | OOF Δo | LB | Δ vs K15 LB 0.6906 | transfer rate |
|---|---|---:|---:|---:|---:|
| `submission_split_e32_soft_ms3.csv` | Soft (w=p_clf raw) | +0.0037 | **0.6910** | **+0.0004** | **11%** |

→ **(작성 시점 #32 기준) 누적 LB v3→현 +0.0232 (1등 0.70까지 −0.0090).** 현재 누적은 +0.0318(#38, "현재 채택 모델" 참조). post-hoc lever (#22 #28 #31 transfer fail 시리즈) 패턴을 **per-sample backbone heterogeneity selector**가 처음으로 깬 사례.

### 핵심 Findings + 교훈 (재사용 자산)

1. **heterogeneous backbone per-sample blend = post-hoc transfer fail 3시리즈(#22 −0.0024/#28 −0.0034/#31 fail) 깬 4번째 lever type**: OOF +0.0037→LB +0.0004 첫 양수 transfer. RESEARCH §10 가드에 "post-hoc shift / model parameter / hetero backbone blend" 3 type 구분 추가, K15/GRU 결합은 새 안전 영역.
2. **Soft 회수율 = oracle ceiling × heterogeneity factor**: K10/K15(homo) Oracle +0.0105→Soft 1% vs K15/GRU(hetero) +0.0221→**17%**(17배). homo 잔차 분산 작아 no-op, hetero는 weak classifier(recovery-AUC 0.5547)여도 ensemble 효과 → pair oracle 사전 측정으로 ROI 추정 가능.
3. **Thr p>0.7/0.8 0 sample** = p_clf [0.5−ε,0.5+ε] 압축(weak classifier). AUC<0.60 환경에선 threshold variant prior 낮음, raw probability scaling(T/cap/shrinkage)이 더 직접적.
4. **Hard p>0.5 net −2** = K15_only/(K15+GRU)=211/432=48.84% random 임계 근처(net≈ceiling×(acc−임계)/(1−임계)). hard switch는 recovery-AUC>0.60 필요한 별도 lever class(soft와 분리 평가).
5. **MinorGRU rule floor +0.0012** = backbone pair invariant minority selector(#31 K10/K15도 +0.0012 동일) = classifier가 acc_norm_last 미사용 신호 → 강화 시 minority_mask strata 통합 가치.
6. **Soft = raw p_clf single point(T=1, no cap, α=1)**: sub-lever space(T/cap/centered shrinkage) 미탐색 → #33 1순위 cost-0 lever(α<1이 K15 끌어당김 prior). **recovery-area AUC = selector sanity 표준**, LB 1일1회+4-gate+floor = over-tuning 차단 작동.

**산출물**: `hr_aware_split_e32.ipynb`, `_gen_split_e32_notebook.py` (있다면), `results/split_e32_meta.json`, `results/split_e32_clf_oof_seed{42,7,123}.npy`, `results/split_e32_clf_test_seed{42,7,123}.npy`, **`submission_split_e32_soft_ms3.csv` (LB 0.6910 = 직전 최고, #33으로 갱신됨)**.

---

## #33 — hr_aware Soft Sweep e33 (L17 K15/GRU Soft 고도화 3-D sub-lever sweep) · 2026-05-28 후반 ✓ LB 0.6914

**설정**: plan.md §4.18. #32 Soft baseline (LB 0.6910) 위에 3축 sub-lever sweep — σ T × max_w Δ × centered shrinkage α = **3×3×3 = 27 cell full grid**. K15/GRU cache (EXP #29 + EXP #25) + **#32 classifier OOF/test cache** 재사용 → 모델 학습 0 (Phase A). Phase B (조건부 V3 26 / Residual / Hybrid classifier 강화)는 Phase A floor 통과 시 자동 skip. 가드 framework은 **G1 informational + Floor primary** (27 cell sweep 시 G1 strict √27 ≈ +0.0073으로 baseline +0.0037 자체 통과 불가 → strict 부적합).

**3-D grid 정의**

| 축 | 값 | 적용 식 |
|---|---|---|
| σ (temperature T) | {0.7, **1.0**, 1.5} | `w_T = sigmoid(logit(p) / T)`. T<1 sharpen / T=1 base / T>1 soften |
| max_w (cap Δ) | {0.10, 0.20, **∞**} | `w_D = clip(w_T, 0.5−Δ, 0.5+Δ)`. Δ=∞ no cap |
| centered shrinkage α | {0.5, 0.75, **1.0**} | `w' = 0.5 + α(w_D − 0.5)`. α=1 base |

baseline cell (#32 Soft-base) = (T=1.0, Δ=∞, α=1.0). 적용 순서 `p → σ → max_w → α → w'`.

**Sanity gates 결과**

| Gate | 기준 | 측정 | Pass? |
|---|---|---|---|
| K15 cache 재현 | EXP #29 비트 일치 | 0.6649 / 0.7312 / 0.3105 | ✓ |
| GRU form auto-detect | Cart vs Frenet | Frenet 0.6659 채택 | ✓ |
| GRU minor sanity | #25 보고 0.3181 | 0.3181 bit-exact | ✓ |
| **#32 classifier 재현** | OOF recovery-AUC 0.5547 | 0.5547 | ✓ |
| **Soft baseline 재현** | OOF 0.6686 (#32 결과) | 0.6686 (Δo +0.0037 vs K15) | ✓ bit-exact |

→ Phase 0 통과 → Phase A 27 cell sweep 진입.

**27 cell HR + 가드 (Δo 분포 [+0.0034, +0.0038], 폭 0.0004)**

| Cell (#) | T | Δ | α | overall | major | minor | Δo | ΔM | Δm | G1i (+0.0073) | G4 (+0.0020) | F (+0.0037) | LB자격 |
|---|:---:|:---:|:---:|---:|---:|---:|---:|---:|---:|:---:|:---:|:---:|:---:|
| 1 (T07_D10_a050) | 0.7 | 0.10 | 0.50 | 0.6686 | 0.7332 | 0.3232 | +0.0037 | +0.0020 | +0.0127 | ✗ | ✓ | ✗ | ✗ |
| 3 (T07_D10_a100) | 0.7 | 0.10 | 1.00 | 0.6683 | 0.7331 | 0.3219 | +0.0034 | +0.0019 | +0.0114 | ✗ | ✓ | ✗ | ✗ |
| **18 ★ (T10_DINF_a100)** | 1.0 | ∞ | 1.00 | 0.6686 | 0.7332 | 0.3232 | +0.0037 | +0.0020 | +0.0127 | ✗ | ✓ | ✗ | ✗ (baseline) |
| **21 (T15_D10_a100)** | 1.5 | 0.10 | 1.00 | 0.6687 | 0.7332 | 0.3238 | **+0.0038** | +0.0020 | +0.0133 | ✗ | ✓ | **✓** | **✓** |
| **24 (T15_D20_a100)** | 1.5 | 0.20 | 1.00 | 0.6687 | 0.7332 | 0.3238 | **+0.0038** | +0.0020 | +0.0133 | ✗ | ✓ | **✓** | **✓** |
| **27 (T15_DINF_a100) ★LB** | 1.5 | ∞ | 1.00 | 0.6687 | 0.7332 | 0.3238 | **+0.0038** | +0.0020 | +0.0133 | ✗ | ✓ | **✓** | **✓** |

(전체 27 cell 표는 `results/soft_sweep_e33_meta.json` 참조. 위 표는 baseline + Floor 통과 3 cell + Bot-5 head)

| 분류 | count |
|---|---:|
| Floor (+0.0037) strict 통과 | **3 / 27** (T15_D10/D20/DINF_a100, 3 cell OOF 비트 일치) |
| 4th guard (+0.0020) 통과 | 27 / 27 (전부 통과, Δo 최저 +0.0034도 통과) |
| G1 informational (+0.0073) 통과 | **0 / 27** (sub-lever sweep 통계적 noise 영역 정량 확인) |
| LB eligible (Floor AND G4 AND G2 AND G3) | 3 / 27 |

**Marginal effect per axis (각 값마다 9 cell 평균 Δo)**

| 축 | 값 | Δo mean | ΔM mean | Δm mean | Floor 통과 / 9 |
|---|:---:|---:|---:|---:|---:|
| σ T | 0.7 | +0.0036 | +0.0019 | +0.0125 | 0 / 9 |
| σ T | 1.0 | +0.0037 | +0.0020 | +0.0124 | 0 / 9 |
| **σ T** | **1.5** | **+0.0037** | **+0.0021** | **+0.0125** | **3 / 9** |
| max_w Δ | 0.10 | +0.0036 | +0.0020 | +0.0123 | 1 / 9 |
| max_w Δ | 0.20 | +0.0037 | +0.0020 | +0.0125 | 1 / 9 |
| max_w Δ | ∞ | +0.0037 | +0.0020 | +0.0125 | 1 / 9 |
| α | 0.50 | +0.0037 | +0.0021 | +0.0123 | 0 / 9 |
| α | 0.75 | +0.0036 | +0.0019 | +0.0124 | 0 / 9 |
| **α** | **1.00** | **+0.0037** | **+0.0019** | **+0.0127** | **3 / 9** |

→ **T=1.5 + α=1.0 단독 조합만 Floor 통과** (3 cell, Δ축 무관). max_w와 α<1는 효과 없음 정량 확인.

**2-축 interaction** (Δo): **T×α에서만 +0.0038 도달**(T=1.5 & α=1.0 단독 cell), Δ축은 모든 panel 평균 동률(cap dead).

**Top-5 + Bot-5 rank (OOF Δo)**

| Rank | cell | Δo | ΔM | Δm | LB자격 |
|---|---|---:|---:|---:|:---:|
| Top-1 | T15_D10_a100 | +0.0038 | +0.0020 | +0.0133 | ✓ |
| Top-2 | T15_DINF_a100 | +0.0038 | +0.0020 | +0.0133 | ✓ |
| Top-3 | T15_D20_a100 | +0.0038 | +0.0020 | +0.0133 | ✓ |
| Top-4 | T10_D10_a050 | +0.0037 | +0.0021 | +0.0120 | ✗ (Floor 동률) |
| Top-5 | T10_DINF_a100 | +0.0037 | +0.0020 | +0.0127 | ✗ (baseline) |
| Bot-5 | T07_D10_a100 | +0.0034 | +0.0019 | +0.0114 | ✗ |

→ Top-3 OOF **비트 일치** (cap Δ 무관 dead 정량 직접 증거). simplicity tiebreak → Δ=∞ (no cap, baseline과 가장 가까운 형태) = **T15_DINF_a100** 채택.

**Recall analysis (K15/GRU recovery: K15_only 211 / GRU_only 221)**

| Mode | hard p>0.5 GRU 선택 | GRU_only 회수 | K15_only 손실 | net hit |
|---|---:|---:|---:|---:|
| Baseline (T=1, Δ=∞, α=1, raw p_clf) | 34 | 1/221 | 3/211 | −2 |
| **Best cell (T15_DINF_a100)** | **34** | **1/221** | **3/211** | **−2** |

→ baseline과 best cell **hard threshold 동일** (GRU 선택 34, net −2). HR Δo +0.0001 차이는 **Soft blend continuous weight가 boundary 근처 sample을 미세하게 다르게 push**한 결과 (minor 영역에서 ~6 sample 추가 hit = Δm +6 bp).

**LB 제출 결과 (2026-05-28 후반)**

| Submission | mode | OOF Δo (vs K15) | LB | Δ vs #32 LB 0.6910 | transfer rate |
|---|---|---:|---:|---:|---:|
| `submission_split_e33_T15_DINF_a100_ms3.csv` | Soft (T=1.5, Δ=∞, α=1.0) | +0.0038 (OOF +0.0001 margin vs #32) | **0.6914** | **+0.0004** | 400% (OOF +0.0001 → LB +0.0004) |

→ **(작성 시점 #33 기준) 누적 v3 → 현 +0.0236, 1등 0.70까지 −0.0086.** 현재 누적은 +0.0318(#38).

### 핵심 Findings + 교훈 (재사용 자산)

1. **T=1.5 soften 단독 dominant lever**: T marginal 0.7(+0.0036)<1.0≤1.5(LB자격 3/9), α=1.0만 LB자격 3/9. weak classifier(recovery-AUC 0.5547)서 T<1 sharpen=noise injection, T>1 soften=0.5쪽 압축→K15/GRU 균등 blend 접근→minor sub-1cm boundary push 추가 회수. **sub-tuning은 T축 우선**(T grid 정교화 → #35).
2. **max_w cap = dead lever 정량 확정**: T15_D10/D20/DINF_a100 3 cell Δo/Δm/ΔM **비트 일치**. T=1.5 후 p_clf [0.4,0.6] 자연 압축으로 cap 효력 0(sharpen-only 환경만 유효) → ablation 제외. simplicity tiebreak Δ=∞ 채택.
3. **centered shrinkage α<1 = T>1과 중복 메커니즘**: α=0.5=α=1.0 marginal(+0.0037), 둘 다 "0.5쪽 압축" 동일 → 하나만 충분, T=1.5가 미세 우월 → T축만 유지, α축 제거.
4. **G1 informational 0/27 = sub-lever sweep 통계적 noise 영역**: G1 strict √27≈+0.0073, 27 cell Δo 폭 0.0004=임계 5.5%. G1 strict 적용 시 baseline 자체 통과 불가 → **Floor(baseline 초과)+4th guard(noise 차단) primary framework**(N>10 cell sweep 표준, G1은 informational).
5. **LB transfer 400%(OOF +0.0001→LB +0.0004) = sub-noise zone 양의 흔들림**: sub-noise zone(OOF margin<combined_std×√2=+0.0020)에서 LB ±0.0003 흔들림, 단일 trial 정량 불가(OOF 신뢰 primary).
6. **#32+#33 cascade = +0.0008 LB**(hetero blend +0.0004 + sub-lever +0.0004 독립 작동): lever pipeline cumulative 작동 → 다음 lever도 위에서 평가 가능. **27 cell submission 전부→best 1개만 정책 반전**(memory `feedback-submission-best-only`, best만 생성+다운로드, 정량 자산은 meta.json 보존).

**산출물**: `hr_aware_soft_sweep_e33.ipynb`, `_gen_soft_sweep_e33_notebook.py`, `results/soft_sweep_e33_meta.json` (27 cell HR + 가드 + marginal + interaction + rank + floor count 전부 보존), **`submission_split_e33_T15_DINF_a100_ms3.csv` (LB 0.6914 = 누적 최고 갱신)**. 27 cell 중 LB 제출본 1개만 생성·다운로드 (memory `feedback-submission-best-only` 적용 후 정책 반전).

---

## #34 — P0 baseline-diversity cheap-screen (CA/damped/EWMA base family) · 2026-05-31 ✗ (prior 하향, LB 미제출)

**동기**: 외부 제안 — CV 단일 base 대신 CA(`p+2v+β·a`)/damped(`p+γ·2v`)/EWMA base family 후보로 per-sample selector(roadmap **L21**). raw CA0.5 HR 0.5993>CV 0.5788 주장. **LB 슬롯 전 로컬 numpy+G_S5-lite로 라인 생사 screen**.

**Q1 raw grid 재현**: CV raw **0.5787**=GPT 일치. CA fine-β(`base=CV+β·a_last·DT_PRED²`) peak β=0.15 **0.6008**(maj 0.6723/min 0.2184), β0.125 0.5993. GPT `a_step=a·DT²`(4× 작은 단위)→GPT β0.5=우리 β0.125. 물리 full CA(β0.5) 0.5131<CV — 작은 계수만 유효. acc-bin winner: 0-2 CV / 2-5 CA β0.25 / 5-10 damped γ0.95 / 10+ **damped γ0.90**(tail은 가속 추가 아닌 속도 damping). simple oracle 0.7355.

**Q4 oracle gain acc-bin 분포** (게이트 tail acc≥10 share>60%→진행): 추가 hit 1568개 중 **2-5 bin 58.6%(919)**, acc≥10 tail **4.9%**(≪40%). gain 78%가 dense majority → LGBM 학습 영역 → 흡수 예상. "tail underfit 생존" 가설 반증.

**Q6 흡수 R²** (delta_base=base_cand−base_CV Frenet 투영 → V3 15 OLS): CA delta_L R²=**1.0**(≡β·DT_PRED²·a_tang_last)/delta_N 0.95(2-5: 0.72)/delta_B 0.018(학습불가 축, HR 무관). damped delta_L R²=**1.0**(≡(γ−1)·DT_PRED·speed_last), delta_N=delta_B=0. 항등식 오차 ~1e-12. → **damped 100% 흡수, CA delta_L 100%**(유일 균열 dense delta_N R²=0.72, 비선형 LGBM 더 흡수).

**Q2/G_S5-lite** (CV vs CA β0.15 corrected OOF, seed42, V3 15): CV 0.6589 / CA 0.6546 → **Δ −0.0043, major −0.0053(G2 fail)**, minor +0.0013(noise). **raw +0.0221 → corrected −0.0043 역전.** #12 Kalman cannibalize 재현 + Q6 항등식으로 "왜 흡수되는지" 확정.

**핵심 Findings (재사용 자산)**: ① **raw HR≠corrected HR** — raw CA 우위 +0.022는 LGBM이 CV 위서 하는 보정과 같은 부분공간(base 선반영 시 major 보정 교란→net 음). base 변경 평가는 raw 금지, **G_S5(corrected OOF Δ vs incumbent) 필수**. ② **흡수 가능성=delta_base feature-span 투영 R²로 사전 측정 가능(신규 도구)** — R²→1이면 dead, damped류(단일 feature 선형) 구조적 R²=1. ③ simple oracle은 corrected 상한 아님(78% dense bin 흡수, ROI 추정 금지). ④ tail은 CA 아닌 damping 유리하나 100% 흡수+영역 5.8%→ROI 낮음. ⑤ **L21 multi-baseline/base 교체 prior 하향(측정 기반)** — base 차이가 LGBM 견딘다는 전제 거짓, "raw +0.022→corrected −0.0043 G2 fail"가 재실험 차단 자산.

**산출물**: `p0_baseline_screen.py`/`p0_absorb_screen.py`/`p0_gs5_lite.py`. 로컬 전용, Colab/LB/cache 미생성.

---

## #35 — hr_aware_t_grid_e34.ipynb (L18 T grid 정교화, sub-lever T 단일축 9-point) · 2026-05-31 ✓ LB 0.6916 (+0.0002, T=0.5 채택)

> **번호 주의**: plan.md §4.1은 본 lever를 "EXP #34"로 라벨했으나, 그 사이 #34가 P0 baseline-diversity screen(위 항목)에 선점됨 → 본 실험 = **#35**. 파일명(`*_e34*`, `submission_split_e34_*`, `t_grid_e34_meta.json`)은 생성 시점 라벨 유지.

**설계**: #33에서 T축 dominant(max_w/α dead) 확정 후 T=1.5가 grid {0.7,1.0,1.5} **경계 최댓값**이라 peak 미확인 → T 단일축 9-point fine grid {0.5~3.0}. Δ=∞·α=1.0 고정, 학습 0(cache 18개 reuse), Floor=#33 best +0.0038 strict + T=1.5 재선택 차단.

**Phase A T 곡선** (OOF Δo / Δm vs K15):

| T | 0.5 | 0.7 | 1.0 | 1.2 | 1.5 | 1.8 | 2.0 | 2.5 | 3.0 |
|---|---|---|---|---|---|---|---|---|---|
| Δo | **+0.0039** | +0.0036 | +0.0037 | +0.0037 | +0.0038 | +0.0037 | +0.0037 | +0.0036 | +0.0037 |
| Δm | +0.0120 | +0.0127 | +0.0127 | +0.0120 | **+0.0133** | +0.0127 | +0.0120 | +0.0114 | +0.0120 |

Δo span=0.0003≪std 0.0014 → **plateau**(T축 ceiling). best(인큐번트 제외) T=0.5 +0.0039(vs T=1.5 +0.0001), peak가 grid 경계(interior optimum 아님). T=0.5 Floor strict 통과 → **LB 0.6916(+0.0002 vs #33)**, T=0.5 채택(LB-best).

**핵심 Findings**: ① **T는 "선택" 아닌 "blend 세기"만 조절** — `sigmoid(logit(p)/T)`는 모든 T에서 0.5 교차 불변 → `w>0.5` 선택은 T 무관, recall(GRU_sel 34/rec 1/lost 3/net −2) T=0.5≡T=1.5 비트 동일, T는 비-flip 9966개 연속 가중치만 바꿈. ② +0.0002는 sub-noise 양의 흔들림 — plateau(span 0.0003) 안 coin-flip, soften/sharpen 구분 무의미 → "sharpen 좋다" prior 갱신 금지(minor는 여전히 T=1.5 우위). ③ **T축 라인 종료**(plateau 확정) → 다음=L19 Classifier 강화(recovery-AUC 0.5547→0.60+).

**산출물**: `hr_aware_t_grid_e34.ipynb`, `results/t_grid_e34_meta.json`(9 T-point 전부 보존), `submission_split_e34_T05_DINF_a100_ms3.csv`(LB 0.6916). 의존 cache=#33 동일 18개. Phase B 미진입.

---

## #36 — hr_aware_rnn_aug_e36.ipynb (L24 내부증강 + L25 per-step Frenet + L22 softhit RNN → 3-way selector) · 2026-06-01 ✓ LB 0.6920 (+0.0004 vs #35, sub-std·rnn_solo 채택)

**설계**: 외부 LB 0.6926 솔루션(RESEARCH 부록 D) 차용 묶음 L24+L22+L25. 신규 **RNN_aug**(GRU 2L h64, step 2..5 per-step Frenet 9ch, MarginSoftRHit α0.35/k350, trajectory 내부 sliding-window 5× 증강+vmax≥1.4 drop)를 K15/GRU에 얹어 **3-way multiclass selector**. 판정 **LB-only**(증강 OOF 크기 상이 → 부록 D).

**Phase 1~2 (OOF, real-window)**: RNN_aug 3-seed solo **0.6742**(maj 0.7335/min 0.3568) — K15(0.6649)·GRU(0.6659) solo 모두 상회, minority 최강. corr(residual) 3쌍 모두 0.983 → **G_RNN2 tripwire(≥0.95) 발동**(near-redundant). 3-way oracle 0.7065, GRU 추가 가치 +0.0093뿐.

**Phase 3 (selector+T sweep, vs floor 0.6688)**: selector(V3-15 multiclass) acc **0.3769≈chance(0.333)** — per-sample 선택 불가.

| mode | overall | major | minor |
|---|---|---|---|
| soft_T0.5 | 0.6725 | 0.7355 | 0.3352 |
| soft_T1.0 | 0.6738 | 0.7370 | 0.3359 |
| soft_T1.5 | 0.6741 | 0.7376 | 0.3346 |
| soft_T2.0 | 0.6741 | 0.7374 | 0.3352 |
| hard | 0.6697 | 0.7326 | 0.3333 |
| **rnn_solo** | **0.6742** | 0.7335 | **0.3568** |

모든 T(0.5~2.0) rnn_solo 미달(soft_T0.5 sharpen=chance-selector 확신 오선택→0.6725 최저). overall 동률대 → minority tiebreak로 **rnn_solo 채택**. **LB 0.6920**(+0.0004 vs #35, sub-std 0.0014).

**핵심 Findings**: ① **OOF→LB lift 축소가 "낮은 점수" 정체** — rnn_solo lift +0.0178 vs #35 +0.0228 → RNN_aug OOF ~0.0050 과대평가, 명목 이득(+0.0054)이 LB +0.0004로 증발(부록 D "OOF 절대비교 금지" 정량 확증). ② **equivalent-but-redundant** — 완전 다른 아키텍처인데 corr 0.983+oracle 0.706+selector chance+T-sweep≤solo → 활용 diversity 없음, 3-way/RNN backbone 라인 소진. ③ **plateau ~0.692 재확인**(#32~#36 전부 +0.000X sub-std) — 돌파엔 **OOF→LB 전이 저상관 신호원** 필요, correlated 모델 추가로는 불가.

**산출물**: `hr_aware_rnn_aug_e36.ipynb`, `results/rnn_aug_e36_meta.json`, `submission_rnn_aug_e36_PICK_rnn_solo.csv`(LB 0.6920). 의존 cache=K15 #29/GRU #25 + 신규 rnn_aug OOF·test seed{42,7,123}.

**채택**: LB-best 정책 → base를 rnn_solo(0.6920)로 교체, #35 보조자산 강등. +0.0004 sub-std라 prior 갱신 아닌 tie-break.

---

## #37 — hr_aware_rnn_axsig_e37.ipynb (L26 axis-σ RNN loss ablation, rnn_solo #36 fork) · 2026-06-01 ✗ 폐기 (axis-σ 무효, 제출 없음)

**설계**: rnn_solo(#36) base 위, MarginSoftRHit L1을 **Frenet 축별 가중** `w_axis=σ_axis^(−p)/mean(σ^(−p))`(mean=1 보존)로 교체. **p=0⇒uniform⇒#36 재현(control)**. 나머지(soft term·증강·α0.35·k350·fold) #36 동결, p-ablation, 판정 LB-only.

**Phase 0 (σ_axis 측정, vmax<1.4 pool)**: residual std L/N/B ratio **[1, 0.80, 0.63]** ≈ 11점 Frenet ref [1,0.81,0.62] 일치(비등방 real, L>N>B). robust(MAD) [1,0.998,0.840](outlier 제거 시 등방 근접).

**Phase 1 (probe {0,1}, seed42)**: p0(control) 0.6719(maj 0.7320/min 0.3505) / p1 0.6715(maj 0.7336/min 0.3390). p0=RNN_SOLO_SEED42 0.6719 정확 일치(코드 sanity). **Δ(p1−p0)=−0.0004**(noise ±0.0014 이내) → Stage 1 skip. **p=0(uniform)=best → axis-σ 무효, 라인 종료, 제출 없음.**

**핵심 Findings**: ① **residual 비등방(1:0.80:0.63) real인데 loss 축별 가중 무효** — 모델이 이미 soft term(α0.35) 등방 거리로 최적화 중 → L1 재가중과 상충/redundant, axis-σ는 loss reweighting으로 추출 불가. ② p1 trade: minor −0.0115/major +0.0016 — B축 강조가 minority 등방 hit 악화(minority는 L축 정확도가 hit 좌우). ③ L26 라인 소진(p0=best, p1 열위, robust-σ 등방), 채택 모델 불변.

**산출물**: `hr_aware_rnn_axsig_e37.ipynb`, `results/rnn_axsig_e37_*`(p0_control 보존). **submission 없음**. 의존 cache=rnn_solo #36.

---

## #38 — hr_aware_rotgate_e38.ipynb (L27 B Rotation-gated 물리 decoder, 외부 0.699 이식) · 2026-06-01 ✓ LB 0.6996 ⭐ (신 SOTA, +0.0076)

**설계**: 외부 LB 0.699+ `HyperPhysics_xy2`(`references/[LB_0.699+] 물리학 기반 추론 모델.ipynb`)를 **충실 이식** + honest 5-fold OOF + LB 직접 판정. **새 물리 = rotation gate**. 아키텍처 상세 = RESEARCH §2·부록 E.

**모델** (단일 MLP physics decoder):
```
pred_local = w_v·exp(−exp_v)·rodrigues(v_EMA, ω) + w_a·exp(−exp_a)·a_local
ω = (ω_hist 해석적 + ω_delta 학습) × rotation_gate
rotation_gate = σ((θ−θ_thr)·10)·σ((speed−speed_thr)·200)   # 저속·직진 회전 차단
```
축별 계수(w_v≈2/w_a≈1 prior), exp_v/exp_a=|v|·θ 다항식 감쇠(softplus≥0). Loss=soft-hit(1.3cm,k408)+129·MSE+0.051·NLL(log_var). 증강=등속외삽 sliding-window(w∈[3,11])+extended target+theta 가중 oversampling(급회전 5×), 학습형 EMA.

**honest 검증**: 5-fold StratifiedKFold(stratify=minority, seed{42,7,123}, 동일 partition). SlidingWindow는 trajectory 내부 윈도우만 → row-level **leakage-free**, val/OOF=real w=11→step-12.

**Phase A 스크린(로컬)**: 해석적 CTRV base가 **모든 gate θ_thr에서 ΔHR<0(overshoot)** — 모기 회전 transient라 CV near-optimal. ref gate(θ=1.09)는 0.4%(37개)만 발화, 흡수 raw R²≈0.99/smoothed 0.83~0.90. **prior 부정적**이나 NN은 ω를 감쇠 학습하므로 LB 직접 판정으로 진행.

**Phase 1~3 (Colab T4 ~75분)**:

| | overall | major | minor |
|---|---|---|---|
| **rotgate (3-seed OOF)** | **0.6765** | **0.7388** | 0.3435 |
| rnn_solo (#36 base) | 0.6742 | 0.7335 | 0.3568 |
| Δ vs rnn_solo | **+0.0023** | **+0.0053** | **−0.0133** |

seed OOF 42/7/123 = 0.6731/0.6752/0.6736(abort gate PASS). 가드: Floor O/4th O/G2 O/**G3(minor) X(−0.0133)** → **G3 hard-stop 의도적 override**(사용자 판정). **LB 0.6996**(+0.0076 vs rnn_solo). 1등 0.70까지 −0.0004.

**핵심 Findings (전부 prior 반전)**: ① **물리 제약 NN은 OOF→LB lift가 더 큼 = #25 "NN 과적합" 반례** — rotgate lift **+0.0231**(0.6765→0.6996)>rnn_solo +0.0178, 명목 OOF +0.0023이 LB +0.0076로 증폭. 강한 물리 prior 위 작은 보정만 학습→train-fold 과적합 적음→generalization 우위. "제약 없는 NN이 진 것이지 NN이 진 게 아니다" 확증. ② **#36 결론("저상관 새 물리 필요") 직접 충족** — 회전=CV와 직교 곡선 신호, plateau를 단일 step +0.0076 돌파(noise-zone lever와 본질 다름). ③ **G3 minority hard-stop은 신 아키텍처에 false-negative** — minor −0.0133(곡률 overshoot)인데 LB +0.0076, structurally-different 모델은 overall+major가 minority 손실 LB 상쇄 → overall-primary 판정(minority hard-stop은 동일 backbone W-schedule에 한정). ④ **회전은 base/feature 아닌 decoder로만 생존** — 고정 CTRV ΔHR<0인데 NN 감쇠 회전 +0.0076, 핵심=ω 감쇠 학습+gate(#34 "raw 회전 흡수" 교훈과 양립).

**산출물**: `hr_aware_rotgate_e38.ipynb`, `submission_rotgate_e38.csv`(LB 0.6996), `results/rotgate_e38_{oof,test}_seed{42,7,123}.npy`(ABSOLUTE pred_global)+meta. Phase A: `p0_ctrv_screen.py`/`p0_ctrv_gate_sweep.py`. **제출 재현=test_seed 3개 평균만으로 가능**(solo, 외부 cache 없음).

**채택**: base를 rotgate(0.6996)로 교체, rnn_solo #36/K15·GRU #35 보조자산 강등. +0.0076=sub-std 5배 이상 → **plateau 돌파·prior 갱신 확정**(tie-break 아님). dual-base 기준도 rotgate로 갱신.

---

## 📌 현재 채택 모델

**Rotation-gated 물리 decoder (#38), LB 0.6996 ⭐** (1등 0.70까지 **−0.0004**). 외부 0.699 `HyperPhysics_xy2` 충실 이식한 **단일 MLP physics decoder** — K15/GRU·RNN_aug와 별개 lineage. plateau(~0.692) 돌파, **prior 갱신 확정**(+0.0076=sub-std 5배 이상).

- **모델**: `pred = p_last + R·[w_v·e^(−exp_v)·rodrigues(v_EMA,ω) + w_a·e^(−exp_a)·a_local]`. ω=(ω_hist+ω_delta)·gate, gate=`σ((θ−θ_thr)·10)·σ((speed−speed_thr)·200)`(저속·직진 회전 차단). 축별 계수(w_v≈2/w_a≈1 prior), exp=|v|·θ 다항식 감쇠. **새 물리=rotation gate**. Loss=soft-hit(1.3cm)+129·MSE+0.051·NLL, 증강=sliding-window+theta oversampling, 학습형 EMA. 식 상세=RESEARCH §2·부록 E.
- **학습**: honest 5-fold(stratify=minority, seed{42,7,123}), val=real w=11, 3-seed test 평균, **solo**(외부 cache 없음). **직전 base(강등)**: rnn_solo #36(0.6920)/K15·GRU #35(0.6916).
- ✅ **채택 근거**: OOF 0.6765(Δo +0.0023)인데 **LB +0.0076** — lift +0.0231(rnn_solo +0.0178보다 큼), 물리 제약이 과적합 적어 generalization 우위(#25 "NN<tree" 반례). minor −0.0133(G3 fail)은 곡률 overshoot이나 major+overall 이득이 LB 압도 → **G3 override 정당**(신 아키텍처는 overall-primary).

자세한 파이프라인·feature·sample weight 메커니즘은 **RESEARCH.md §2~§6**(K15/GRU 보조자산) / **부록 E**(rotgate 아키텍처) 참조. 다음 lever roadmap은 **plan.md §4** 참조.

**Cumulative LB**: v3(#10) 0.6678 → … → ★#38 rotgate **0.6996** (누적 **+0.0318**, plateau 돌파·신 SOTA). 단계별 = 상단 「Cumulative LB」 표. 1등 0.70까지 −0.0004.

**산출물 (현 채택 #38)**: `submission_rotgate_e38.csv`(LB 0.6996), `hr_aware_rotgate_e38.ipynb`+meta, Phase A `p0_ctrv_screen.py`/`p0_ctrv_gate_sweep.py`. **제출 재현**=`results/rotgate_e38_test_seed{42,7,123}.npy` 3개 평균만으로 가능(solo). **노트북 재현**=`traj_train/y_train/traj_test.npy`만 필요.

**보조 캐시**(폐기 lever 재시도 보존): `submission_split_e34_T05_DINF_a100_ms3.csv`(0.6916, #35) / `_e33_T15_`(0.6914) / `submission_t_ax_C1_ms3.csv`(0.6906, K26) / `_t_v3_`(0.6888) / `_e32_soft_`(0.6910) / `topk_K15`(0.6906).
