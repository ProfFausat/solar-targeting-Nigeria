# Feature Harvest Plan — Solar Targeting (Ensemble Round)

*Planning artifact. Written before any ensemble code, on the Findex principle: enumerate every eligible variable, screen by explicit criteria, then let the forest rank and prune by evidence. Nothing here is modelled yet — this is the decision record for what enters the candidate matrix and why.*

**Target (unchanged):** `E_3` — willingness to pay the randomly offered price upfront for a solar device. 29% positive.

---

## The screening rule (apply to every candidate)

A variable earns a place in the candidate matrix only if it passes **all three** tests:

1. **Knowable before outreach.** The solar company must be able to obtain or estimate it for a household it has *not yet contacted*. If the only way to know it is to have already run the sales conversation, it cannot be a predictor — it is an outcome.
2. **Not downstream of the target.** It must not be recorded as a consequence of, or conditional on, the `E_3` willingness question. This is the leakage test, and Section E is where it bites hardest.
3. **Plausible mechanism.** There is a defensible reason — economic, infrastructural, or behavioural — why it would move willingness to pay. We screen wide, but not blind; every candidate carries a one-line rationale.

Tree ensembles tolerate wide, correlated, mixed-type feature sets gracefully, so the bar for tests 1–3 is "passes," not "is among the best." The *forest* decides what is best; our job is to hand it an honest, leakage-free menu.

---

## Verdict A — KEEP (carry into the candidate matrix)

### Already in the single-tree model (7) — retain
| Feature | Source | Rationale |
|---|---|---|
| Offered price (randomized) | `solar_price`, Sec E | The experimental driver; dominates by design (imp. 0.67). |
| Urban / rural | Sec B | Infrastructure and cash-economy proxy (added no split power alone — keep; forests may find interactions). |
| Head's education | `A9`, Sec A1 → head | Human-capital / income proxy. |
| Grid connection | `E1` / `H1` | Baseline electrification; shapes substitute value of solar. |
| Household size | roster | Demand scale and budget pressure. |
| Formal bank account | `B16` | Financial inclusion → ability to transact / finance. |
| Mobile money account | `B21` | Directly relevant: PAYG solar is billed over mobile money. |

### New candidates — dwelling quality & SES (Section B)
| Feature | Source | Rationale |
|---|---|---|
| Dwelling type | `B5` | Standard SES marker. |
| Owns dwelling | `B7` | Asset security; owners invest in fixed improvements. |
| Number of rooms | `B9` | Wealth / crowding proxy; scales lighting need. |
| Wall material | `B10` | Classic SES ladder (you know these well). |
| Roof material | `B11` | SES; also proxies roof suitability for solar. |
| Floor material | `B12` | SES. |
| Toilet facility | `B13` | SES / living standard. |
| Main water source | `B14` | SES / infrastructure access. |
| Years in dwelling | `B4` | Residential stability → willingness to invest. |

### New candidates — financial depth & credit (Section B)
| Feature | Source | Rationale |
|---|---|---|
| Informal savings group | `B18`/`B19` | **PAYG-critical:** habit of periodic contributions predicts capacity for instalment payment. |
| Loan / credit access | `B20` | Access to credit → affordability of upfront or financed purchase. |

### New candidates — revealed lighting demand (Section F) — *the client's sales argument*
| Feature | Source | Rationale |
|---|---|---|
| Uses candle / wick / hurricane / pressurized lamp | `F3__1–4` | A household already buying fuel-based light has *revealed* illumination demand. |
| Monthly fuel-for-lighting spend | `F20` (+ `F21` share) | A revealed budget line solar can capture — the strongest commercial signal in the data. |
| Lighting source for children studying | `F_15` | Demand intensity: education-driven lighting need. |

### New candidates — assets, incl. the phone variable (Section N)
| Feature | Source | Rationale |
|---|---|---|
| Asset count / index | `N_electrical_*` | Aggregate ownership = wealth proxy; robust and forest-friendly. |
| Smartphone | `N_electrical_25` | **The phone variable we hunted:** connectivity + mobile-money capability. |
| Regular phone / charger | `N_electrical_26` | Charging need is a core off-grid solar use case. |
| TV / radio / fan | `N_electrical_6,8,27–29` | Appliance demand implies value placed on power. |

### New candidates — spending power & composition (Sections L, A1)
| Feature | Source | Rationale |
|---|---|---|
| Total household consumption | Sec L_CONSUMPTION | Direct ability-to-pay proxy — the income measure we located and never merged. |
| Head age | `A5` → head | Life-cycle effects on adoption. |
| Head gender | `A3` → head | Adoption signal **and** required for the planned fairness diagnostics. |
| Head occupation | `A14` → head | Income type / stability. |
| Number of children / dependents | roster | Composition beyond raw size; lighting-for-study demand. |

### New candidate — supply reliability (Section H)
| Feature | Source | Rationale |
|---|---|---|
| Grid hours / reliability (12 mo) | `H_2__*` | **Sharper than the binary grid flag:** connected-but-unreliable households are prime solar buyers. |

### New candidate — geography
| Feature | Source | Rationale |
|---|---|---|
| State / LGA | identification | Regional demand variation; encode carefully (target/ordinal, watch cardinality). |

---

## Verdict B — DROP (fails a test)

| Variable | Why dropped |
|---|---|
| `E_3b`, `E_4`, `E_5`, `E_6`, `E_7` (+ `_b`/`_oth`) | **Leakage — the decisive exclusion.** These are the price-bargaining follow-ups and "why did you refuse" fields, recorded *after and conditional on* `E_3`. They exist only because the outcome already happened. Including them would inflate the score and collapse in deployment, where they don't exist. |
| `solar_full_price`, `solar_system` (tier) | Design constants collinear with `solar_price`; add noise, not signal. |
| Current solar ownership (Sec C_SOLAR) | 1.2% prevalence, near-zero information; also conceptually the rejected target. |
| `missing_*` counts, `*_start`/`*_end`, enumerator IDs | Survey-administration artifacts, not household attributes. |
| `*_oth` free-text specify fields | Sparse, unstructured; no modelling value without manual coding. |

---

## Two data-engineering cautions before merging

1. **Unit of analysis.** Section A1 is *person-level* (22,597 rows over 3,669 households). Every A1 feature must be reduced to the household — filter to the head (via `A4` relationship) for head attributes, or aggregate (counts, means) for composition. Merging it raw will multiply rows and silently corrupt the target join. This is the same row-count discipline from notebook 02.
2. **Long-format fuel table.** `F_FUEL` has one row per fuel type per household. Pivot to household level (e.g. total `F20` spend) before merging — do not join long.

---

## What happens next (the modelling discipline)

1. Build the candidate matrix from Verdict-A variables, applying the two cautions above; re-run the row-count assertion after every merge.
2. **Log every candidate set and model run in MLflow** — this feature sweep across tree / random forest / gradient boosting is exactly the experiment-tracking problem MLflow exists for.
3. Fit the three models; read feature importances **and** a permutation-importance check (importances on wide correlated sets can mislead).
4. **Prune by evidence, not by taste:** drop only what the forest ranks near-zero *and* has no business rationale to retain.
5. Judge everything by the standing business metric — **precision at the top-20%** — never raw accuracy.

**Open question to hold:** with fuel spend (`F20`) now in the matrix, watch whether it rivals the offered price for importance. If it does, that is not just a modelling result — it is the single most persuasive line in the client brief.
