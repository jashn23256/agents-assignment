# LiveKit Intelligent Interruption Agent ("Kelly")

This repository contains a LiveKit Voice Agent customized to handle "Intelligent Interruptions." The primary goal is to solve the "Backchannel Problem"—where users saying "Yeah," "Okay," or "Uh-huh" accidentally cut off the agent mid-sentence.

##  The Logic Core

The agent uses a **Hybrid VAD + Semantic Brain** approach to decide when to stop speaking.

### 1. The VAD "Safety Net" (Time-Based)
The first line of defense is the Voice Activity Detector (VAD) configuration.
- **Logic:** We set `min_interruption_duration` to a high threshold (e.g., `2.0s`).
- **Effect:** The hardware VAD is effectively "blind" to short utterances. If a user says anything short (like "Stop" or "Okay"), the agent **never** pauses automatically. This guarantees zero audio "hiccups."

### 2. The Semantic Brain (Meaning-Based)
Since the VAD ignores everything, we rely on the Speech-to-Text (STT) transcript to manually control the conversation flow.

**The Decision Matrix:**
Every time the user finishes speaking, the `my_custom_turn_completed` function runs:

1.  **Check 1: Exact Match**
    * Is the word exactly in our `IGNORE_WORDS` list? (e.g., "yeah", "ok", "sure").
    * *Result:* If **YES**, the agent ignores it completely and keeps talking.

2.  **Check 2: Semantic Similarity (Embeddings)**
    * We use `SentenceTransformer` ('all-MiniLM-L6-v2') to vectorize the user's input.
    * We compare it against the embeddings of our ignore list using **Cosine Similarity**.
    * *Result:* If the similarity score is > `0.60`, we treat it as a passive acknowledgement ("Backchanneling") and ignore it.

3.  **Check 3: The "Stop" Command**
    * If the input is **NOT** an ignore word (e.g., "Stop", "Wait", "Change topic"), the code manually triggers the interruption.
    * **Action:** It cancels the current LLM task and forces the agent to acknowledge the stop immediately (e.g., "Stopping."), effectively clearing the audio buffer.

### 3. State Awareness
* **Agent Speaking:** The logic above applies. Passive words are ignored; commands stop the agent.
* **Agent Silent:** No special logic is applied. If the user says "Yeah" while the agent is listening, the agent treats it as a normal conversational turn (e.g., "Glad you agree!").

---

##  Setup & Installation

### 1. Prerequisites
* Python 3.9+
* A LiveKit Cloud Project (or local instance)
* API Keys for Groq (LLM/STT) and Murf (TTS)

### 2. Install Dependencies
```bash
pip install -r requirements.txt
