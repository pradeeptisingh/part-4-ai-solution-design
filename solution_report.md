# AI Solution Design Report
## Medical Image Triage System - Healthcare Domain

**Role:** AI Business Analyst  
**Reference files used:** AI use case reference catalog and business KPI sample  
**Selected catalog entry:** Healthcare - Medical image triage

---

## Task 1: Business Domain

**Selected domain:** Healthcare

Healthcare was selected because delays in clinical diagnosis can directly affect patient safety. The reference catalog lists Healthcare / Medical image triage as an image classification use case using X-ray or scan images, with recall, sensitivity, and review-time reduction as key evaluation measures.

---

## Task 2: Business Problem Definition

### Problem

Radiology departments receive a high volume of X-rays, CT scans, and other medical images. In many hospitals, scans enter a worklist in the order they are received. A critical case can therefore wait behind many routine cases.

The problem being solved is delayed prioritization of critical medical images.

### Users and Stakeholders

| Stakeholder | Role |
|-------------|------|
| Radiologists | Review scans and make final clinical decisions |
| Emergency physicians | Need fast results for urgent patients |
| Patients | Benefit from faster diagnosis and treatment |
| Hospital operations teams | Monitor throughput, queues, and service quality |
| Hospital IT / PACS team | Integrate the AI system into existing imaging workflows |
| Compliance and governance teams | Ensure safety, privacy, and auditability |

### Current Process

1. A scan is captured and stored in the hospital PACS.
2. The scan enters a radiology worklist.
3. A radiologist manually reviews scans, often in time order.
4. A report is written and sent to the treating physician.
5. Treatment decisions are made after the report is available.

### Limitations of the Current Process

- Critical scans may wait behind routine scans.
- Manual prioritization is slow during peak workload.
- Radiologists may face fatigue and backlog pressure.
- There is limited early warning before a scan is formally reviewed.
- The KPI sample shows baseline averages of 453.83 manual processing hours/month, 30.27 hours average resolution time, 6.96% error rate, and 7.04/10 customer satisfaction.

---

## Task 3: AI Task Type

**Selected AI task type:** Image classification

The model receives a medical image and predicts one of three priority classes:

- `normal`
- `urgent`
- `critical`

Image classification is suitable because the input is visual pixel data and the output is a fixed category. A CNN-based model can learn spatial patterns such as abnormal opacity, bleeding signs, fractures, or collapsed lung indicators.

If the hospital later needs exact abnormal-region localization, object detection or segmentation can be added. For this assignment, classification is the clearest fit for triage.

---

## Task 4: Data Requirement Plan

### Data Needed

| Data | Structure | Purpose |
|------|-----------|---------|
| X-ray or CT images | Unstructured image data | Main model input |
| Radiologist triage label | Structured categorical label | Target variable |
| Radiologist report | Unstructured text | Supports label creation and audit |
| Patient metadata | Structured tabular data | Optional model/context feature |
| Urgent referral flag | Structured binary data | Optional workflow signal |

### Input Features

Primary image input:

- DICOM image converted to model-ready image format
- Resized image, for example 224 x 224 pixels
- Normalized pixel values
- Optional contrast enhancement

Optional metadata:

- Age group
- Sex
- Modality type
- Scanner type
- Referring department
- Urgent referral flag

### Target Variable

`severity_class`: `normal`, `urgent`, or `critical`

Labels should be created from historical radiologist decisions and reviewed by clinical experts. Critical cases should receive special attention because they are rarer and higher risk.

### Data Collection Method

1. Extract historical images and reports from PACS/RIS systems.
2. Remove patient identifiers from DICOM metadata.
3. Map historical reports and triage outcomes to severity labels.
4. Ask radiologists to validate a sample or high-risk labels.
5. Continue collecting radiologist overrides after deployment for future model improvement.

### Data Quality Risks

| Risk | Mitigation |
|------|------------|
| Label noise | Use radiologist consensus for difficult cases |
| Class imbalance | Use class weights and stratified sampling |
| Different scanner quality | Include scanner diversity in training data |
| Missing or corrupted DICOM files | Add automated input quality checks |
| Privacy leakage | De-identify metadata and remove burned-in patient text |
| Distribution shift | Monitor performance by site and scanner type |

---

## Task 5: Model Recommendation

**Recommended model:** ResNet-50 with transfer learning

### Architecture

```text
Input medical image
-> Resize and normalize
-> ResNet-50 convolutional backbone
-> Global average pooling
-> Dense layer with ReLU
-> Dropout
-> Dense output layer with softmax
-> Output probabilities for normal, urgent, critical
```

### Why This Model Is Appropriate

- CNNs are designed for image pattern recognition.
- ResNet-50 is deep but stable because residual connections reduce vanishing-gradient problems.
- Transfer learning reduces the amount of labeled medical data required.
- The model can be fine-tuned for hospital-specific image patterns.
- Softmax output gives a probability for each triage class, which supports confidence thresholds and human review.

### Training Plan

- Start with ImageNet-pretrained weights.
- Freeze the backbone first and train the classification head.
- Fine-tune later layers using a smaller learning rate.
- Use class-weighted categorical cross-entropy to address rare critical cases.
- Use data augmentation such as rotation, zoom, and brightness variation.

---

## Task 6: Evaluation Plan

### Technical Metrics

| Metric | Purpose |
|--------|---------|
| Critical-class recall / sensitivity | Measures how often critical cases are caught |
| Critical-class precision | Measures false-alert burden |
| Macro F1 score | Balances performance across all classes |
| AUC-ROC | Measures class separation |
| Confusion matrix | Shows which categories are confused |
| Inference latency | Ensures the model does not slow the workflow |

The most important metric is critical-class recall because missing a critical scan is the highest-risk failure.

### Business Metrics

Using the KPI sample baseline:

| KPI | Baseline average | Target after AI |
|-----|------------------|-----------------|
| Manual processing hours/month | 453.83 hours | about 150 hours |
| Average resolution time | 30.27 hours | under 8 hours |
| Error rate | 6.96% | under 2% |
| Customer satisfaction score | 7.04 / 10 | 8.5+ / 10 |

### Possible Failure Cases

- Critical case predicted as normal
- Normal case predicted as critical, causing alert fatigue
- Poor image quality causing unreliable output
- Model underperformance on a specific scanner or patient subgroup
- Workflow failure where alerts are not delivered to the correct reviewer

### Human Review and Validation

- All scans are still reviewed by radiologists.
- Critical AI flags trigger priority review, not automatic diagnosis.
- Low-confidence predictions are escalated.
- Radiologist overrides are logged.
- The system should run in shadow mode before live deployment.
- A clinical governance team should review monthly performance reports.

---

## Task 7: Responsible AI Considerations

### Bias in Data

The model may perform differently across age groups, sex, hospital sites, scanner models, or underrepresented populations.

**Mitigation:** Evaluate metrics by subgroup before deployment and continue monitoring after deployment.

### Incorrect Predictions

False negatives can delay care, while false positives can create alert fatigue.

**Mitigation:** Tune thresholds for high critical-case recall, keep radiologists in control, and audit false negatives.

### Privacy Concerns

Medical images and DICOM metadata may contain protected health information.

**Mitigation:** De-identify DICOM metadata, remove burned-in text where needed, encrypt data, and maintain access controls.

### Over-Reliance on AI

Users may trust the AI suggestion too much.

**Mitigation:** Present AI output as suggested priority, not diagnosis. Require human sign-off for clinical decisions.

### Impact on Users

The system should reduce workload pressure without replacing clinical judgment or creating unfair productivity tracking.

**Mitigation:** Involve radiologists in design, training, and governance. Do not use AI outputs to punish individual staff.

### Human Oversight

Healthcare AI requires clear accountability.

**Mitigation:** Maintain audit logs, model versioning, rollback plans, and a clinical AI governance committee.

---

## Task 8: Final One-Page Solution Summary

### Problem

Radiology worklists are often reviewed in time order, which can delay critical scans. The business problem is slow identification and prioritization of urgent medical images.

### Proposed AI Solution

Deploy an AI triage layer that classifies incoming medical images as `normal`, `urgent`, or `critical`. The output is used to prioritize the radiologist worklist. The system supports clinicians but does not replace final human diagnosis.

### Required Data

The solution requires de-identified X-ray or CT images, severity labels from radiologist review, optional patient/scanner metadata, and historical workflow outcomes.

### Model Recommendation

Use ResNet-50 with transfer learning. This CNN architecture is appropriate for image classification, works well with limited labeled data compared with training from scratch, and can be fine-tuned for medical imaging.

### Expected Business Impact

The AI system is expected to reduce manual prioritization effort, shorten average resolution time, lower routing or prioritization errors, and improve patient and physician satisfaction.

### Risks and Mitigation

| Risk | Mitigation |
|------|------------|
| Missed critical case | High recall threshold and mandatory radiologist review |
| False alerts | Precision monitoring and threshold tuning |
| Bias | Subgroup evaluation and fairness audits |
| Privacy leakage | DICOM de-identification and secure storage |
| Over-reliance | Human-in-the-loop workflow and user training |
| Model drift | Shadow mode, monitoring, retraining, and rollback process |

### Architecture Diagram

See `diagrams/solution_architecture.png`.
