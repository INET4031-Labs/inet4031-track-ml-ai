# QA Report: Final Sprint, Machine Learning and AI

| | |
|---|---|
| **Section** | Shared VM (team ___) / Local VM |
| **Test author(s)** | ______________________ (Shared VM: QA) |
| **Tester** | ______________________ (Local VM: you are author and tester) |
| **Commit tested** | `git rev-parse --short HEAD` = ________ |
| **Dates** | Cases written: ________ Cases run: ________ |

## How to use this report

1. **Write the test cases first,** in Week 11, before the work they test exists. A test written after the fact tends to test what was built instead of what was asked for.
2. **Write the expected result before you run anything.** If the actual result surprises you, that is a finding, not a typo to fix.
3. **Paste real output or point to an evidence file.** "Works" is not a result. A command and its output is.
4. **Cover every success criterion with at least one case,** and include at least one negative test (something that must fail or be blocked) and the rebuild proof of life.
5. **Re-run every case in Week 13** on the rebuilt VM and update the status. A case that passed only on the old VM is a rebuild gap.

If your team replaced a criterion through [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md), test the replacement instead, and link the approved design brief.

---

## Test Cases

Copy the block for each new case. Six slots are a starting point, not a limit.

<details>
<summary>Example of a filled-in case (delete when you start)</summary>

**TC-00: Service rejects traffic from an unlisted pod**

| Field | Value |
|---|---|
| Criterion | 4 |
| Steps | Apply `blocked-client` (memory-limited), run `wget -T 5` against the service, save output, delete the pod |
| Expected (written before running) | The request times out; the allowed client still gets a category, severity, and confidence |
| Actual | See `capstone/ml-ai/evidence/<file>`; summary: ... |
| Status | Pass |
| Run on | Original VM, YYYY-MM-DD; rebuilt VM, YYYY-MM-DD |

</details>

### TC-01: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-02: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-03: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-04: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-05: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-06: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

---

## Success Criteria Verification

Evidence paths are relative to `capstone/ml-ai/` and come from [EVIDENCE-CHECKLIST](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/EVIDENCE-CHECKLIST.md). Change a path if your file is named differently.

| Criterion | Evidence | Test case(s) | Met? |
|---|---|---|---|
| 1. Labeled synthetic dataset of 5,000+ rows, 4+ categories, 3+ severities, reproducible from a seed | the dataset file, `evidence/dataset-hash.txt` | | [ ] Yes [ ] No |
| 2. Held-out evaluation: severity accuracy 0.80 or higher, macro F1 for both targets, baseline recorded | `metrics.json` | | [ ] Yes [ ] No |
| 3. Service in the cluster with memory requests and limits, answering in under 500 ms | `evidence/describe-deploy.txt`, `evidence/timing.txt` | | [ ] Yes [ ] No |
| 4. Only intended traffic reaches the service; an unauthorized pod is blocked | `evidence/netpol-blocked.txt` | | [ ] Yes [ ] No |
| 5. Additions under 512Mi of limits, image under 600 MB | `evidence/memory.txt`, `evidence/image-size.txt` | | [ ] Yes [ ] No |

---

## Rebuild Verification (Week 13)

| Check | Result |
|---|---|
| Rebuild date and VM (new VirtualBox VM, or Shared VM queue slot) | |
| Started from commit | |
| Proof of life, Shared VM: The generator reproduces the same dataset from the seed (`sha256sum` matches); the model loads; one request returns a category, a severity, and a confidence | [ ] Pass [ ] Fail |
| Proof of life, Local VM: Same as the Shared VM | [ ] Pass [ ] Fail |
| Every app change came back (schema, routes, services, images) | [ ] Yes [ ] No |
| Gaps logged in `capstone/ml-ai/REBUILD-GAPS.md` and fixed by pull request | ___ gaps found, ___ fixed |
| All test cases above re-run on the rebuilt VM | [ ] Yes [ ] No |

---

## Deliverables Verification

- [ ] Implementation committed under `capstone/ml-ai/` (merged by pull request)
- [ ] `capstone/ml-ai/RUNBOOK.md`: operate, check, and recover
- [ ] `capstone/ml-ai/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages)
- [ ] `capstone/ml-ai/ARCHITECTURE.md` with a diagram and a "What we changed and why" section
- [ ] "How to run this" section in the top-level README
- [ ] `capstone/ml-ai/REBUILD-GAPS.md` from the Week 13 rebuild
- [ ] This QA report, with every case run and signed
- [ ] Shared VM: `docs/final-sprint-retrospective.md`. Local VM: the Individual Reflection section
- [ ] `capstone/ml-ai/DESIGN-BRIEF.md`, only if a criterion was replaced, with the approval noted
- [ ] Discussion answers in the Google Doc
- [ ] Final presentation and Week 14 rebuild presentation prepared (live on the Shared VM, recorded on the Local VM)
- [ ] Track-specific: Generator, training, and evaluation scripts with a fixed seed
- [ ] Track-specific: Saved model file (small) or a documented command that recreates it
- [ ] Track-specific: `capstone/ml-ai/service/` with Dockerfile, `requirements.txt`, and service code
- [ ] Track-specific: Deployment, Service, and NetworkPolicy (Local VM: in `week-5/`)
- [ ] Track-specific: One documented wrong prediction and what it taught you

---

## Rework Required

| # | Failed case or criterion | What is wrong | Fix and owner | Re-test date | Re-test result |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

---

## Sign-Off

**Overall status:** [ ] All criteria met [ ] Met with documented exceptions [ ] Rework required

**Exceptions** (criteria not met, and why):

```

```

| Role | Name | Date |
|---|---|---|
| Test author | | |
| Tester (Local VM: same person) | | |

**Notes:** observations about technical quality and, on the Shared VM, team coordination.
