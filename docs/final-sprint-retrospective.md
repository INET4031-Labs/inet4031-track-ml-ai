# Final Sprint Retrospective

**Sprint:** Final
**Date:** TODO
**Scrum Master:** TODO

## Sprint Goals

- Generate a labeled synthetic incident dataset (5,000+ rows, 4+ categories, 3+ severity levels) reproducibly from a seed
- Train a model and commit evaluation metrics with a baseline comparison (severity accuracy at least 0.80)
- Deploy the inference service in the k3d cluster with memory requests/limits, answering in under 500 ms
- Protect the service with its own NetworkPolicy and show blocked-traffic evidence
- Keep additions under 512Mi of memory limits and the image under 600 MB

## What Went Well

TODO: List 2-3 things the team did well this sprint.

## What Could Improve

TODO: List 2-3 things that were difficult or took longer than expected.

## Process Change for Next Sprint

TODO: If we had another sprint, describe one specific change the team would make based on what you learned.

## Blockers Encountered and Resolved

TODO: List any technical or process blockers. How did the team resolve them? Did they require help from a TA?

## Metrics Achieved

- TODO: Dataset rows generated: ___
- TODO: Baseline severity accuracy: ___
- TODO: Model severity accuracy: ___
- TODO: Macro F1 (category / severity): ___ / ___
- TODO: Request latency measured: ___ ms
- TODO: Memory limit total of added pods: ___ Mi
- TODO: Image size: ___ MB
- TODO: Blocked-traffic test result: ___

## Team Communication During Sprint

TODO: How often did the team check in? Did async communication work well? Any scheduling conflicts?

## Decisions Made This Sprint

TODO: List major technical or team decisions (labeling rules, model choice, who may call the service, memory limit values).

## Team Contributions

| Role | Name | Primary Contribution |
| ---- | ---- | -------------------- |
| Scrum Master | TODO | TODO |
| System Admin | TODO | TODO |
| Developer | TODO | TODO |
| Developer | TODO | TODO |
| QA | TODO | TODO |

## Individual Reflection (Local VM only)

1. TODO: What did you build?
2. TODO: What surprised you?
3. TODO: What would you do next?
4. TODO: Which success criteria did you not meet, and why?
