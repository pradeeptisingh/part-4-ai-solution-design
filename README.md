# Part 4: AI Solution Design for a Business Problem

## Project Overview

This repository contains an AI solution design for a healthcare business problem: medical image triage in radiology departments.

The solution uses the Healthcare entry from the AI use case reference catalog and connects the design to the provided business KPI sample.

## Selected Use Case

| Item | Choice |
|------|--------|
| Domain | Healthcare |
| Business problem | Medical image triage |
| AI task type | Image classification |
| Recommended model | ResNet-50 transfer learning model |
| Main users | Radiologists, emergency physicians, hospital operations teams |

## Repository Structure

```text
part-4-ai-solution-design/
├── README.md
├── solution_report.md
└── diagrams/
    └── solution_architecture.png
```

## Solution Summary

Radiology departments often review scans in a first-in-first-out queue. This can delay critical findings such as pneumothorax, intracranial bleeding, or pulmonary embolism. The proposed AI system classifies incoming X-ray or CT images into `normal`, `urgent`, or `critical`, then prioritizes the radiologist worklist.

The system is designed as decision support. It does not replace radiologists and does not make final diagnoses.

## Expected Business Impact

The provided KPI sample shows the following baseline averages:

| KPI | Baseline average | Target after AI |
|-----|------------------|-----------------|
| Manual processing hours/month | 453.83 hours | about 150 hours |
| Average resolution time | 30.27 hours | under 8 hours |
| Error rate | 6.96% | under 2% |
| Customer satisfaction score | 7.04 / 10 | 8.5+ / 10 |

## Key Responsible AI Controls

- Human-in-the-loop review for all scans
- High recall threshold for critical cases
- Bias audits by age, sex, scanner type, and site
- DICOM de-identification and privacy controls
- Audit logs for every prediction and radiologist override
- Shadow-mode validation before production deployment

## Files

- `solution_report.md`: Full eight-task assignment response
- `diagrams/solution_architecture.png`: End-to-end architecture diagram
