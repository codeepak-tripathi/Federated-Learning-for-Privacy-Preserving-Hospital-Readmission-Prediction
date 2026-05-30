Federated Learning for Privacy-Preserving Hospital Readmission Prediction
Research-Level System Architecture & Working Explanation
Based on your notebook, the project builds a Federated Learning (FL) framework for predicting 30-day patient readmission risk using Electronic Health Records (EHRs) from multiple hospitals while preserving patient privacy.

1. Research Problem
Traditional Approach (Centralized ML)
Normally, hospitals send all patient records to a central server.
Hospital A ──┐
Hospital B ──┼──► Central Data Warehouse ► AI Model
Hospital C ──┘

Problems:
Privacy violation
HIPAA/GDPR compliance issues
Data leakage risk
Ownership concerns
Cross-hospital data sharing restrictions

Proposed Solution
Federated Learning
Data never leaves hospital
Only model parameters are shared


Overall Architecture
                        ┌─────────────────────────┐
                         │      Master Server      │
                         │   Federated Aggregator  │
                         └──────────┬──────────────┘
                                    │
                     Model Weights / Gradients
                                    │
          ┌──────────────┬──────────┴──────────┬──────────────┐
          │              │                     │              │
          ▼              ▼                     ▼              ▼

 ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
 │   Hospital 1   │ │   Hospital 2   │ │   Hospital 3   │
 │ Local Dataset  │ │ Local Dataset  │ │ Local Dataset  │
 └───────┬────────┘ └───────┬────────┘ └───────┬────────┘
         │                  │                  │
         ▼                  ▼                  ▼

  Local Model       Local Model       Local Model
  Training          Training          Training

         │                  │                  │
         ▼                  ▼                  ▼

  Model Parameters  Model Parameters  Model Parameters

         └──────────────┬──────────────┘
                        ▼

                Master Server

                FedAvg Aggregation

                        ▼

                 Global Model

                        ▼

          Sent back to all Hospitals


2. Dataset Flow
Your project uses:
Dataset:
SyntheticMass Synthea EHR Dataset
Contains:
Patients
Conditions
Encounters
Observations

Raw Data Files
patients.csv
conditions.csv
encounters.csv
observations.csv


3. Data Engineering Pipeline
Step 1: Patient Data
Input:
Patient ID
Birth Date
Gender
Race
Ethnicity
Death Date

Generated Features:
Age
Gender
Race
Ethnicity
Is_Dead


Step 2: Encounter Data
Input:
Encounter Start
Encounter Stop
Encounter Class
Costs
Coverage

Generated Features:
Encounter Duration
Encounter Type
Base Encounter Cost
Total Claim Cost
Payer Coverage


Step 3: Condition Data
Input:
Disease Records

Generated Feature:
Condition Count


Feature Engineering Flow
Patients.csv
       │
       ▼
Patient Features
       │
       ├── Age
       ├── Gender
       ├── Race
       ├── Ethnicity
       └── Death Status

Conditions.csv
       │
       ▼
Condition Count

Encounters.csv
       │
       ▼
Encounter Features
       │
       ├── Duration
       ├── Encounter Type
       ├── Costs
       └── Coverage

                    ▼
             Feature Fusion

                    ▼

             Final Dataset


4. Target Variable Creation
Research Goal:
Predict:
Readmitted Within 30 Days?
Your notebook calculates:
Next Visit Date
-
Current Visit End Date

If:
Days <= 30

Then:
READMITTED_30 = 1

Otherwise:
READMITTED_30 = 0


Target Generation Flow
Encounter 1
      │
      ▼

Encounter 2

Days Between Visits

      │
      ▼

 <=30 Days ?


 YES            NO
  │              │
  ▼              ▼

Readmit=1    Readmit=0


5. Hospital Partitioning
Patients are randomly divided into:
Hospital 1
Hospital 2
Hospital 3

This simulates independent hospitals.
Patient Pool
      │
      ▼

 ┌─────────────┐
 │ Shuffle IDs │
 └──────┬──────┘
        │
 ┌──────┼──────┐
 ▼      ▼      ▼

H1     H2     H3


6. Local Hospital Infrastructure
Each hospital contains:
EHR Database
Feature Engineering
Data Preprocessing
Scaler
Local AI Training

Architecture:
┌───────────────────────┐
│ Hospital Local Node   │
├───────────────────────┤
│ EHR Database          │
│ Feature Extraction    │
│ StandardScaler        │
│ ML Model              │
│ Local Evaluation      │
└──────────┬────────────┘
           │
           ▼

Model Parameters Only


7. Machine Learning Models
Your notebook uses:
Model 1
Logistic Regression
Used in FedAvg experiment.

Model 2
MLPClassifier
Architecture:
Input Layer

     │

64 Neurons

     │

32 Neurons

     │

Output Layer
(Readmission Risk)


8. Federated Learning Workflow
Round 1
Master Server sends initial model.
Global Model V1

to all hospitals.

Local Training
Hospital 1:
Train on H1 Data

Hospital 2:
Train on H2 Data

Hospital 3:
Train on H3 Data

No data sharing occurs.

Parameter Upload
Each hospital uploads:
Weights
Biases
Model Parameters

NOT:
Patient Data
Medical Records


Federated Round Flow
Global Model

      │
      ▼

 ┌────┼────┐
 ▼    ▼    ▼

H1   H2   H3

Local Training

 ▼    ▼    ▼

W1   W2   W3

      │
      ▼

 Master Server

      │
      ▼

 FedAvg

      │
      ▼

 Updated Global Model


9. FedAvg Algorithm
Core aggregation:
Your notebook performs weighted averaging.
Concept:
W_{global}=\sum_{k=1}^{N}\frac{n_k}{n}W_k
Where:
(W_k) = Local model weights
(n_k) = Samples in hospital k
(n) = Total samples

Aggregation Example
Hospital 1 : 1000 patients
Hospital 2 : 2000 patients
Hospital 3 : 3000 patients

Weights:
16.7%
33.3%
50.0%

Global model becomes weighted average.

10. Master Server Responsibilities
The Master Server:
Communication Layer
Receives:
Model Parameters


Aggregation Layer
Executes:
FedAvg


Global Model Manager
Stores:
Round 1 Model
Round 2 Model
Round 3 Model
...
Round N Model


Distribution Layer
Broadcasts:
Updated Global Model

back to hospitals.

11. Privacy Protection Layer
Data Never Leaves Hospital
Patient Records - NO

Medical Images -NO

EHR Tables -NO

Lab Reports -NO

Shared:
Weights -YES

Gradients -YES

Biases -YES


12. Evaluation Framework
Metric used:
ROC-AUC
Measures ability to distinguish:
Readmitted
vs
Not Readmitted

Range:
0.50 = Random
0.70 = Good
0.80 = Very Good
0.90 = Excellent

Your project reports:
Hospital AUC
Hospital B AUC
Hospital C AUC
Average AUC

per round.

13. Centralized Baseline
Research comparison:
Federated Learning
vs
Centralized Learning

Centralized:
All Data Combined

H1 + H2 + H3

       ▼

Single MLP

Used as upper-bound benchmark.

Complete End-to-End Research Pipeline
Synthea Dataset
       │
       ▼

Data Ingestion
       │
       ▼

Feature Engineering
       │
       ▼

Readmission Label Creation
       │
       ▼

Hospital Partitioning
       │
       ▼

 ┌─────────┬─────────┬─────────┐
 ▼         ▼         ▼

H1        H2        H3

Local Training
(Logistic + MLP)

 ▼         ▼         ▼

Local Weights

 └─────┬─────┬───────┘
       ▼

 Master Server

       ▼

 FedAvg Aggregation

       ▼

 Global Model

       ▼

 Redistribute

       ▼

 Next Round

       ▼

 Final Global Model

       ▼

 ROC-AUC Evaluation

       ▼

 FL vs Centralized Comparison

Research Contributions
Privacy-preserving healthcare analytics.
Cross-hospital collaboration without data sharing.
Readmission prediction using EHR records.
Federated Logistic Regression and Federated Neural Network implementation.
FedAvg-based distributed optimization.
Comparison with centralized deep learning baseline.
Scalable architecture suitable for multi-hospital healthcare networks.
For a PhD-level thesis or IEEE paper, the next extension would be to add:
Secure Aggregation
Differential Privacy
Homomorphic Encryption
Non-IID hospital data simulation
Federated XGBoost
Federated Transformer models for EHRs
Explainable AI (SHAP/LIME)
Blockchain-based audit trail for model updates
These additions would elevate the work from a basic FL implementation to a publishable healthcare federated AI research system.

