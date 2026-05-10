# Project Structure & Agent Overview

This document provides a detailed breakdown of the **InterviuX** folder structure, its core AI agents, and how they communicate to deliver a high-fidelity mock interview experience.

---

## 📂 Folder Structure

```text
InterviuX/
├── backend/               # Backend Server (Node.js/Express)
│   ├── server.js          # Main API Logic
│   ├── package.json       # Dependencies
│   └── .env               # API Keys
├── frontend/              # Frontend Assets (Vanilla JS/CSS)
│   ├── index.html         # Main Application Entry Point
│   ├── app.js             # Frontend Logic (Multimodal Capturing)
│   ├── styles.css         # Glassmorphism UI & Animations
│   └── models/            # face-api.js pre-trained weights
├── sample_inputs/         # Example resumes for testing
├── sample_outputs/        # Mock reports and sample scores
├── architecture.md        # High-level System Design
├── README.md              # Setup & Overview
└── STRUCTURE.md           # [This File] Agent & Communication Detail
```

---

## 🤖 Main Modules & Agents

InterviuX operates as a distributed multi-agent system where intelligence is split between the **Client (Browser)** and the **Server (Groq LLM)**.

### 1. Resume Parser Agent (Backend)
- **Tech:** `pdf-parse` + `llama-3.3-70b`
- **Role:** Extracts raw text from uploaded PDFs and performs "Entity & Skill Extraction." It identifies suggested roles, seniority levels, and critical "skill gaps" which inform the interview's starting difficulty.

### 2. Interview Agent / Orchestrator (Backend/Frontend)
- **Tech:** Node.js + Groq Chat Completions
- **Role:** The "Brain" of the interview. It manages a state machine (Warm-up → Core → Deep-dive) and generates context-aware questions. It uses **Adaptive Difficulty**: if a user struggles, it pivots to foundational questions in real-time.

### 3. Audio & Linguistic Agent (Frontend)
- **Tech:** Web Audio API + Web Speech API
- **Role:** Analyzes the candidate's speech patterns. It calculates **Words Per Minute (WPM)**, detects **Filler Words** (um, uh, like), and measures **Hesitation Gaps** (>1.2s of silence) to score communication confidence.

### 4. Computer Vision (CV) Agent (Frontend)
- **Tech:** `face-api.js`
- **Role:** Monitors visual engagement during the interview. It tracks eye contact, facial expressions (smile/stress), and detects "slouching" or "off-center" posture using bounding box heuristics.

### 5. Feedback & Report Generator (Backend)
- **Tech:** Multi-dimensional Scoring Rubric (LLM-based)
- **Role:** Aggregates data from all other agents (technical accuracy + communication + visual engagement) to produce a holistic **Coaching Report**. It acts as a "Strict Senior Lead" to ensure high-fidelity evaluation.

---

## 🔌 How Components Communicate

The system follows an **Asynchronous RESTful Flow** with client-side state persistence:

1.  **JSON Exchanges:** All communication between the Frontend and Backend occurs via `POST` requests with JSON payloads.
2.  **Multimodal Pipelines:**
    *   **Visual data** is processed entirely on the client (for privacy and speed).
    *   **Audio data** is transcribed on the client and the text is sent to the backend for linguistic evaluation.
3.  **State Management:** The interview state (current question index, session scores) is tracked in the Backend's memory and persisted in the Frontend's `localStorage` for session recovery.
4.  **Fallback Chain:** The Backend implements a model fallback chain (e.g., if 70b is rate-limited, it automatically falls back to 8b) to ensure the interview is never interrupted.
