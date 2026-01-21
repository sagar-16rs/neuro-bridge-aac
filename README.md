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
```

## How to Use

### Step 1: Calibration (Teaching Mode)
The system needs to learn *your* face and specific gestures.
```bash
python teach_gestures.py
```
> Enter Label: Type IDLE -> Sit still for 3 seconds.

> Enter Label: Type WATER -> Perform your gesture (e.g., Puff Cheeks).

> Enter Label: Type YES -> Look UP. (These vectors are saved locally to ./neuro_bridge_memory).

### Step 2: Run the Cognitive Agent
Start the live communication system.
```bash
python main_agent_pro_2.py
```
> Green HUD: System is Scanning.

> Orange HUD: System is Negotiating (Active Learning).

> Voice: Speaks when confidence > 88%.

## 🧠 The "Thinking" Engine (Traceable Reasoning)

Unlike "Black Box" AI, Neuro-Bridge follows a transparent, deterministic reasoning path. Every 33ms, the **Cognitive Agent** executes the following logic chain to prevent false positives and ensure safety.

### 1. The Decision Logic Flow
```mermaid
graph TD
    A[Webcam Input] -->|MediaPipe| B(99D Face Vector)
    B --> C{Fatigue Analysis}
    C -->|Alert User| D[High Threshold 0.88]
    C -->|Tired User| E[Low Threshold 0.70]
    D --> F[Qdrant Search]
    E --> F
    F --> G{Similarity Score?}
    G -->|Score > Threshold| H[EXECUTE Command]
    G -->|Score 0.45 - Threshold| I[NEGOTIATE: Ask User]
    G -->|Score < 0.45| J[Ignore: IDLE]
    I -->|User Blinks| K[LEARN: Update Qdrant]
```

## 2. Sample Reasoning Trace (Log Extract)
How the system decides to speak vs. wait based on Fatigue metrics:

> [T=0.0s] Input: Face Vector received. EAR=0.30 (User Alert). [T=0.1s] Retrieval: Nearest match "WATER". Similarity=0.92. Required=0.88. [T=0.1s] Decision: 0.92 > 0.88 → High Confidence. [T=0.1s] Action: EXECUTE "I need Water."

> (... 2 hours later, patient is tired ...)

> [T=7200s] Input: Face Vector received. EAR=0.15 (User Fatigued). [T=7200.1s] Reasoning: Fatigue Detected. Lowering Threshold to 0.75. [T=7200.1s] Retrieval: Nearest match "WATER". Similarity=0.78. [T=7200.1s] Decision: 0.78 > 0.75 (Adaptive) → Success. [T=7200.1s] Note: Without adaptation, this would have failed (0.78 < 0.88).

## 3. The Consistency Manager (Debouncing)
To filter out ALS fasciculations (involuntary muscle spasms), we do not act on a single frame.

Buffer: Stores the last 8 decisions.

Rule: An Action is only taken if it appears in >60% of the buffer.

Result: Random twitches are ignored; only deliberate, sustained gestures trigger the voice.
