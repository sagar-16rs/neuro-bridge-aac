# Neuro-Bridge 9.0: Fatigue-Adaptive AAC System

> **Convolve 4.0 (Pan-IIT AI/ML Hackathon)**
> **Track:** Healthcare & Accessibility



## The Problem
Patients with **ALS (Amyotrophic Lateral Sclerosis)**, **Locked-In Syndrome (LIS)**, and severe paralysis often retain full cognitive awareness but lose the ability to speak or move.
* **Commercial Eye-Trackers ($5,000+)** are expensive and rigid.
* **The Critical Failure:** Existing solutions fail when a patient gets tired. As eyelids droop (**ptosis**) due to exhaustion, standard trackers stop working, leaving the patient isolated when they are most vulnerable.

## The Solution: Neuro-Bridge
Neuro-Bridge is a **Neuro-Symbolic AI Agent** that runs on a standard laptop webcam. It uses **Qdrant** to store "memories" of facial micro-gestures and **adapts** to the patient's fatigue level in real-time.

### Key Innovations
1.  **Fatigue Adaptation:** The system monitors the **Eye Aspect Ratio (EAR)**. As the patient tires, the AI automatically lowers the detection threshold, making it easier to trigger commands.
2.  **One-Shot Learning:** Using Qdrant's vector search, the system can learn a new gesture (e.g., "Itch") in <2 seconds.
3.  **Consistency Manager:** A temporal voting system filters out muscle spasms (fasciculations) to prevent false triggers.
4.  **Privacy First:** 100% Edge Computing. No video data leaves the device.

---

## Demo & Screenshots

### 1. State: IDLE (Scanning)
The system establishes a baseline. The **Yellow Line** tracks the patient's fatigue (eye openness).
<img width="400" height="400" alt="Screenshot 2026-01-21 151330" src="https://github.com/user-attachments/assets/f3e11652-3ad8-4aa2-b66d-e0cd59299341" />


### 2. State: NEGOTIATION (Active Learning)
The user makes a messy/ambiguous gesture. The system asks **"CONFIRM?"**. If the user blinks, the system **Upserts** this new vector to Qdrant, learning the user's specific "accent" of movement.
<img width="400" height="400" alt="Screenshot 2026-01-21 151454" src="https://github.com/user-attachments/assets/68ab8251-937d-4412-a7fe-2e2f4a6293b2" />


### 3. State: EXECUTION (Success)
High-confidence match (>88%). The system speaks: **"I need Water."**
<img width="400" height="400" alt="Screenshot 2026-01-21 190800" src="https://github.com/user-attachments/assets/e4d461cb-3777-4a4d-8021-7b48b433ad9f" />


---

## System Architecture

The system follows a 4-Layer Neuro-Symbolic pipeline:

| Layer | Technology | Function |
| :--- | :--- | :--- |
| **1. Perception** | **MediaPipe** | Extracts 478 facial landmarks and converts them into a **Normalized 99D Vector** (invariant to rotation). |
| **2. Memory** | **Qdrant** | Stores semantic embeddings of gestures. Enables **Cosine Similarity** search (<30ms latency). |
| **3. Reasoning** | **Cognitive Agent** | Manages state (IDLE/EXECUTE). Implements the **Fatigue Algorithm**: $T_{adapt} = T_{base} - (Fatigue \times 0.5)$. |
| **4. Actuation** | **PyTTSx3** | Thread-safe Text-to-Speech engine for voice output. |

---

## Getting Started

### Prerequisites
* Python 3.9+
* A webcam
* Windows/Linux/Mac

### Installation
```bash
# 1. Clone the repository
git clone [https://github.com/your-username/neuro-bridge-aac.git](https://github.com/your-username/neuro-bridge-aac.git)
cd neuro-bridge-aac

# 2. Create a virtual environment (Optional but recommended)
python -m venv venv
# Windows: venv\Scripts\activate
# Mac/Linux: source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt
