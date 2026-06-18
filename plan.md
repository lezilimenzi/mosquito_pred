# 모기 비행 궤적 예측 — 대회 회고 (완료)

> 🏁 **대회 종료 (2026-06-12): 최종 LB 0.7002 · 최종 순위 12등.**

> 본 문서는 종료된 대회의 **회고 요약**이다. 진행 중에는 구현 스펙 + 다음 lever roadmap 문서였으나, 대회 종료로 회고로 정리됨. **설계·아키텍처 상세 = [RESEARCH.md](RESEARCH.md), 전체 실험 로그·실험별 protocol = [EXPERIMENTS.md](EXPERIMENTS.md), 실제 코드 = 각 노트북.** 트림된 셀 구성·미시도 roadmap의 고유 정보는 이 세 곳에 모두 보존된다.

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

> *#35의 옛 파일명은 생성 시점 라벨 `e34` 유지. 미채택 실험의 노트북·Colab 캐시는 저장소에 없음 — EXPERIMENTS.md 각 entry의 `산출물` 기록으로만 보존.

---

## 1. 결과 요약

- **최종 LB 0.7002, 최종 순위 12등.** 최종 채택 모델 = **Rotation-gated 물리 decoder** (단일 MLP physics decoder, EXP #38 / `10_rotgate.ipynb`). 아키텍처 상세 = [RESEARCH.md](RESEARCH.md) §2.
- 두 국면으로 전개: **(A) cascade 라인**(#14~#35, LGBM Frenet residual + per-sample selector)이 데이터 이해로 LB를 ~0.692까지 끌어올렸고, **(B) rotgate**(#36·#38, 별개 단일 MLP lineage)가 **회전(곡선 외삽) 물리**로 plateau를 돌파했다.

**LB 사다리 (v3 → 최종)**

```
v3 baseline (#10)                          LB 0.6678
 → Frenet target (#14)                     0.6878  (+0.0200)
 → multi-seed (#15)                        0.6888  (+0.0010)
 → axis-wise σ 가중 (#17)                   0.6906  (+0.0018)
 → V3 26→15 simplicity (#29)               0.6906  (±0.0000)
 → K15/GRU Soft selector (#32)             0.6910  (+0.0004)
 → T=1.5 soften (#33)                      0.6914  (+0.0004)
 → T=0.5 (#35)                             0.6916  (+0.0002)
 → RNN_aug solo (#36, 별개 lineage)         0.6920  (+0.0004)
 → ★ Rotation-gated decoder (#38)          0.6996  (+0.0076, plateau 돌파)
 → 최종 제출                                0.7002  (12등)
```

---

## 2. 효과 있던 lever (채택 경로)

| 실험 # | 저장소 파일 | lever | 왜 효과 | LB Δ |
|---|---|---|---|---:|
| #14 | `02_frenet_target.ipynb` | 잔차를 motion-aligned(Frenet L/N/B) 축에서 학습 | feature·target 정렬로 학습 신호 강화 — 단일 lever 최대 효과 | +0.0200 |
| #15 | `03_frenet_multiseed.ipynb` | fold seed 3개 ensemble + feature 축소 | OOF 분산↓, 동률이면 단순한 쪽 채택 | +0.0010 |
| #17 | `04_axiswise_sigma.ipynb` | Frenet 잔차 std 비대칭(L:N:B=1:0.81:0.62)에 비례한 boundary 가중 | 대칭 Cart엔 무효였던 가중을 비대칭이 활성화 | +0.0018 |
| #29 | `05_topk_k15.ipynb` | feature 26→15 (gain 상위만), LB 동률 | 핵심 신호는 상위 15개로 충분 — 42% 단순화 | ±0.0000 |
| #32 | `06_split_selector.ipynb` | LGBM⊕GRU per-sample Soft blend | 이질 backbone(corr<0.95) sample별 가치 추출 | +0.0004 |
| #33 | `07_soft_sweep.ipynb` | selector 가중 logit-space T=1.5 soften | 약한 classifier 환경에서 균등 blend 접근 | +0.0004 |
| #35 | `08_t_grid.ipynb` | T 9-point fine grid → T=0.5 | T축 plateau 확정 | +0.0002 |
| #36 | `09_rnn_aug.ipynb` | per-step Frenet GRU + 내부 sliding-window 증강 (별개 lineage) | 단일 RNN이 cascade와 LB 동급 → backbone 교체(tie-break) | +0.0004 |
| **#38** | **`10_rotgate.ipynb`** ⭐ | **회전 gate + 감쇠 물리 decoder 단일 MLP** | **신 SOTA. 없던 곡선 외삽 물리로 minority 직격, OOF→LB lift +0.0231로 일반화 우위** | **+0.0076** |

> 상세 설계·근거 = RESEARCH §2(rotgate)·부록 C(cascade). 각 실험의 OOF/가드/산출물 = EXPERIMENTS.md 해당 #.

---

## 3. 폐기 lever 핵심 교훈 (재사용 자산)

대회 중 폐기한 lever들에서 얻은 일반 규칙. 상세 수치 = EXPERIMENTS.md, 메커니즘 상세 = RESEARCH 부록 B.

- **base 변경 = 후속 잔차 모델이 흡수(cannibalize)** (#12 Kalman / #19 Latency / #34 base-diversity). base 차이가 v/a에 선형이면 feature span에 흡수 R²≈1 → 구조적 dead. rotgate는 rotation_gate로 직진·저속을 꺼서 이를 *구조적으로 회피*하며 곡선 보정만 추가.
- **EDA-feature 추가 = OOF→LB transfer fail 3연속** (#11 D6 / #13 F / #18 F-INT). 수학적 새 정보 평면도 모델이 기존 feature 조합으로 재구성 → no-op/noise. marginal corr·univariate R²만으론 채택 금지.
- **invariant 변환 증강 = 단순 upweighting** (#16 T-AUG). Frenet basis가 trajectory에서 계산되면 회전/미러 시 (X,y) 모두 불변 → 가중과 충돌. 유효 조건 = 진짜 새 trajectory(rotgate 내부 sliding-window).
- **hetero-backbone blend 양방향 closed** (#25 NN / #26 tree-tree). blend LB ≈ component 단독 LB의 선형 평균 → 하나라도 base 미달이면 blend ≤ base. (per-sample selector는 별개 메커니즘 — #32 채택.)
- **post-hoc lever(calibration/TTA) transfer fail** (#22 / #28). OOF micro-positive(Δo < noise)는 LB transfer 보장 안 함 — LB 시도 자격 = backbone 동질 + Δo 충분 + **model parameter 변경**, 셋 다 충족.

---

## 4. 프로세스 회고

- **LB 1일 1회 제약** → OOF가 1차 판정자, LB는 채택 확정용. OOF→LB lift는 동일 family 내 상대 비교로만 신뢰.
- **OOF→LB transfer는 lever type·backbone family에 의존** — "4-가드"(backbone 동질 · Δo>noise · model parameter 변경 · Δo>std·√2) framework으로 약한 lever의 LB 슬롯 낭비를 막았다.
- **simplicity tiebreak (Karpathy)** — OOF noise zone 동률이면 단순한 쪽. #15(38→26)·#29(26→15) feature 축소가 LB로 정당화됨.
- **증강 OOF는 낙관** → 채택은 LB로만 판정, OOF 절대비교 금지.
- **plateau 돌파는 미세 튜닝이 아니라 새 물리에서 나왔다** — sub-lever 누적(#32~#35) 합계 +0.0010 vs 회전 물리 한 방(#38) +0.0076. 정체 구간에선 같은 라인 미세조정보다 *구조적으로 없던 신호*(여기선 곡선 외삽)를 찾는 것이 ROI가 압도적.
